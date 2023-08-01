---
title: "Avoid shared state on the server in SvelteKit"
seoTitle: "State Management in SvelteKit"
seoDescription: "How to use stores in SvelteKit. Avoid shared state on server in SvelteKit. How to use stores in SSR flow in SvelteKit. Data Leak in SvelteKit in SSR flow."
datePublished: Sun Jul 09 2023 17:27:47 GMT+0000 (Coordinated Universal Time)
cuid: cljvpjlzf000i09jqecjj64bd
slug: avoid-shared-state-on-the-server-in-sveltekit
tags: ssr, state-management, sveltekit

---

State management is a crucial part when working with complex web applications. Svelte does provide us with elegant native stores that can be used in such scenarios. However, we must be cautious while using them otherwise our application may result in unwanted behavior and could produce bugs that are difficult to trace and fix!

One such example is using stores in the backend i.e. in the SSR flow. This article will focus on a few edge cases where we must be extremely cautious in the way we use stores or rather state management in general.

### What is the problem?

**Using stores in the backend (SSR mode) causes data leaks between clients.**

**But Why?**

Servers are stateless i.e. one common space on the server is shared by multiple clients (users). In other words, the state on Server is global by default that will be shared by all of its clients. The servers are often long-lived and shared by multiple users. For that reason, it's important not to store data in shared variables. For example, consider the following scenario:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688891933795/8861edca-c071-469d-9e2e-df40f070168e.png align="center")

Multiple users have been logged into the system and they are interacting with a common application server. Now each user interacts with the server, independently, via the browser. **A store is contextual to each instance of your app**. This essentially means that the state on the client side is always stateful. If we save any store value on the client side for a particular user, it will remain local to that user and will not be shared with other users.

On the other hand, the state of the server is shared with all the users simultaneously and hence if user "A" updates the store value, that will be reflected on user "B" as well, thus leaking data! In addition, when user "A" returns to the site later in the day, the server may have restarted, losing their data. **The main thing to understand is that as soon as you create a store, it becomes global server-side in an SSR context (= your store is a singleton in memory server-side, so it is shared by all HTTP requests hitting your server).**

### How to fix this problem?

***DO NOT USE STORES ON THE SERVER SIDE AT ALL***.

* Avoid using stores in the endpoint (*+server.ts* files) and in any particular [server-specific actions](https://kit.svelte.dev/docs/form-actions). If you still want to pass some custom data, check out the `event.locals` It takes care of this and is one simple way to share data on the server side that is unique to the client session. `event.locals` are what their name says, they are states localized to the lifespan of individual server-request events. They are independent, i.e. they are not persisted across requests (but they are persisted across handlers running in the context of the same request, i.e. handlers sequenced by your handle hook.
    
* Avoid using stores in [universal and server load functions](https://kit.svelte.dev/docs/load#universal-vs-server). You could technically make an exception for a universal load function where you check for the `browser` environment, but in reality, if you have the correct configuration, everything you need should be available in the `$page` variable.
    

Here are a few alternatives using which we can mitigate this problem:

**(1) Avoid side effects in the load functions.**

* This is an example from [SvelteKit's documentation](https://kit.svelte.dev/docs/state-management#no-side-effects-in-load) on how we should avoid setting store values within the load function. Instead, we can return data from the load function and let the client handle data accordingly.
    
* AVOID DOING THIS:
    
    ```typescript
    export async function load({ fetch }) {
      const response = await fetch('/api/user');
      // NEVER DO THIS!
      user.set(await response.json());
    }
    ```
    
* INSTEAD DO THIS:
    
    ```typescript
    export async function load({ fetch }) {
      const response = await fetch('/api/user');
      // INSTEAD DO THIS!
      return {
        user: await response.json()
      }
    }
    ```
    
    ```svelte
    <script lang="ts">
      export let data: PageData;
    
      // APPROACH - 1
      onMount(() => {
        user.set(data.user);
      })
    
      // APPROACH -2
      if (browser) {
        user.set(data.user);
      }
    </script>
    ```
    

**(2) Use context API**

* [SvelteKit uses context API](https://kit.svelte.dev/docs/state-management#using-stores-with-context) behind the scenes for application stores like `$page` `$navigation` etc.
    
* Using context API, the store is attached to the component tree with `setContext`, and when you subscribe you retrieve it with `getContext`
    
* Example: In the file *src/routes/+layout.svelte*
    
    ```svelte
    <script>
      import { setContext } from 'svelte';
      import { writable } from 'svelte/store';
      import type { LayoutData } from './$types';
    
      export let data: LayoutData;
      // Create a store and update it when necessary...
      const user = writable();
      $: user.set(data.user);
      // ...and add it to the context for child components to access
      setContext('user', user);
    </script>
    ```
    
* Now that we have set the context in the parent component, all the direct child components can get the corresponding context. For example, in the file *src/routes/user/+page.svelte*
    
    ```svelte
    <script>
      import { getContext } from 'svelte';
      // Retrieve user store from context
      const user = getContext('user');
    </script>
    
    <p>Welcome {$user.name}</p>
    ```
    
* However, there are a few scenarios that must be taken into consideration while using context API:
    
    * Context API can only be used within components (*.svelte* files), we cannot use them in load functions (or any *.ts* / *.js* files)
        
    * Context API must be used during the component initialization phase only.
        
    * The context is passed to the child components only, you cannot do `getContext` on a sibling component.
        

**(3) Using Custom Stores with slight modifications to ensure stores are not global**

* Another solution is to create a custom store in such a way that it is unique for every client. Although I haven't tried these solutions personally, they are worth sharing:
    
    * [Creating a custom store with $page context](https://github.com/sveltejs/kit/discussions/4339#discussioncomment-2473390) by *Sumit Rai*.
        
    * [Creating a custom sandboxed store](https://github.com/sveltejs/kit/discussions/4339#discussioncomment-3258927) by *Radu*.
        
    * [Creating a custom store with WeakMap](https://github.com/sveltejs/kit/discussions/4339#discussioncomment-5639707) by *Bae Junehyeon.*
        
    * [Creating a custom store and safe variables by isolating the stores across requests, enabling file-based stores to exist](https://github.com/sveltejs/kit/discussions/9878) by *Albert Marashi.*
        

**(4) Using Asynchronous Local Storage**

* If there is a situation where you must preserve state per client then NodeJS native [Asynchronous Local Storage](https://nodejs.org/api/async_context.html#new-asynclocalstorage) can be useful. However, this should be viewed as the last alternative when none of the solutions worked for you.
    

### Will the SvelteKit core team resolve this problem?

As of today, July'23 with SvelteKit v1.22.0, I don't see any roadmap highlighting this issue and that generally means, we as developers, have to use stores cautiously in SSR mode.

There are a couple of threads in GitHub where the community is discussing this issue in detail and the solution they have adopted for overcoming this behavior. I have referenced those issues in the next section.

### References

* [State Management in SvelteKit](https://kit.svelte.dev/docs/state-management#avoid-shared-state-on-the-server) from official SvelteKit documentation.
    
* [Dangerous Store behavior with SSR](https://github.com/sveltejs/kit/discussions/4339) - GitHub issue.
    
* [Implementing safe & secure file-based stores](https://github.com/sveltejs/kit/discussions/9878) - GitHub issue.
    
* [SSR rendered page retains old store value in dev mode](https://github.com/sveltejs/kit/issues/8614) - GitHub issue.
    
* [Clarification on safely using stores in Sveltekit](https://github.com/sveltejs/kit/discussions/6098) - GitHub issue. Very old discussion but worth reading.
    
* [Helper in creating stores](https://github.com/sveltejs/kit/issues/7105) - GitHub issue. Very old discussion but worth reading.
    
* [Svelte Store Shared State - Ensuring no cross-client data leak](https://discord.com/channels/457912077277855764/1107384981921210418) - Discord Chat.
    
* [The correct way to use stores in SvelteKit](https://dev.to/jdgamble555/the-correct-way-to-use-stores-in-sveltekit-3h6i) by Jonathan Gamble.
    
* [Safe SvelteKit stores for SSR](https://dev.to/brendanmatkin/safe-sveltekit-stores-for-ssr-5a0h) by Brendan Matkin.