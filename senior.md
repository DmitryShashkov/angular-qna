# Senior

### 1. What are JIT and AOT? Pros and cons.

Just-in-Time (JIT) is a type of compilation that compiles your app in the browser at runtime. JIT compilation is the default when you run the ng build (build only) or ng serve (build and serve locally) CLI commands. i.e, the below commands used for JIT compilation,
```
ng build
ng serve
```
Ahead-of-Time (AOT) is a type of compilation that compiles your app at build time. For AOT compilation, include the --aot option with the ng build or ng serve command as below,
```
ng build --aot
ng serve --aot
```
Below are the list of AOT benefits:

- Faster rendering: The browser downloads a pre-compiled version of the application. So it can render the application immediately without compiling the app.
- Fewer asynchronous requests: It inlines external HTML templates and CSS style sheets within the application javascript which eliminates separate ajax requests.
- Smaller Angular framework download size: Doesn't require downloading the Angular compiler. Hence it dramatically reduces the application payload.
- Detect template errors earlier: Detects and reports template binding errors during the build step itself
- Better security: It compiles HTML templates and components into JavaScript. So there won't be any injection attacks.

### 2. What is Angular Universal? Pros and cons. What are Angular Elements?

Angular Universal is a server-side rendering module for Angular applications in various scenarios. This is a community driven project and available under @angular/platform-server package. Recently Angular Universal is integrated with Angular CLI.

https://angular.io/guide/universal

Restrictions: https://github.com/angular/universal/blob/master/docs/gotchas.md

### 3. NgZone? Render behaviour and optimization (OnPush detection strategy).

##### Change Detection
On each asynchronous event Angular performs change detection over the entire component tree. Although the code which detects for changes is optimized for inline-caching, this still can be a heavy computation in complex applications. A way to improve the performance of the change detection is to not perform it for subtrees which are not supposed to be changed based on the recent actions.

##### ChangeDetectionStrategy.OnPush
The OnPush change detection strategy allows us to disable the change detection mechanism for subtrees of the component tree. By setting the change detection strategy to any component to the value ChangeDetectionStrategy.OnPush, will make the change detection perform only when the component have received different inputs. Angular will consider inputs as different when it compares them with the previous inputs by reference, and the result of the reference check is false. In combination with immutable data structures OnPush can bring great performance implications for such "pure" components.

["Change Detection in Angular"](https://vsavkin.com/change-detection-in-angular-2-4f216b855d4c)

["Everything you need to know about change detection in Angular"](https://blog.angularindepth.com/everything-you-need-to-know-about-change-detection-in-angular-8006c51d206f)

##### Detaching the Change Detector
Another way of implementing a custom change detection mechanism is by detaching and reattaching the change detector (CD) for given component. Once we detach the CD Angular will not perform check for the entire component subtree.

This practice is typically used when user actions or interactions with an external services trigger the change detection more often than required. In such cases we may want to consider detaching the change detector and reattaching it only when performing change detection is required.

##### Run outside Angular

The Angular's change detection mechanism is being triggered thanks to [zone.js](https://github.com/angular/zone.js). Zone.js monkey patches all asynchronous APIs in the browser and triggers the change detection in the end of the execution of any async callback. In rare cases we may want given code to be executed outside the context of the Angular Zone and thus, without running change detection mechanism. In such cases we can use the method runOutsideAngular of the NgZone instance.

In the snippet below, you can see an example for a component which uses this practice. When the _incrementPoints method is called the component will start incrementing the _points property every 10ms (by default). The incrementation will make the illusion of an animation. Since in this case we don't want to trigger the change detection mechanism for the entire component tree, every 10ms, we can run _incrementPoints outside the context of the Angular's zone and update the DOM manually (see the points setter).

```
@Component({
  template: '<span #label></span>'
})
class PointAnimationComponent {

  @Input() duration = 1000;
  @Input() stepDuration = 10;
  @ViewChild('label') label: ElementRef;

  @Input() set points(val: number) {
    this._points = val;
    if (this.label) {
      this.label.nativeElement.innerText = this._pipe.transform(this.points, '1.0-0');
    }
  }
  get points() {
    return this._points;
  }

  private _incrementInterval: any;
  private _points: number = 0;

  constructor(private _zone: NgZone, private _pipe: DecimalPipe) {}

  ngOnChanges(changes: any) {
    const change = changes.points;
    if (!change) {
      return;
    }
    if (typeof change.previousValue !== 'number') {
      this.points = change.currentValue;
    } else {
      this.points = change.previousValue;
      this._ngZone.runOutsideAngular(() => {
        this._incrementPoints(change.currentValue);
      });
    }
  }

  private _incrementPoints(newVal: number) {
    const diff = newVal - this.points;
    const step = this.stepDuration * (diff / this.duration);
    const initialPoints = this.points;
    this._incrementInterval = setInterval(() => {
      let nextPoints = Math.ceil(initialPoints + diff);
      if (this.points >= nextPoints) {
        this.points = initialPoints + diff;
        clearInterval(this._incrementInterval);
      } else {
        this.points += step;
      }
    }, this.stepDuration);
  }
}
```
Warning: Use this practice very carefully only when you're sure what you are doing because if not used properly it can lead to an inconsistent state of the DOM. Also note that the code above is not going to run in WebWorkers. In order to make it WebWorker-compatible, you need to set the label's value by using the Angular's renderer.

### 4. DI details: DI providers, multiple service instances, custom providers (useValue, useClass, useExisting).

https://angular.io/guide/dependency-injection-providers
