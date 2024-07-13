---
title: "Managing Shared Sessions Across Multiple Applications in SvelteKitAuth"
seoTitle: "Shared Sessions in SvelteKitAuth Apps"
seoDescription: "Learn how to effectively manage shared sessions across multiple applications using SvelteKitAuth, covering both same and different domain scenarios"
datePublished: Sat Jul 13 2024 20:59:45 GMT+0000 (Coordinated Universal Time)
cuid: clykm1ed700060aig6p9aamc1
slug: managing-shared-sessions-across-multiple-applications-in-sveltekitauth
tags: auth0, sveltekit, sveltekitauth, preview-url, preview-domain

---

In this article, we will explore how to manage shared sessions across multiple applications using SvelteKitAuth. We will cover scenarios where applications are hosted on the same domain as well as on different domains, providing practical examples and configurations to achieve seamless session sharing.

### Sharing session with applications on the same domain

Let's take an example where my application is hosted on `myapp.com` and we perform authentication on `auth.myapp.com`. In this example, the domain `myapp.com` is shared among two applications, making it easy to share the session between them. The session is stored in the `session-token` cookie. If the applications share the same domain, we can simply update the *domain* property in the cookies. This is the example from the `SvelteKitAuthConfig`:

```typescript
// hooks.server.ts

import { dev } from '$app/environment';
import type { Handle } from '@sveltejs/kit';
import { SvelteKitAuth, type SvelteKitAuthConfig } from '@auth/sveltekit';

const { handle: getAuthConfig } = SvelteKitAuth(async (event) => {
  const useSecureCookies = !dev;
  const config: SvelteKitAuthConfig = {
    ...
    cookies: {
      callbackUrl: {
        name: `${useSecureCookies ? '__Secure-' : ''}authjs.callback-url`,
        options: {
          httpOnly: true,
          sameSite: useSecureCookies ? 'none' : 'lax',
          path: '/',
          secure: useSecureCookies,
          domain: '.myapp.com'
        }
      },
      pkceCodeVerifier: {
        name: `${useSecureCookies ? '__Secure-' : ''}authjs.pkce.code_verifier`,
        options: {
          httpOnly: true,
          sameSite: useSecureCookies ? 'none' : 'lax',
          path: '/',
          secure: useSecureCookies,
          domain: '.myapp.com'
        }
      },
      sessionToken: {
        name: `${useSecureCookies ? '__Secure-' : ''}authjs.session-token`,
        options: {
          httpOnly: true,
          sameSite: useSecureCookies ? 'none' : 'lax',
          path: '/',
          secure: useSecureCookies,
          domain: '.myapp.com'
        }
      },
      state: {
        name: `${useSecureCookies ? '__Secure-' : ''}authjs.state`,
        options: {
          httpOnly: true,
          sameSite: useSecureCookies ? 'none' : 'lax',
          path: '/',
          secure: useSecureCookies,
          domain: '.myapp.com'
        }
      }
    },
  }
});

export const handle = getAuthConfig;
```

With this simple setup, we can share the session between two applications having a common domain. In the next section, we will learn how to share sessions with multiple applications that do not share a common domain.

### Sharing session with applications in different domains

Sharing sessions with applications on different domains is very tricky to implement in Authjs as they don't allow having a dynamic `callbackURL`. When we configure the `SvelteKitAuthConfig` object, we don't have any option to manually set the `callbackURL`. It is automatically set by Authjs and defaults to `${url.origin}/${base_path}/auth/callback/${provider-id}` (e.g., `http://localhost:4200/base/auth/callback/auth0`). Since it always picks up the current origin, we cannot customize the `callbackURL`, resulting in an authentication error:

```typescript
error {
  error: 'unauthorized_client',
  error_description: 'The redirect URI is wrong. You sent http://localhost:4201, and we expected http://localhost:4200'
}
ERROR in AUTH:  {
  "name":"CallbackRouteError",
  "type":"CallbackRouteError",
  "kind":"error",
  "message":"Read more at https://errors.authjs.dev#callbackrouteerror"
}
```

The solution is to use our own proxy identity server and configure it with the `redirectProxyURL` property. The `redirectProxyURL` property in Auth.js is used to specify a proxy server that handles the redirection process during authentication. This is particularly useful when dealing with scenarios where applications are hosted on different domains and you need to manage cross-domain authentication. By setting the `redirectProxyURL`, you can centralize the callback handling to a single domain, which then forwards the authentication response to the appropriate application. This helps in overcoming the limitation of having a dynamic `callbackURL` in Auth.js.

Let's understand how this works **for the application on the same domain**:

* It makes a request to the OAuth provider by passing the authorization URL and a whitelisted `redirectURL`.
    
* The OAuth provider validates the authentication and passes the authorization code to the application via the `redirectURL` that was provided earlier.
    
* In this scenario, since the application was performing and maintaining the session in the same domain, the auto-generated `callbackURL` by Authjs is the same as the whitelisted `redirectURL` provided by us, and hence authentication was successful.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720899260992/a58d9fd6-1ad7-4552-a812-b78d88962ab4.png align="center")

**For the application on a different domain**:

* It makes a request to the OAuth provider by passing the authorization URL and a `redirectURL` that is not whitelisted.
    
* Since the auto-generated `callbackURL` by Authjs is NOT the same as the whitelisted `redirectURL` provided by us, authentication fails.
    
* To correct this, we must first create a temporary Identity server that will impersonate our application and interact with the OAuth provider.
    
* We will first whitelist the `redirectURL` of this temporary identity server so that it is able to communicate seamlessly with the OAuth provider.
    
* Our application passes the authorization request to this identity server instead of the actual OAuth. In the authorization request, it passes the `source_url`, which is the current application URL that triggers the authentication process. Our fake identity server will give back control to this particular URL.
    
* Apart from passing the authorization request, our application also sets the `redirectProxyURL` property, which holds the value of the origin of the fake identity server.
    
* The identity server then interacts with the OAuth provider and requests authentication.
    
* The OAuth server validates the request and sends back the authorization code via the whitelisted `redirectURL` property of the fake identity server.
    
* Once the authorization code is received, the fake identity server transfers back control to our application. It picks up the `source_url` that was passed earlier.
    
* Since the `redirectProxyURL` property was set, our application realizes that the authentication was performed via a proxy source and it lets the session continue on our application.
    

#### Creating Identity Server

The identity server must be configured keeping the pattern of `authorizeURL` and the whitelisted `redirectURL` in mind. For example, let's consider the following pattern for our OAuth provider where the value of the `authorizeURL` is `https://oauth-provider/authorize` and for the whitelisted `redirectURL` is `https://my-app/auth/callback/oauth`. The folder structure of our new application will be:

```apache
routes/
├── authorize/
│   └── +page.svelte
└── auth/
    └── callback/
        └── oauth/
            └── +page.svelte
```

Consider the value of the origin of our identity server is `https://oauth-identity.com/`. Our application will make a call to `https://oauth-identity.com/authorize?source_url=https://my-app.com/`. Please note that we are passing the `source_url` where we expect the control back from our identity server once the authentication is done.

The authorization code in the identity server will do the following:

* It will save the `source_url` in the browser's local storage, where the key could be a random string like "state" and the value will be `{ redirect_uri: https://my-app.com }`.
    
* It will then invoke the OAuth provider's authorization URL by passing in the whitelisted URL of the fake identity server.
    

Once the authentication is completed, the control will come back to our identity server's whitelisted URL, i.e., `https://oauth-identity.com/auth/callback/oauth/+page.svelte`. This file will perform the following operations:

* It will pick the `source_url` from the browser's local storage that was saved in the previous step.
    
* It will append `/auth/callback/<provider-id>` as a suffix to the `source_url` so that when the control is passed back to our application, Authjs is able to recognize this callback pattern.
    
* It will append the authorization code that was received from the OAuth and send back control to our application, i.e., `https://my-app.com/auth/callback/auth0?code=abcd1234`.
    

Finally, control comes back to our application, and since the `redirectProxyURL` property was configured, Authjs will realize that the authentication was performed by a proxy server. It will pick up the authorization code and make a token and userinfo call and proceed ahead to create and save the `session-token` cookie.

Here is the gist of the `SvelteKitAuthConfig` object:

```typescript
// hooks.server.ts

import { dev } from '$app/environment';
import type { Handle } from '@sveltejs/kit';
import { SvelteKitAuth, type SvelteKitAuthConfig } from '@auth/sveltekit';

const { handle: getAuthConfig } = SvelteKitAuth(async (event) => {
  const useSecureCookies = !dev;
  const config: SvelteKitAuthConfig = {
    providers: [
      id: 'auth0',
      authorization: {
        url: `https://sveltekit-auth-identity-server.vercel.app/authorize?source_url=${event.url.origin}`,
        params: {
          scope: 'openid name email profile',
          redirect_uri: `https://sveltekit-auth-identity-server.vercel.app/auth/callback/auth1`,
        }
      },
      token: `${ISSUER}oauth/token`,
      userinfo: `${ISSUER}userinfo`,
      redirectProxyUrl: 'https://sveltekit-auth-identity-server.vercel.app/auth'
    ]
  }
});

export const handle = getAuthConfig;
```

While working on the local machine, you can run code in two IDEs: one will execute the code of the main application, say, on port 4201, and one will execute the code of the identity server on port 4200. In that use case, you can configure:

```typescript
authorization: {
  url: `http://localhost:4200/authorize?source_url=${event.url.origin}`,
  params: {
    scope: 'openid name email profile',
    redirect_uri: `http://localhost:4200/auth/callback/auth0`
  }
},
redirectProxyUrl: 'http://localhost:4200/auth'
```

With this configuration, we are able to share sessions among multiple applications spread across different domains.

### Conclusion

In conclusion, managing shared sessions across multiple applications using SvelteKitAuth can be effectively achieved through careful configuration and understanding of domain-specific challenges. For applications sharing the same domain, a straightforward update to the cookie's domain property ensures seamless session sharing. However, for applications on different domains, implementing a proxy identity server becomes essential. This server handles the redirection process, allowing cross-domain authentication by centralizing callback handling. By following the outlined steps and configurations, developers can ensure secure and efficient session management across diverse application environments.

Here is the link to the GitHub repository with the codebase for the [main application](https://github.com/aakash14goplani/SvelteKitAuth/tree/proxy-identity-config) and the one with the [identity application](https://github.com/aakash14goplani/SvelteKitAuth-Authorize-Server). Here is the [link](https://sveltekit-auth-preview.vercel.app/) to view a live demo of this use case. In the next article, we will explore the shortcomings or the [limitations of the SvelteKitAuth](https://blog.aakashgoplani.in/drawbacks-of-sveltekitauth-you-should-know).