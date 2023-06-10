---
title: "Generate Breadcrumb and Navigation in SvelteKit"
seoTitle: "How to generate breadcrumb and navigation bar in SvelteKit."
seoDescription: "How to generate breadcrumb and navigation bar in SvelteKit."
datePublished: Thu Jun 08 2023 20:54:04 GMT+0000 (Coordinated Universal Time)
cuid: clinm9hd6000108l4arfn5mru
slug: generate-breadcrumb-and-navigation-in-sveltekit
tags: navigation, sveltekit, breadcrumb

---

### Building Dynamic Navigation

As we know that in SvelteKit, `+page.svelte` files point to the actual web page. So, to generate navigation, we need to figure out a way to get the list of all `+page.svelte` files. Currently, SvelteKit (v1.20.2) does not have any built-in mechanism to give us a list of all pages. The solution for this is to import all the `+page.svelte` files from `src/routes` using [Vite's Glob imports](https://vitejs.dev/guide/features.html#glob-import) and generate a list out of it.

Here is how we can do that - In ***src/routes/+layout.svelte*** file, we fetch all the `+page.svelte` files using Vite. The output is in the form of an object with key-value pairs where the key is the path to `+page.svelte` files and value is a Promise object for dynamically importing modules.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686392222038/dc25c6af-5c77-450f-90c5-0636d80aa339.png align="center")

We will pass this object to our Navigation component that will perform a series of operations to generate navigation.

```svelte
<script lang="ts">
  import Navigation from '$lib/components/Navigation.svelte';

  const modules = import.meta.glob('./**/+page.svelte');
</script>

<Navigation { modules } />
```

Let's focus on ***src/lib/components/Navigation.svelte*** file:

```svelte
<script lang="ts">
  import { page } from '$app/stores';
  import { onMount } from 'svelte';

  export let modules: Record<string, () => Promise<unknown>>;
  let menu: Array<{ link: string; title: string }> = [];

  onMount(() => {
    for (let path in modules) {
      let pathSanitized = path.replace('.svelte', '').replace('./', '/');

      // for group layouts
      if (pathSanitized.includes('/(')) {
        pathSanitized = pathSanitized.substring(pathSanitized.indexOf(')/') + 1);
      }

      // for dynamic paths -> needs more triaging
      if (pathSanitized.includes('[')) {
        pathSanitized = pathSanitized.replaceAll('[', '').replaceAll(']', '');
      }

      pathSanitized = pathSanitized.replace('/+page', '');

      menu = [
        ...menu,
        {
          title: pathSanitized
            ? pathSanitized.substring(pathSanitized.lastIndexOf('/') + 1)
            : 'home',
          link: pathSanitized ? pathSanitized : '/'
        }
      ];
    }
  });
</script>

<div>
  <ul>
    {#each menu as item}
      <li>
        <a href={item.link} class:active={$page.url.pathname === item.link}>{item.title}</a>
      </li>
    {/each}
  </ul>
</div>

<style lang="scss">
  .nav-list {
    margin: 0 1rem;
    padding: 1rem;

    ul > li {
      display: inline-block;
    }

    ul, li {
      margin: 0;
      padding: 0;
    }

    a {
      padding: 1rem;
      color: red;
      text-decoration: none;

      &.active {
        color: blue;
      }
    }
  }
</style>
```

* This component iterates over the object and generates an array link & title.
    
    ```typescript
    [
      {
        "title": "profile",
        "link": "/profile"
      },
      ...
    ]
    ```
    
* In this process, it ignores the *group layouts* and *dynamic paths.* Now, this step is quite opinionated, one could override logic according to their requirements. Do let me know how you plan to deal with group and dynamic paths in the comments section!
    

With this configuration, our Navigation Bar is ready. You can see the demo at [Stackblitz](https://stackblitz.com/edit/stackblitz-starters-aev7ds). In the next section, we will work our way through Breadcrumbs!

### Building Breadcrumbs

SvelteKit uses a filesystem-based router. Files named `+layout.svelte` determines the layout for the current page and pages below itself in the file hierarchy. We can use SvelteKit’s store `page` to determine what the current path is and pass that location to a breadcrumb component which we add to the layout.

***src/routes/+layout.svelte***

```svelte
<Breadcrumb path={$page.url.pathname} />
```

***src/lib/components/Breadcrumb.svelte***

```svelte
<script lang="ts">
  export let path: string;
  let crumbs: Array<{ label: string, href: string }> = [];

  $: {
    // Remove zero-length tokens.
    const tokens = path.split('/').filter((t) => t !== '');

    // Create { label, href } pairs for each token.
    let tokenPath = '';
    crumbs = tokens.map((t) => {
      tokenPath += '/' + t;
      t = t.charAt(0).toUpperCase() + t.slice(1);
      return { label: t, href: tokenPath };
    });

    // Add a way to get home too.
    crumbs.unshift({ label: 'Home', href: '/' });
  }
</script>

<div class="breadcrumb">
  {#each crumbs as c, i}
    {#if i == crumbs.length - 1}
      <span class="label">
        {c.label}
      </span>
    {:else}
      <a href={c.href}>{c.label}</a> &gt;&nbsp;
    {/if}
  {/each}
</div>

<style lang="scss">
  .breadcrumb {
    margin: 0 1.5rem;
    padding: 1rem 2rem;

    a {
      display: inline-block;
      color: red;
      padding: 0 0.5rem;
    }

    .label {
      padding-left: 0.5rem;
      color: blue;
    }
  }
</style>
```

### Wrapping Up

* You can see the demo at [Stackblitz](https://stackblitz.com/edit/stackblitz-starters-aev7ds).
    
* I would like to give some credit here, the idea behind generating Navigation using Vite is from a Youtube video by [Web Jeda](https://youtu.be/Y_NE2R3HuOU) and the idea for generating Breadcrumb using SveleKit's store is from [Dean Fogarty's blog](https://df.id.au/technical/svelte/breadcrumbs/).