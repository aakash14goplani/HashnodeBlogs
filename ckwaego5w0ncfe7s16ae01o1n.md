---
title: "How to display app version from package.json file in Angular"
seoTitle: "How to display app version from package.json file in Angular"
seoDescription: "How to display app version from package.json file in Angular"
datePublished: Mon Nov 22 2021 08:21:10 GMT+0000 (Coordinated Universal Time)
cuid: ckwaego5w0ncfe7s16ae01o1n
slug: how-to-display-app-version-from-packagejson-file-in-angular
tags: angular

---

I had this requirement to display or keep track of application version that we are currently using in our environments. Typically any mid to large scale organization have multiple dedicated environments where we deploy our application. There is chance that different version could have been deployed in one environment then other. Arguably, this isn't any problem, but only way to know which version is deployed to what environment is to check it on the console / dashboard of the deployment tool (like uDeploy or similar...) that we are using!

We thought it would be easy if we have that handy within application itself instead of every time checking that on dashboard of tool. To achieve this, we thought of reading the version number from `package.json` file and storing that as global variable.

**package.json file:** File that has version number that we are interested in
```json
{
   "name": "my-awesome-app",
   "version": "4.1.0-RC3"
}
```

**app.component.ts** Reads value from package file and store this is global variable
```js
declare const require: (path: string) => any;

@Component({...})
export class AppComponent implements OnInit {
  ngOnInit(): void {
    const APP_VERSION = require('../../package.json').version;
    if (APP_VERSION) {
      (window as any).APP_VERSION = APP_VERSION;
    }
  }
}
```

Access this in browser console:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1637568822757/ZcJxT9HJa.png)

