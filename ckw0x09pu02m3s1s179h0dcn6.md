---
title: "Manage list of supported browsers for your application in Angular"
seoTitle: "Manage list of supported browsers for your application in Angular"
seoDescription: "Manage list of supported browsers for your application in Angular"
datePublished: Mon Nov 15 2021 17:02:36 GMT+0000 (Coordinated Universal Time)
cuid: ckw0x09pu02m3s1s179h0dcn6
slug: manage-list-of-supported-browsers-for-your-application-in-angular
tags: browser, angular

---

Often we have a need to disable contents of our application for old browsers (like Internet Explorer) due to various reasons, some of which could be browser is no longer supported, you application's features (say animations) are very performant-specific and could only be supported by latest version of browsers.

In such scenarios we need a mechanism that enables us to target or cherry-pick certain range of browsers to load our applications and may be hide or show warning message when our applications loads on lower version of browsers.

This scenario is very easy to implement with [browserslist-useragent-regexp](https://github.com/browserslist/browserslist-useragent-regexp) package. This package along with [browserlist](https://github.com/browserslist/browserslist) (already provided by angular) enables us to query / cherry-pick browsers according to their version numbers.

**Installing dependencies:**

Step one will be to install `browserslist-useragent-regexp` package. I would also suggest to install [detect-browser](https://github.com/DamonOehlman/detect-browser) package as it helps us to very good information on what current version and type of browser that our application is running on, but again that's optional.
```
npm i browserslist-useragent-regexp detect-browser
```

**Writing Query for browsers that we want to target:**

Once we have installed required dependency, we can now query / cherry pick which type of browsers we want to allow and which version we would like to have in `.browserlistrc` file placed at root folder of our project. Example:
```
last 2 versions
not dead
not IE > 0
not IE_Mob > 0
```
- select last two stable versions of all the major browsers available out there. Example - if current version of stable google chrome browser is v92 then this query will support v92 and v91
- browsers should have official support i.e. they are not dead
- ignore IE browser
- ignore IE mobile browser

You can read more on queries [here](https://github.com/browserslist/browserslist#queries)

**Create script to generate regex for supported browsers:**

Once we have queries we can now add one script in `package.json` file to generate the regex with help of `browserslist-useragent-regexp` package which will then be used to conditionally hide / display content.
```json
"scripts": {
  "supportedBrowsers": "(echo module.exports = && browserslist-useragent-regexp --allowHigherVersions) > src/supportedBrowsers.js"
}
```

Before executing the script, please create an empty javascript file named `supportedBrowsers.js` within `src` folder. then we can `npm run supportedBrowsers`. This command will generate a RegEx which we can use with browser's `userAgent` to display / hide content.
```js
module.exports = 
/((CPU[ +]OS|iPhone[ +]OS|CPU[ +]iPhone|CPU IPhone OS)[ +]+(14[_.]5|14[_.](......)/
```

**Conditionally displaying data:**

Once we have all the configurations in place we can import `supportedBrowsers.js` file into our component and write the logic to hide / display data
```js
import * as supportedBrowsers from '../supportedBrowsers';
import { detect } from 'detect-browser';
...
export class AppComponent implements OnInit {
  browserSupported = '';
  title = 'Browser Support';
  message = '';

  ngOnInit(): void {
    this.browserSupported = supportedBrowsers.test(navigator.userAgent) ? '' : 'not';
    this.message = `Your current browser ${detect()?.name} version ${detect()?.version} is ${this.browserSupported} supported`;
  }
}
```
```html
<main>
  <div class="app-main">
    <h2>{{ title }}</h2>
    <p>{{ message }}</p>
  </div>
</main>
```

> **Note:** You may get compiler warning while importing `.js` files in `.ts` files - to resolve this simply add `"allowJs": true` within `compilerOptions` of *tsconfig.json* file.

[Stackbliz link](https://stackblitz.com/edit/angular-ivy-fe2mdj?file=src/app/app.component.ts)



