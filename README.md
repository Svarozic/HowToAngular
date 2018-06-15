# HowToAngular2


### Deserialize HTTP Json reposense in object
- https://nehalist.io/working-with-models-in-angular/


### Template Slot system
- `list.ts`
```html
<!-- PARENT -->
<celum-list>
  <!-- name template for container in celum-list -->
  <div top-list-item></div>
</celum-list>


<!-- CHILD celum-list -->
<div>
  TMPL content
  <!-- slot with specific select -->
  <ng-content select="[top-list-item]"></ng-content>
</div>
```



### window.requestAnimationFrame
- https://developer.mozilla.org/de/docs/Web/API/window/requestAnimationFrame
- https://github.com/angular/angular/issues/8804
```js
this.ngZone.runOutsideAngular(() => {
      requestAnimationFrame(() => {
        //code
      });
});
```



### Window resize event

#### Simple with debounce in component
```
private windowResizeEvent$: BehaviorSubject<number> = new BehaviorSubject(window.innerHeight);

@HostListener('window:resize', ['$event.target.innerHeight'])
onWindowResize(innerHeight: number) {
  this.windowResizeEvent$.next(innerHeight);
}

this.windowResizeEvent$
     .debounceTime(10) //GRYF throttling !
     .subscribe((windowInnerHeight) => {
       this.calculateIFrameHeight(windowInnerHeight);
     });
```

#### As angular service with EventManager
- https://stackoverflow.com/questions/35527456/angular-window-resize-event
  - plus improvement: Observables is not created each time getter is called, it is created once in constructor

window-events.service.ts
```js
import { EventManager } from '@angular/platform-browser';
import { Observable } from 'rxjs/Observable';
import { Subject } from 'rxjs/Subject';
import { Injectable } from '@angular/core';

@Injectable()
export class WindowEventsService {

  private static RESIZE_DEBOUNCE_TIME_MS = 10;

  private _resizeSubject$: Subject<Window>;
  /**
   * Used to provide {@linkcode _resizeSubject$}
   *  - in safe way out of service
   *  - with filtered values with debounce and distinct
   */
  private resize$: Observable<Window>;

  constructor(private eventManager: EventManager) {
    this._resizeSubject$ = new Subject();
    this.resize$ = this._resizeSubject$.asObservable()
                       .distinctUntilChanged()
                       .debounceTime(WindowEventsService.RESIZE_DEBOUNCE_TIME_MS);

    this.eventManager.addGlobalEventListener('window', 'resize', this.onWindowResize.bind(this));
  }

  private onWindowResize(event: UIEvent) {
    this._resizeSubject$.next(<Window>event.target);
  }

  get onResize$(): Observable<Window> {
    return this.resize$;
  }
}
```

usage in component
```js
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'my-component',
  template: ``,
  styles: [``]
})
export class MyComponent implements OnInit {

  private resizeSubscription: Subscription;

  constructor(private windowEventsService: WindowEventsService) { }

  ngOnInit() {
     // if ChangeDetectionStrategy.OnPush is used in component, markForCheck ChangeDetector !!
    this.resizeSubscription = this.windowEventsService.onResize$
      .subscribe(size => console.log(size));
  }

  ngOnDestroy() {
    if (this.resizeSubscription) {
      this.resizeSubscription.unsubscribe();
    }
  }
}
```



### ChangeDetectionStrategy
- Change Detection Differences between Angular and AngularJS - Mike Giambalvo https://youtu.be/CMkKzQmOnyg
- 44 min NG-NL 2016: Pascal Precht - Angular 2 Change Detection Explained https://www.youtube.com/watch?v=CUxD91DWkGM

#### ChangeDetectorRef.detectChanges() vs ChangeDetectorRef.markForCheck()
https://stackoverflow.com/questions/41364386/whats-the-difference-between-markforcheck-and-detectchanges
So in short :
- Use detectChanges() when you've updated the model after angular has run it's change detection, or if the update hasn't been in angular world at all.
- Use markForCheck() if you're using OnPush and you're bypassing the ChangeDetectionStrategy by mutating some data or you've updated the model inside a setTimeout;

https://alligator.io/angular/change-detection-strategy/



### HTML Base Tag
#### for production on build
- https://github.com/angular/angular-cli/blob/master/docs/documentation/build.md#base-tag-handling-in-indexhtml
```
# Sets base tag href to /myUrl/ in your index.html
ng build --base-href /myUrl/
ng build --bh /myUrl/
```
#### for NgRouter
- https://angular.io/guide/router#set-the-base-href
- You only need this trick for the live example, not production code
```html
<script>document.write('<base href="' + document.location + '" />');</script>
```



### Elegant way to unsubscribe Observables
- https://www.reddit.com/r/Angular2/comments/67q5us/when_to_unsubscribe_in_angular/?st=j26h7w73&sh=96e72432

```typescript
@Component({
  selector: 'blah',
  template: 'blah',
})
export class MyComponent {
  private onDestroy$ = new Subject();

  constructor(http: Http) {
    // Use .takeUntil() instead of tracking subscriptions manually. This should be the last
    // operator before .subscribe().
    http.get('/url').takeUntil(this.onDestroy$).subscribe(...);
  }

  ngOnDestroy() {
    // Clean up all subscriptions at once:
    this.onDestroy$.next();
    this.onDestroy$.complete();
  }
}
```



### How to create Angula library
SYNC version:
- http://stackoverflow.com/questions/40089316/how-to-share-service-between-two-modules-ngmodule-in-angular2

```js
/// some.module.ts
import { NgModule } from '@angular/core';

import { SomeComponent }   from './some.component';

@NgModule({
    imports: [],
    exports: [],
    declarations: [SomeComponent],
    providers: [ MyService ], // <======================= PROVIDE THE SERVICE
})
export class SomeModule { }
```

```js
/// some-other.module.ts
import { NgModule } from '@angular/core';

import { SomeModule } from 'path/to/some.module'; // <=== IMPORT THE JSMODULE

import { SomeOtherComponent }   from './some.other.component';

@NgModule({
    imports: [ SomeModule ], // <======================== IMPORT THE NG MODULE
    exports: [],
    declarations: [SomeOtherComponent],
    providers: [],
})
export class SomeOtherModule { }
```

ASYNC version (lazy loading) with forRoot pattern:
- http://blog.angular-university.io/angular2-ngmodule/
- http://www.dzurico.com/how-to-create-an-angular-library/
- http://blog.angular-university.io/how-to-create-an-angular-2-library-and-how-to-consume-it-jspm-vs-webpack/



### How create custom Input/Output property, ngModel
- http://stackoverflow.com/questions/35327929/angular-2-ngmodel-in-child-component-updates-parent-component-property
```js
import {Component, EventEmitter, Input, Output} from 'angular2/core'

@Component({
    selector: 'child',
    template: `
        <p>Child sharedVar: {{sharedVar}}</p>
        <input [ngModel]="sharedVar" (ngModelChange)="change($event)">
    `
})
export class ChildComponent {
    @Input() sharedVar: string;
    @Output() sharedVarChange = new EventEmitter();
    change(newValue) {
      console.log('newvalue', newValue)
      this.sharedVar = newValue;
      this.sharedVarChange.emit(newValue);
    }
}

@Component({
    selector: 'parent',
    template: `
        <div>Parent sharedVarParent: {{sharedVarParent}}</div>
        <child [(sharedVar)]="sharedVarParent"></child>
    `,
    directives: [ChildComponent]
})
export class ParentComponent {
    sharedVarParent ='hello';
    constructor() { console.clear(); }
}
```



### How expose angular 2 methods / call from outside of angular2 ?
- http://stackoverflow.com/questions/35276291/how-do-expose-angular-2-methods-publicly/35276652?noredirect=1#comment58266532_35276652
- http://stackoverflow.com/questions/35296704/angular2-how-to-call-component-function-from-outside-the-app



### Pre-Bootstraping template
- vsetko v angular directive bude vymazane po na-bootstrap-ovani angular appky
- logo/image ako byte array https://www.base64-image.de/ - vynecha dodatocny request pre obrazok v boostrap template-e

#### Blinking WR logo Example

```html
<tbm-root>

  <!--Pre-Bootstrap template START-->
  <!--Will be removed once Angular code bootstraped and content of tmb-root directive is resolved-->
  <style>

    body, html {
      margin: 0;
      padding: 0;
    }

    .tbm-ng-pre-bootstrap-loading-wrapper {
      position: absolute;
      width: 100%;
      height: 100%;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
    }

    .tbm-ng-pre-bootstrap-loading-wrapper_loading-image {
      animation: fading 3s infinite linear;
      height: 150px;
      width: 150px;
    }

    @keyframes fading {
      0% {
        opacity: 1;
      }
      50% {
        opacity: 0.4;
      }
      100% {
        opacity: 1;
      }
    }
  </style>
  <div class="tbm-ng-pre-bootstrap-loading-wrapper">
    <img class="tbm-ng-pre-bootstrap-loading-wrapper_loading-image"
         src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAJYAAACWCAMAAAAL34HQAAAC91BMVEUAAABZq8UAkrherscXUoMAkrgXUoNZq8UWVIVvscgonr8Bk7hwssgCk7k0osEBjLRgwtAJd6JewM9vsskAkrhwschwscgAkrg0osE/qsVwscg0osEWUoMGlLk0osEBkbgVmbxwschwscgXUoMAhK9stsoJlLldwM9fwtAXU4QBkbczocEWVoYHfacwoL8toMAUWYlNtMoKdaEdm70SX45DrsdwschgwtAAYZczosEAYZcAYZdUuswAkrhQt8tgwtA5psNmvM07qMQjnL5wschwschwschwscg0osE0osEAYZcLlrpwscgNbpsAaZxwscgAkrhwschwschwschwschutMlgwtAAkbgIlbozosElpcJIs8kAYZdwschwscgAkrgAkrhttMoOlboLlbomnb4ElLkXV4dvsslvsskEhq4AhrARl7s2pMIPlrpwscgAZZo8pMIZmbwLcp4AcKEMbZlAssgQZZMIXJBwscgAkrhwscgAkrgAkrgFYJVttcoAYZdnr8cLmLthrcZZvc4Ai7MFha5stcpwscgXnb4OlrofaZUAY5hbwM8xocEAfqpWvc4OlrojdZ80osFiwM8AdqVLuMsNbpoQZpQAkrg0osE0osEAkrhwscgvocFwscgAkrgAkrgAYJdwscgAYZdqt8sAj7ZgwtAbX41gwtARVohrt8sAgq1iwc9pucwOWIswqsQFYJQAkrgAkrgBZZkAkLdwscguoMBgwtA0osFwschwschwscgAkrhgwtA0osFgwtAAkrhlvM4dmr1avs4UVIZWu81gwtBcwM9gwtBNqMMGlLkeocACYJYxmrsAeagvlrgtjrIog6kAkrhgwtA0osFHq8U0osEAYZcAkrhJp8M0osEAYZcAYZc0osEAkrg0osEAkrg0osFaq8VhwdAAkrgAkrg0osFgwtBNucsGgqpPqMMegahhwdBgwtAAYZcTmLtgwtBwschgwtA0osFgwtAAkrgAkrgXUoNwscg0osFgwtAAYZcAkLYQQi7YAAAA9nRSTlMAynACPzAw5v7+z/35+fn33M0cCOvN7vTpykb7+fTw59acMyAPDQb77+3j4t7c2dPT0tLPzMpq+vjt6uXh2tnT0c/OzsFXIBT18+/m39TRvLGoi4VSJBTv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">
  </div>
  <!--Pre-Bootstrap template END-->

</tbm-root>
```

#### Spinner Example

```html
<app>
       <style type="text/css">
           * {
               outline: 0!important
           }

           .app-loader {
               display: table;
               height: 100vh;
               width: 100%
           }

           .app-loader .loader-pusher {
               display: table-cell;
               text-align: center;
               vertical-align: middle
           }

           .app-loader .loader-pusher .spinner {
               background: url('/assets/logo.svg') no-repeat 50% 50% #f2f2f2;
               background-size: 80%;
               border-radius: 50%;
               display: inline-block;
               height: 80px;
               position: relative;
               width: 80px;
               line-height: 80px;
               text-align: center;
               color: #fff;
           }

           .app-loader .loader-pusher .spinner .spinner-tail {
               -webkit-animation: spinner .86s linear infinite;
               animation: spinner .86s linear infinite;
               background: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMzgiIGhlaWdodD0iMzgiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PGRlZnM+PGxpbmVhckdyYWRpZW50IHgxPSI4LjA0MiUiIHkxPSIwJSIgeDI9IjY1LjY4MiUiIHkyPSIyMy44NjUlIiBpZD0iYSI+PHN0b3Agc3RvcC1jb2xvcj0iI2JlYmViZSIgc3RvcC1vcGFjaXR5PSIwIiBvZmZzZXQ9IjAlIi8+PHN0b3Agc3RvcC1jb2xvcj0iI2JlYmViZSIgc3RvcC1vcGFjaXR5PSIuNjMxIiBvZmZzZXQ9IjYzLjE0NiUiLz48c3RvcCBzdG9wLWNvbG9yPSIjYmViZWJlIiBvZmZzZXQ9IjEwMCUiLz48L2xpbmVhckdyYWRpZW50PjwvZGVmcz48ZyB0cmFuc2Zvcm09InRyYW5zbGF0ZSgxIDEpIiBmaWxsPSJub25lIj48cGF0aCBkPSJNMzYgMThjMC05Ljk0LTguMDYtMTgtMTgtMTgiIHN0cm9rZT0idXJsKCNhKSIgc3Ryb2tlLXdpZHRoPSIyIi8+PGNpcmNsZSBmaWxsPSIjYmViZWJlIiBjeD0iMzYiIGN5PSIxOCIgcj0iMSIvPjwvZz48L3N2Zz4=) 0 0/cover no-repeat;
               bottom: -10px;
               left: -10px;
               position: absolute;
               right: -10px;
               top: -10px;
           }

           @-webkit-keyframes spinner {
               0% {
                   -webkit-transform: rotate(0);
                   transform: rotate(0)
               }
               100% {
                   -webkit-transform: rotate(360deg);
                   transform: rotate(360deg)
               }
           }

           @keyframes spinner {
               0% {
                   -webkit-transform: rotate(0);
                   transform: rotate(0)
               }
               100% {
                   -webkit-transform: rotate(360deg);
                   transform: rotate(360deg)
               }
           }
       </style>
       <div class="app-loader">
           <div class="loader-pusher">
               <div class="spinner">
                   <div class="spinner-tail"></div>
               </div>
           </div>
       </div>
   </app>
```



### Real debounce on input element
- http://stackoverflow.com/questions/32051273/angular2-and-debounce
- NeMoX filterbar



### Ako pouzit jQuery v komponente
```js
declare var jQuery: any;
//...
@ViewChild('checkbox') checkbox: ElementRef;
//...
ngAfterViewInit() {
  jQuery(this.checkbox.nativeElement).checkbox(this.isPredefined ? 'set checked' : 'set unchecked');
}
```



### How load config from server before bootstraping
- https://github.com/angular/angular/issues/23279
- https://hackernoon.com/hook-into-angular-initialization-process-add41a6b7e
- http://stackoverflow.com/questions/39033835/angularjs2-preload-server-configuration-before-the-application-starts/39033958

- code:
```js
@Injectable()
export class ConfigService {
  static DASHBOARD_CONFIG_ADDRESS = 'dashboard-configuration.json';

  private _config: Config;

  constructor(private http: Http) { }

  loadConfig(): Promise<Config> {
    const serviceThat = this;

    return new Promise((resolve) => {
      this.http.get(ConfigService.DASHBOARD_CONFIG_ADDRESS).map(res => res.json()).toPromise()
        .then((configResponse) => {
          serviceThat._config = configResponse;
          console.info('[ConfigService] dashboard configuration: ' + JSON.stringify(configResponse));
          resolve();
        })
        .catch(() => {
          console.error('[ConfigService] could not get dashboard configuration !');
          resolve();
        });
    });
  }

  getConfig(): Config {
    return this._config;
  }

}

//module js file
export function loadConfigFn(configService: ConfigService) {
  return () => configService.loadConfig();
}

@NgModule({
providers: [
    FetchingService,
    ConfigService,
    {
      provide: APP_INITIALIZER,
      useFactory: loadConfigFn,
      deps: [ConfigService],
      multi: true,
    }
  ]
})

```



### [angular-cli] Webpack Warning that export 'INTERFACE' was not found
- workaround https://github.com/angular/angular-cli/issues/2034
  - interfaces folder with index.ts that export all interfaces manually
  - split each interface into separate file



### [angular-cli] Integration of Font-Awesome (External css lib with url to fonts)
- pridal som v `angular-cli.json` style hodnotu, cesta na `node_modules` min css, URLs vo vnutry css-iek sa po kompilacii angular-cli poriesili a dotahali fonty ako assets



### [Chrome] debug Devpanel/Devtools ako chrome extension ?
- http://stackoverflow.com/questions/27661243/how-to-debug-chrome-devtools-panel-extension



### [Chrome] Angular app ako chrome extension (devpanel) nefunguje bootstraping ?
- `
Unhandled Promise rejection: Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source of script in the following Content Security Policy directive: "script-src 'self' blob: filesystem: chrome-extension-resource:".
; Zone: <root> ; Task: Promise.then ; Value: EvalError: Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source of script in the following Content Security Policy directive: "script-src 'self' blob: filesystem: chrome-extension-resource:".
`
- https://www.sitepoint.com/chrome-extension-angular-2/
- `"content_security_policy": "script-src 'self' 'unsafe-eval'; object-src 'self'"` vyriesilo moj problem



### [ImmutableJS] object as Observable value trick
- https://github.com/ngrx/store/issues/233
- `_store.select('counter').map((immuObj: List<any>) => immuObj.toJS())`



### [ImmutableJS] neviem pracovat s Typescript getters() setters() !!
```js
class foo {
    private _bar:boolean = false;
    get bar():boolean {
        return this._bar;
    }
    set bar(theBar:boolean) {
        this._bar = theBar;
    }
}
```



### [RxJs] How to import RxJs
```js
import { Observable } from 'rxjs/Observable';
import { Subject } from 'rxjs/Subject';

import 'rxjs/add/operator/map';
```



### [TypeScript] Declare global variable if it is not recognized
`declare var navigator;`



### [TypeScript] Ako explicitne setnem hodnotu na `window` objekt ?
- http://stackoverflow.com/questions/12709074/how-do-you-explicitly-set-a-new-property-on-window-in-typescript
- `(<any>window).WHATEVER=0`



### [Typescript] nepozna `chrome` window objekt
- `declare var chrome: any;` at top and ignore it
