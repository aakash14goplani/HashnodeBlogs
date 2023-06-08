---
title: "Get List of all pages in SvelteKit"
seoTitle: "Get List of all pages in SvelteKit."
seoDescription: "Get List of all pages in SvelteKit. Create NAvigation bar in SvelteKit."
datePublished: Thu Jun 08 2023 20:54:04 GMT+0000 (Coordinated Universal Time)
cuid: clinm9hd6000108l4arfn5mru
slug: get-list-of-all-pages-in-sveltekit
tags: navigation, sveltekit

---

Currently, SvelteKit (v1.20.2) gives us a list of build files but not any list of pages. It would be great if there was a way to get the list of pages. The solution for this is to import all the `+page.svelte` files from `src/routes` and generate a list out of it. One use case of this could be to generate a **Navigation Bar**.

Here is how we can do that:

**Folder Structure**

```bash
lib
|--components
|  |--Navigation.svelte
src
|--routes
|  |--(app)
|  |  |--profile
|  |  |  |--admin
|  |  |  |  +layout.svelte
|  |  |  |  +page.svelte
|  |  |  |--user
|  |  |  |  +layout.svelte
|  |  |  |  +page.svelte
|  |  |  +layout.svelte
|  |  |  +page.svelte
|  |  |--settings
|  |  |  |--[country]
|  |  |  |  |--[language]
|  |  |  |  |  +layout.svelte
|  |  |  |  |  +page.svelte
|  |  |  |--general
|  |  |  |  +layout.svelte
|  |  |  |  +page.svelte
|  |  |  +layout.svelte
|  |  |  +page.svelte
|  |  +layout.svelte
|  |--(reset)
|  |  |--about
|  |  |  |--about
|  |  |  |  +layout.svelte
|  |  |  |  +page.svelte
|  |  +layout.svelte
|  +layout.svelte
|  +page.svelte
|  +error.svelte
```

***src/routes/+layout.svelte***

```svelte
<script lang="ts">
  import Navigation from '$lib/components/Navigation.svelte';

  const modules = import.meta.glob('./**/+page.svelte');
</script>

<Navigation { modules } />
```

***src/lib/components/Navigation.svelte***

```svelte
<script lang="ts">
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
        <a href={item.link}>{item.title}</a>
      </li>
    {/each}
  </ul>
</div>
```

**Output**

```json
[
    {
        "title": "profile",
        "link": "/profile"
    },
    {
        "title": "admin",
        "link": "/profile/admin"
    },
    {
        "title": "user",
        "link": "/profile/user"
    },
    {
        "title": "settings",
        "link": "/settings"
    },
    {
        "title": "language",
        "link": "/settings/country/language"
    },
    {
        "title": "general",
        "link": "/settings/general"
    },
    {
        "title": "about",
        "link": "/about"
    },
    {
        "title": "organization",
        "link": "/about/organization"
    },
    {
        "title": "product",
        "link": "/about/product"
    },
    {
        "title": "later",
        "link": "/delete/later"
    },
    {
        "title": "home",
        "link": "/"
    }
]
```

### Reference

* The original idea behind this snippet is from a Youtube channel [Web Jedda](https://youtu.be/Y_NE2R3HuOU)
    
* Alternate approach from [GitHub thread](https://github.com/sveltejs/kit/issues/923#issuecomment-1567052262).