# Trainee

### 1. What are directives? What are pipes?

Directives add behaviour to an existing DOM element or an existing component instance.
```
import { Directive, ElementRef, Input } from '@angular/core';

@Directive({ selector: '[myHighlight]' })
export class HighlightDirective {
    constructor(el: ElementRef) {
       el.nativeElement.style.backgroundColor = 'yellow';
    }
}
```
Now this directive extends HTML element behavior with a yellow background as below
```
<p myHighlight>Highlight me!</p>
```

A pipe takes in data as input and transforms it to a desired output. A pipe can accept any number of optional parameters to fine-tune its output. The parameterized pipe can be created by declaring the pipe name with a colon (:) and then the parameter value. If the pipe accepts multiple parameters, separate the values with colons. Let's take a birthday example with a particular format(dd/mm/yyyy):
```
import { Component } from '@angular/core';

@Component({
  selector: 'app-birthday',
  template: `<p>Birthday is {{ birthday | date:'dd/mm/yyyy'}}</p>` // 18/06/1987
})
export class BirthdayComponent {
  birthday = new Date(1987, 6, 18);
}
```
Note: The parameter value can be any valid template expression, such as a string literal or a component property.

You can chain pipes together in potentially useful combinations as per the needs.

Apart from built-in pipes, you can write your own custom pipe with the below key characteristics:

- A pipe is a class decorated with pipe metadata @Pipe decorator, which you import from the core Angular library. For example: `@Pipe({name: 'myCustomPipe'})`
- The pipe class implements the PipeTransform interface's transform method that accepts an input value followed by optional parameters and returns the transformed value. The structure of pipeTransform would be as below:
    ```
    interface PipeTransform {
        transform(value: any, ...args: any[]): any
    }
    ```
- The @Pipe decorator allows you to define the pipe name that you'll use within template expressions. It must be a valid JavaScript identifier.
    ```
    template: `{{someInputValue | myCustomPipe: someOtherValue}}`
    ```

### 2. What are components and modules?
Components are the most basic UI building block of an Angular app which formed a tree of Angular components. These components are subset of directives. Unlike directives, components always have a template and only one component can be instantiated per an element in a template. Let's see a simple example of Angular component
```
import { Component } from '@angular/core';

@Component ({
   selector: 'my-app',
   template: ` <div>
      <h1>{{title}}</h1>
      <div>Learn Angular6 with examples</div>
   </div> `,
})

export class AppComponent {
   title: string = 'Welcome to Angular world';
}
```
Modules are logical boundaries in your application and the application is divided into separate modules to separate the functionality of your application. Lets take an example of app.module.ts root module declared with @NgModule decorator as below,
```
import { NgModule }      from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent }  from './app.component';

@NgModule ({
   imports:      [ BrowserModule ],
   declarations: [ AppComponent ],
   bootstrap:    [ AppComponent ]
})
export class AppModule { }
```
The @NgModule() decorator is a function that takes a single metadata object, whose properties describe the module. The most important properties are as follows.
- declarations: The components, directives, and pipes that belong to this NgModule.
- exports: The subset of declarations that should be visible and usable in the component templates of other NgModules.
- imports: Other modules whose exported classes are needed by component templates declared in this NgModule.
- providers: Creators of services that this NgModule contributes to the global collection of services; they become accessible in all parts of the app. (You can also specify providers at the component level, which is often preferred.)
- bootstrap: The main application view, called the root component, which hosts all other app views. Only the root NgModule should set the bootstrap property.

### 3. Lifecycle hooks
Angular application goes through an entire set of processes or has a lifecycle right from its initiation to the end of the application. The representation of lifecycle in pictorial representation as follows:

![NG lifecycle](https://github.com/sudheerj/angular-interview-questions/raw/master/images/lifecycle.png)

The description of each lifecycle method is as below^

- ngOnChanges: When the value of a data bound property changes, then this method is called.
- ngOnInit: This is called whenever the initialization of the directive/component after Angular first displays the data-bound properties happens.
- ngDoCheck: This is for the detection and to act on changes that Angular can't or won't detect on its own.
- ngAfterContentInit: This is called in response after Angular projects external content into the component's view.
- ngAfterContentChecked: This is called in response after Angular checks the content projected into the component.
- ngAfterViewInit: This is called in response after Angular initializes the component's views and child views.
- ngAfterViewChecked: This is called in response after Angular checks the component's views and child views.
- ngOnDestroy: This is the cleanup phase just before Angular destroys the directive/component.

### 4. Types of data binding
Data binding is a core concept in Angular and allows to define communication between a component and the DOM, making it very easy to define interactive applications without worrying about pushing and pulling data. There are four forms of data binding(divided as 3 categories) which differ in the way the data is flowing.

##### From the Component to the DOM:
Interpolation: {{ value }}: Adds the value of a property from the component
```
<li>Name: {{ user.name }}</li>
<li>Address: {{ user.address }}</li>
```
Property binding: `[property]=”value”`: The value is passed from the component to the specified property or simple HTML attribute
```
<input type="email" [value]="user.email">
```
##### From the DOM to the Component:
Event binding: (event)=”function”: When a specific DOM event happens (eg.: click, change, keyup), call the specified method in the component
```
<button (click)="logout()"></button>
```
##### Two-way binding:
Two-way data binding: `[(ngModel)]=”value”`: Two-way data binding allows to have the data flow both ways. For example, in the below code snippet, both the email DOM input and component email property are in sync
```
<input type="email" [(ngModel)]="user.email">
```

##### Mention also:
[HostListener](https://angular.io/api/core/HostListener)
Declares a host listener. Angular will invoke the decorated method when the host element emits the specified event. Will listen to the event emitted by host element, declared with @HostListener.

[HostBinding](https://angular.io/api/core/HostBinding)
Declares a host property binding. Angular automatically checks host property bindings during change detection. If a binding changes, it will update the host element of the directive. Will bind property to host element, If a binding changes, HostBinding will update the host element.

### 5. Structural directives(ngIf, ngFor). Template expressions and interpolation

The Angular ngIf directive inserts or removes an element based on a truthy/falsy condition. Let's take an example to display a message if the user age is more than 18,
```
<p *ngIf="user.age > 18">You are not eligible for student pass!</p>
```
Note: Angular isn't showing and hiding the message. It is adding and removing the paragraph element from the DOM. That improves performance, especially in the larger projects with many data bindings.

Angular ngFor directive is used in the template to display each item in the list. For example, here we iterate over list of users,
```
<li *ngFor="let user of users">
  {{ user }}
</li>
```
The user variable in the ngFor double-quoted instruction is a template input variable.

Note: you cannot use two structural directives on one DOM element.

Interpolation is a special syntax that Angular converts into property binding. It’s a convenient alternative to property binding. It is represented by double curly braces({{}}). The text between the braces is often the name of a component property. Angular replaces that name with the string value of the corresponding component property. Let's take an example,

```
<h3>
  {{title}}
  <img src="{{url}}" style="height:30px">
</h3>
```
In the example above, Angular evaluates the title and url properties and fills in the blanks, first displaying a bold application title and then a URL.

A template expression produces a value similar to any Javascript expression. Angular executes the expression and assigns it to a property of a binding target; the target might be an HTML element, a component, or a directive. In the property binding, a template expression appears in quotes to the right of the = symbol as in [property]="expression". In interpolation syntax, the template expression is surrounded by double curly braces. For example, in the below interpolation, the template expression is {{username}},
```
<h3>{{username}}, welcome to Angular</h3>
```
The below javascript expressions are prohibited in template expression:
- assignments (=, +=, -=, ...)
- new
- chaining expressions with ; or ,
- increment and decrement operators (++ and --)
