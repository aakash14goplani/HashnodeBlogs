---
title: "Simple configuration hacks to improve unit test case performance using Jasmine and Karma in Angular"
seoTitle: "Simple configuration hacks to improve unit test case performance using"
seoDescription: "Simple configuration hacks to improve unit test case performance using Jasmine and Karma in Angular"
datePublished: Sun Jan 09 2022 18:20:39 GMT+0000 (Coordinated Universal Time)
cuid: cky7l0hm80mey5ns1aszzf91u
slug: simple-configuration-hacks-to-improve-unit-test-case-performance-using-jasmine-and-karma-in-angular
tags: unit-testing, angular, jasmine

---

### Make use of Headless Chrome

Headless Chrome is a way to run the Chrome browser in a headless environment without the full browser UI. One of the benefits of using Headless Chrome (as opposed to testing directly in Node) is that your JavaScript tests will be executed in the same environment as users of your site. Headless Chrome gives you a real browser context without the memory overhead of running a full version of Chrome.

Configuration in *karma.conf.js*
```js
browsers: ['Chrome', 'ChromeHeadlessNoSandbox']
customLaunchers: {
  ChromeHeadlessNoSandbox: {
     base: 'ChromeHeadless',
     flags: [
       '--no-sandbox',
       '--disable-gpu',
       '--remote-debugging-port=9222',
       '--disable-site-isolation-trials',
     ]
  }
}
```

More reading on this configuration from [developers.google.com](https://developers.google.com/web/updates/2017/04/headless-chrome)

Configurations in *package.json* file:
```json
{
  "test-fast": "node --max_old_space_size=8192 node_modules/@angular/cli/bin/ng test --browsers=ChromeHeadlessNoSandbox --watch=true --codeCoverage=true --source-map=false"
}
```
`npm run test-fast`

---

### Execute test cases in parallel

We can utilize [karma-parallel](https://www.npmjs.com/package/karma-parallel) plugin to execute test cases parallely. This npm package splits your unit tests into multiple suites that run in parallel with each other, on different threads of your processor. It's highly customizable straight from your karma config file with how many threads you want to use, and how it splits your tests up.

Configuration details:
1. Install package as dev dependency `npm i karma-parallel --save-dev`
2. Changes in *karma.conf.js*
   ```js
   frameworks: ['parallel', 'jasmine', ...others] // <- 'parallel' should be first one here
   plugin: [
     ... others,
     require: ('karma-parallel')
   ],
   parallelOptions: {
     executors: require('os') ? Math.ceil(require('os').cpus().length / 2) : 1
     shardStrategy: 'round-robin'
   }
   ```

NOTE:
* Though this plugin is awesome, the only problem that I have faced with this is inconsistency in calculation code coverage.
* I used this plugin until Angular v11. Starting v12, Angular has made many performance boost to Testing Module and since then, I did not felt use to include this!

---

### Style Cleanup & Reseting Testing Module

If you adjust your karma config to run without a headless browser, and using the inspector, inspect your `<head>` tag, you will notice hundreds if not thousands of `<style>` tags appended to your body. One more point, after running every test case angular recompiles our test bed configuration, meaning we spent 70% of total time in compilation rather than actual execution of test case. To overcome that we need to override default test bed rest module. 

[Credits](https://medium.com/angular-in-depth/angular-unit-testing-performance-34363b7345ba)   

NOTE:
* This section make sense till Angular v11.
* Post v12 Angular has made many performance boost to Testing Module and we can use (in *test.ts* file):
   ```js
   getTestBed().initTestEnvironment(
     BrowserDynamicTestingModule(),
     { teardown: { destroyAfterEach: true } } // <- this is the new entry
   );
   ```
* This is default in v13 and onwards to no need to make any changes in *test.ts*, you reap this feature by default (unless you choose to opt-out during upgrade)

Create new file *override-reset-test-module-jasmine.ts*:
```js
import { getTestBed, TestBed, ComponentFixture } from '@angular/core/testing';

export function overrideResetTestModule() {
  const testBedApi: any = getTestBed();
  const originReset TestBed.resetTestingModule;

  beforeAll(() => {
    TestBed.resetTestingModule();
    TestBed.resetTestingModule = () => TestBed;
  });

  afterEach(() => {
    testBedApi._activeFixtures.forEach((fixture: ComponentFixture<any>) => fixture.destroy());
    testBedApi._instantiated = false;
    cleanStylesFromDOM();
  });

  afterAll(() => {
    TestBed.resetTestingModule = originReset;
    TestBed.resetTestingModule();
  });

  function cleanStylesfromDOM(): void {
    const head: HTMLHeadElement = document.getElementsByTagName('head')[0];
    const styles: HTMLCollectionOf<HTMLStyleElement> I [] = document.getElementsByTagName('style');
    for (let i = 0; i < styles.length; i++) {
      head.removeChild(styles[i]); 
    }
  }
```

In you spec file, import this helper file:
```js
describe('', => {
  overrideResetTestModule();
  // rest of code
});
```

---

### Mock Data Correctly

-  Declare components instead of importing modules
- Always mock data. Especially if your service is interacting with HTTP or Router.
- This is how I prefer to mock [http](https://stackoverflow.com/a/69809726/3411606) and [router](https://stackoverflow.com/a/69809024/3411606)
- This is very good and detailed [article](https://medium.com/ngxp/optimize-angular-component-test-performance-ed1b261a2d97) that explains more on importance of mocking and how to mock data

---

### Best Practice

- A very good [article](https://vsavkin.com/three-ways-to-test-angular-2-components-dcea8e90bd8d) that I refered back in 2017 when I started writing unit test cases.
- Another best article on [Resolving common technical debt to speed up Angular development](https://www.devbridge.com/articles/resolving-common-technical-debt-to-speed-up-angular-development/)

---

I follow all these practices on my medium sized project in Angular v13 having around 1500 test cases and execution time is less than a minute (~50 seconds)! Yes less than a minute!!
