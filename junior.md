# Junior

### 1. What is RxJS? How to chain Rx functions? What is subscription? Shorthand notation for subscription.

RxJS is a library for composing asynchronous and callback-based code in a functional, reactive style using Observables. Many APIs such as HttpClient produce and consume RxJS Observables and also uses operators for processing observables. For example, you can import observables and operators for using HttpClient as below:
```
import { Observable, throwError } from 'rxjs';
import { catchError, retry } from 'rxjs/operators';
```

As of RxJS version 5.5 and above, RxJS operators may be chained using `pipe` method:
```
of(1,2,3).pipe(
  map(x => x + 1),
  filter(x => x > 2)
);
```
An Observable instance begins publishing values only when someone subscribes to it. So you need to subscribe by calling the subscribe() method of the instance, passing a handler to receive the notifications.
```
dialogRef.content.onGenerated.subscribe(() => {
    this.promoCodesTable.refresh();
});
```
The subscribe() method can accept callback function definitions in line, for next, error, and complete handlers, which is known as shorthand notation or Subscribe method with positional arguments. For example, you can define subscribe method as below:
```
myObservable.subscribe(
  x => console.log('Observer got a next value: ' + x),
  err => console.error('Observer got an error: ' + err),
  () => console.log('Observer got a complete notification')
);
```

### 2. How to use HttpClient module? (explain how to setup it, make GET/POST request and process full response)
Below are the steps needed to be followed for the usage of HttpClient.

- Import HttpClient into root module:
```
import { HttpClientModule } from '@angular/common/http';
@NgModule({
  imports: [
    BrowserModule,
    // import HttpClientModule after BrowserModule.
    HttpClientModule,
  ],
  ......
  })
 export class AppModule {}
 ```

- Inject the HttpClient into the application: Let's create a userProfileService(userprofile.service.ts) as an example. It also defines get method of HttpClient
```
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

const userProfileUrl: string = 'assets/data/profile.json';

@Injectable()
export class UserProfileService {
  constructor(private http: HttpClient) { }

  getUserProfile() {
    return this.http.get(this.userProfileUrl);
  }
}
```
- Create a component for subscribing service: Let's create a component called UserProfileComponent(userprofile.component.ts) which inject UserProfileService and invokes the service method,
```
fetchUserProfile() {
  this.userProfileService.getUserProfile()
    .subscribe((data: User) => this.user = {
        id: data['userId'],
        name: data['firstName'],
        city:  data['city']
    });
}
```
Since the above service method returns an Observable which needs to be subscribed in the component.

The response body doesn't return full response data because sometimes servers also return special headers or status code which which are important for the application workflow. Inorder to get full response, you should use observe option from HttpClient,
```
getUserResponse(): Observable<HttpResponse<User>> {
  return this.http.get<User>(
    this.userUrl, { observe: 'response' });
}
```
Now HttpClient.get() method returns an Observable of typed HttpResponse rather than just the JSON data.

### 3. Interface, Type, Class - what's the difference?
[Class vs. Interface](https://stackoverflow.com/questions/40973074/difference-between-interfaces-and-classes-in-typescript)

[Type vs. Interface](https://medium.com/@martin_hotell/interface-vs-type-alias-in-typescript-2-7-2a8f1777af4c)

### 4. Lazy loading. Description and sample of usage.
Lazy loading is one of the most useful concepts of Angular Routing. It helps us to download the web pages in chunks instead of downloading everything in a big bundle. It is used for lazy loading by asynchronously loading the feature module for routing whenever required using the property loadChildren. Let's load both Customer and Order feature modules lazily as below,
```
const routes: Routes = [
  {
    path: 'customers',
    loadChildren: () => import('./customers/customers.module').then(module => module.CustomersModule)
  },
  {
    path: 'orders',
    loadChildren: () => import('./orders/orders.module').then(module => module.OrdersModule)
  },
  {
    path: '',
    redirectTo: '',
    pathMatch: 'full'
  }
];
```

### 5. Dependency injection in Angular. Samples of injections (Services, @Optional decorator...)
Dependency injection (DI), is an important application design pattern in which a class asks for dependencies from external sources rather than creating them itself. Angular comes with its own dependency injection framework for resolving dependencies( services or objects that a class needs to perform its function).So you can have your services depend on other services throughout your application.
```
import { Injectable } from '@angular/core';
import { Http } from '@angular/http';

@Injectable({ // The Injectable decorator is required for dependency injection to work
  // providedIn option registers the service with a specific NgModule
  providedIn: 'root',  // This declares the service with the root app (AppModule)
})
export class RepoService{
  constructor(private http: Http){
  }

  fetchAll(){
    return this.http.get('https://api.github.com/repositories');
  }
}
```

See also: [@Self vs. @Optional vs. @Host](https://medium.com/frontend-coach/self-or-optional-host-the-visual-guide-to-angular-di-decorators-73fbbb5c8658)
