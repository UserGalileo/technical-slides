slidenumbers: true
slidecount: true
class: slide-separator

<img style="width: 50px; margin-right: 1em; float: left;
  filter: drop-shadow(1px 1px 0 white)
        drop-shadow(-1px 1px 0 white)
        drop-shadow(1px -1px 0 white)
          " src="../images/logo-lit.png">
<h3>Lit</h3>
# Using Lit to create SPAs

---
class: slide-separator

<img src="../images/avatar-michele.jpeg" style="width: 200px; border-radius: 50%; float: right">
## Michele Stieven
Freelance Front-End Consultant
<ul style="padding-top: .5em">
<li><h3 style="">Angular <b>GDE</b> <img src="../images/logo-gde.svg" style="width: 2rem; vertical-align: top"></h3></li>
<li><h3 style="">Founder of <b>Accademia.dev</b></h3></li>
</ul>

---
## Roadmap

1. Web Components
2. Lit
3. Reactive Controllers
4. Directives
5. Lit Labs

---
class: slide-separator

## Web Components

---
## The broken promise?

- Browser Support
- Framework Support
- UI Components (eg. Ionic)
- Adoption
  - Google (YouTube)
  - Microsoft (FastElement)
  - SpaceX (Dragon)
  - Salesforce
  - Adobe (Spectrum, Photoshop, Illustrator)
  - IBM (Carbon Web Components)
  - Apple
  - GitHub
  - Amazon
  - Reddit
  - ...

---
## What's a Web Component?

```js
// HTML Templates
const template = document.createElement('template');
template.innerHTML = `<h1>Hello Verona!</h1>`;

class HelloComponent extends HTMLElement {

  constructor() {
    super();
    // Shadow DOM
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.appendChild(template.content.cloneNode(true));
  }
}

// Custom Elements
customElements.define('my-hello', HelloComponent);
```

---
## Lifecycle

<h3 style="padding-bottom: 5px; margin-bottom: 5px">connectedCallback</h3>
The element has been added to the DOM

<h3 style="padding-bottom: 5px; margin-bottom: 5px">disconnectedCallback</h3>
The element has been removed form the DOM

<h3 style="padding-bottom: 5px; margin-bottom: 5px">adoptedCallback</h3>
The element has moved to a new document 🤔

<h3 style="padding-bottom: 5px; margin-bottom: 5px">attributeChangedCallback</h3>
An attribute has changed

---
## Purposes

- Reusable Components
  - UI Libraries / Design Systems (Eg. Ionic, MDC Web, Shoelace...)
  - Other
- Migrations
- Enhancements (eg. WordPress)
- ...Applications? 😮

---
## The Adobe's case

**Photoshop**, on the Web
- <img style="height: 20px;" src="../images/logo-lit.png"> Spectrum Web Components
- <img style="height: 20px;" src="../images/logo-lit.png"> Web App
  - React islands by other teams
- WebAssembly
- File System Access
- Service Worker (Workbox)

<img src="../images/photoshop-web.avif" style="width: 100%;">

---
class: slide-separator

## Components

---
## The Component Model


.cols[
.half[
### <img style="height: 1.8rem;" src="../images/logo-react.png"> React
- Props
- Children
- States
- Declarative Template (JSX)
]
.half[
### <img style="height: 1.8rem;" src="../images/logo-lit.png"> Lit
- Properties, Attributes
- Slots
- States
- Declarative Template (Tagged Template Literals)
- Events
]
]

---
## LitElement

```ts
import { html, css, LitElement } from 'lit';
import { customElement, property, state } from 'lit/decorators.js';

@customElement('simple-greeting')
export class SimpleGreeting extends LitElement {

  static styles = css`p { color: blue }`;

  @property() name = 'Somebody';

  render() {
    return html`<p>Hello, ${this.name}!</p>`;
  }
}
```

```html
<simple-greeting name="World"></simple-greeting>
```

---
## LitElement

```ts
@customElement('my-counter')
export class Counter extends LitElement {

  @state() count = 0;

  render() {
    return html`
      Counter: ${this.count}
      <button @click=${() => this.count++}>+</button>
      <button @click=${this.dec}>-</button>
    `;
  }

  dec() {
    this.count--;
  }
}
```

---
### Tagged Template Literals

```ts
const city = 'Verona';
html`Hello ${city}!`;
```

```ts
function html(strings, ...holes) {
  console.log(strings); // ['Hello ', '!']
  console.log(holes); // ['Verona']
}
```

---
## Why Lit?


- Tiny (5kb gzip)
- Fast
- Compiler-less (no build system, no CLI)
- Framework agnostic
- Written in TypeScript
- No need for specific DevTools
- SSR (Work in progress)

---
class: slide-separator
## Reactive Controllers

---
## React Hooks
What are they? "Reusable Stateful Functions".

.cols[
.half[
### <img style="height: 1.8rem;" src="../images/logo-react.png"> React
- useState
- useEffect
- useRef
- useCallback
- useMemo
]
.half[
### <img style="height: 1.8rem;" src="../images/logo-lit.png"> Lit
- `@state`
- Lifecycle
- `ref`, `@query`, `@queryAll`, `@queryAsync`
- -
- -
]
]

---
## Reactive Controllers
Reusable Classes bound to an Element

```ts
interface ReactiveController {
  hostConnected(): void;
  hostUpdate(): void;
  hostUpdated(): void;
  hostDisconnected(): void;
}
```

---
## Eg. Mouse

```ts
export class MouseController implements ReactiveController {
  pos = { x: 0, y: 0 };

  private listener = ({ clientX, clientY }: MouseEvent) => {
    this.pos = { x: clientX, y: clientY };
*   this.host.requestUpdate();
  };

  constructor(private host: ReactiveControllerHost) {
    this.host.addController(this);
  }

  hostConnected() {
    window.addEventListener('mousemove', this.listener);
  }

  hostDisconnected() {
    window.removeEventListener('mousemove', this.listener);
  }
}
```

---
## Eg. Mouse

```ts
class MyElement extends LitElement {

  private mouse = new MouseController(this);

  render() {
    return html`
      <h3>The mouse is at:</h3>
      <pre>
        x: ${this.mouse.pos.x}
        y: ${this.mouse.pos.y}
      </pre>
    `;
  }
}

```
<mouse-tracker></mouse-tracker>

---
class: slide-separator
## Directives

---
## Class Directives

- Manipulate the DOM (add, remove, reorder...)
- Persist state between renders
- Update the DOM asynchronously (eg. react to Observables)

```ts
class MaxDirective extends Directive {

  render(a: number, b: number) {
    return Math.max(a, b);
  }
}

const max = directive(MaxDirective);
```
```ts
// Usage
html`<div>${max(10, 20)}</div>`;
```

---
## Async Directives

```ts
class Foo extends AsyncDirective {

  update(part, params) {
    // ...
    this.render(params);
  }
    
  render(params) {
    // ...
    return noChange;
  }

* disconnected() {
*   // ...
* }
  
* reconnected() {
*   // ...
* }
}
export const foo = directive(Foo);
```

---
## Built-in Directives

.cols[
.half[
- Styling
  - classMap
  - styleMap
- Loops and conditionals
  - when
  - choose
  - map
  - repeat
  - join
  - range
  - ifDefined
]
.half[
- Caching and change detection
  - cache
  - guard
  - live
- Rendering special values
  - templateContent
  - unsafeHTML
  - unsafeSVG
- Referencing rendered DOM
  - ref
- Asynchronous rendering
  - until
  - asyncAppend
  - asyncReplace
]
]

---
## Eg. Forms (experiment)

---
class: slide-separator
## Lit Labs

---
## @lit/localize
Provides localization/internationalization support for Lit templates.

```ts
import { msg } from '@lit/localize';

class MyGreeter extends LitElement {

  @property() who = 'World';

  render() {
    return msg(html`Hello <b>${this.who}</b>`);
  }
}
```

- Supports expressions and HTML markup
- Locale switch detection
- 1.27 KB gzip
- Optional: compile for each locale (0 KB)
- Generates XLIFF files

---
## @lit-labs/motion
Lit directives for making things move.

```ts
import { animate } from '@lit-labs/motion';

class MyElement extends LitElement {

  @state() shifted = false;

  render() {
    return html`
      <button @click=${this.move}>Move</button>
*     <div class="box ${this.shifted ? 'shifted' : ''}" ${animate()}></div>
    `;
  }

  move() {
    this.shifted = !this.shifted;
  }
}
```

---
## @lit-labs/task
A controller for Lit that renders asynchronous tasks.

```ts
class MyElement extends LitElement {
  @state() userId: number;

  private apiTask = new Task(this,
    ([userId]) => fetch(`/api/user/${userId}`).then(res => res.json()),
    () => [this.userId]
  );

  render() {
    return html`
      ${this.apiTask.render({
        initial: () => html`...`,
        pending: () => html`Loading...`,
        error: (e) => html`...`,
        complete: (user) => html`${user.name}`
      })}
    `;
  }
}
```
---
## @lit-labs/router
A Router for Lit.

```ts
class MyElement extends LitElement {

  private router = new Router(this, [
    { path: '/', render: () => html`<h1>Home</h1>` },
    { path: '/projects', render: () => html`<h1>Projects</h1>` },
    { path: '/about', render: () => html`<h1>About</h3>` },
  ]);

  render() {
    return html`
      <header>...</header>
      <main>${this.router.outlet()}</main>
      <footer>...</footer>
    `;
  }
}
```

---
## Render callbacks

```ts
{
  path: '/profile/:id',
  render: ({ id }) => html`<my-profile .profileId=${id}></my-profile>`
}
```

- When the ID changes, only the property gets updated
- The component isn't aware of being a container
 - No `useParams`, no Mixin, no dependencies

---
## Enter callbacks

```ts
{
  path: '/profile/:id',
  render: ({ id }) => html`<my-profile .profileId=${id}></my-profile>`,
  enter: async (params) => {
    await import('./my-profile.js');
  },
}
```

---
## @lit-labs

- **motion**: animations
- **task**: asynchronous tasks
- **router**: routing & navigation
- **ssr**, **ssr-client**: SSR support
- **eleventy-plugin-lit**: Lit templates & components with e11y
- **react**: React wrappers for Web Components and Reactive Controllers
- **observers**: Reactive Controllers for *IntersectionObserver*, *ResizeObserver*, *MutationObserver*...
- **analyzer**: syntax analyzer for IDEs & tools
- **scoped-registry-mixin**: mixin for the Scoped Custom Element Registries proposal

---
class: slide-separator
## Thanks ❤️

<h3 style="opacity: .7">https://github.com/UserGalileo/talks</h3>

#### Useful links
- **Lit docs**: https://lit.dev
- **Form Experiment**: https://github.com/UserGalileo/lit-reactive-forms
- **Lit Framework Discussion**: https://github.com/lit/lit/discussions/2462
- **Lit for React Devs (Codelab)**: https://codelabs.developers.google.com/codelabs/lit-2-for-react-devs
