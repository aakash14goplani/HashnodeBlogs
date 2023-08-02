---
title: "Internationalization in SvelteKit with typesafe-i18n"
seoTitle: "Internationalization in SvelteKit with typesafe-i18n"
seoDescription: "Internationalization in SvelteKit with typesafe-i18n, i18n in SvelteKit, Localization in SvelteKit, Translations in Sveltekit, typesafe-i18n"
datePublished: Tue Aug 01 2023 17:20:25 GMT+0000 (Coordinated Universal Time)
cuid: clkskeqrt000k09mge57j3zrr
slug: internationalization-in-sveltekit-with-typesafe-i18n
tags: internationalization, i18n, localization, sveltekit, typesafe-i18n

---

This is the final article of [three-part series](https://blog.aakashgoplani.in/series/i18n-in-sveltekit) to demonstrate i18n in SvelteKit. In the [previous article](https://blog.aakashgoplani.in/internationalization-in-sveltekit-with-sveltekit-i18n), we worked our way with *sveltekit-i18n* and in this article, we will work on integrating *typesafe-i18n* with SvelteKit.

The *typesafe-i18n* is a fully type-safe and lightweight internationalization library for all your TypeScript and JavaScript projects. One thing that makes *typesafe-i18n* stand out is that it is a generic library and not limited to Svelte, you can use it with any of your JavaScript or TypeScript projects.

This library introduces a full-fledged flow for maintaining localization in your application. The moment you install this library, it generates a specific folder structure encompassing all the i18n-specific files.

## Understanding *typesafe-i18n* ecosystem and nomenclature

There are six main packages that I want to explain before we start with actual implementation.

1. **Generators**:
    
    * [Generators](https://github.com/ivanhofer/typesafe-i18n/tree/main/packages/generator), as the name says generate boilerplate code specific to i18n and look for changes that you make to locale files.
        
    * Once we have installed *typesafe-i18n* and run the command `typesafe-i18n`, the generator kicks in and will generate the following project structure within the *src/i18n* directory. By default locales for `de` and `en` will be auto-generated.
        
        ```abap
        src/
          i18n/
            en/
              index.ts
            de/
              index.ts
            custom-types.ts
            formatters.ts
            i18n-types.ts
            i18n-util.async.ts
            i18n-util.sync.ts
            i18n-util.ts
        ```
        
    * Let's go through these generated files as they will be very useful when we localize our application:
        
        * `en/index.ts`: If 'en' is your base locale, the file *src/i18n/en/index.ts* will contain your translations. Whenever you make changes to this file, the generator will create updated type definitions.
            
        * `custom-types.ts`  
            To define types that are unknown to *typesafe-i18n*.
            
        * `formatters.ts`  
            In this file, you can configure the formatters to use inside your translations. The `Formatters` type gets generated automatically by reading all your translations from the base locale file.
            
        * `i18n-types.ts`  
            Type definitions are generated in this file. You don't have to understand them. They are just here to help TypeScript understand, how you need to call the translation functions.
            
        * `i18n-util.async.ts`  
            This file contains the logic to load individual locales asynchronously. **This is the preferred approach** and we will be using this approach while localizing our application.
            
        * `i18n-util.sync.ts`  
            This file contains the logic to load your locales synchronously.
            
        * `i18n-util.ts`  
            This file contains wrappers with type information around the base i18n functions.
            
    * These files will be auto-updated (except *formatters.ts* and *custom-types.ts*) every time you make changes to your locale, if you manually make any changes to these files, they will be over-written!
        
    * We can configure generators as per our requirements. Here is the [list of available options](https://github.com/ivanhofer/typesafe-i18n/tree/main/packages/generator) that can be utilized to customize generators. In this article, I'll be keeping things simple and won't be customizing them.
        
2. **Detectors:**
    
    * Locale detection is a key part of any i18n solution. Therefore *typesafe-i18n* provides a solution to detect a user's locale.
        
    * [Dectotors](https://github.com/ivanhofer/typesafe-i18n/tree/main/packages/detectors) exports multiple utility methods that enable us to detect locale on the client and server side. We will see a few of them in sections **Detect the locale on the server side** and **Detect the locale on the client side.**
        
3. **Runtime:**
    
    * The [runtime package](https://github.com/ivanhofer/typesafe-i18n/tree/main/packages/runtime) provides the means of writing and using translations in *typesafe-i18n*. It exposes wrappers that enable us to pluralize, format and translate our messages.
        
    * From amongst many utilities exported by this package, three important ones that I want to highlight are:
        
        * ***i18nString***: It is represented as **LLL**. It is used for simple string interpolation.
            
        * ***i18nObject***: It is represented as **LL**. It is useful in scenarios when we want to provide translations for a particular locale asynchronously i.e. load one locale at a time. *We will be using this method in our application*.
            
        * ***i18n***: It is represented as **L**. It is useful in scenarios when we want to load all the locales synchronously.
            
    * As mentioned before, I'll be using the **LL** approach in this article. Here is the [link](https://github.com/ivanhofer/typesafe-i18n/tree/main/packages/runtime) to official documentation that explains how to use the other two approaches.
        
4. **Formatters**
    
    * This [package](https://github.com/ivanhofer/typesafe-i18n/tree/main/packages/formatters) exports multiple utility methods useful for formatting. We will see a few of them in the section on **Formatting.**
        
5. **Importers:**
    
    * This [package](https://github.com/ivanhofer/typesafe-i18n/tree/main/packages/importer) is used for importing language files that come from an API, spreadsheet or JSON files. I won't be covering details about this package in this article.
        
6. **Exporters:**
    
    * This [package](https://github.com/ivanhofer/typesafe-i18n/tree/main/packages/exporter) is used for exporting language files to a service or an API. I won't be covering details about this package in this article.
        

That's it with the fundamentals, let's start with localizing our application.

## Application Structure

Before we begin, I want to give you a detailed walkthrough of the application that we will be working on. You can find the code in the [GitHub repository](https://github.com/aakash14goplani/sveltekit-with-typesafe18n).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690910323040/d14c6da9-0986-485f-b88f-7bd56b4a3c0a.png align="center")

As we can see from the image, this will be a simple single-page application demonstrating i18n features like *locale switching*, *pluralization* and *formatting*.

In the next section, we will work on integrating the *typesafe-i18n* library with SvelteKit.

## Integration with SvelteKit

1. **Install and Configure the *typesafe-i18n* library**
    
    * This first step is to install this library as a dependency in our SvelteKit project. We will also need to install `npm-run-all` package as the dev-dependency. **NOTE**: Use *npm*, I tried with *pnpm* but was not able to execute this command.
        
        ```abap
        npm i --save-dev npm-run-all
        npx typesafe-i18n --setup-auto
        ```
        
    * This command will run the setup process and **automatically detect** the config needed. If you want to manually setup the configuration, the command for that is: `npx typesafe-i18n --setup`
        
    * This will generate a config file *.typesafe-i18n.json.* In this file, we will add a new property `outputPath` to generate folder structure within the *src/lib/i18n* (default is *src/i18n*) so that we can access those files with SvelteKit `$lib` feature.
        
        ```json
        {
          ...,
          "outputPath": "./src/lib/i18n/"
        }
        ```
        
    * The next step is to update the *package.json* file to include the script for running the *typesafe-i18n* CLI along with the Vite development server.
        
        ```json
        "scripts" : {
          "dev": "npm-run-all --parallel vite typesafe-i18n",
          "vite": "vite dev",
          "typesafe-i18n": "typesafe-i18n",
          ...
        }
        ```
        
    * If we run the script `npm run dev`, it will generate a folder structure for your locales inside the `src/lib/i18n` folder. By default locales for `de` and `en` will be auto-generated, we will customize this in the next section.
        
2. **Define Locale Dictionary**
    
    * Now that we have installed this library, it's time to identify the areas in our demo application that need localization. We then define the **locale dictionaries** within the *src/lib/i18n* folder generated by *typesafe-i18n*. A locale dictionary is a regular JSON object which contains message definitions for a certain language.
        
    * The best thing about *typesafe-i18n* is all the dictionaries will be typed. We can use `BaseTranslation` type for the same.
        
    * In our example application, I want five fields to be localized: The main heading, Lable for locale switching, Body text (the paragraph) and Text for pluralization.
        
    * For the sake of this application, I will define dictionaries in three languages - English (*en.json*), Hindi (*hi.json*) and French (*fr.json*) within the *src/lib/i18n* folder.
        
    * By default `en/index.ts` and `de/index.ts` will be pre-configured. We will rename `de` to `hi` and create a new entry for the `fr/index.ts` language.
        
        ```typescript
        // file -> src/lib/i18n/en/index.ts
        import type { BaseTranslation } from '../i18n-types';
        
        const en = {
          heading: 'Internationalization in SvelteKit',
          toggle_label: 'Select Locale',
          body_text:
        		'This is a small example to demonstrate i18n functionality in SvelteKit using typesafe-i18n library. typesafe-i18n is a fully type-safe and lightweight internationalization library for all your TypeScript and JavaScript projects. typesafe-i18n comes with an API that allows other services to read and update translations. Total number of npm downloads per week as of {date:Date|simpleDate} are {download:number|simpleNumber}.',
          awards: 'You have {{ not won any awards | won exactly ?? award | won ?? awards }}!',
          time: '{value:Date|simpleTime}',
          date: '{value:Date|simpleDate}',
          currency: '{value:number|simpleCurrency}'
        } satisfies BaseTranslation;
        
        export default en;
        
        // file -> src/lib/i18n/hi/index.ts
        import type { BaseTranslation } from '../i18n-types';
        
        const hi = {
          heading: 'SvelteKit में अंतर्राष्ट्रीयकरण',
          toggle_label: 'भाषा चुने',
          body_text:
        		'यह typesafe-i18n लाइब्रेरी का उपयोग करके SvelteKit में i18n कार्यक्षमता प्रदर्शित करने के लिए एक छोटा सा उदाहरण है। typesafe-i18n आपके सभी टाइपस्क्रिप्ट और जावास्क्रिप्ट प्रोजेक्ट्स के लिए पूरी तरह से टाइप-सुरक्षित और हल्के अंतर्राष्ट्रीयकरण लाइब्रेरी है। typesafe-i18n एक एपीआई के साथ आता है जो अन्य सेवाओं को अनुवाद पढ़ने और अपडेट करने की अनुमति देता है।। {date:Date|simpleDate} तक प्रति सप्ताह npm डाउनलोड की कुल संख्या {download:number|simpleNumber} है|',
          awards: 'आपने {{ कोई पुरस्कार नहीं जीता | बिल्कुल ?? पुरस्कार जीता | ?? पुरस्कार जीते }} है|',
          time: '{value:Date|simpleTime}',
          date: '{value:Date|simpleDate}',
          currency: '{value:number|simpleCurrency}'
        } satisfies BaseTranslation;
        
        export default hi;
        
        // etc... all other languages that you wish to support
        ```
        
    * Pay attention to the syntax `{value: <type>|<formatter}` Let's break this syntax down into three parts:
        
        * `{value}` is a placeholder that will be populated with a value of a particular locale during runtime. As the locale changes, this field will be recomputed.
            
        * `{value: <type>}` is used to specify the data type of value e.g. *number*, *Date* etc.
            
        * `{value: <type>|<formatter>}` is used to specify a value with a particular data type along with a formatter to format value based on a particular locale.
            
    * As soon you update these locale dictionary (translation) files, typesafe-i18n will recompute all the files within the `src/lib/i18n` directory.
        
3. **Detect the locale on the server side**
    
    * Now that our library is installed and the locale dictionary is ready, it is time to create an entry point that will load the assets based on the user locale and initialize the library with a specific locale.
        
    * We can make use of [detectors](https://github.com/ivanhofer/typesafe-i18n/tree/main/packages/detectors) to detect the locale. Now this package exports multiple utilities to detect locale and I cannot cover every of those in this article, I'll advise readers to go through every method and pick the best one as per requirement.
        
    * In the below code snippet within the *hooks.server.ts* file, I am querying request headers using `initAcceptLanguageHeaderDetector()` to determine if the locale is present. If yes, then I call `detectLocale(...)` that returns the final locale to be used. If no locale is returned from headers, I query cookies for the same using `initRequestCookiesDetector()` If cookies don't return anything, I use the default locale i.e. `en`
        
        ```typescript
        import type { Handle } from '@sveltejs/kit';
        import { loadLocaleAsync } from '$lib/i18n/i18n-util.async';
        import { setLocale } from '$lib/i18n/i18n-svelte';
        import { detectLocale } from '$lib/i18n/i18n-util';
        import {
          initAcceptLanguageHeaderDetector,
          initRequestCookiesDetector
        } from 'typesafe-i18n/detectors';
        
        export const handle: Handle = async ({ event, resolve }) => {
          let deafultLocale = 'en';
        
          const acceptLanguageHeaderDetector = initAcceptLanguageHeaderDetector(event.request);
          const localeFromHeaders = detectLocale(acceptLanguageHeaderDetector);
        
          if (!localeFromHeaders) {
            // if locale is not present in headers, try fetching it from cookies
            const requestCookiesDetector = initRequestCookiesDetector({
              cookies: event.cookies.get('lang') || ''
            });
            const localeFromCookies = detectLocale(requestCookiesDetector);
            if (localeFromCookies) {
              deafultLocale = localeFromCookies;
            } else {
              // add in cookes
              event.cookies.set('lang', deafultLocale, { path: '/' });
            }
          } else {
            deafultLocale = localeFromHeaders;
          }
          const locale = detectLocale(() => [deafultLocale]);
          // Load it
          await loadLocaleAsync(locale);
          // Set it
          setLocale(locale);
          // [OPTIONAL] set locale within locals property
          event.locals.locale = deafultLocale;
          return resolve(event);
        };
        ```
        
    * \[OPTIONAL\] If you see any typescript error in the line `event.locals.locale = deafultLocale;` then update the file *src/app.d.ts* to include `locale` property.
        
        ```typescript
        // See https://kit.svelte.dev/docs/types#app
        // for information about these interfaces
        declare global {
          namespace App {
            // interface Error {}
            interface Locals {
              locale: string;
            }
            // interface PageData {}
            // interface Platform {}
          }
        }
        
        export {};
        ```
        
    * Once the locale is detected, we asynchronously load the corresponding translation messages for that particular locale using `loadLocaleAsync(locale)`. Finally, we set the store `setLocale(locale)` to save the current locale across the server.
        
4. **Detect the locale on the client side**
    
    * Once we have the locale set up on the server side, it's time to load the locale on the client side as well. In the file *+layout.ts*, once the client-side is initialized, we detect the locale from session storage using `sessionStorageDetector()`, if the locale is not found we pick the locale from the user's browser configuration using `navigatorDetector()`
        
        ```typescript
        import { browser } from '$app/environment';
        import type { LayoutLoad } from './$types';
        import { setLocale } from '$lib/i18n/i18n-svelte';
        import { detectLocale } from '$lib/i18n/i18n-util';
        import { loadLocaleAsync } from '$lib/i18n/i18n-util.async';
        import { sessionStorageDetector, navigatorDetector } from 'typesafe-i18n/detectors';
        
        export const load: LayoutLoad = async (event) => {
          if (browser) {
            const deafultLocale = 'en';
            const locale =
              detectLocale(sessionStorageDetector) || detectLocale(navigatorDetector) || deafultLocale;
        	
            await loadLocaleAsync(locale);
            setLocale(locale);
          }
          return event.data;
        };
        ```
        
    * Similar to the server side, once the locale is detected, we asynchronously load the corresponding translation messages for that particular locale using `loadLocaleAsync(locale)`. Finally, we set the store `setLocale(locale)` to save the current locale across the application.
        

## Localizing Application

Now that we have configured the library and have the initial locale set, we're ready to start localizing our app. To do that we simply import `$LL` and pass message id in any component that needs to be translated.

Let's take a look at our locale file:

```typescript
const en = {
  heading: 'Internationalization in SvelteKit',
  toggle_label: 'Select Locale',
  body_text: '... {date:Date|simpleDate} are {download:number|simpleNumber}.',
  ...
} satisfies BaseTranslation;
```

To set the main heading, the label for locale switching, we simply invoke `$LL.<message-id>()`:

```svelte
<h1>{$LL.heading()}</h1>
<span>{$LL.toggle_label()}: </span>
```

We can also pass additional parameters to `$LL` to populate values at runtime based on the current locale. For example, `body_text` message-id has two variables `date` of type *Date* and `download` of type *number*. We can pass the same variables as an object in `$LL`

```svelte
<p>{$LL.body_text({
  download: 16711,
  date: new Date(2023, 6, 14, 0, 0, 0, 0)
})}</p>
```

## Locale Switching

To switch between locales, we must first load the translations asynchronously for the new locale using `loadLocaleAsync()` and then update the store using `setLocale()`

```svelte
<script lang="ts">
  import { browser } from '$app/environment';
  import { onDestroy, onMount } from 'svelte';
  import { LL, setLocale } from '$lib/i18n/i18n-svelte';
  import { loadLocaleAsync } from '$lib/i18n/i18n-util.async';

  let value: Locales = 'en';

  async function handleLocaleChange(event: Event) {
    event.preventDefault();
    value = event?.target?.value;
    await loadLocaleAsync(value);
    setLocale(value);
    sessionStorage.setItem('lang', value);
  }

  onMount(() => {
    const valueFromSession = sessionStorage.getItem('lang') || 'en';
    value = valueFromSession as Locales;
    sessionStorage.setItem('lang', valueFromSession);
  })

  onDestroy(() => {
    if (browser) {
      sessionStorage.removeItem('lang');
    }
  })
</script>

<div class="container__toggle">
  <span>{$LL.toggle_label()}: </span>
  <select {value} on:change={handleLocaleChange}>
    <option value="en">English</option>
    <option value="hi">Hindi</option>
    <option value="fr">French</option>
  </select>
</div>
```

One more point, in the last section, we detected locale in the *+layout.ts* file using `sessionStorageDetector()` and hence we must update the session storage to reflect the latest locale value. Also, the key used in session storage must be "**lang**" only, this is a requirement from *typesafe-i18n*.

## Pluralization

The typesafe-i18n provides multiple ways in which we can pluralize a string. I would encourage readers to go through [examples](https://github.com/ivanhofer/typesafe-i18n/tree/main/packages/runtime#plural) in their documentation as it would be not possible for me to cover them all.

For this example application, the syntax for pluralization used will be:

```typescript
{{ zero | one | other }}
```

Consider the following message-id from our translation file:

```typescript
awards: 'You have {{ not won any awards | won exactly ?? award | won ?? awards }}!',
```

This is how we are going to localize:

```svelte
<span>{$LL.awards(randomNumber)}</span>
```

Now if we compare the message-id with the plural syntax that we have used, this is how things will break up:

* if `randomNumber = 0`, the output will be "*not won any awards*"
    
* if `randomNumber = 1`, the output will be "*won exactly 1 award*". Here `??` is the value of the argument that was passed to `$LL()`
    
* if `randomNumber > 1`, the output will be "*won 5 awards*" (assuming `randomNumber = 5`)
    

## Formatting

Coming to the last section of the article where I will discuss formatting Date, Time, Number and Currency using inbuild Formatters.

The typesafe-i18n has one entire [package](https://github.com/ivanhofer/typesafe-i18n/tree/main/packages/formatters) dedicated to formatters. It enables us to use inbuild formatters as well as provides utilities to create custom formatters.

We will write all formatter utilities within the file *src/lib/i8n/formatters.ts*. We will use `time()`, `date()` and `number()` formatters from `typesafe-i18n/formatters`. These methods are wrappers around the [Intl](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl) namespace.

The idea here is to create an object with the custom keys and export it and later while localizing, chain them with respective message-id.

As a first step, we export an object with our custom format options.

```typescript
import type { FormattersInitializer } from 'typesafe-i18n';
import type { Locales, Formatters } from './i18n-types';
import { date, time, number } from 'typesafe-i18n/formatters';

export const initFormatters: FormattersInitializer<Locales, Formatters> = (locale: Locales) => {
  const formatters: Formatters = {
    simpleDate: date(locale, { year: 'numeric', month: 'long', day: 'numeric' }),
    simpleTime: time(locale, { hour: 'numeric', minute: 'numeric' }),
    simpleNumber: number(locale),
    simpleCurrency: number(locale, { style: 'currency', currency: getCurrencyCode(locale) })
  };

  return formatters;
};

function getCurrencyCode(value: Locales): string {
  switch (value) {
    case 'en':
      return 'USD';
    case 'hi':
      return 'INR';
    case 'fr':
      return 'EUR';
    default:
      return 'USD';
  }
}
```

Now we can use those keys to the chain with our message-id in our translation files.

```typescript
const en = {
  ...
  time: '{value:Date|simpleTime}',
  date: '{value:Date|simpleDate}',
  currency: '{value:number|simpleCurrency}'
} satisfies BaseTranslation;
```

The last step is to just invoke `$LL()` and pass relevant arguments.

```svelte
<span>Time: { $LL.time({ value: new Date() }) }</span>
<span>Date: { $LL.date({ value: new Date() }) }</span>
<span>Currency: { $LL.currency({ value: 16711 }) }</span>
```

## Conclusion

Finally, we were able to localize our application using *typesafe-i18n*. You can find the code in the [GitHub repo](https://github.com/aakash14goplani/sveltekit-with-typesafe18n) and [link](https://sveltekit-with-typesafei18n.vercel.app/) to the live demo.

This was the final article of [three-part series](https://blog.aakashgoplani.in/series/i18n-in-sveltekit) to demonstrate i18n in SvelteKit. In the [next article](https://blog.aakashgoplani.in/comparing-i18n-libraries-in-sveltekit-svelte-i18n-sveltekit-i18n-and-typesafe-i18n), I'll be comparing three libraries: svelte-i18n, sveltekit-i18n and typesafe-i18n.

## References

* [Official Documentation](https://github.com/ivanhofer/typesafe-i18n/tree/main)
    
* [Util Functions](https://github.com/ivanhofer/typesafe-i18n/tree/main/packages/utils)