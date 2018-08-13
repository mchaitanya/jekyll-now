---
layout: post
title: Angular as HTML++ - Part 1 - Components & Directives
---

In this series of posts, we'll see how Angular helps us develop and maintain web apps by introducing neat abstractions that build on top of the existing HTML/CSS/DOM APIs. The strength of the web lies in its declarative and reactive nature. In most cases, Angular lets us move away from cumbersome, imperative DOM APIs to a much easier-to-use declarative style of coding. And Angular's embrace of rxjs let us move towards a more reactive style of coding, not just for DOM events, but for other data flows in the app.

## Component as HTML element++
Components are the basic building blocks of an app - higher-level domain-specific constructs that are built up from the basic web primitives. By primitives, I mean all the elements, properties etc. that are defined in the HTML spec (like `h1`, `p`, `div`, `span` etc.). If nothing else, components vastly improve the readability of the code - the markup contains elements that are problem-domain analogs. 

A component is comparable to the idea of a class that is at the core of all object-oriented languages. The class is a blueprint that is composed from language primitives like `int`s, `boolean`s etc. and other classes. For example, consider a class in Java:
```java
public class Person { 
    String name;
    int age;
    Address address;
}
```
Now, imagine a Person component with this template. Here, primitives like `h1` & `div` are combined with the `address` component.
```html
<h1>{{ person.name }}</h1>
<div class="age">{{ person.age }}</div>
<app-address [address]="person.address"></app-address>
```

Now in addition to the presentational aspect (that goes in the template & styles), a component also has internal state. This goes inside the controller class (with the `@Component` decorator). The controller and the template communicate through bindings. This way state can flow into the DOM properties and events from the DOM can flow back into the controller and update the state.

Just like a class is used to create objects that can be manipulated during the course of a program, the component can be used to create views that can be inserted/removed/hidden etc. during the lifetime of the app. To use a component, you just drop it in the template of another component. You are spared the trouble of working directly with the DOM API; you don't have to tell Angular how to create the view with methods like `createElement`, `appendChild` etc. Also, you just set up the bindings in the template and Angular's change detection will re-render the DOM any time the state changes. Again all declarative; you don't have to query for an element to modify its `classList`, `addEventListener` etc.

## Directive as CSS class++
The css class is meant for declaratively modifying the appearance of an element. The same class can be applied to many elements; styling should be a concern orthogonal to the content. Now, over time, the humble `class` attribute has come to be used for many purposes. It can act as a marker for:
 1. applying styles
 2. signifying semantic information  
 Consider `<table class="products">` as opposed to `<table class="striped">`, which is purely presentational.
 3. for DOM queries  
 You can use the class name later in your JS code to do `getElementsByClassName('class')` or `querySelectorAll('.class')`. Sometimes, developers prefix class names with `'js-'` to denote that this is what the class is for.

The directive generalizes the css class to not only declaratively modify the style of an element but also its properties/behavior/structure. Maxim Koretskyi (from the [AngularInDepth](https://blog.angularindepth.com/) blog) [explains here](https://www.youtube.com/watch?v=qWmqiYDrnDc) that, ideally, components should be in charge of presentation logic & directives in charge of rendering logic. This rendering logic might involve just simple style changes as in regular css (think `NgStyle` or `NgClass`), changes to DOM properties or more heavy-duty changes to the DOM structure (think `NgIf` or `NgFor`). Let's see how each of the above applications of the `class` attribute can be dealt with in Angular: 
 1. It's best to just continue to use css classes to apply styles. It is hard to replicate the power of css selectors & combinators with directives. And there are a lot of other techniques today to write modular CSS. As an example of an overkill directive, you can do this just to set the font size to 24px on the host element:  
     ```ts
     @Directive({
       selector: '[standout]'
     })
     export class StandoutDirective {
       @HostBinding('style.fontSize.px') fontSize = 24;
     }
     ```
 2. This use of classes should go down a lot, since the component itself is a natural choice as a semantic unit. Perhaps at the subcomponent level, it might still make sense to describe portions of the template semantically with classes. For example, inside the template of a `HeaderComponent`, you might find:  
     ```html
     <div class="logo">
       <img src="logo.jpg" alt="Logo" />
     </div>
     <div class="banner">
       <h1>App Title</h1>
     </div>
     ```
 3. You almost never have to query for elements in Angular. If you apply a directive on an element, that element can just be injected into the directive with Dependency Injection. Once you have a reference to the element, you can modify its properties, nest other elements inside it etc. For example:  
     ```ts
     @Directive({
       selector: '[highlight]'
     })
     export class HighlightDirective implements OnInit {
       @Input('highlight') color: string;
       defaultColor = 'yellow';
       constructor(private el: ElementRef, private renderer: Renderer2) { }
       
       ngOnInit() {
         this.renderer.setStyle(this.el.nativeElement, 'backgroundColor', this.color || this.defaultColor);
       }
     }
     ```

     Here's how we might use this directive. 
     ```html
     <p>blah blah ... <span highlight>Color me default</span>, <span highlight="lightblue">Color me lightblue</span> ... blah blah</p>
     ```

     Let's see how we might implement the above directive in a vanilla JS app. This might not seem so bad, but keep in mind that this is imperative code: we query for all elements that have the `data-highlight` attribute and then iterate through the list, applying the background color to each element.
     ```js
     var defaultColor = 'yellow';
     var elems = document.querySelectorAll('[data-highlight]');
     elems.forEach(function(elem) {
       var color = elem.getAttribute('data-highlight');
       elem.style.backgroundColor = color || defaultColor;
     });
     ```
     ```html
     <p>blah blah ... <span data-highlight>Color me default</span>, <span data-highlight="lightblue">Color me lightblue</span> ... blah blah</p>
     ```
