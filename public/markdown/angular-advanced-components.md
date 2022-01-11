slidenumbers: true
slidecount: true
class: slide-separator

<img style="float: left;" src="../images/logo-angular.png" width="60px">
### Angular
# Advanced Components

---
class: slide-separator

<img src="../images/avatar-michele.jpeg" style="width: 150px; border-radius: 50%; float: right">
## Michele Stieven

- #### Angular **GDE** <img src="../images/logo-gde.svg" style="width: 2rem; vertical-align: top">
- #### Autore di **Accademia.dev**

---
## Roadmap

1. Content Projection
2. Direttive Strutturali
3. *ngTemplateOutlet
4. Dynamic Components
5. Lazy Loading
6. [Preview] Standalone Components


---
class: slide-separator

## Content Projection

---
## Content Projection

Puoi inserire (o _proiettare_) del contenuto HTML ad un componente figlio con il tag `<ng-content>`:

```ts
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <ng-content></ng-content>
    </div>
  `
})
export class CardComponent {}
```

```html
<app-card>Ciao!</app-card>
```

Renderizza:

```html
<div class="card">Ciao!</div>
```

---
## Content Projection

Puoi suddividere il contenuto in vari slot:

```ts
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <ng-content></ng-content>
      <hr>
      <ng-content select="[foo]"></ng-content>
    </div>
  `
})
export class CardComponent {}
```

- `select` supporta i selettori CSS
- Tutto ciò che non è preso di mira da uno slot finisce nel contenuto generico

---
## Content Projection

Puoi suddividere il contenuto in vari slot:

```html
<app-card>
  <p foo>Io sono lo slot "foo"</p>
  Io sono il contenuto generico!
</app-card>
```

Renderizza:

```html
<div class="card">
  Io sono il contenuto generico!
  <hr>
  <p foo>Io sono lo slot "foo"</p>
</div>
```

---
class: slide-separator

## Direttive Strutturali

---
## ng-template

Un `<ng-template>` contiene codice HTML che non è subito presente nel DOM.

Questo codice HTML non renderizza nulla:

```html
<ng-template>
  <span>Ciao</span>
</ng-template>
```


---
## Direttive Strutturali

Una Direttiva Strutturale è una direttiva utilizzata con il prefisso `*`, ad esempio `*myDirective`.

### A cosa serve?

Se una Direttiva d'Attributo semplicemente si aggancia ad un elemento o componente, una Direttiva Strutturale può rimuovere o aggiungere l'elemento al DOM.  
- Esempi: `*ngIf`, `*ngFor`, `*ngSwitch`

### Perché l'asterisco?

L'asterisco è una convenzione di Angular, per evitare di scrivere un `<ng-template>` manualmente ed altre scomodità.

---
## Direttive Strutturali - L'asterisco

Quando applichi una direttiva utilizzando l'asterisco:

```html
<p *myDirective class="example">
  Hello World
</p>
```

In automatico Angular interpreta il template in questo modo:

```html
<ng-template myDirective>
  <p class="example">Hello World</p>
</ng-template>
```

Per questo motivo, un elemento sul quale è applicata una Direttiva Strutturale non è inizialmente nel DOM.

Per lo stesso motivo, non è possibile usare più direttive strutturali sullo stesso elemento: Angular non saprebbe dare una priorità.

---
## Direttive Strutturali - L'asterisco

L'asterisco semplifica anche la scrittura di proprietà di **contesto**:

```html
<div *ngFor="let person of people; let i=index; let odd=odd; trackBy: trackById">
  {{ i }} {{ person.name }}
</div>

<ng-template
  ngFor
  [ngForOf]="people"
  [ngForTrackBy]="trackById"
  let-person
  let-i="index"
  let-odd="odd"
>
  <div>{{ i }} {{ person.name }}</div>
</ng-template>
```

- `person`, `i` e `odd` sono proprietà di contesto: ci vengono date dalla direttiva. Ogni contesto ha una proprietà implicita e per questa non serve indicare il nome (nel nostro caso è `person`).

---
## Direttive Strutturali - `*ngIf`

Direttiva `*ngIf` a fini d'esempio:

```ts
@Directive({ selector: '[ngIf]' })
export class NgIfDirective {
  private visible = false;

  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef) { }

  @Input() set ngIf(condition: boolean) {
    if (condition && !this.visible) {
*     this.viewContainer.createEmbeddedView(this.templateRef);
      this.visible = true;
    } else if (!condition && this.visible) {
*     this.viewContainer.clear();
      this.visible = false;
    }
  }
}
```

---
## ViewContainerRef

Puoi manipolare le view (`ViewRef`) con vari metodi:

```ts
class ViewContainerRef {
  // Recupera la ViewRef
  get(index: number): ViewRef | null
  // Rimuovi tutte le ViewRef
  clear(): void
  // Inserisci una ViewRef
  insert(viewRef: ViewRef, index?: number): ViewRef
  // Sposta una ViewRef
  move(viewRef: ViewRef, currentIndex: number): ViewRef
  // Ottieni l'indice di una ViewRef
  indexOf(viewRef: ViewRef): number
  // Rimuovi una ViewRef
  remove(index?: number): void
  // Rimuovi una ViewRef senza distruggerla
  detach(index?: number): ViewRef | null
  // Crea una ViewRef
  createEmbeddedView<C>(templateRef: TemplateRef<C>, context?: C, index?: number): EmbeddedViewRef<C>
}
```

---
class: slide-separator

## *ngTemplateOutlet

---
## ng-template

Un `<ng-template>` contiene codice HTML che non è subito presente nel DOM.

Questo codice HTML non renderizza nulla:

```html
<ng-template>
  <span>Ciao</span>
</ng-template>
```

---
## ngTemplateOutlet

Puoi rendere visibile il contenuto di un `<ng-template>` flaggandolo con una variabile di template (ottenendo un `TemplateRef`) e passandola ad un contenitore con la direttiva `*ngTemplateOutlet`:

```html
<ng-template #saluto>
  <span>Ciao</span>
</ng-template>

<div *ngTemplateOutlet="saluto"></div>
```

Risultato:

```html
<div>
  <span>Ciao</span>
</div>
```

---
## Template Context

Puoi passare un contesto ad ogni `<ng-template>`:

```ts
myContext = { name: 'Mario' };
```

```html
<ng-template #saluto let-nome="name">
  <span>Ciao {{ nome }}</span>
</ng-template>

<div
  *ngTemplateOutlet="saluto"
  [ngTemplateOutletContext]="myContext"
></div>
```

--

```html
<!-- shorthand -->
<div
  *ngTemplateOutlet="saluto; context: myContext"
></div>
```

---
## Template Context

Un contesto permette una proprietà implicita:

```ts
myContext = { name: 'Mario', $implicit: 'Rossi' };
```

Puoi recuperarla con la sintassi `let-nome_a_piacimento`:

```html
<ng-template #saluto let-nome="name" let-foo>
  <span>Ciao {{ nome }} {{ foo }}</span>
</ng-template>
```

Renderizza:

```html
<span>Ciao Mario Rossi</span>
```

---
## Passare un Template ad un componente (1)

Puoi passare un `<ng-template>` ad un componente, che lo utilizzerà (magari N volte) al suo interno. In altri framework potresti implementare questa feature con delle _Render Props_.

```html
<ng-template #cell>
  <label>Hello!</label>
</ng-template>

<app-grid [data]="data" [cellTemplate]="cell"></app-grid>
```
--
```ts
@Component({
  template: `
    <ng-container *ngTemplateOutlet="cellTemplate"></ng-container>
  `
})
export class GridComponent<T> {
  @Input() data: T[] = [];
  @Input() cellTemplate!: TemplateRef<any>;
}
```

---
## Passare un Template ad un componente (1)

```html
<ng-template #cell let-item>
  <label>{{ item.name }}</label>
</ng-template>
```

```ts
@Component({
  template: `
    <li *ngFor="let row of data">
      <ng-container
        *ngTemplateOutlet="cellTemplate"
        [ngTemplateOutletContext]="{ $implicit: row }"
      ></ng-container>
    </li>
  `
})
export class GridComponent<T> {
  @Input() data: T[] = [];
  @Input() cellTemplate!: TemplateRef<any>;
}
```

---
## Passare un Template ad un componente (2)

Al posto di passare un template via Input, puoi passarlo come `<ng-content>`:

```html
<app-grid [data]="data">
  <div *gridCell="let cell">
    <span>{{ cell }}</span>
  </div>
</app-grid>
```

- `gridCell` è una Direttiva vuota, che funge solo da query

```ts
@Directive({
  selector: '[gridCell]'
})
export class GridCellDirective {}
```

---
## Passare un Template ad un componente (2)

Al posto di passare un template via Input, puoi passarlo come `<ng-content>`:

```ts
export class GridComponent<T> {
  // Array di elementi
  @Input() data: T[] = [];
  // Template del singolo elemento
  @ContentChild(GridCellDirective, {read: TemplateRef}) gridCellTemplate;
}
```

```html
<li *ngFor="let row of data">
  <ng-container
    *ngTemplateOutlet="gridCellTemplate"
    [ngTemplateOutletContext]="{ $implicit: row }"
  >
  </ng-container>
</li>
```

---
class: slide-separator

## Dynamic Components

---
## Dynamic Components

Istanziare un componente a runtime via TypeScript, senza alcun template

```html
<app-ad></app-ad>
```
--
#### _Angular < 13_

```ts
@Component({ ... })
export class AdComponent {
  constructor(
    private viewContainer: ViewContainerRef,
    private resolver: ComponentFactoryResolver
  ) {
    const componentFactory = this.resolver.resolveComponentFactory(FirstAdComponent);
    this.viewContainer.createComponent<FirstAdComponent>(componentFactory);
  }
}
```

---
## Dynamic Components

Istanziare un componente a runtime via TypeScript, senza alcun template

```html
<app-ad></app-ad>
```

#### _Angular >= 13_

```ts
@Component({ ... })
export class AdComponent {
  constructor(private viewContainer: ViewContainerRef) {
    
    this.viewContainer.createComponent(FirstAdComponent);
  }
}
```

---
## Dynamic Components

Un componente dinamico può essere istanziato da un `Component` oppure da una `Directive`

```html
<div #ad></div>
```

#### _Angular >= 13_

```ts
@Directive({ ... })
export class AdDirective {
  constructor(private viewContainer: ViewContainerRef) {

    this.viewContainer.createComponent(FirstAdComponent);
  }
}
```
---
## Dynamic Components

Puoi rimuovere il componente dinamico con il metodo `clear()`

```ts
@Component({ ... })
export class AdComponent {
  constructor(private viewContainer: ViewContainerRef) {}

  load() {
    this.viewContainer.createComponent(FirstAdComponent);
  }
  
  unload() {
    this.viewContainer.clear();
  }
}
```

---
## AdComponent
Esempio

```ts
export class AdComponent implements OnChanges, OnDestroy {
  
  @Input() ads: Type<any>[] = [];
  private sub: Subcription | null = null;
  
  constructor(private viewContainer: ViewContainerRef) {}
  
  ngOnChanges() {
    this.sub?.unsubscribe();
    
    this.sub = timer(0, 5000).pipe(take(this.ads.length), repeat()).subscribe(i => {
      this.viewContainer.clear();
      this.viewContainer.createComponent(this.ads[i]);
    })
  }
  
  ngOnDestroy() {
    this.sub?.unsubscribe();
  }
}
```

---
## Dynamic Components

Il metodo createComponent restituisce una `ComponentRef`

```ts
class ComponentRef<C> {
  // Elemento HTML che contiene il nuovo componente
  location: ElementRef;
  // Injector del nuovo componente
  injector: Injector;
  // Istanza del nuovo componente
  instance: C;
  // Host view del nuovo componente
  hostView: ViewRef;
  // Change Detector del nuovo componente
  changeDetectorRef: ChangeDetectorRef;
  // Tipo del componente
  componentType: Type<any>;
  // Metodo che distrugge il componente
  destroy(): void;
  // Metodo che aggiunge logica al destroy del componente
  onDestroy(callback: Function): void;
}
```

---
## Dynamic Components

Puoi interagire con il componente creato tramite la proprietà `instance`

```ts
const componentRef = this.viewContainer.createComponent(FirstAdComponent);
const instance = componentRef.instance;
```

Qualche esempio:

```ts
// Es: Passare un @Input o settare una proprietà pubblica
instance.item = ...;

// Es: Chiamare un metodo pubblico
instance.doSomething();

// Es: Ascoltare un @Output
instance.titleChange.subscribe(title => ...)
```

---
## Istanziare in un punto preciso

```ts
@Component({
  selector: 'app-ad',
  template: `
    Ads:
    <div #ads></div>
  `
})
export class AdComponent {
  
  @ViewChild('ads', { static: true, read: ViewContainerRef }) ads!: ViewContainerRef;
}
```
--
Si utilizza `ViewContainerRef` come sempre:
```ts
// Creazione
this.ads.createComponent(...);

// Rimozione
this.ads.clear();
```

---
## ngComponentOutlet

Un'alternativa **dichiarativa** all'utilizzo di `ViewContainerRef`.

Ideale per casi semplici in cui non serve interagire con il componente dinamico (Input, Output, metodi...).

```ts
export class AdComponent {

  activeAd: Type<any> | null = FirstAdComponent;
}
```

```html
<div *ngComponentOutlet="activeAd"></div>
```


---
## ngComponentOutlet

La direttiva fornisce questi `Input`:

- `ngComponentOutletInjector`

Un **Injector** custom. Di default è quello del padre.
- `ngComponentOutletContent`

Dei nodi HTML da proiettare all'interno del componente (se c'è un `ng-content`).
- `ngComponentOutletNgModuleFactory`

Una factory per caricare un modulo dinamicamente, prima di istanziare il componente. `NgModuleFactory` è **deprecata** nelle ultime versioni di Angular.

---
## ngComponentOutlet
Esempio completo:

```html
<div *ngComponentOutlet="activeAd; content: content; injector: customInjector"></div>
```

```ts
export class AdComponent {
  // Il componente attivo
  activeAd = FirstAdComponent;
  // Il contenuto da proiettare
  content = [[document.createTextNode('Hello')], [document.createTextNode('World')]];
  // Un Injector custom
  customInjector = Injector.create({
    providers: [{ provide: BANNER_TITLE, useValue: 'Pentole in acciaio inox' }],
    parent: this.injector
  });

  constructor(private injector: Injector) {}
}
```

---
## Ricevere dati dall'Injector

Come Best Practice, in assenza di classi si fa il provide del dato con un `InjectionToken`:

```ts
import { InjectionToken } from '@angular/core';

export const BANNER_TITLE = new InjectionToken<string>(`Il titolo dell'ad`);
```

E si inietta in questo modo:

```ts
import { Inject } from '@angular/core';

export class FirstAdComponent {

  constructor(@Inject(BANNER_TITLE) public title: string) {}
}
```

---
class: slide-separator

## Lazy Loading Components

---
## Lazy Loading Components

```ts
this.viewContainer.createComponent(MyComponent);
```

### Domanda:
Cosa succede se quel componente è dichiarato in un altro modulo? O in nessun modulo? O se ha delle dipendenze?

---
## Lazy Loading Components

Un componente non ha **per forza** bisogno di un modulo! Questo è un componente perfettamente valido:

```ts
@Component({
  selector: 'app-hello',
  template: `Hello!`
})
export class HelloComponent {}
```

Per usarlo in un template deve essere dichiarato (`declarations`), ma non è così per i componenti caricati in modo Lazy.

---
## Lazy Loading Components
Esempio:

```html
<div *ngComponentOutlet="hello"></div>
```

```ts
export class AppComponent {
  
  hello: Type<HelloComponent> | null = null;
  
  async load() {
    const module = await import('./hello.component');
    this.hello = module.HelloComponent;
  }
}
```

<small>Chiaramente il discorso non cambia utilizzando `ViewContainerRef.createComponent()`</small>

---
## Lazy Loading Components

Cambiamo un po' il componente:

```ts
@Component({
  selector: 'app-hello',
  template: `
    <div *ngIf="true">Hello!</div>
  `
})
export class HelloComponent {}
```

--
```html
Can't bind to 'ngIf' since it isn't a known property of 'div'.
```

Il componente ora ha bisogno di `CommonModule` per funzionare (contiene `ngIf`).

---
## Lazy Loading Components

Approccio **SCAM** (*Single Component Angular Modules*):

```ts
// Componente
@Component({
  selector: 'app-hello',
  template: `
    <div *ngIf="true">Hello!</div>
  `
})
export class HelloComponent {}

// Modulo
@NgModule({
  imports: [CommonModule],
  declarations: [HelloComponent]
})
export class HelloModule {}
```

Funziona? **Sì, funziona!** ...Ma come?

---
### Lazy Loading Components

```ts
export class AppComponent {

  hello: Type<HelloComponent> | null = null;

  async load() {
    const module = await import('./hello.component');
    this.hello = module.HelloComponent;
  }
}
```

Non abbiamo nemmeno importato `HelloModule`... Come può funzionare?

Il compilatore di Angular è abbastanza intelligente da capire, in fase di analisi, che quel componente è parte di quel modulo, e automaticamente include le dipendenze **all'interno del componente** compilato. Quindi non dobbiamo fare nulla: siamo a cavallo!

---
class: slide-separator

## RFC: Standalone Components, Directives and Pipes

---
## Standalone Components

Problema:

```ts
this.viewContainerRef.createComponent(HelloComponent);
```
--
Siamo sicuri funzioni? E se...

```ts
@Component({...})
export class HelloComponent {
  constructor(service: HelloService) {}
}
```
```ts
@NgModule({
  declarations: [HelloComponent],
  imports: [...],
  providers: [HelloService],
})
export class HelloModule {}
```

---
## Standalone Components

Componenti indipendenti, riutilizzabili senza la necessità di importare alcun `NgModule`.

```ts
@Component({
  selector: 'app-hello',
  template: `Hello!`,
* standalone: true,  
})
export class HelloComponent {}
```

Stessa sintassi per **Direttive** e **Pipe**!

---
## Standalone Components

Pensate come se fossero l'unione di un componente con un `NgModule` nascosto.

Diventa possibile importare moduli e componenti standalone direttamente nel componente:

```ts
@Component({
  selector: 'app-hello',
  template: `Hello!`,
  standalone: true,
  imports: [CommonModule, FormsModule, WorldComponent],
})
export class HelloComponent {}
```

---
## Standalone Components

Grazie agli Standalone Components può diventare più facile insegnare Angular, introducendo il concetto di `NgModule` in un secondo momento.

#### Bootstrap

```ts
import { bootstrapComponent } from '@angular/core';

bootstrapComponent(HelloComponent);
```

#### Router

```ts
RouterModule.forRoot([
  {
    path: '/hello', 
    loadComponent: () => import('./hello.component').then(m => m.HelloComponent)
  }
]);
```

---
class: slide-separator
## Grazie ❤️