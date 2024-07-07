---
title: "Step-by-Step Guide to Using built-in (Auth0) OAuth provider with SvelteKitAuth"
seoTitle: "Auth0 OAuth Setup with SvelteKitAuth Authjs guide"
seoDescription: "Integrate Auth0 OAuth with SvelteKit. Learn to configure properties, set environment variables, and ensure secure authentication for your application"
datePublished: Sun Jul 07 2024 18:49:21 GMT+0000 (Coordinated Universal Time)
cuid: clybwql78000c09jq655t3bbm
slug: step-by-step-guide-to-using-built-in-auth0-oauth-provider-with-sveltekitauth
tags: oauth, auth0, sveltekit, sveltekitauth

---

In the [previous article](https://blog.aakashgoplani.in/setting-up-auth0-and-adding-sveltekitauth-to-your-app), we covered the basics of SvelteKitAuth. In this article, we will delve into integrating the built-in OAuth provider with SvelteKit.

### Basic Configuration

Below is the code snippet for using the built-in Auth0 OAuth provider:

```typescript
// hooks.server.ts

import Auth0Provider from "@auth/core/providers/auth0";
import type { SvelteKitAuthConfig } from '@auth/sveltekit';

const config: SvelteKitAuthConfig = {
  providers: [
    Auth0Provider({
      id: 'auth0',
      name: 'Auth0',
      clientId: '-client-id-',
      clientSecret: '-client-secret-',
      issuer: 'https://dev-****.auth0.com/',  // <- remember to add trailing `/` 
      wellKnown: 'https://dev-****.auth0.com/.well-known/openid-configuration'
    }) as Provider
  ],
  secret: 'random-32-bit-hexadecimal-string',
  session: {
    strategy: 'jwt',
    maxAge: 3600 
  },
  trustHost: true
  ...
}
```

Let discuss about these properties in detail:

* **id**: It is a string value that you can assign to uniquely identify the provider. As we can see from the syntax `providers[{...}]` is an array, and hence this id property helps in referencing a particular provider in the case where we have multiple providers. Also, this particular id is used in the callback URL pattern. Our callback URL is [***localhost:4000/auth/callback/auth0***](http://localhost:4000/auth/callback/auth0); here "auth0" comes from this "id" parameter. It is an optional parameter; if we skip this, it defaults to the in-built provider that we are using, in this case, "Auth0".
    
* **name**: This is an optional property wherein you can provide any string value, preferred one is the name of the OAuth provider you're using i.e., "Auth0"
    
* **clientId**: is a unique identifier that is assigned to an application when it registers with an OAuth 2.0 service provider. The `clientId` is used by the application to authenticate itself to the service provider and to obtain access tokens. Client Id value can be obtained from the OAuth application. In your Auth0 application, go to -&gt; *Settings* tab -&gt; *Basic Information Section* -&gt; Get *Client ID* string.
    
* **clientSecret**: is a secret key that is used to protect the client ID. The `clientSecret` is not shared with the service provider and must be kept confidential by the application. Client Secret value, similar to Client ID, can be retrieved from the Basic Information Section of your Auth0 application.
    
* **issuer**: The issuer is the domain of your Auth0 application, which can be fetched from the Basic Information Section of your Auth0 application.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690471007022/6c04ed69-0697-40f3-89d2-f156e4336a40.png?auto=compress,format&format=webp align="left")
    
* **wellKnown**: The *wellKnown* URL is a preassigned stable endpoint that a server uses every time it runs. It contains information about where to fetch tokens and user information post-authentication.
    
* **secret**: This is a random string used as a salt for encryption. It should be a 32-character Hexadecimal string. You can generate one from [**this site**](https://generate-secret.vercel.app/32).
    
* **trustHost**: Auth.js relies on the incoming request’s `host` header to function correctly. For this reason, this property needs to be set to `true`.
    
    Make sure that your deployment platform sets the `host` header safely.
    
* **session**: This sets the strategy used for authentication. Available options are "*database*" and "*jwt*". It defaults to "*jwt*". For this tutorial series, we will be using "*jwt*". You can read more about [session strategy here](https://authjs.dev/concepts/session-strategies).
    

These were the core properties that should suffice the basic requirement for authentication. However, we will discuss a few more properties that enhance the developer experience to manage authentication.

* **debug**: This will use the `console` methods to log out many details about the authentication process, including requests, responses, errors, and database requests and responses.
    
* **basePath**: If your application has a base path configured, then you must set the same base path here as well.
    
* **cookies**: SvelteKitAuth sets a minimum of 3 and a maximum of 5 cookies based on the configuration of the provider.
    
    ```typescript
    import { dev } from '$app/environment';
    
    const useSecureCookies = !dev;
    
    cookies: {
      callbackUrl: {
        name: `${useSecureCookies ? '__Secure-' : ''}authjs.callback-url`,
        options: {
          httpOnly: true,
          sameSite: useSecureCookies ? 'none' : 'lax',
          path: '/',
          secure: useSecureCookies
        }
      },
      ...
    }
    ```
    
    1. ***callbackUrl***: This property saves callbackUrl
        
    2. ***csrfToken***: This property saves the CSRF token that is generated by SvelteKit (by default, if on else by SvelteKitAuth)
        
    3. ***pkceCodeVerifier***: This sets pkce cookie only once check for PKCE is enabled
        
    4. **sessionToken**: This is a very important cookie and sets the actual session of the user.
        
    5. ***state***: This sets pkce cookie only once check for STATE is enabled
        
    
    You can change these default configurations, however, **This is an advanced option.** Advanced options are passed the same way as basic options, but **may have complex implications** or side effects. You should **try to avoid using advanced options** unless you are very comfortable using them. [More Reading](https://authjs.dev/reference/core#cookies)...
    
* **callbacks**: Callbacks are asynchronous functions you can use to control what happens when an action is performed. Callbacks are *extremely powerful*, especially in scenarios involving JSON Web Tokens as they **allow you to implement access controls without a database** and to **integrate with external databases or APIs**.
    
    There are 4 types of callbacks: (1) *jwt,* (2) *session,* (3) *redirect* and (4) *signIn.*
    
    ```typescript
    optional callbacks: {
      jwt: (params) => Awaitable<null | JWT>;
      redirect: (params) => Awaitable<string>;
      session: (params) => Awaitable<Session | DefaultSession>;
      signIn: (params) => Awaitable<string | boolean>;
    };
    ```
    
    We will discuss some of these in detail in our next tutorial on creating a custom OAuth provider. However, if you want to utilize these options within in-built providers, you can read the details from [official documentation](https://authjs.dev/reference/sveltekit/types#callbacks)
    
* **logger**: You can customize the logging output by providing your own logger. This is useful if you want to send logs to a logging service or if you want to customize the format of the logs.
    
    ```typescript
    logger: {
      error(code, ...message) {
        console.error(code, message)
      },
      warn(code, ...message) {
        console.warn(code, message)
      },
      debug(code, ...message) {
        console.debug(code, message)
      },
    },
    ```
    
* **events**: Events are asynchronous functions that do not return a response; they are useful for audit logging. You can specify a handler for any of these events below - e.g., for debugging or to create an audit log. The content of the message object varies depending on the flow (e.g., OAuth or Email authentication flow, JWT or database sessions, etc.), but typically contains a user object and/or contents of the JSON Web Token and other information relevant to the event.
    
    ```typescript
    events: {
      createUser: (message) => Awaitable<void>;
      linkAccount: (message) => Awaitable<void>;
      session: (message) => Awaitable<void>;
      signIn: (message) => Awaitable<void>;
      signOut: (message) => Awaitable<void>;
      updateUser: (message) => Awaitable<void>;
    };
    ```
    
* **pages**: Specify URLs to be used if you want to create custom sign-in, sign-out, and error pages. Pages specified will override the corresponding built-in page. We have 4 types of pages:
    
    ```typescript
    pages: {
      signIn: '/auth/signin',
      signOut: '/auth/signout',
      error: '/auth/error',
      verifyRequest: '/auth/verify-request',
      newUser: '/auth/new-user'
    }
    ```
    
    Here are a few examples from the official documentation:
    
    1. [Built-in pages](https://authjs.dev/guides/pages/built-in-pages)
        
    2. [Custom sign-in page](https://authjs.dev/guides/pages/signin)
        
    3. [Custom sign-out page](https://authjs.dev/guides/pages/signout)
        
    4. [Custom Error page](https://authjs.dev/guides/pages/error)
        
    
    We will go through a few examples in depth later in this tutorial series.
    

### Environment Variables

We must set a couple of mandatory environment variables:

1. AUTH\_URL: This environment variable is mostly unnecessary with v5 as the host is inferred from the request headers. However, if you are using a different base path, you can set this environment variable as well. For example, `AUTH_URL=`[`http://localhost:3000/web/auth`](http://localhost:3000/web/auth) or `AUTH_URL=`[`https://company.com/app1/auth`](https://company.com/app1/auth)
    
2. CLIENT\_ID and CLIENT\_SECRET: It is recommended to save CLIENT\_ID and CLIENT\_SECRET in the *.env* file and access via `$env/static/private`.
    
3. ISSUER and WELL\_KNOWN: Same strategy as above for these two variables as well.
    

Here are few other [environment variables](https://authjs.dev/getting-started/deployment#environment-variables) that you may find useful.

### Final Setup

With these additional properties, out updated configuration will be:

```typescript
import { AUTH_SECRET, CLIENT_ID, CLIENT_SECRET, ISSUER, WELL_KNOWN } from '$env/static/private';

const config: SvelteKitAuthConfig = {
  providers: [
    Auth0Provider({
      id: 'auth0',
      name: 'Auth0',
      clientId: CLIENT_ID,
      clientSecret: CLIENT_SECRET,
      issuer: ISSUER,
      wellKnown: WELL_KNOWN
    }) as Provider
  ],
  secret: AUTH_SECRET,
  session: {
    strategy: 'jwt',
    maxAge: 3600 
  },
  trustHost: true,
  debug: true,
  logger: {
    error: async (error: any) => {
      console.log('Error trace from SvelteKitAuth:', error);
    }
  }
}
```

We must use this piece of code in the *hooks.server.ts* file. *Why?* Because we want authentication to happen on every network request we make. We want to ensure that protected resources are not accessed if the user is not authenticated. In the SvelteKit ecosystem, hooks act as a middleware that taps request and response. Hence, this is the ideal place where we want our Authentication code to be integrated.

```typescript
// hooks.server.ts

import type { Handle } from '@sveltejs/kit';
import Auth0Provider from "@auth/core/providers/auth0";
import { SvelteKitAuth, type SvelteKitAuthConfig } from '@auth/sveltekit';

const { handle: getAuthConfig } = SvelteKitAuth(async (event) => {
  const config: SvelteKitAuthConfig = {...};
  return config;
});

export const handle = getAuthConfig;
```

`SvelteKitAuth(() => {})` is the callback expose by `SvelteKitAuth` that return three functions:

* *handle*: This must be used within handle function in hooks
    
* *signIn*: User for server side sign-in.
    
* *signOut*: Used for server side sign-out.
    

For the sake of this tutorial, we only need the `handle` callback that will be consumed by our hooks file. We will discuss the other two callbacks in detail in the later part of the tutorial.

Integrating the built-in Auth0 OAuth provider with SvelteKit can significantly streamline the authentication process in your application. By understanding and configuring the essential properties such as `id`, `clientId`, `clientSecret`, and `issuer`, you can ensure a secure and efficient authentication flow. Additionally, leveraging advanced options like `callbacks`, `logger`, and `events` can enhance the developer experience and provide greater control over the authentication process. Setting up the necessary environment variables and placing the authentication code in the `hooks.server.ts` file ensures that protected resources are accessed only by authenticated users. With this setup, you are well-equipped to manage authentication in your SvelteKit application, paving the way for more advanced configurations and custom providers in the [next tutorial](https://blog.aakashgoplani.in/step-by-step-guide-to-using-custom-oauth-provider-with-sveltekitauth).