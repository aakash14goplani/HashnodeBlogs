---
title: "Streamlining Client-Side Sign-In and Sign-Out Processes"
seoTitle: "Optimizing User Sign-In and Sign-Out on client side"
seoDescription: "Enhance client-side sign-in and sign-out processes using SvelteKitAuth for a seamless user experience with flexible authentication management"
datePublished: Sun Jul 07 2024 19:59:10 GMT+0000 (Coordinated Universal Time)
cuid: clybz8czj00060al092r4bmcs
slug: streamlining-client-side-sign-in-and-sign-out-processes
tags: auth0, sveltekit, sveltekitauth, authjs

---

In the previous articles, we covered the basics of SvelteKitAuth and learned how to configure OAuth applications using both built-in and custom OAuth providers. In this article, we will focus on enhancing the user's sign-in and sign-out experience.

There are two ways to initiate the authentication flow: client-side and server-side. In this article, we will go through a step-by-step approach for client-side authentication.

We have two methods, `signIn()` and `signOut()`, from `@auth/sveltekit/client` to perform client-side sign-in and sign-out actions, respectively.

### Sign-in Flow

`signIn()` is the client-side method to initiate a sign-in flow or send the user to the sign-in page listing all possible providers. It automatically adds the CSRF token to the request.

Use the `signIn()` method with the following properties:

* **providerId**: This is the "id" property that we specified in the previous sections. This is optional; if omitted, it defaults to the first id property specified in the SvelteKit configuration.
    
* **options**: This is an optional property where we can specify the *callbackURL*, i.e., the URL to which the user should be redirected once sign-in is successful. In some cases, you might want to handle the sign-in response on the same page and disable the default redirection. For example, if an error occurs (like wrong credentials given by the user), you might want to handle the error on the same page. For that, you can pass `redirect: false` in the second parameter object.
    
* **Additional parameters**: It is also possible to pass additional parameters to the `/authorize` endpoint through the third argument of `signIn()`.
    

Although this is more than enough, if you still want to dive deeper into more options, you can read more configuration details in the [**official documentation**](https://next-auth.js.org/getting-started/client#redirects-to-sign-in-page-when-clicked).

```xml
<script lang="ts">
  import { signIn } from '@auth/sveltekit/client';
</script>

<button on:click={() => signIn(
  'auth0', {
    redirect: false,
    callbackUrl: 'http://localhost:4000/about'
  },
  {
    scope: 'api openid profile email'
  }
)}>Sign In with Auth0</button>
```

### Sign-out Flow

The `signOut()` method logs the user out by removing the session cookie. It automatically adds the CSRF token to the request.

Like the `signIn()` method, you can pass a *callbackURL* and *redirect* option. More details are available in the [**official documentation**](https://next-auth.js.org/getting-started/client#signout).

```xml
<script lang="ts">
  import { signOut } from '@auth/sveltekit/client';
</script>

<button on:click={() => signOut()} class="button">Sign out</button>
<!-- OR -->
<button on:click={() => signOut({
  redirect: true,
  callbackUrl: 'url-post-logout'
})} class="button">Sign out with optional params</button>
```

**NOTE**: If you don't provide the *callbackUrl* option, it will redirect you to the page that initiated the sign-in request.

In this article, we explored the client-side sign-in and sign-out processes using SvelteKitAuth. By leveraging the `signIn()` and `signOut()` methods from `@auth/sveltekit/client`, we can streamline the authentication flow, providing a seamless user experience. Whether specifying a provider, handling callbacks, or managing errors, these methods offer flexibility and control.

Here is the link to the [**GitHub repository**](https://github.com/aakash14goplani/SvelteKitAuth) with the codebase. In the next section, we will go through server-side sign-in and sign-out flow.