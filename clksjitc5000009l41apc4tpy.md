---
title: "Internationalization in SvelteKit with svelte-i18n"
seoTitle: "Internationalization in SvelteKit with svelte-i18n"
seoDescription: "Internationalization in SvelteKit with svelte-i18n, i18n in SvelteKit, Localization in SvelteKit, Translations in Sveltekit, svelte-i18n"
datePublished: Tue Aug 01 2023 16:55:36 GMT+0000 (Coordinated Universal Time)
cuid: clksjitc5000009l41apc4tpy
slug: internationalization-in-sveltekit-with-svelte-i18n
tags: internationalization, i18n, localization, sveltekit, svelte-i18n

---

This is the first article of [four-part series](https://blog.aakashgoplani.in/series/i18n-in-sveltekit) to demonstrate i18n in SvelteKit. In this article, we will work on integrating *svelte-i18n* with SvelteKit.

*svelte-i18n* helps you localize your app using the reactive tools Svelte provides. By using stores to keep track of the current `locale`, `dictionary` of messages and `format` messages, it keeps everything neat, in sync and easy to use on svelte files.

Under the hood, *svelte-i18n* uses [formatjs](https://formatjs.io/) for localizing messages. It allows *svelte-i18n* to support the ICU message syntax.

### Application Structure

Before we begin, I want to give you a detailed walkthrough of the application that we will be working on. You can find the code in the [GitHub repository](https://github.com/aakash14goplani/sveltekit-with-sveltei18n).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690286047702/54c0dc32-f5c3-453e-85b0-7f986af6c5d7.png align="center")

As we can see from the image, this will be a simple single-page application demonstrating i18n features like *locale switching*, *pluralization* and *formatting*.

In the next section, we will work on integrating the *svelte-i18n* library with SvelteKit.

### Integration with SvelteKit

1. **Install *svelte-i18n* library**
    
    * This first step is to install this library as a dependency in our SvelteKit project:
        
        ```typescript
        pnpm i svelte-i18n
        ```
        
    * \[OPTIONAL\] If you're using `VSCode` and want to have your messages previewed alongside your components, check out the [i18n-ally](https://github.com/antfu/i18n-ally) and their [FAQ](https://github.com/antfu/i18n-ally/wiki/FAQ) to see how to set it up.
        
2. **Define Locale Dictionary**
    
    * Now that we have installed this library, it's time to identify the areas in our application that need localization. We then define the **locale dictionaries** within the *src/lib/lang* folder. A locale dictionary is a regular JSON object which contains message definitions for a certain language.
        
    * In our example application, I want five fields to be localized: The main heading, Lable for locale switching, Button label text, Body text (the paragraph) and Text for pluralization.
        
    * We also have the requirement for formatting date, time and currency but for that, we don't need a special entry in the locale dictionary. We will implement the same using inbuild methods provided by the *svelte-i18n* for formatting.
        
    * For the sake of this application, I will define dictionaries in three languages - English (*en.json*), Hindi (*hi.json*) and French (*fr.json*) within the *src/lib/lang* folder.
        
        ```json
        // en.json
        {
          "heading": "Internationalization in SvelteKit",
          "toggle_label": "Select Locale",
          "button_label": "Generate Awards",
          "body_text": "This is a small example to demonstrate i18n functionality in SvelteKit using svelte-i18n library. svelte-i18n helps you localize your app using the reactive tools Svelte provides. By using stores to keep track of the current locale, dictionary of messages and to format messages, we keep everything neat, in sync and easy to use on your svelte files. Total number of npm downloads per week as of {date} are {download}.",
          "awards": "You have {n, plural, =0 { not won any awards } one { won exactly # award } other { won # awards }}!"
        }
        
        // hi.json
        {
          "heading": "SvelteKit में अंतर्राष्ट्रीयकरण",
          "toggle_label": "भाषा चुने",
          "button_label": "पुरस्कार उत्पन्न करें",
          "body_text": "svelte-i18n लाइब्रेरी का उपयोग करके SvelteKit में i18n कार्यक्षमता प्रदर्शित करने के लिए यह एक छोटा सा उदाहरण है। svelte-i18n, Svelte द्वारा प्रदान किए गए प्रतिक्रियाशील टूल का उपयोग करके आपके ऐप को स्थानीयकृत करने में आपकी सहायता करता है। वर्तमान स्थान, संदेशों के शब्दकोश पर नज़र रखने और संदेशों को प्रारूपित करने के लिए स्टोर का उपयोग करके, हम सब कुछ साफ-सुथरा, सिंक में रखते हैं और आपकी विस्तृत फ़ाइलों पर उपयोग में आसान रखते हैं। {date} तक प्रति सप्ताह NPM डाउनलोड की कुल संख्या {download} है|",
          "awards": "आपने {n, plural, =0 { कोई पुरस्कार नहीं जीता } one { बिल्कुल # पुरस्कार जीता } other { # पुरस्कार जीते }} है|"
        }
        
        // etc... all other languages that you wish to support
        ```
        
    * Pay attention to the syntax `{value}` Here `{value}` is a placeholder that will be populated with a value of a particular locale during runtime. As the locale changes, this field will be recomputed.
        
3. **Defining entry point and mode for initializing *svelte-i18n* library**
    
    * Now that our library is installed and the locale dictionary is ready, it is time to create an entry point that will load the assets based on the user locale and initialize the library with a specific locale.
        
    * This entry point will be invoked as soon as the application bootstraps - once on the client side and once on the server side.
        
    * We will create the helper methods in the *src/lib/i18n.ts* file.
        
        ```typescript
        import { browser } from '$app/environment';
        import { init, register } from 'svelte-i18n';
        
        const defaultLocale = 'en';
        
        register('en', () => import('./lang/en.json'));
        register('hi', () => import('./lang/hi.json'));
        register('fr', () => import('./lang/fr.json'));
        
        init({
          fallbackLocale: defaultLocale,
          initialLocale: browser ? window.navigator.language : defaultLocale
        });
        ```
        
    * The above code snippet loads the locale files (*en.json*, *hi.json* and *fr.json*) and registers them with the library. Now when the user switches the locale, the corresponding translation file will be used. We also provide `initialLocale` and a `fallbackLocale.` **Please note that the** `fallbackLocale` **is always loaded, independent of the current locale, since only some messages can be missing**. Let's go through each method in detail.
        
    * [`init()`](https://github.com/kaisermann/svelte-i18n/blob/46b025ceebeb9bd68df0a2f30cc3c0775049ed85/docs/Methods.md#init) is responsible for configuring some of the library behaviors such as the global fallback and initial locales. Must be called before setting a locale and displaying your view.
        
    * [`register()`](https://github.com/kaisermann/svelte-i18n/blob/46b025ceebeb9bd68df0a2f30cc3c0775049ed85/docs/Methods.md#register) adds a new dictionary of messages to a certain locale i.e. it will register your local dictionaries (*en.json* etc.) and get all translation keys ready for you.
        
    * Now there are two ways to register/add dictionaries to your locale:
        
        * the **synchronous way** using [`addMessages()`](https://github.com/kaisermann/svelte-i18n/blob/46b025ceebeb9bd68df0a2f30cc3c0775049ed85/docs/Methods.md#addmessages) that imports your locale JSON files
            
        * the **asynchronous way** using `register()` The asynchronous way is a more performant way to load your dictionaries. This way, only the files registered to the current locale will be loaded. As the locale value changes, it will automatically load the registered loaders for the new locale.
            
4. **Initialize the library on the server side**
    
    * The rule is to initialize the *svelte-i18n* library with a locale as soon as the application boots up. For SSR we need to tell the server what language is being used. This could use *cookies* or the *accept-language* header for example. The easiest way to set the locale is in the server hook.
        
        ```typescript
        import type { Handle } from '@sveltejs/kit';
        import { locale } from 'svelte-i18n';
        
        export const handle: Handle = async ({ event, resolve }) => {
          const lang = event.request.headers.get('accept-language')?.split(',')[0];
          if (lang) {
            locale.set(lang);
          }
          return resolve(event);
        };
        ```
        
    * In the above example, we intercept every request and look for a header with the key "*accept-language*" and set it as the current locale.
        
        > For the sake of explanation, I am keeping things simple and limiting them to query headers only but you can extend this logic to query cookies and URL parameters to compute the locale and set it accordingly!
        
5. **Initialize the library on the client side**
    
    * Once the backend is all wired up, we will now proceed with client-side initialization in the *+layout.ts*
        
        ```typescript
        import { browser } from '$app/environment';
        import '$lib/i18n'; // Import to initialize. Very Important!
        import { locale, waitLocale } from 'svelte-i18n';
        import type { LayoutLoad } from './$types';
        
        export const load: LayoutLoad = async () => {
          if (browser) {
            locale.set(window.navigator.language);
          }
          await waitLocale();
        };
        ```
        
    * In the above code example, as soon as the browser is initialized, we set locale to the default language configured in the user's browser.
        
    * After that we invoke [`waitLocale()`](https://github.com/kaisermann/svelte-i18n/blob/46b025ceebeb9bd68df0a2f30cc3c0775049ed85/docs/Methods.md#waitlocale) This method executes the queue of the locale. If the queue isn't resolved yet, the same promise is returned.
        
        > For the sake of explanation, I am keeping things simple and limiting them to use the default language configured in the user's browser but you can extend this logic to set locale returned by parent layout or by any other means and set it accordingly!
        

### Localizing Application

Before we start with localization, it is important to ensure the default locale is set up and that we do have an initial set of key-value message pairs for translations. To do so we can implement a derived store variable in the `src/lib/i18n.ts` file and react to it accordingly:

```typescript
import { derived } from 'svelte/store';
import { locale } from 'svelte-i18n';

export const isLocaleLoaded = derived(locale, ($locale) => typeof $locale === 'string');
```

```svelte
<div class="content">
  {#if $isLocaleLoaded}
    <h1>{$_('heading')}</h1>
  {:else}
    <div>Locale initializing...</div>
  {/if}
</div>
```

Now that we have configured the library and have the initial locale set, we're ready to start localizing our app. To do that we simply import `$_` and pass message id in any component that needs to be translated.

The `$_` is the implementation of `format()` from [formnatjs](https://formatjs.io/). To format a message is as simple as executing the `$_` method and passing the message-id:

```svelte
<script>
  import { _ } from 'svelte-i18n';
</script>

<h1>{$_('page_title')}</h1>
```

It has two aliases `$t` and `$format`. So we can do any of the following:

```svelte
<h1>{$_('page_title')}</h1>
<h1>{$t('page_title')}</h1>
<h1>{$format('page_title')}</h1>
```

From the context of our application, let us revisit our translation file:

```json
{
  "heading": "Internationalization in SvelteKit",
  "toggle_label": "Select Locale",
  "button_label": "Generate Awards",
  "body_text": "This is a small example to demonstrate i18n functionality in SvelteKit using svelte-i18n library. svelte-i18n helps you localize your app using the reactive tools Svelte provides. By using stores to keep track of the current locale, dictionary of messages and to format messages, we keep everything neat, in sync and easy to use on your svelte files. Total number of npm downloads per week as of {date} are {download}.",
  "awards": "..."
}
```

To set the main heading, the label for locale switching and the button label text, we simply invoke `$_()` and pass in the message-id as explained above:

```svelte
<h1>{$_('heading')}</h1>
<span>{$_('toggle_label')}: </span>
<button>{$_('button_label')}</button>
```

We can also pass additional parameters to `$_()` and the syntax is:

```typescript
$_(messageId: string, options?: MessageObject): string
$_(options: MessageObject): string

interface MessageObject {
  id?: string;
  locale?: string;
  format?: string;
  default?: string;
  values?: Record<string, string | number | Date>;
}
```

In the message-id, we had something like:

```json
"body_text": "...  {date} are {download}."
```

We can use the new syntax that we just saw and fill in the values of the *date* and the *download* **placeholders** (`{ ... }`) dynamically:

```svelte
<p>{$_('body_text', {
  values: {
    download: $number(30242),
    date: $date(Date.UTC(2023, 6, 14, 0, 0, 0, 0), { year: "numeric", month: "long", day: "numeric" })
  }
})}</p>
```

For now, ignore `$number()` and `$date()` I'll discuss them in detail, in the following section about the Formatters. For now, switching to the next section where we will learn how to switch between locales!

### Locale Switching

To switch between locales, we make use of the `$locale` store variable.

The `locale` store defines what is the current locale. When its value is changed, before updating the actual stored value, *svelte-i18n* sees if there are any message loaders registered for the new locale:

* If yes, changing the `locale` is an async operation.
    
* If no, the locale's dictionary is fully loaded and changing the locale is a sync operation.
    

```svelte
<script lang="ts">
  import { locale } from 'svelte-i18n';

  let value: string = 'en';

  function handleLocaleChange(event: Event) {
    event.preventDefault();
    value = event?.target?.value;
    $locale = value;
  }
</script>

<select {value} on:change={handleLocaleChange}>
  <option value="en" selected>English</option>
  <option value="hi">Hindi</option>
  <option value="fr">French</option>
</select>
```

We can get the list of locales available in our application using the `$locales`

```svelte
{#each $locales as locale, i}
  <option value={locale}>{locale.toUpperCase()}</option>
{/each}
<!-- $locales = ['en', 'hi', 'fr'] -->
```

While changing the locales, we can also make use of the `$loading` store that can detect if the app is currently fetching any enqueued message definitions.

```svelte
<script>
  import { isLoading } from 'svelte-i18n'
</script>

{#if $isLoading}
  Please wait...
{:else}
  <Nav />
  <Main />
{/if}
```

We can also check the list of all messages with a particular locale using `$dictionary` The `$dictionary` store is responsible for holding all loaded message definitions for each locale.

```svelte
<script>
  import { dictionary } from 'svelte-i18n'

  console.log($dictionary); // { en: {...}, hi: {...}, fr: {...} }
</script>
```

### Pluralization

As I have mentioned before that *svelte-i18n* uses [ICU syntax](https://formatjs.io/docs/core-concepts/icu-syntax/#plural-format), and we can leverage that to implement Pluralization.

Consider the following message-id:

```json
"awards": "You have {n, plural, =0 { not won any awards } one { won exactly # award } other { won # awards }}!"
```

If we break that into a simple format and compare it with ICU syntax `{ key, plural, matches }`

```abap
{ n, plural, =0 { some-message } one { some-message # } other { some-message # } }
```

Here `n` is the *key* that will be passed via `$_()` followed by a *plural* keyword. *matches* resemble the rest of the conditions where if:

* `n=0` we can output custom messages within `{...}` Example: `=0 { not won any awards }`
    
* `n=1` for plural category `one` we can output custom messages within `{...}` Here `#` represents the value of `n` which will be `1` in this case. Example: `one { won exactly # award }`
    
* `n=any-number` for plural category `other` we can output custom messages within `{...}` Example: `other { won # awards }`
    

Coming back to our example:

```svelte
<span>{$_('awards', { values: { n: randomNumber } })}</span>
```

* if `n = 0`, output = *You have not won any awards!*
    
* if `n = 1`, output = *You have won exactly 1 award!*
    
* if `n = 10`, output = *You have won 10 awards!*
    

### Formatting

Coming to the last section of the article where I will discuss formatting Date, Time, Number and Currency using inbuild Formatters.

*svelte-i18n* provides inbuild formatters as well as provisions to create a custom formatter. For the sake of simplicity, I will only be explaining how to use inbuild formatters. You can refer to [this guide](https://github.com/kaisermann/svelte-i18n/blob/46b025ceebeb9bd68df0a2f30cc3c0775049ed85/docs/Formatting.md) on how to create custom formatters.

svelte-i18n provides `$time()`, `$date()` and `$number()` formatters that we can use to format data accordingly.

The following are the available Formats:

**Number:**

* `currency`: `{ style: 'currency' }`
    
* `percent`: `{ style: 'percent' }`
    
* `scientific`: `{ notation: 'scientific' }`
    
* `engineering`: `{ notation: 'engineering' }`
    
* `compactLong`: `{ notation: 'compact', compactDisplay: 'long' }`
    
* `compactShort`: `{ notation: 'compact', compactDisplay: 'short' }`
    

**Date:**

* `short`: `{ month: 'numeric', day: 'numeric', year: '2-digit' }`
    
* `medium`: `{ month: 'short', day: 'numeric', year: 'numeric' }`
    
* `long`: `{ month: 'long', day: 'numeric', year: 'numeric' }`
    
* `full`: `{ weekday: 'long', month: 'long', day: 'numeric', year: 'numeric' }`
    

**Time:**

* `short`: `{ hour: 'numeric', minute: 'numeric' }`
    
* `medium`: `{ hour: 'numeric', minute: 'numeric', second: 'numeric' }`
    
* `long`: `{ hour: 'numeric', minute: 'numeric', second: 'numeric', timeZoneName: 'short' }`
    
* `full`: `{ hour: 'numeric', minute: 'numeric', second: 'numeric', timeZoneName: 'short' }`
    

Coming back to our example application, we can now format date, time and currency:

```svelte
<script lang="ts">
  import { time, date, number } from 'svelte-i18n';
</script>

<span>Time: { $time(new Date(), { hour: "numeric", minute: "numeric", second: "numeric" }) }</span>
<span>Date: { $date(new Date(), { year: "numeric", month: "long", day: "numeric" }) }</span>
<span>Currency: { $number(2, { style: "currency", currency: "INR" }) }</span>
<span>Number: { $number(2) }</span>
```

### Conclusion

Finally, we were able to localize our application using *svelte-i18n*. You can find the code in the [GitHub repo](https://github.com/aakash14goplani/sveltekit-with-sveltei18n) and [link](https://sveltekit-with-sveltei18n.vercel.app/) to the live demo.

This was the first article of [four-part series](https://blog.aakashgoplani.in/series/i18n-in-sveltekit) to demonstrate i18n in SvelteKit. In the next article, I'll be explaining [i18n in SvelteKit with *sveltekit-i18n* library](https://blog.aakashgoplani.in/internationalization-in-sveltekit-with-sveltekit-i18n).

### References

* [List of Methods](https://github.com/kaisermann/svelte-i18n/blob/46b025ceebeb9bd68df0a2f30cc3c0775049ed85/docs/Methods.md) in svelte-i18n
    
* [CLI commands](https://github.com/kaisermann/svelte-i18n/blob/46b025ceebeb9bd68df0a2f30cc3c0775049ed85/docs/CLI.md) reference
    
* [Formatters](https://github.com/kaisermann/svelte-i18n/blob/46b025ceebeb9bd68df0a2f30cc3c0775049ed85/docs/Formatting.md)