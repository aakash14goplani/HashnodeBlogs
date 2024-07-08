---
title: "How to Exchange Data Between Client and Server Using SvelteKitAuth"
seoTitle: "Data Exchange in SvelteKitAuth"
seoDescription: "Learn how to exchange data between client and server using SvelteKitAuth with JWT and session callbacks for seamless and secure synchronization"
datePublished: Mon Jul 08 2024 19:59:24 GMT+0000 (Coordinated Universal Time)
cuid: clydeoili000608le8notc8tn
slug: how-to-exchange-data-between-client-and-server-using-sveltekitauth
tags: sveltekit, client-server-communication, sveltekitauth, authjs

---

In this article, we will explore how to exchange data between the client and server using SvelteKitAuth. We will delve into the importance of JWT and session callbacks, and how to synchronize and store sessions effectively.

### Updating and Syncing data from the Client to the Server

Since the `SvelteKitAuthConfig` is attached to the hooks, it triggers with every API call that is intercepted by the hooks. The `SvelteKitAuthConfig` comes with two important callback functions, i.e., `jwt` and `session`, that trigger on every network request intercepted by the hooks.

The `jwt` callback holds the `token` property, which includes the authenticated user's token and profile data. This information constitutes the session information that will be shared site-wide. This information is stored in an encrypted format in the `session-token` cookie.

On every network request, this callback decrypts the `session-token` cookie, reads the information, performs operations, and then encrypts it back and saves it in the `session-token` cookie. This token is then passed to the `session` callback, which forms the session object that will be shared site-wide and can be accessed on the server as `const session = await locals.auth()` and on the client-side as `const session = $page.data.session`.

Hence, in order to update the user information, we must use the `jwt` callback so that it is preserved in the `session-token` cookie and later pass the information to the `session` callback so that the information is accessible on both the server and the client side.

#### **The First Step: Updating the Config Object**

With this knowledge in hand, let's configure the `jwt` and the `session` callback in the `SvelteKitAuthConfig` object.

```typescript
{
  callbacks: {
    async jwt({ token, account, profile }) {
      ...
      const userQuery = event.request.headers.get('query') || event.url.searchParams.get('query');
      if (userQuery === 'update-user-data') {
        try {
          const clonedRequest = event.request.clone();
          const clonedBody = clonedRequest.body;
          const response = await new Response(clonedBody);
          const body = await response.json();
          if (!isEmpty(body)) {
            token = {
              ...token,
              ...body
            };
          }
        } catch (ex: any) {
          console.log('Unable to update user data for url', ex?.message);
        }
      }
      return token;
    },
    async session({ session, token }) {
      if (session.user) {
        if (token?.access_token) {
          session.user = { ...session.user, ...token } as any;
        }
      }
      return session;
    }
  }
}
```

Let's take a moment to understand what the above code snippet actually does:

* Since the `jwt` callback is invoked for every API request intercepted by the hooks, we cherry-pick the ones that have a particular flag passed via the header or the query params, which will instruct us that we must perform data communication operations. For the sake of this example, we use the string *update-user-data*.
    
* Once we get hold of this flag, we will fetch the payload from the request and update the `token` property of the `jwt` callback. Remember, this is the same property that will be preserved in an encrypted format in the `session-cookie`. Side Note: cookie size up to 4kb could be saved at any given time due to the browser's storage limitation. However, if we pass too much data into the `token` property, they will be chunked into multiple cookies, i.e., `session-cookie_0`, `session-cookie_1`, and so on.
    
* Finally, we pass the `token` property to the `session` callback so that it is available for use on both the client and the server side.
    

#### **The Second Step: Creating the Update Endpoint**

As we discussed in the last section, we need to intercept specific API requests that have flags which will enable us to kickstart the data operation. So in this section, we will create an API route that will be used by the client to send the payload which must be synced and updated on the server side.

So we create an API route in the file *src/routes/api/update-user-data/+server.ts*. The aim of this API route is to pass data to the `jwt` callback. The API route then sends the updated data back, which we can catch by invoking the `auth()` method. We then return the updated object back to the client.

```typescript
import type { RequestHandler } from '@sveltejs/kit';
import { isEmpty } from 'lodash-es';

export const POST = (async ({ locals }) => {
  const session = await locals.auth();
  const user = session?.user;

  if (!isEmpty(user) && !isEmpty(session)) {
    try {
      return new Response(JSON.stringify({
        data: user
      }), { status: 200 });
    } catch (error: any) {
      console.log('Error while updating user data: ', error?.message, '. Sending existing data');
      return new Response(JSON.stringify({ data: user }), { status: 200 });
    }
  } else {
    return new Response(JSON.stringify({ data: 'user is not authorized to update data' }), { status: 401 });
  }
}) satisfies RequestHandler;
```

#### **The Third Step: Client Initiates the Update Request**

The client triggers the API route that we created in the previous section and sends the payload object along with it.

```svelte
<script lang="ts">
  let userData = $page.data.session?.user?.fav_num || '';

  async function updateUserData() {
    const request = await fetch('/api/update-user-data?query=update-user-data', {
      method: 'POST',
      body: JSON.stringify({
        fav_num: `My favourite number is: ${Math.ceil(Math.random() * 100)}`
      })
    });
    const response = await request.json();
    userData = response?.data?.fav_num;
  }
</script>

<button on:click={updateUserData} class="button">Update user data</button>

{#if userData}
  <p>Updated user-data {userData}</p>
{/if}
```

With these three steps, the data is synced and updated by the client to the server. In the next section, we will learn how to update and sync the data from the server to the client.

### **Updating and Syncing Data from the Server to the Client**

The only way for the server to send the information back to the client is via hydration. The server load functions, i.e., *+layout.server.ts* and *+page.server.ts*, can send data to the client by returning an object. Only objects that can be serialized using [**devalue**](https://github.com/rich-harris/devalue) can be used. Here is an example where the server [load fu](https://github.com/rich-harris/devalue)nction returns the session property to the client.

**NOTE**: Properties returned by the layout server load function will be av[ailable](https://github.com/rich-harris/devalue) to all the child routes, whereas the properties r[eturned](https://github.com/rich-harris/devalue) by the page server load will only be available to the current route. If both functions return the same property, the one that returns the last will precede over others.

```typescript
// server load functions

import type { LayoutServerLoad } from './$types';

export const load: LayoutServerLoad = async (event) => {
	const session = await event.locals.auth();
	return { session };
};
```

The client can access this information using the `$page.data` property.

```svelte
<script lang="ts">
  import { page } from '$app/stores';

  let session = $page.data.session;
  let user = $page.data.session.user;
</script>
```

### Conclusion

In this article, we explored the process of exchanging data between the client and server using SvelteKitAuth. We delved into the importance of JWT and session callbacks, and how to effectively synchronize and store sessions. By configuring the `jwt` and `session` callbacks, creating an update endpoint, and initiating update requests from the client, we ensured seamless data synchronization from the client to the server. Additionally, we discussed how the server can send information back to the client through hydration using server load functions. By following these steps, you can maintain a consistent and secure data exchange between the client and server in your SvelteKit applications.

Here is the link to the [**GitHub repository**](https://github.com/aakash14goplani/SvelteKitAuth) with the codebase. In the next article, we will learn to how to build custom pages and handle events in SvelteKitAuth.