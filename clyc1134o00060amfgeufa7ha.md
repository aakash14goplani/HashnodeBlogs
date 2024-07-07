---
title: "How to Implement Refresh Token Rotation in SvelteKitAuth"
seoTitle: "Refresh Token Rotation in SvelteKitAuth"
seoDescription: "Learn how to implement refresh token rotation in SvelteKitAuth for secure and seamless user sessions"
datePublished: Sun Jul 07 2024 20:49:29 GMT+0000 (Coordinated Universal Time)
cuid: clyc1134o00060amfgeufa7ha
slug: how-to-implement-refresh-token-rotation-in-sveltekitauth
tags: sveltekit, access-token, refresh-token, sveltekitauth, authjs, token-rotation

---

Refresh token rotation is the practice of updating an `access_token` on behalf of the user, without requiring interaction (eg.: re-sign in). `access_token` are usually issued for a limited time. After they expire, the service verifying them will ignore the value. Instead of asking the user to sign in again to obtain a new `access_token`, certain providers support exchanging a `refresh_token` for a new `access_token`, renewing the expiry time. Refreshing your `access_token` with other providers will look very similar, you will just need to adjust the endpoint and potentially the contents of the body being sent to them in the request.

Using `jwt` and `session` callbacks in the provider configuration, we can persist OAuth tokens and refresh them when they expire. These callbacks trigger every time a call is made to `auth()`, i.e., `event.locals.auth()`.

The **first step** is to create an API route that will refresh the token. In the file *src/routes/api/renew-token/+server.ts*:

```typescript
import { isEmpty } from 'lodash-es';
import type { RequestHandler } from '@sveltejs/kit';
import { CLIENT_ID, CLIENT_SECRET, ISSUER, API_IDENTIFIER } from '$env/static/private';

export const POST = (async ({ fetch, locals }) => {
  const session = await locals.auth();
  const user = session?.user;

  if (!isEmpty(user)) {
    try {
      const request = await fetch(`${ISSUER}oauth/token`, {
        method: 'POST',
        headers: {
          'content-type': 'application/x-www-form-urlencoded'
        },
        body: new URLSearchParams({
          grant_type: 'client_credentials',
          client_id: CLIENT_ID,
          client_secret: CLIENT_SECRET,
          audience: API_IDENTIFIER // ${ISSUER}/api/v2/
        })
      });
      await request.json();
      console.log(`Response from API: ${JSON.stringify(response)}`);
      return new Response(JSON.stringify(response), { status: 200 });
    } catch (error: any) {
      console.log(`Error while updating token data: ${error?.message}. Reusing existing tokens!`);
      return new Response(JSON.stringify({
        access_token: user.access_token,
        expires_in: user.expires_in,
        token_type: 'Bearer'
      }), { status: 200 });
    }
  } else {
    return new Response(JSON.stringify({
      message: 'User is not authorized to rotate access-token'
    }), { status: 401 });
  }
}) satisfies RequestHandler;
```

Now, the **second step** is to configure the `jwt` and `session` callbacks in the provider's configuration to trigger token rotation upon invocation of the target API route:

```typescript
// hooks.server.ts
export const { handle: getAuthConfig } = SvelteKitAuth(async (event) => {
  const config: SvelteKitAuthConfig = {
    ....,
    callbacks: {
      async jwt({ token, account, profile }) {
        /**
         * This callback triggers multiple times. For the very first time,
         * token -> { name, email, picture, sub }
         * account -> { all_tokens }
         * profile -> { all_user_details_and_custom-attributes }
         * trigger -> { signin, signut, update }
         * For second and successive times,
         * token -> { name, email, picture, sub, iat, exp, jti }
         * account -> undefined
         * profile -> undefined
         * trigger -> undefined
        */
        // store init values that must be passed to session cb, if this line is skipped then
        // { name, email, picture, sub, iat, exp, jti } will always be undefined in session cb
        try {
          if (!isEmpty(account)) {
            token = { ...token, ...account };
          }
          if (!isEmpty(profile)) {
            token = { ...token, ...(profile as any) };
          }
					
          // update user data on request
          const userQuery = event.request.headers.get('query') || event.url.searchParams.get('query');
					
          // refresh token post 30 minutes
          if (
            token &&
            (isTokenRefreshRequired(token.token_expires_in as string) || userQuery === 'update-token-data')
          ) {
            const tokenRequest = await event.fetch(
              event.url.origin + '/api/renew-token',
              { method: 'POST' }
            );
            const updatedToken = await tokenRequest.json();
            if (updatedToken.access_token) {
              token = {
                ...token,
                ...updatedToken
              };
            }
          }
        } catch (e: any) {
          console.log('ERROR in AUTH JWT CALLBACK: ', e?.message);
        }
        return token;
      },
      async session({ session, token }) {
        try {
          // This callback triggers multiple times
          if (session.user) {
            if (token?.access_token) {
              session.user = { ...session.user, ...token } as any;
            }
          }
        } catch (e: any) {
          console.log('ERROR in AUTH SESSION CALLBACK: ', e?.message);
        }
        return session;
      }
    },
  }
});

function isTokenRefreshRequired(issued_at: string) {
  return +difference([new Date(+issued_at), new Date(), 'minutes']) > 29;
}
```

In the `jwt` callback, we trigger the token rotation request in two scenarios:

1. **Automatically**, after every 30 minutes (just for the sake of example).
    
2. Once the user has **manually** requested token rotation.
    

It is important to save the updated `access_token` in the `token` property, as the contents of this `token` property are encrypted and saved in the *session-token* cookie. So when a new request is made, the same token is decrypted, and hence the value must be preserved.

As we know that the `jwt` and `session` callbacks trigger every time a call to `auth()` is made, to trigger the refresh token, we will have to create a dummy API route that will trigger the renew token process:

```typescript
// FILE -> src/routes/api/trigger-renew-token/+server.ts

import type { RequestHandler } from '@sveltejs/kit';
import { isEmpty } from 'lodash-es';

export const POST = (async ({ locals }) => {
  const session = await locals.auth();
  const returnValue = !isEmpty(session)
    ? { status: 200 }
    : { status: 401 };
  return new Response(JSON.stringify(returnValue), { status: 200 });
}) satisfies RequestHandler;
```

Finally, we invoke it from the client side:

```svelte
<!-- some *.svelte file -->
<script lang="ts">
  async function refreshToken() {
    const request = await fetch(
      '/api/trigger-renew-token?query=update-token-data',
      { method: 'POST' }
    });
  }
</script>
<button on:click={refreshToken} class="button">Refresh Token</button>
```

In conclusion, implementing refresh token rotation in SvelteKitAuth is a crucial practice for maintaining secure and seamless user sessions. By leveraging `jwt` and `session` callbacks, we can efficiently manage OAuth tokens, ensuring they are refreshed without user intervention. This approach not only enhances the user experience by avoiding frequent re-authentication but also maintains the integrity and security of the session. By following the outlined steps, including creating an API route for token renewal and configuring the necessary callbacks, developers can ensure that access tokens are consistently updated and preserved, providing a robust authentication mechanism in their SvelteKit applications.

Here is the link to the [**GitHub repository**](https://github.com/aakash14goplani/SvelteKitAuth) with the codebase. In the next article, we will learn how to effectively communicate data between client and the server in SvelteKitAuth.