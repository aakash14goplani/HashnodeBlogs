---
title: "Migration Guide from Routify to SvelteKit Router"
seoTitle: "Routify to SvelteKit migration guide"
seoDescription: "Routify to SvelteKit migration guide. Migrate routing from Routify to SvelteKit"
datePublished: Sat Jun 10 2023 19:22:39 GMT+0000 (Coordinated Universal Time)
cuid: cliqdvn1d01xdemnv3vxf571n
slug: migration-guide-from-routify-to-sveltekit-router
tags: migration, sveltekit, routify

---

### What is the need for migration?

* I had a few personal projects created using Svelte which I am now migrating to SvelteKit.
    
* Svelte did not provide any out-of-box navigation mechanism and hence I was forced to look out for third-party libraries.
    
* SvelteKit, on the other hand, has a built-in mechanism for routing and is constantly evolving for good. Since we have a built-in solution available now, it makes sense to adhere to it.
    

> Before we proceed ahead, I want to stress on this point that Routify is an excellent library. I have used that across many personal and professional projects and the experience has been awesome. The only reason I am migrating from Routify to SvelteKit's built-in router is just my personal opinion which is - I installed Routify only because Svelte didn't have any routing mechanism but SvelteKit does.

### Does Routify support SvelteKit?

* The current version of Routify i.e. v2.18 DOES NOT support SvelteKit.
    
* We will need to upgrade that to [v3-next](https://v3.routify.dev/) and as the name suggests, that is still in beta state.
    
* Just upgrading the package will not be sufficient, we will need to do a couple of more configurations that are specified in their [documentation](https://v3.ci.routify.dev/docs/guide/introduction). I am sure when they release a stable v3 version, they will have a migration guide (from v2 to v3) ready by then.
    

### Process of Migration

Now that we are done with the introduction, let's go through the migration process.

#### Rearrange folder structure

* In Routify, all the pages are placed within the `pages` directory but in SvelteKit, they have to be placed within the `routes` directory.
    
* In SvelteKit, we do not have the facility to reset the layout, the only way to do that is to group the paths with specific layouts. Let's understand this with an example, consider this project structure:
    
    ```bash
    src
       routes
          (reset)
             folders_with_different_layouts
             +layout.svelte
          (main)
             rest_of_folders
             +layout.svelte
       +layout.svelte
       +page.svelte
       +error.svelte
    ```
    
* The root layout file i.e. *src/routes/+layout.svelte* will be inherited by all the routes no matter what. So this file should contain the bare minimum logic (like just an empty file having only `<slot />`)or the code that will be used by all the pages.
    
* The next step will be to create two groups - *(reset)*: that will have a layout that is different from the root one. *(app)*: that will have a layout for pages that does not requires a total reset.
    

#### Rename Files

| ROUTIFY | SVELTEKIT |
| --- | --- |
| *\_folder.svelte* or *\_layout.svelte* | *+layout.svelte* |
| *index.svelte* | *+page.svelte* |
| *\_reset.svelte* | Not possible as discussed in the previous section |
| *\_fallback.svelte* | Needs special handling as discussed in the next section |

#### Special Handling of Error pages

* In Routify, we have *\_fallback.svelte* file which acts as a "catch" for 404 URLs. We can have fallback at the root and per route basis as well.
    
* In SvelteKit, this is not possible. All the 404 are handled by the root error page i.e. *src/routes/+error.svelte*. So the logic must be adjusted to handle 404 at the root level than that of the per-route level.
    
* If we must need to have an error page at the route level, we will have to create a corresponding load function in *layout.ts* or *layout.server.ts* at that route level and throw an error from there. You can read more on this in SvelteKit's official [documentation](https://kit.svelte.dev/docs/routing#error).
    

#### Update Utilities

Routify provides tons of helper functions and on the contrary SvelteKit provides just a few. So we need to adjust the logic accordingly, create new helper functions etc to main the routing consistency. In this section, I will be discussing 9 such utilities/helper methods that I have extensively used in my projects. Based on that, feel free to extend logic for other methods as well.

1. `goto()`
    
    * SvelteKit has a built-in method `goto()`. We have to do 2 changes here is:
        
        * change imports from `import { goto } from '@roxi/routify';` to `import { goto } from '$app/navigation';`
            
        * rename `$goto()` to `goto()`
            
    * One thing that we have to take care of is the way we pass parameters in `goto()`. In Routify, we have, `$goto('url', { ...params } )` & in SvelteKit, we have to pass params as query-parameters `goto('url?a=b&c=d')`
        
2. `isActive()`
    
    * SvelteKit does not provide a utility to check if the link is active or not, we have to write our custom logic for that, which is quite simple using SvelteKit's store. Example:
        
        ```svelte
        {#each menu as item}
          <li>
            <a href={item.link} class:active={$page.url.pathname === item.link}>{item.title}</a>
          </li>
        {/each}
        ```
        
    * Alternatively, I have written a helper function for the same. The only caution we have to take is to pass the value of the `path` properly.
        
        ```typescript
        export function isActive(currentUrl: URL, path: string) {
          if (path === '/') {
            return currentUrl.pathname === path;
          } else {
            return path.match(currentUrl.pathname + '($|\\/)') != null;
          }
        }
        
        // invocation
        const currentUrl = new URL(window.location.href);
        <a href="{link}" class:active={isActive(currentUrl, '/app/settings')}></a> 
        ```
        
3. `url()`
    
    * Before we proceed ahead. let's have a walkthrough on how the navigation works in Routify and SvelteKit. Assume the following folder structure:
        
        ```bash
        pages
           about
              product
                 index.svelte <- We are at this page
              index.svelte
        index.svelte <- We want to navigate here
        ```
        
    * Navigating from `/about` to `/`, in Routify, we have to use `<a href={$url('../../')}>HOME</a>` and `<a href={$url('../')}>ABOUT</a>` whereas in SvelteKit, it would be `<a href="../">HOME</a>` and `<a href="./">ABOUT</a>`
        
    * From the above example, it is evident that in Routify when we want to navigate from page-A to page-B, page-A will be considered as the root page & navigation path should be calculated from that node. Whereas in SvelteKit, *src/routes/+page.svelte* is always considered as the root page and navigation will be calculated from that node.
        
    * Because of this distinction, we will have to remove all occurrences of `$url` and replace with absolute path calculated from root page.
        
    * I was not able to come up with a generic helper function for this use case, if you have any ideas, do mention that in the comment section below!
        
4. `params`
    
    * Let's discuss `params` with the help of an example. Assume the following folder structure:
        
        ```bash
        pages
           [country]
              [currency]
                 index.svelte <- we are here
        ```
        
    * The given URL is: [`https://google.com/in/inr?a=b&c=d`](https://se.com/myschneider/ww/en?a=b&c=d)
        
        * In Routify, `$params` will be an object `{ country: in, currency: inr, a: b, c: d }`
            
        * In SvelteKit, for getting that same output, we will have to use the `$page` store. Example: `$page.params` will be an object `{ country: in, currency: inr }` and `$page.url.searchParams('a')` will output `b` or `$`[`page.url.search`](http://page.url.search) will give us `?a=b&c=d`
            
    * Alternatively, I have written a helper function:
        
        ```typescript
        export function getParams(page: Readable<Page<Record<string, string>, string | null>>) {
          const pageValue = get(page);
          let returnValue = {};
        
          const optionalParams = pageValue.params;
          if (Object.keys(optionalParams).length > 0) {
            returnValue = { ...returnValue, ...optionalParams };
          }
        
          const searchParams = pageValue.url.search;
          if (searchParams.length > 1) {
            const temp = Object.fromEntries(new URLSearchParams(searchParams));
            returnValue = { ...returnValue, ...temp };
          }
        
          return returnValue;
        }
        ```
        
        To invoke this function:
        
        ```typescript
        import { page } from "$app/stores";
        import { getParams } from "$lib/utils/router-helper";
        
        const params = getParams(page);
        ```
        
5. `afterPageLoad()`
    
    * SvelteKit has a built-in method `afterNavigate` which we can import from `$app/navigation` to replace Routify's `$afterPageLoad(...)`
        
6. `beforeUrlChange()`
    
    * SvelteKit has a built-in method `beforeNavigate` which we can import from `$app/navigation` to replace Routify's `$beforeUrlChange(...)`
        
7. `redirect()`
    
    * SvelteKit has a built-in method `redirect` which we can import from `@sveltejs/kit` to replace Routify's `$redirect(...)`
        
    * One thing to keep in mind is, Routify's `$routify("URL")` just has one parameter which is the URL to be redirected to, whereas, in SvelteKit, we will have to provide HTTP status as well, For example, `redirect(302, "URL")`
        
8. `$page`
    
    * SvelteKit has equivalent `$page` store variable.
        
    * In Routify, we mostly use the `$page` to access *meta*, *title* and *parent* property. Since we don't have these properties with the `$page` of SvelteKit, we will have to adjust the logic in those files.
        
    * `title` corresponds to the last fragment of the URL, i.e., if URL = [`https://www.google.com/settings`](https://se.com/myschneider/settings), title = "settings".
        
    * `meta` is something we will have to fetch from the corresponding *layout.ts* or *layout.server.ts* file which I have discussed briefly in the next section.
        
    * `parent` is something we won't be able to compute. Routify maintains a tree hierarchy behind the scenes and such a feature is not available in SvelteKit, so we will have to rewrite the logic in those files.
        
9. `$layout`
    
    * `$layout`, like `$page` is used to access `children` and `meta` properties. We don't have any equivalent functionality in SvelteKit.
        
    * I have discussed briefly `meta` in the next section, now I will walk you through the process of fetching `children` of the given layout:
        
        ```typescript
        export function getChildren(modules: Record<string, () => Promise<unknown>>) {
          let returnValue: Array<{ path: string; title: string }> = [];
        
          for (const [key, value] of Object.entries(modules)) {
            const keyStartIndex = key.indexOf('./') + 1;
            const keyEndIndex = key.indexOf('/+page');
            if (keyEndIndex > keyStartIndex) {
              // +1 to remove `/` from title
              let title = key.substring(keyStartIndex + 1, keyEndIndex);
              if (title.includes('/')) {
                // in case of optional params /[country]/[currency]
                title = title.substring(title.lastIndexOf('/') + 1);
              }
              const tempValue = value.toString();
              const valueEndIndex = tempValue.indexOf('/+page');
        
              let valueStartIndex = 0;
              let path = '';
        
              valueStartIndex = tempValue.includes('/(')
                ? (valueStartIndex = tempValue.indexOf(')/') + 1)
                : (valueStartIndex = tempValue.indexOf('routes') + 7); // routes -> 6, +1 as usual for substring = 7
        
              path = tempValue.substring(valueStartIndex, valueEndIndex);
        
              returnValue = [...returnValue, { path, title }];
            }
          }
        
          return returnValue;
        }
        ```
        
        Invoking this function only with *+layout.svelte* files:
        
        ```svelte
        <script lang="ts">
        import { getChildren } from "$lib/utility/router-helper";
        import { onMount } from "svelte";
        
        onMount(() => {
          const modules = import.meta.glob('./**/+page.svelte');
          const children = getChildren(modules);
        })
        </script>
        ```
        
        The output will be something like the below for the project structure:
        
        ```bash
        src
          routes
            profile
              admin
                +page.svelte
              user
                +page.svelte
            +page.svelte
            +layout.svelte
        ```
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686416055215/e7905bfb-6b1c-416d-bf59-83afef011802.png align="center")
        
10. `leftover`
    
    * `$leftover` is only used in *\_fallback.svelte* files (in Routify whose equivalent is *+error.svelte* in SvelteKit). The value of `$leftover` is the unused part in the URL.
        
    * Unfortunately, we don't have any such functionality in SvelteKit and the rough part is that we won't be able to create a utility/helper for this due to the reason SvelteKit handles 404 pages.
        
    * Assume the following folder structure:
        
        ```svelte
        src
           pages
              admin
                 user
                    index.svelte
                    _fallback.svelte
                 index.svelte
                 _fallback.svelte
              index.svelte
              _fallback.svelte
        ```
        
    * If we enter the URL `/admin/user/foo-bar`, in Routify, *\_fallback* within the *user* will be invoked and the value of `$leftover` will be "foo-bar". Similarly, for URL `/admin/foo-bar`, *\_fallback* within *admin* will be invoked and the value of `$leftover` will be "foo-bar" and so on...
        
    * In SvelteKit, if 404 occurs, control always goes to *src/routes/+error.svelte*, there is no concept of redirecting users to specific error pages in 404 scenarios and hence implementing a helper/utility for this feature is not possible. If you come across any hack, do let me know in the comments section!
        

#### Breadcrumb and Dynamic Navigation

In Routify, to generate the breadcrumbs and a Navigation bar, we have traversed the `$page.parent` and `$layout.children` in a recursive manner. As this is not possible in SvelteKit, I have written a [blog post](https://blog.aakashgoplani.in/generate-breadcrumb-and-navigation-in-sveltekit) showcasing how to generate breadcrumbs and dynamic navigation.

#### Meta Tags

* Finally coming to the last section :) that is Meta tags. In Routify, we specify meta options as `<!--routify:options key=value -->` and access them in CLOSEST layout or page file as `$layout.meta` or `$page.meta`
    
* In SvelteKit, we can export an object from either *layout.ts*, *layout.server.ts*, *page.ts*, or *page.server.ts* and access them in the CLOSEST layout or page file. Example:
    
    ```typescript
    // file layout.ts, layout.server.ts, page.ts, or page.server.ts
    return {
       title: 'Hello World',
       category: 'Blog',
    }
    ```
    
    Access the meta values in the CLOSEST *+layout.svelte* or *+page.svelte* file as:
    
    ```typescript
    export let data;
    const pageCategory = data.category; // outputs "Blog"
    ```
    

### Wrapping Up

That's all folks! If you have any ideas/suggestions to add, do let me know in the comments section!