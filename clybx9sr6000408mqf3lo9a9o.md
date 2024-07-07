---
title: "Step-by-Step Guide to using custom OAuth provider with SvelteKitAuth"
seoTitle: "Custom OAuth with SvelteKitAuth: Step-by-Step"
seoDescription: "Guide for integrating a custom OAuth provider with SvelteKitAuth for extensive authentication customization and flexibility"
datePublished: Sun Jul 07 2024 19:04:17 GMT+0000 (Coordinated Universal Time)
cuid: clybx9sr6000408mqf3lo9a9o
slug: step-by-step-guide-to-using-custom-oauth-provider-with-sveltekitauth
tags: auth0, sveltekit, sveltekitauth, authjs

---

In the [**previous article**](https://blog.aakashgoplani.in/step-by-step-guide-to-using-built-in-auth0-oauth-provider-with-sveltekitauth), we covered the integration of a built-in OAuth provider with SvelteKitAuth. In this article, we will delve into integrating a custom OAuth provider with SvelteKitAuth.

A custom provider can be particularly useful when (a) you require extensive customization in your authentication mechanism, or (b) the authentication provider you're using is not currently supported by Auth.js.

Here is the code snippet for the custom provider. Let us now deep dive into custom providers and the properties that are used within the provider.

```typescript
// hooks.server.ts

import type { SvelteKitAuthConfig } from '@auth/sveltekit';
import { VERCEL_SECRET, CLIENT_ID, CLIENT_SECRET, ISSUER, WELL_KNOWN } from '$env/static/private';

const config: SvelteKitAuthConfig = {
  providers: [{
    id: 'auth0',
    name: 'custom-oauth-provider',
    type: 'oidc',
    client: {
      token_endpoint_auth_method: 'client_secret_post'
    },
    clientId: CLIENT_ID,
    clientSecret: CLIENT_SECRET,
    issuer: ISSUER,
    wellKnown: WELL_KNOWN,
    checks: ['pkce'],
    authorization: {
      url: `${ISSUER}authorize`, // 'http://localhost:4200/authorize
      params: {
        scope: 'openid name email profile',
        redirect_uri: `${event.url.origin}/auth/callback/auth0`
      }
    },
    token: `${ISSUER}oauth/token`,
    userinfo: `${ISSUER}userinfo`
  }],
  secret: VERCEL_SECRET,
  debug: true,
  session: {
    maxAge: 1800 // 30 mins
  }
};

export const handle = SvelteKitAuth(config) satisfies Handle;
```

* `id`, `name`, `clientId`, `clientSecret`, `issuer` and `wellKnown` - these configurations remain the same as they were discussed in the last section. The focus here will be on the `type` property. The `type` property specifies the type of authentication mechanism, allowed values are: "*oidc*", "*oauth*", "*credentials*", and "*email*".
    
* If your provider is OpenID Connect (OIDC) compliant, the recommendation is to use the `wellKnown` option instead. OIDC usually returns an `id_token` from the `token` endpoint. `SvelteKitAuth` can decode the `id_token` to get the user information, instead of making an additional request to the `userinfo` endpoint.
    
* In case your provider is not OIDC compliant, we have the option to customize the configuration by using a combination of the following properties. You can find more information in the [**docs**](https://next-auth.js.org/configuration/providers/oauth).
    
    * **authorization**: This is the URL for authentication. There are two ways to use this option:
        
        1. You can either set `authorization` to be a full URL, like `"https://example.com/oauth/authorization?scope=email"`.
            
        2. Use an object with `url` and `params` like so
            
            ```typescript
             authorization: {
               url: "https://example.com/oauth/authorization",
               params: { scope: "email" }
             }
            ```
            
    * **token:** This is the URL that will fetch token information. There are three ways to use this option:
        
        1. You can either set `token` to be a full URL, like `"https://example.com/oauth/token?some=param"`.
            
        2. Use an object with `url` and `params` like so
            
            ```typescript
             token: {
               url: "https://example.com/oauth/token",
               params: { some: "param" }
             }
            ```
            
        3. Completely take control of the request:
            
            ```typescript
             token: {
               url: "https://example.com/oauth/token",
               async conform(response) {
                 if (response.ok) {
                   const body = await response.clone().json()
                   if (body?.response?.access_token) {
                     return new Response(JSON.stringify(body.response), response)
                   } else if (body?.access_token) {
                     console.warn("Token response conforms to the standard, workaround not needed.")
                   }
                 }
                 return response
               }
             }
            ```
            
    * **userinfo**: A `userinfo` endpoint returns information about the logged-in user. It is not part of the OAuth specification but is usually available for most providers. There are three ways to use this option:
        
        1. You can either set `userinfo` to be a full URL, like `"https://example.com/oauth/userinfo?some=param"`.
            
        2. Use an object with `url` and `params` like so
            
            ```typescript
             userinfo: {
               url: "https://example.com/oauth/userinfo",
               params: { some: "param" }
             }
            ```
            
        3. Completely take control of the request:
            
            ```typescript
             userinfo: {
               url: "https://example.com/oauth/userinfo",
               // The result of this method will be the input to the `profile` callback.
               async conform(response) {
                 if (response.ok) {
                   const body = await response.clone().json()
                   if (body?.response?.access_token) {
                     return new Response(JSON.stringify(body.response), response)
                   } else if (body?.access_token) {
                     console.warn("Token response conforms to the standard, workaround not needed.")
                   }
                 }
                 return response
               }
             }
            ```
            

And that's it! We are ready with our custom OAuth Auth0 provider. Here is the final snippet that we will be using in the *hooks.server.ts* file:

```typescript
import type { Handle } from '@sveltejs/kit';
import { SvelteKitAuth, type SvelteKitAuthConfig } from '@auth/sveltekit';
import { VERCEL_SECRET, CLIENT_ID, CLIENT_SECRET, ISSUER, WELL_KNOWN } from '$env/static/private';

const { handle: getAuthConfig } = SvelteKitAuth(async (event) => {
  const config: SvelteKitAuthConfig = {
    providers: [{
      id: 'auth0',
      name: 'custom-oauth-provider',
      type: 'oidc',
      client: {
        token_endpoint_auth_method: 'client_secret_post'
      },
      clientId: CLIENT_ID,
      clientSecret: CLIENT_SECRET,
      issuer: ISSUER,
      wellKnown: WELL_KNOWN,
      checks: ['pkce'],
      authorization: {
        url: `${ISSUER}authorize`, // 'http://localhost:4200/authorize
        params: {
          scope: 'openid name email profile',
          redirect_uri: `${event.url.origin}/auth/callback/auth0`
        }
      },
      token: `${ISSUER}oauth/token`,
      userinfo: `${ISSUER}userinfo`
    }],
    secret: VERCEL_SECRET,
    debug: true,
    trustHost: true,
    session: {
      strategy: 'jwt',
      maxAge: 1800 // 30 mins
    },
    logger: {
      error: async (error: any) => {
        console.log('Error trace from SvelteKitAuth:', error);
      }
    }
  };
  return config;
});

export const handle = SvelteKitAuth(getAuthConfig) satisfies Handle;
```

In conclusion, integrating a custom OAuth provider with SvelteKitAuth offers flexibility and extensive customization options for your authentication mechanism. By understanding and configuring properties such as `authorization`, `token`, and `userinfo`, you can tailor the authentication process to meet your specific needs. Additionally, the ability to include multiple providers in the same configuration enhances the versatility of your application. With these steps, you are well-equipped to implement a robust and customized authentication system using SvelteKitAuth.

Here is the link to the [GitHub repository](https://github.com/aakash14goplani/SvelteKitAuth) with the codebase. In the next article, we will go through the steps on [configuring multiple providers with SvelteKitAuth](https://blog.aakashgoplani.in/how-to-integrate-multiple-oauth-providers-in-sveltekitauth).