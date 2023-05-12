---
title: "Angular Dynamic Form Validation - Template and Reactive Forms"
seoTitle: "Angular Dynamic Form Validation - Template and Reactive Forms"
seoDescription: "Angular Dynamic Form Validation - Template and Reactive Forms"
datePublished: Tue Nov 16 2021 20:25:08 GMT+0000 (Coordinated Universal Time)
cuid: ckw2jokr902p7qes17pbi93jb
slug: angular-dynamic-form-validation-template-and-reactive-forms
tags: forms, angular, validation

---

In this article I will present a way to validate Angular forms, both - model driven and template driven.

For this we will need two directives:   
(1) *form-control-validation:* validates single input (form) control   
(2) *form-group-directive:* validates group of form controls.   
These directives will be attached to input elements & hence we can easily access core `ngModel` / `FormControl` instances and validate them.

We will use concept of dynamic components to create Error Component that has custom error validation message & inject it in DOM.

This is general overview of what we are trying to implement:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1637004182096/7PwzcyVqA.png)

**DOM Structure:**

As mentioned we will be injecting those directives on input elements within DOM, one thing that we need to take care of is DOM structure. The whole POC is designed with the concept that directive will search for the nearest div / span with class "form-group" and attach the dynamic component with error message as child of that div / span.

So all our input elements must be wrapped within "form-group" class. Example:
```html
<!-- snippet from template form -->
<div class="form-group user-data" ngModelGroup="userData" appFormGroupValidation validationMsgId="userData">
  <div class="firstname">
    <label for="firstname">First Name:</label>
    <input type="text" class="form-control" ngModel name="firstname" required [pattern]="validationPatterns.name" />
  </div>
  <div class="lastname">
    <label for="lastName">Last Name:</label>
    <input type="text" class="form-control" ngModel name="lastName" required [pattern]="validationPatterns.name" />
  </div>
</div>
<!-- snippet from reactive form -->
<div class="form-group address">
  <label for="address" for="address">Address:</label>
  <textarea formControlName="address" rows="3" class="form-control" appFormControlValidation validationMsgId="address" [pattern]="validationPatterns.address" required></textarea>
</div>
```

**Directives:**

We will be creating two directives - `appFormControlValidation` and `appFormGroupValidation`.

`appFormControlValidation` - validates single input element i.e. `ngModel` (in case of template driven form) and `FormControlName` (in case of model driven form). It iterates over the object to check for "validator" property (only if "errors" property is defined) & sets the error to the input control in case of falsy value and removes the error in case of truthy value. 
```js
if (this.control?.control?.validator && (this.control.control.validator(this.control.control) && this.control?.control?.validator(this.control?.control)?.hasOwnProperty('required'))) {
  // Need to add required error - template driven forms.
  const targetFormControl = this.control.control;
  targetFormControl.setErrors({ ...targetFormControl.errors, required: true });
} else if (this.control.validator && (this.control.validator(this.control as any) && this.control?.validator(this.control as any)?.hasOwnProperty('required'))) {
  // Need to add required error - reactive forms.
  const targetFormControl = this.control?.control;
  targetFormControl?.setErrors({ ...targetFormControl?.errors, required: true });
}
```

`appFormGroupValidation` - validates group of input elements i.e. `FormGroup`. It works exactly same as previous directive i.e. iterates over the object to check for "validator" property (only if "errors" property is defined) & sets the error to the input control in case of falsy value and removes the error in case of truthy value. 
```js
this.targetFormGroup = (this.container as NgModelGroup).control;
if (this.targetFormGroup?.statusChanges && !this.statusChangeSubscription) {
  this.statusChangeSubscription = this.targetFormGroup.statusChanges.subscribe(
  (status) => {
    if (status === 'INVALID' && (this.targetFormGroup.touched)) {
      this.showError();
    } else {
      this.removeError();
    }
  });
}
```

**Dynamic component:**

Once the validation is done, we display error messages dynamically using concept of Angular Dynamic Components.

As soon as the validation is falsy i.e. user typed incorrect input, we invoke component factory resolver with view reference of given input element & inject the error component dynamically. Once the user has corrected the input value, we remove that reference (dynamic component) from the DOM.
```js
if (dynamicItem.component) {
  const componentFactory = 
  this.componentFactoryResolver.resolveComponentFactory(dynamicItem.component);
  const parent = parentNode || vcr.element.nativeElement;
  if (parent.innerHTML.indexOf(componentFactory.selector) < 0) {
    vcr.clear();
    const componentRef = vcr.createComponent(componentFactory);
    const newChild = componentRef.injector.get(ErrorComponent).elementRef.nativeElement;
    this.renderer.appendChild(parentNode || vcr.element.nativeElement, newChild);
    (componentRef.instance as DynamicComponent).data = dynamicItem.data;
  }
}
```   

**Helper Functions:**

In this POC, we need to helper utilities:

1. **Dynamic error message generation service:**
   - It enables to generate error message dynamically.
   - We can provide the key via `validationMsgId` input property of directive. This key is use to construct error message which we then inject in DOM.
   -  Example: If the given field is required, then `ngModel` / `FormControlName` will have error property set to `{ required: true }`. This service picks up the `required` key and appends it to `validationMsgId` (say for example address) provided by user i.e. address-required. If the given field has any pattern and id provided by user is "firstname", so generate key will be "firstname-pattern"
   - If id is not provided by user, it defaults to "generic-required".
   ```html
   <input type="text" appFormControlValidation validationMsgId="address" required />
   ```
   ```js
   const errorMessages = {
     'generic-required': 'This field is required',
     'generic-pattern': 'Pattern does not match',
     'address-required': 'Please enter complete postal address',
     'address-pattern': 'Only Alpha-Numeric values and characters like `_ . / &` are allowed'
   };
   getValidationMessage(id = '-required'): string {
       return this.errorMessages[id] || this.errorMessages[`generic-${id.split('-')[1]}`];
   }
   ```

2. **Validation on form submit:**
   - Our directives only get invoked in case of "blur" or "change" event i.e. only when user interacts with the field. What if user never interacts with fields and directly tries to submit the form?
   - In that use case, we need a mechanism to let user know to provide valid input before submission.
   - On every submission, we can loop over `FormGroup`, mark all inputs as "dirty" and "touched" and re-validate them using `updateValueAndValidity()`, this enables Angular to perform validation once more.
   - To optimize this, we can add `{ onlySelf: true }` to update value and validity for given control only and not its parent elements.
   ```js
   const validateForm = (group: FormGroup): void => {
     Object.keys(group.controls).forEach((key: string) => {
       const value: AbstractControl = group.controls[key];
       if (value instanceof FormGroup) {
         value.markAsDirty();
         value.markAllAsTouched();
         validateForm(value);
         value.updateValueAndValidity({ onlySelf: true });
       } else {
         value.markAsDirty();
         value.markAsTouched();
         value.updateValueAndValidity({ onlySelf: true });
       }
    });
   };
   ```

[StackBliz Example](https://stackblitz.com/edit/angular-ivy-m4cjn4?file=src/app/app.component.ts)

