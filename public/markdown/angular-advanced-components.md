slidenumbers: true
slidecount: true
class: slide-separator

<img style="float: left;" src="../images/logo-angular.png" width="60px">
### Angular
# Advanced Components

---
## Roadmap

1. Direttive Strutturali
2. Dynamic Components
3. Content Projection
4. *ngTemplateOutlet

---
class: slide-separator

## Direttive Strutturali

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
class: slide-separator

## Dynamic Components

---
## Dynamic Components

Istanziare un componente a runtime via TypeScript, senza alcun template

```html
<app-ad></app-ad>
```

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

#### _Angular < 13_

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
## ng-template

Puoi rendere visibile il contenuto di un `<ng-template>` flaggandolo con una variabile di template (ottenendo un `TemplateRef`) e passandola ad un contenitore con la direttiva `*ngTemplateOutlet`:

```html
<ng-template #saluto>
  <span>Ciao</span>
</ng-template>

<ng-container *ngTemplateOutlet="saluto"></ng-container>
```

Risultato:

```html
<ng-container>
  <span>Ciao</span>
</ng-container>
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

<ng-container
  *ngTemplateOutlet="saluto"
  [ngTemplateOutletContext]="myContext"
></ng-container>
```

--

```html
<!-- shorthand -->
<ng-container
  *ngTemplateOutlet="saluto; context: myContext"
></ng-container>
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
  <label>{{ item.name }}</label>
</ng-template>

<app-grid [data]="data" [cellTemplate]="cell"></app-grid>
```

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
