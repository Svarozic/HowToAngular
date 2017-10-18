# HowToAngular2
spoznamkovane problemy pri tvorbe angular2 appiek pomocou angular-cli alebo inych seedov

#### [Angular] Window resize event

```
import 'rxjs/add/operator/debounceTime';
import 'reflect-metadata';

// in component
@HostListener('window:resize', ['$event'])
  onWindowResize(event) {
    // this.windowResizeEvent$.next(event);
  }
```

#### [Angular] NgRouter and dynamic base location trick
- https://angular.io/docs/ts/latest/guide/router.html#!#base-href

```html
<script>document.write('<base href="' + document.location + '" />');</script>
```

#### [Angular] Elegant way to unsubscribe Observables
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
  }
}
```

#### [Angular] How to create Angula library
SYNC version:
- http://stackoverflow.com/questions/40089316/how-to-share-service-between-two-modules-ngmodule-in-angular2

```typescript
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

```typescript
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


#### [Angular] How create custom Input/Output property, ngModel
- http://stackoverflow.com/questions/35327929/angular-2-ngmodel-in-child-component-updates-parent-component-property
```typescript
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


#### [Angular] How trigger change detection when 'ChangeDetectionStrategy.OnPush'
- `this._changeDetectorRef.detectChanges();`
- observable or ngZone.run() trigger tick and change automatically 

#### [Angular] How expose angular 2 methods / call from outside of angular2 ?
- http://stackoverflow.com/questions/35276291/how-do-expose-angular-2-methods-publicly/35276652?noredirect=1#comment58266532_35276652
- http://stackoverflow.com/questions/35296704/angular2-how-to-call-component-function-from-outside-the-app


#### [Angular] Pre-Bootstraping template
- vsetko v angular directive bude vymazane po na-bootstrap-ovani angular appky
- logo/image ako byte array https://www.base64-image.de/ - vynecha dodatocny request pre obrazok v boostrap template-e


#### [Angular] Real debounce on input element
- http://stackoverflow.com/questions/32051273/angular2-and-debounce
- NeMoX filterbar


#### [Angular] Ako pouzit jQuery v komponente
```js
declare var jQuery: any;
//...
@ViewChild('checkbox') checkbox: ElementRef;
//...
ngAfterViewInit() {
  jQuery(this.checkbox.nativeElement).checkbox(this.isPredefined ? 'set checked' : 'set unchecked');
}
```


#### [Angular] How load config from server before bootstraping
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


#### [angular-cli] Webpack Warning that export 'INTERFACE' was not found
- workaround https://github.com/angular/angular-cli/issues/2034 
  - interfaces folder with index.ts that export all interfaces manually
  - split each interface into separate file

#### [angular-cli] Integration of Font-Awesome (External css lib with url to fonts)
- pridal som v `angular-cli.json` style hodnotu, cesta na `node_modules` min css, URLs vo vnutry css-iek sa po kompilacii angular-cli poriesili a dotahali fonty ako assets


#### [Chrome] debug Devpanel/Devtools ako chrome extension ?
- http://stackoverflow.com/questions/27661243/how-to-debug-chrome-devtools-panel-extension


#### [Chrome] Angular app ako chrome extension (devpanel) nefunguje bootstraping ?
- `
Unhandled Promise rejection: Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source of script in the following Content Security Policy directive: "script-src 'self' blob: filesystem: chrome-extension-resource:".
; Zone: <root> ; Task: Promise.then ; Value: EvalError: Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source of script in the following Content Security Policy directive: "script-src 'self' blob: filesystem: chrome-extension-resource:".
`
- https://www.sitepoint.com/chrome-extension-angular-2/
- `"content_security_policy": "script-src 'self' 'unsafe-eval'; object-src 'self'"` vyriesilo moj problem


#### [ImmutableJS] object as Observable value trick
- https://github.com/ngrx/store/issues/233
- `_store.select('counter').map((immuObj: List<any>) => immuObj.toJS())`


#### [ImmutableJS] neviem pracovat s Typescript getters() setters() !!
```typescript
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


#### [TypeScript]  Property 'map' does not exist on type 'Observable<Response>'
- check if I have import on Angular Response type and apropriate rxjs operators
```
import { Response } from '@angular/http';

import 'rxjs/add/operator/map';
import 'rxjs/add/operator/toPromise';
```


#### [TypeScript] Ako explicitne setnem hodnotu na `window` objekt ?
- http://stackoverflow.com/questions/12709074/how-do-you-explicitly-set-a-new-property-on-window-in-typescript
- `(<any>window).WHATEVER=0`


#### [Typescript] nepozna `chrome` window objekt
- `declare var chrome: any;` at top and ignore it
