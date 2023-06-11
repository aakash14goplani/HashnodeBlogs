---
title: "SvelteKitAuth with Salesforce OAuth provider"
seoTitle: "SvelteKit authentication with SvelteKitAuth and Salesforce OAuth provi"
seoDescription: "SvelteKit authentication with SvelteKitAuth and Salesforce OAuth provider. Authentication in SvelteKit using SvelteKitAuth and Salesforce."
datePublished: Sun Jun 04 2023 10:25:47 GMT+0000 (Coordinated Universal Time)
cuid: cliha23oe000309jt3q1icgt2
slug: sveltekitauth-with-salesforce-oauth-provider
tags: salesforce, sveltekit, sveltekitauth

---

Before we proceed ahead with this article, I would recommend reading my [previous blog post](https://blog.aakashgoplani.in/sveltekit-authentication-using-sveltekitauth-and-oauth-providers-a-comprehensive-guide) which gives a detailed idea of what SvelteKitAuth is, how it works and basic configuration details. In this blog post, we will go through the configuration for the Salesforce OAuth provider.

We will begin this post by installing the required dependencies:

```json
{
  "@auth/core": "latest",
  "@auth/sveltekit": "latest"
}
```

Once we have the required dependencies installed, we will go through two configurations i.e.,

1. **Configuring using a built-in provider:** SvelteKitAuth provides [60+ built-in](https://github.com/nextauthjs/next-auth/tree/main/packages/next-auth/src/providers) providers. If our requirement regarding the configuration is straightforward or in other words does not require any customization, we can select this approach.
    
2. **Configuration using the custom provider**: If our requirement needs some tweaks i.e., we have a custom URL setup for fetching tokens or user details, we will select custom providers.
    

### Working with a built-in provider

SvelteKitAuth provides a built-in [*SalesforceProvider*](https://github.com/nextauthjs/next-auth/blob/main/packages/next-auth/src/providers/salesforce.ts). Let's walk through the required configuration:

```typescript
import SalesforceProvider from "@auth/core/providers/salesforce";
...
providers: [
  SalesforceProvider({
    clientId: import.meta.env.VITE_CLIENT_ID,
    clientSecret: import.meta.env.VITE_CLIENT_SECRET,
    wellKnown: 'https://my.salesforce.com/.well-known/openid-configuration'
  })
]
```

With the built-in provider, we need minimal configuration details like **clientId**, **clientSecret** and **wellKnown**. With this minimal configuration, SvelteKitAuth will complete the authentication process and fetch the required tokens and user details as mentioned in `wellKnown` config file. I have already covered a detailed explanation of these properties in my [previous article](https://blog.aakashgoplani.in/sveltekit-authentication-using-sveltekitauth-and-oauth-providers-a-comprehensive-guide#heading-configuring-sveltekitauth-built-in-auht0-provider).

In the next section, we will go through the configuration required to set up a custom provider.

### Working with a custom provider

In many scenarios, our organization would have configured dedicated URLs for authorization, token and user information which will not be available in standard `wllKnown` configuration file, in such scenarios we must use custom providers.

```typescript
providers: [{
  id: 'salesforce',
  name: 'Salesforce',
  type: 'oidc',
  client: { token_endpoint_auth_method: 'client_secret_post' },
  clientId: import.meta.env.VITE_CLIENT_ID,
  clientSecret: import.meta.env.VITE_CLIENT_SECRET,
  issuer: 'https://my-custom-issuer-url',
  authorization: {
    url: 'https://my-custom-authorization-url',
    params: {
      app: 'application-name',
      scope: 'refresh_token web openid id api',
      response_type: 'code',
      response_mode: 'query',
      redirect_uri: 'https://my-custom-redirect-url'
    }
  },
  token: 'https://my-custom-token-url',
  userinfo: 'https://my-custom-user-information-url',
  checks: ['pkce']
}]
```

I have already covered a detailed explanation of every property that is used within the custom provider configuration in my [previous article](https://blog.aakashgoplani.in/sveltekit-authentication-using-sveltekitauth-and-oauth-providers-a-comprehensive-guide#heading-configuring-sveltekitauth-custom-provider), in this blog post, I'll cover a few important configurations that are required specifically for the Salesforce custom provider.

1. For the Salesforce custom provider, we have to set the `type` property to **oidc.** For the built-in provider, it is set to **oauth.**
    
2. Next, we have to specify the *client token endpoint authentication method* which is generally "*client\_secret\_post*". This particular configuration is subjective and may or may-not-be required in your use case.
    
3. Finally, the last required property here is `checks` which are set to **pkce**.
    

### Wrapping Up

Now that we have our `providers` configuration ready, we can put all the required configurations together in ***src/hooks.server.ts*** file.

```typescript
import { SvelteKitAuth } from '@auth/sveltekit';
import type { Handle } from '@sveltejs/kit';

const configuration: SvelteKitAuthConfig = {
  providers: [...],
  debug: import.meta.env.VITE_VERCEL_ENV !== 'production',
  secret: import.meta.env.VITE_VERCEL_SECRET,
  session: {...},
  callbacks: {...},
  logger: {...},
  events: {
    async signOut(message) {
	  // revoke token
	  const accessToken = message.token.access_token;
	  await fetch(`https://revoke-token-url?token=${accessToken}`, {
        method: 'POST'
	  })
	}
  }
};

export const handle = SvelteKitAuth(configuration) satisfies Handle;
```

One important point to highlight is revoking the access token as soon as the user logs out. Salesforce by default does not revoke the token, it is our responsibility as a developer to trigger the event (via POST request) and let Salesforce know that the user has logged out and we should revoke the access token.

For the explanation on the rest of the above-mentioned properties and a few of the other important details like callback URL, redirect URL, sign-in, sign-out and managing session all of which I have covered in detail in my [previous article](https://blog.aakashgoplani.in/sveltekit-authentication-using-sveltekitauth-and-oauth-providers-a-comprehensive-guide).