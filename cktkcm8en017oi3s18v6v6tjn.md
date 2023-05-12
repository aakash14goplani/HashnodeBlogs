---
title: "A simple approach towards handling errors in Angular"
seoTitle: "Error Handling in Angular"
seoDescription: "A simple approach towards handling server-side and client-side errors in Angular"
datePublished: Tue Sep 14 2021 17:28:05 GMT+0000 (Coordinated Universal Time)
cuid: cktkcm8en017oi3s18v6v6tjn
slug: a-simple-approach-towards-handling-errors-in-angular
tags: angular, error-handling

---

Angular errors can be broadly classified into two types:
   1. HTTP Errors
   2. Client Errors

HTTP errors occurs whenever we deal with external APIs example, we made call to an endpoint and the network went down or while making the call server was not able to process the request properly and in return sends back error etc. All such scenarios which involve server's response status of 5xx and 4xx  role comes under this category. In Angular, they are identified by `HttpErrorResponse`.

![image](https://user-images.githubusercontent.com/13452115/133273630-c3c291bc-fede-4fa0-9245-2b88852003c7.png)

Client (Browser) Error is the error that mostly occurs at runtime because of developers mistake while writing the code. Types of these errors are: `EvalError`, `InternalError`, `RangeError`, `ReferenceError`, `SyntaxError`, `URIError`, `TypeError`. One such example:
```js
(windows as any).abc.pqr = '';
// here property `abc` is not defined on global window object so
// `(windows as any).abc` will result into undefined and
// undefined.pqr will throw TypeError: stating that we are
// trying to set something on a property that does not exists
```
So any such errors that are induced by developers comes under Client (Browser) Error category.

![image](https://user-images.githubusercontent.com/13452115/133273671-1d3f0f89-8a19-4640-9525-315f0ede9553.png)

Under both the circumstances its the end user that suffer the most. Whenever any such error occurs, execution of JavaScript stops and the screen freezes giving end user a bad experience. So, the good practice is to handle such errors and perform a relevant action like routing users to error page and displaying some custom message like `Something Went Wrong! Please try again later! `

Angular comes up with class `ErrorHandler` that provides default method `handleError(error: Error)` which we can utilize to catch those errors.
```js
@Injectable()
class MyErrorHandler implements ErrorHandler {
  handleError(error: Error) {
    // do something with the exception like router.navigate(['/error-page']);
  }
}
```
We can use `handleError(error: Error)` to catch the error and redirect user to a generic `error-page`. One problem here is how do we inject the helper service in our custom `ErrorHandler` implementation?

If we inject the service as we generally do
```js
constructor(private router: Router){}
```
This will throw error:
> Error: NG0200: Circular Dependency in DI detected for ErrorHandler
> ![image](https://user-images.githubusercontent.com/13452115/133276587-a1a039d8-47b8-4394-a26e-d0662cd73bd0.png)

Angular creates `ErrorHandler` before providers otherwise it won't be able to catch errors that occurs in early phase of application. Hence the providers will not be available to `ErrorHandler`. So, we need to inject dependent services using injectors.
```js
@Injectable()
class MyErrorHandler implements ErrorHandler {
  constructor(private injector: Injector){}
  
  handleError(error: Error) {
    const router = this.injector.get(Router);
    router.navigate(['/error-page']);
  }
}
```

This solves one problem but leads to another.
> Navigation triggered outside Angular zone, did you forget to call `ngZone.run()`
> ![image](https://user-images.githubusercontent.com/13452115/133276468-7d04b25d-c24d-4c9e-a34c-f64532efc152.png)

The problem here is exactly same as before while injecting our helper services. ErrorHandle runs outside of regular ngZone. So, the navigation should take place outside of the zone so that regular flow of Change Detection is not hampered

```js
@Injectable()
class MyErrorHandler implements ErrorHandler {
  constructor(
    private injector: Injector,
    private zone: NgZone,
  ){}
  
  handleError(error: Error) {
    const router = this.injector.get(Router);
    this.zone.run(() => router.navigate(['/error-page']));
    console.error('Error Caught: ', error);
  }
}
```

Once we had achieved this, we need to provide this service to root module, example AppModule:
```js
@NgModule({
  providers: [{provide: ErrorHandler, useClass: MyErrorHandler}]
})
class MyModule {}
```

We can add more customization to above `handleError` method
```js
handleError(error: Error) {
  if (error instanceof HttpErrorResponse) {
    // HTTP related error
  } else if (error instanceof TypeError || error instanceof ReferenceError) {
    // Runtime exceptions mostly induced by Developer's code
  } else {
    // catch-all: catch rest of errors
  }
}
```

**Edit 10/30**

When working with iframes, we had to handle special use case - The iframe use to refresh after every 3 minutes of non-interactivity & generate new security token (for security purpose), when the refresh use to happen, for very small window, we got CORS issue in console, which falls under `SecurityError` and because of which we were redirected to errorpage. In this scenario we wanted to stay on the same page as the refresh activity, takes only a second or two, so we had to add an additional condition to above code to ignore `SecurityError` 
```js
if (e.name !== 'SecurityError') {
   // proceed with logic
}
```
