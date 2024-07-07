---
title: "How to Integrate Multiple OAuth Providers in SvelteKitAuth"
seoTitle: "Integrating Multiple OAuth Providers in SvelteKit"
seoDescription: "Learn to integrate multiple OAuth providers in SvelteKitAuth for enhanced user convenience and flexibility"
datePublished: Sun Jul 07 2024 19:27:49 GMT+0000 (Coordinated Universal Time)
cuid: clyby42bb00000al04rbqak7z
slug: how-to-integrate-multiple-oauth-providers-in-sveltekitauth
tags: auth0, sveltekit, sveltekitauth, authjs

---

In previous articles, we explored how to integrate both [built-in](https://blog.aakashgoplani.in/step-by-step-guide-to-using-built-in-auth0-oauth-provider-with-sveltekitauth) and [custom](https://blog.aakashgoplani.in/step-by-step-guide-to-using-custom-oauth-provider-with-sveltekitauth) providers in SvelteKit. In this section, we'll demonstrate how to configure multiple providers simultaneously.

Let's first discuss the scenarios where such a configuration is beneficial. For instance, offering multiple sign-in options like Google, GitHub, or LinkedIn can enhance user convenience and flexibility.

Here’s how to achieve this:

```typescript
// hooks.server.ts

import type { Handle } from '@sveltejs/kit';
import Auth0Provider from "@auth/core/providers/auth0";
import { SvelteKitAuth, type SvelteKitAuthConfig } from '@auth/sveltekit';
import { VERCEL_SECRET, CLIENT_ID, CLIENT_SECRET, ISSUER, WELL_KNOWN } from '$env/static/private';

const { handle: getAuthConfig } = SvelteKitAuth(async (event) => {
  const config: SvelteKitAuthConfig = {
    providers: [{
      Auth0Provider({
        id: 'auth0',
        name: 'built-in-oauth-provider',
        clientId: CLIENT_ID,
        clientSecret: CLIENT_SECRET,
        issuer: ISSUER
      }),
      id: 'auth1',
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
          redirect_uri: `${event.url.origin}/auth/callback/auth1`
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

export const handle = SvelteKitAuth(config) satisfies Handle;
```

The main differentiating factor is the `id` property. The `id` property uniquely identifies which provider should be used for authentication. The `name` property acts as a label or a detailed description that helps us differentiate the providers from each other.

We can switch the providers based on the `id` example:

```xml
<button on:click={() => signIn('auth0')} >
  <span>Sign In with Built-in Auth0 Provider</span>
</button>

<button on:click={() => signIn('auth1')} >
  <span>Sign In with Custom Auth0 Provider</span>
</button>
```

In conclusion, integrating multiple OAuth providers in SvelteKitAuth significantly enhances user experience by offering diverse sign-in options. This flexibility not only improves user convenience but also broadens the potential user base by accommodating various preferences. By following the outlined steps, developers can seamlessly configure multiple providers, ensuring a smooth and secure authentication process.

Here is the link to the [**GitHub repository**](https://github.com/aakash14goplani/SvelteKitAuth) with the codebase. In the next article, we will learn how to enhance [custom typings with SvelteKitAuth](https://blog.aakashgoplani.in/enhancing-sveltekitauth-with-custom-type-additions).