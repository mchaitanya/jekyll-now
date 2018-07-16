---
layout: post
title: Passing TemplateRefs Around in Angular
---

In our Angular apps, we pass data from one component to another all the time. We can facilitate the exchange through a shared service or just use simple input/output bindings if the components are in a direct parent-child relationship. Most often, the data are just plain JavaScript primitives/objects. 

But there are interesing use cases when we might want to pass a template/chunk of HTML from one component to another. This is a pretty powerful pattern: we can define the general structure of common UI component and switch in/out dynamic content based on our app's logic. 

Let's explore an example built with the [ng-bootstrap package](https://ng-bootstrap.github.io/). We have a set of reusable modal components to display warnings, prompts etc. and any component in the app can launch one of these modals and insert their custom content into the modal body.

<iframe style="width:100%; height:400px" src="https://stackblitz.com/edit/angular-passing-templaterefs?embed=1&file=src/app/app.component.html&hidedevtools=1"></iframe>

Here's the general pattern. The template is defined in a component with the `ng-template` tag. For example, in `Example1Component`, we have:
```html {% raw %}
<!-- Example1Component -->
<button (click)="warn(tmpl)">Display Simple Warning</button>
...
<ng-template #tmpl>
  <p>This is a simple warning from {{ cmpName }}</p>
</ng-template>
{% endraw %}```

We pass a reference to this template to the click handler of the button. Note that we can treat this templateRef like any other variable. The templateRef is passed to the `displayWarningModal` method in the `ModalService` and then on to the `WarningModalComponent`.
```ts
// ModalService
displayWarningModal(tmpl: TemplateRef<any>) {
    const modalRef = this.modalService.open(WarningModalComponent);
    modalRef.componentInstance.tmpl = tmpl;
}
```
In the `WarningModalComponent`, we can embed this templateRef with the convenient [`*ngTemplateOutlet` directive](https://angular.io/api/common/NgTemplateOutlet). Dealing with the templateRefs in this way keeps our component class clean of `ViewChild` annotations to get access to the templateRef or to the view container in which to insert them.
```html
<!-- WarningModalComponent -->
<ng-container *ngTemplateOutlet="tmpl"></ng-container>
```

Now, the templateRefs keep all our bindings intact. The bindings are evaluated in the context of the original component instance, where they are defined. For example, the string interpolation {% raw %}`{{ cmpName }}`{% endraw %} in `Example1Component`'s template evaluates to 'example1' and not 'warning modal' as defined in the `WarningModalComponent`. Note that the styling defined in `Example2Component` (`.special { color: orange }`) is applied to its template even though it is instantiated in the `WarningModalComponent`. 

The fact that the template maintains a connection to the original component state can be leveraged in interesting ways. In `Example3Component`, we set up two-way binding to the `fontSize` and `fontColor` properties. We can now bring up the settings modal, adjust the options there and instantly see the changes applied to the text. 

***

Also check out:
* [Ng-Bootstrap's Modal Component](https://ng-bootstrap.github.io/#/components/modal/examples)
* [Minko Gechev's article, 'Understanding Dynamic Scoping and TemplateRef' ](https://blog.mgechev.com/2017/10/01/angular-template-ref-dynamic-scoping-custom-templates/)  
This demonstrates a really nice way to use `ng-template` with content projection.
* [Alex Rickabaugh's AngularMIX talk, 'Advanced Angular Concepts' (HIGHLY RECOMMENDED!)](https://www.youtube.com/watch?v=rKbY1t39dHU)  
The first part of the video covers this topic in the context of building a left nav, a portion of which depends on the currently loaded route. 
