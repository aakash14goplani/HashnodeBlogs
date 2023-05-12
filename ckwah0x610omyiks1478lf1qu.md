---
title: "Disabling browser back navigation in Angular"
seoTitle: "Disabling browser back navigation in Angular"
seoDescription: "Disabling browser back navigation in Angular"
datePublished: Mon Nov 22 2021 09:32:54 GMT+0000 (Coordinated Universal Time)
cuid: ckwah0x610omyiks1478lf1qu
slug: disabling-browser-back-navigation-in-angular
tags: angular

---

To prevent browse back navigation we make use of `History.pushState()`web api. As soon as the user lands on the component where we want to disable back navigation, we set `history.pushState(null, '')`.

As per [MDN docs](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState), `pushState()` has three parameters:
   1. The first parameter passed to `pushState()` is `state` which is an JavaScript object that holds the history details. To prevent using going back to previous state, we set this to `null`.
   2. The second parameter i.e. `title` does not play much role in this functionality and can be set to empty string.
   3. Final parameter is `url` which is optional that allows you to define the new history entry’s URL. It defaults to current document URL if not provided. This is what exactly we want - as soon as user lands on this component, they should kind of stick to current document URL and should not be able to navigate away!

When user navigates to a new state / page, a `popstate` event is fired. Here we take help of RxJs `fromEvent` and listen to `popstate` events whenever user tries to hit browser's back button & we can disable that action accordingly.

> Note: please remember to unsubscribe from the observable once you're done else that will lead to memory leak

Here is code example:

```js
import { fromEvent, Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

private unsubscriber : Subject<void> = new Subject<void>();

ngOnInit(): void {
  history.pushState(null, '');

  fromEvent(window, 'popstate').pipe(
    takeUntil(this.unsubscriber)
  ).subscribe((_) => {
    history.pushState(null, '');
    this.showErrorModal(`You can't make changes or go back at this time.`);
  });
}

ngOnDestroy(): void {
  this.unsubscriber.next();
  this.unsubscriber.complete();
}
```

[Stackbliz Example](https://stackblitz.com/edit/angular-ivy-zxu8hh?file=src/app/app.component.ts)