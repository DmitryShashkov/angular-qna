# Middle

### 1. Router in angular. How can you setup routing system in Angular. Wildcard route and it's usage. Router changes detection.

Angular Router is a mechanism in which navigation happens from one view to the next as users perform application tasks. It borrows the concepts or model of browser's application navigation.

A router must be configured with a list of route definitions. You configures the router with routes via the RouterModule.forRoot() method, and adds the result to the AppModule's imports array.
```
const appRoutes: Routes = [
 { path: 'todo/:id',      component: TodoDetailComponent },
 {
   path: 'todos',
   component: TodosListComponent,
   data: { title: 'Todos List' }
 },
 { path: '',
   redirectTo: '/todos',
   pathMatch: 'full'
 },
 { path: '**', component: PageNotFoundComponent }
];

@NgModule({
 imports: [
   RouterModule.forRoot(
     appRoutes,
     { enableTracing: true } // <-- debugging purposes only
   )
   // other imports here
 ],
 ...
})
export class AppModule { }
```
If the URL doesn't match any predefined routes then it causes the router to throw an error and crash the app. In this case, you can use wildcard route. A wildcard route has a path consisting of two asterisks to match every URL. For example, you can define PageNotFoundComponent for wildcard route as below:
```
{ path: '**', component: PageNotFoundComponent }
```
You can subscribe to router to detect the changes. The subscription for router events would be as below:
```
this.router.events.subscribe((event: Event) => {})
```
Let's take a simple component to detect router changes
```
import { Component } from '@angular/core';
import { Router, Event, NavigationStart, NavigationEnd, NavigationError } from '@angular/router';

@Component({
    selector: 'app-root',
    template: `<router-outlet></router-outlet>`
})
export class AppComponent {

    constructor(private router: Router) {

        this.router.events.subscribe((event: Event) => {
            if (event instanceof NavigationStart) {
                // Show loading indicator and perform an action
            }

            if (event instanceof NavigationEnd) {
                // Hide loading indicator and perform an action
            }

            if (event instanceof NavigationError) {
                // Hide loading indicator and perform an action
                console.log(event.error); // It logs an error for debugging
            }
        });
   }
}
```

### 2. How can you select an element in component template? (@ViewChild, 2 types of parameters).
You can use @ViewChild directive to access elements in the view directly. Let's take input element with a reference,
```
<input #uname>
```
and define view child directive and access it in ngAfterViewInit lifecycle hook
```
@ViewChild('uname') input;

ngAfterViewInit() {
  console.log(this.input.nativeElement.value);
}
```
Since Angular 8, the ViewChild and ContentChild decorators now must have a new option called static. Let me explain why with a very simple example using a ViewChild:
```
<div *ngIf="true">
  <div #dynamicDiv>dynamic</div>
</div>
```
Let’s get that element in our component and log it in the lifecycle hooks ngOnInit and ngAfterViewInit:
```
@ViewChild('dynamicDiv') dynamicDiv: ElementRef<HTMLDivElement>;

ngOnInit() {
  console.log('init dynamic', this.dynamicDiv); // undefined
}

ngAfterViewInit() {
  console.log('after view init dynamic', this.dynamicDiv); // div
}
```
Makes sense as AfterViewInit is called when the template initialization is done.

But in fact, if the queried element is static (not wrapped in an ngIf or an ngFor), then it is available in ngOnInit also:
```
<h1 #staticDiv>static</h1>
```
gives:
```
@ViewChild('staticDiv') staticDiv: ElementRef<HTMLDivElement>;

ngOnInit() {
  console.log('init static', this.staticDiv); // div
}

ngAfterViewInit() {
  console.log('after view init static', this.staticDiv); // div
}
```
This was not documented, or recommended, but that’s how it currently works.

With Ivy though, the behavior changes to be more consistent:
```
ngOnInit() {
  console.log('init static', this.staticDiv); // undefined (changed)
}

ngAfterViewInit() {
  console.log('after view init static', this.staticDiv); // div
}
```
A new static flag has been introduced to not break existing applications, so if you want to keep the old behavior even when you’ll switch to Ivy, you can write:
```
@ViewChild('static', { static: true }) static: ElementRef<HTMLDivElement>;
```
and the behavior will be the same as the current one (the element is also accessible in ngOnInit).

Note that if you add static: true on a dynamic element (wrapped in a condition or a loop), then it will not be accessible in ngOnInit nor in ngAfterViewInit!

### 3. ng-container, ng-template, ng-content. What's the difference.

https://blog.angular-university.io/angular-ng-template-ng-container-ngtemplateoutlet/

https://www.freecodecamp.org/news/everything-you-need-to-know-about-ng-template-ng-content-ng-container-and-ngtemplateoutlet-4b7b51223691/

### 4. Observable creation functions? Name some of RxJS pipes and describe its usage (switchMap, flatMap, share, shareReplay, forkJoin, retry...). How do you perform error handling in observables? (pipe and subscribe).

https://www.learnrxjs.io/operators/creation/

##### switchMap

https://www.learnrxjs.io/operators/transformation/switchmap.html

Map to observable, complete previous inner observable, emit values.

EXAMPLE : RESTART INTERVAL ON EVERY CLICK
```
// RxJS v6+
import { interval, fromEvent } from 'rxjs';
import { switchMap } from 'rxjs/operators';

fromEvent(document, 'click')
  .pipe(
    // restart counter on every click
    switchMap(() => interval(1000))
  )
  .subscribe(console.log);
```

##### mergeMap

https://www.learnrxjs.io/operators/transformation/mergemap.html

Map to observable, emit values.

EXAMPLE : MERGEMAP WITH OBSERVABLE
```
// RxJS v6+
import { of } from 'rxjs';
import { mergeMap } from 'rxjs/operators';

//emit 'Hello'
const source = of('Hello');
//map to inner observable and flatten
const example = source.pipe(mergeMap(val => of(`${val} World!`)));
//output: 'Hello World!'
const subscribe = example.subscribe(val => console.log(val));
```

##### share

https://www.learnrxjs.io/operators/multicasting/share.html

Share source among multiple subscribers.

EXAMPLE : MULTIPLE SUBSCRIBERS SHARING SOURCE
```
// RxJS v6+
import { timer } from 'rxjs';
import { tap, mapTo, share } from 'rxjs/operators';

//emit value in 1s
const source = timer(1000);
//log side effect, emit result
const example = source.pipe(
  tap(() => console.log('***SIDE EFFECT***')),
  mapTo('***RESULT***')
);

/*
  ***NOT SHARED, SIDE EFFECT WILL BE EXECUTED TWICE***
  output:
  "***SIDE EFFECT***"
  "***RESULT***"
  "***SIDE EFFECT***"
  "***RESULT***"
*/
const subscribe = example.subscribe(val => console.log(val));
const subscribeTwo = example.subscribe(val => console.log(val));

//share observable among subscribers
const sharedExample = example.pipe(share());
/*
  ***SHARED, SIDE EFFECT EXECUTED ONCE***
  output:
  "***SIDE EFFECT***"
  "***RESULT***"
  "***RESULT***"
*/
const subscribeThree = sharedExample.subscribe(val => console.log(val));
const subscribeFour = sharedExample.subscribe(val => console.log(val));
```

##### shareReplay

https://www.learnrxjs.io/operators/multicasting/sharereplay.html

Share source and replay specified number of emissions on subscription.

EXAMPLE: MULTIPLE SUBSCRIBERS SHARING SOURCE
```
// RxJS v6+
import { Subject, ReplaySubject } from 'rxjs';
import { pluck, share, shareReplay, tap } from 'rxjs/operators';

// simulate url change with subject
const routeEnd = new Subject<{data: any, url: string}>();
// grab url and share with subscribers
const lastUrl = routeEnd.pipe(
  tap(_ => console.log('executed')),
  pluck('url'),
  // defaults to all values so we set it to just keep and replay last one
  shareReplay(1)
);
// requires initial subscription
const initialSubscriber = lastUrl.subscribe(console.log)
// simulate route change
// logged: 'executed', 'my-path'
routeEnd.next({data: {}, url: 'my-path'});
// logged: 'my-path'
const lateSubscriber = lastUrl.subscribe(console.log);
```

##### forkJoin

https://www.learnrxjs.io/operators/combination/forkjoin.html

When all observables complete, emit the last emitted value from each.

EXAMPLE: (V6.5+) USING A DICTIONARY OF SOURCES
```
// RxJS v6.5+
import { ajax } from 'rxjs/ajax';
import { forkJoin } from 'rxjs';

/*
  when all observables complete, provide the last
  emitted value from each as dictionary
*/
forkJoin(
  // as of RxJS 6.5+ we can use a dictionary of sources
  {
    google: ajax.getJSON('https://api.github.com/users/google'),
    microsoft: ajax.getJSON('https://api.github.com/users/microsoft'),
    users: ajax.getJSON('https://api.github.com/users')
  }
)
  // { google: object, microsoft: object, users: array }
  .subscribe(console.log);
```

##### retry

https://www.learnrxjs.io/operators/error_handling/retry.html

Retry an observable sequence a specific number of times should an error occur.

Error handling in RxJS may be performed either with `catchError` operator, or with `subscribe` error callback

https://blog.angular-university.io/rxjs-error-handling/

### 5. Reactive Forms and Model Driven forms. FormGroup and FormControl. Could be nice to hear something about FormBuilder, custom validation and custom inputs (ControlValueAccessor).

https://angular.io/guide/forms

https://angular.io/guide/reactive-forms
