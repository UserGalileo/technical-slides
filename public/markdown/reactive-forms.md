slidenumbers: true
slidecount: true
class: slide-separator

<img style="float: left;" src="../images/logo-angular.png" width="60px">
### Angular
# Reactive Forms

---
## Roadmap

1. Differenze fra Template-driven e Reactive Form
2. Reactive Form: Creazione
3. Reactive Form: Validazione
4. Reactive Form: Manipolazione
5. Validatori custom
6. Reattività
7. Validatori asincroni
8. Cross-field validation


---
## Differenze

.cols[
.half[
### Template-driven Forms
- Vivono nel template grazie a direttive
- Richiedono meno codice
- Difficili da manipolare
- Difficili da testare
- ...ideali per casi semplici
]
.half[
### Reactive Forms
- Vivono nella classe
- Facili da testare
- Facili da manipolare
- Facili da monitorare (RxJS)
- Facili da validare
- ...ideali per casi complessi
    - Campi dinamici
    - Validatori dinamici
    - Interazioni con servizi...
]
]

---
## Creazione Reactive Forms

Importa `ReactiveFormsModule` da `@angular/forms` nel modulo in cui li utilizzi

#### _src/app/app.module.ts_
```ts
import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
  declarations: [],
  imports: [
*   ReactiveFormsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

---
## Creazione Reactive Forms

Utilizza le API di Angular per modellare il tuo form

```ts
import { FormControl, FormGroup, FormArray } from '@angular/forms';

profile = new FormGroup({
  name: new FormControl('Michele'),
  surname: new FormControl('Stieven'),
  phones: new FormArray([
    new FormControl('+39 123 4567890'),
    new FormControl('+39 123 4567890'),
  ])
})
```

- Un **FormControl** rappresenta un singolo campo di un form
- Un **FormGroup** rappresenta un intero form
- Un **FormArray** rappresenta un elenco numerato di controlli
- Tutti estendono una classe chiamata **AbstractControl**

---
## Creazione Reactive Forms

Utilizza le API di Angular per modellare il tuo form

```html
<form [formGroup]="profile" (ngSubmit)="onSubmit()">
  <input formControlName="name">
  <input formControlName="surname">
  <button>Invia</button>
</form>
```
- Ogni `button` di default è di tipo `submit`
- Puoi ascoltare il submit dell'intero form con `(ngSubmit)`

_Vedremo poi come mettere in binding un FormArray!_

---
## Creazione Reactive Forms

Non è necessario raggruppare tutto in un FormGroup, possiamo avere dei campi stand-alone

```ts
name = new FormControl('Michele')
```

```html
<input [formControl]="name">
```

Per avere il valore attuale:
```ts
const username = this.name.value;
```
---
## Manipolazione dei valori
Puoi modificare il valore attuale di un controllo con due metodi:

```ts
this.profile.setValue({
  name: 'Michele',
  surname: 'Stieven'
});
```
```ts
this.profile.patchValue({
  name: 'Fabio'
})
```
- **setValue** va in errore se la struttura del form è sbagliata o parziale
- **patchValue** accetta un oggetto parziale (e scarta proprietà aggiuntive)
- Puoi usare setValue per aggiornare un singolo _FormControl_

---
## Reset
Puoi resettare un controllo o un intero form con `reset()`

```ts
this.profile.reset();
```
### Cosa fa
- Assegna il valore _null_ ai controlli del form
- Il controllo torna ad essere _untouched_ e _pristine_
- Puoi resettare il controllo ad un valore specifico

### Cosa non fa
- Non rimuove i controlli in un _FormArray_, li pulisce solamente

---
## FormGroup dinamico
Puoi aggiungere o rimuovere controlli in un FormGroup con **addControl** e **removeControl**

```ts
togglePartitaIva() {
  if (this.isPartitaIva) {
    this.profile.addControl('partitaIva', new FormControl(''))
  } else {
    this.profile.removeControl('partitaIva')
  }
}
```
- Il metodo **setControl** sovrascrive un controllo esistente
- Il metodo **registerControl** è come addControl ma non aggiorna il valore o la validità del controllo

---
## FormArray
Puoi usare un getter per facilitarti la vita e la tipizzazione!

```ts
get phones() {
  return this.profile.get('phones')! as FormArray;
}
```

```html
<form [formGroup]="profile" (ngSubmit)="onSubmit()">
  ...
* <div formArrayName="phones">
    <div *ngFor="let phone of phones.controls; let i = index">
      <input [formControlName]="i" >
    </div>
  </div>
</form>
```

---
## FormArray
Puoi manipolare i controlli in un _FormArray_ con questi metodi:

```ts
phones.at(index)
```
```ts
phones.push(new FormControl(''))
```
```ts
phones.insert(index, new FormControl(''))
```
```ts
phones.removeAt(index)
```
```ts
phones.clear()
```
---
##FormBuilder
Un aiuto per scrivere meno codice ed evitare i `new`

```ts
import { FormBuilder } from '@angular/forms';

export class AppComponent {

  profile = this.fb.group({
    name: [''],
    surname: [''],
    phones: this.fb.array([])
  })

  constructor(private fb: FormBuilder) {}
}
```
- Esiste il metodo `control()`, ma in un FormGroup è sottinteso
---
## Stato di un controllo

```ts
profile.get('name')!.status // VALID, INVALID, PENDING, DISABLED
profile.get('name')!.valid
profile.get('name')!.invalid
profile.get('name')!.pending
profile.get('name')!.disabled
```
Disabilitare un controllo manualmente:
```ts
profile.get('name')!.enable();
profile.get('name')!.disable();
```
UI status:
```ts
profile.get('name')!.touched
profile.get('name')!.untouched
profile.get('name')!.dirty
profile.get('name')!.pristine
```
---
## markAs*
Puoi manipolare lo stato di un controllo con i metodi _markAs_ e _markAllAs_

```ts
name.markAsDirty();
name.markAsPristine();
```
```ts
name.markAsTouched();
name.markAsUntouched();
```
```ts
profile.markAllAsUntouched();
```
```ts
name.markAsPending();
```
---
## Validatori

Built-in Validators: `required`, `maxLength`, `minLength`, `pattern`, ecc...

```ts
import { Validators } from '@angular/forms';
```

```ts
// Sintassi standard
name = new FormControl('', [Validators.required]);
```

```ts
// FormBuilder
profile = this.fb.group({
  name: ['', Validators.required],
  surname: ['', [Validators.required, Validators.minLength(2)]],
})
```
I validatori *non* vanno riapplicati nel template!

---
## Controllo degli errori

Controlla se il campo ha degli errori

```ts
get name() {
  return this.profile.get('name')! as FormControl;
}
```

```html
<ul *ngIf="name.invalid && name.touched">
* <li *ngIf="name.hasError('maxlength')">
    Non ci sta nel database! 😵
  </li>
* <li *ngIf="name.hasError('required')">
    Chi sei? 🤌
  </li>
</ul>
```

- _In alternativa esiste la proprietà `errors` in ogni AbstractControl, un oggetto_
---
## Validatori custom

Puoi creare un tuo validatore personale che ritorni:
- `null` se il valore è valido
- `{ validatorName: errorText }` se non lo è

```ts
import { AbstractControl } from '@angular/form';

export function noMicheleValidator(control: AbstractControl) {
  return control?.value === 'michele'
    ? { noMichele: 'Michele non è il benvenuto ⛔' }
    : null;
}
```
```ts
name = new FormControl('', [
  required,
* noMicheleValidator
])
```

---
## Validatori custom (factory)
Puoi generalizzare i tuoi validatori con una funzione intermedia (_higher-order_)

#### _forbidden.validator.ts_
```ts
import { AbstractControl } from '@angular/form';

export function forbiddenValidator(name: string) {
  return (control: AbstractControl) => {
    return control?.value === name
      ? { forbidden: `${name} non è il benvenuto ⛔` }
      : null;
  }
}
```
```ts
name = new FormControl('', [
  required,
* forbiddenValidator('michele')
])
```

---
## Stato di un form
- Un FormGroup è _invalid_ se almeno uno dei suoi controlli è _invalid_...
- ...in questo caso gli errori rimangono sui singoli controlli

Volendo puoi usare una funzione helper, ad esempio:
```ts
function collectErrors(form: FormGroup) {
    let errors: Record<string, string> = {};
    
    Object.keys(form.controls).forEach(key => {
      const error = form.get(key)!.errors;
      if (error) {
        errors[key] = error;
      }
    });
    
    return errors;
}
```
---
## Reattività
Un AbstractControl espone due Observable: _valueChanges_ e _statusChanges_

```ts
get name() {
  return this.profile.get('name')! as FormControl;
}
```
```ts
this.name.valueChanges.subscribe(value => {
  console.log(`Nuovo valore: ${value}`)
})
```

```ts
this.name.statusChanges.subscribe(status => {
  console.log(`Nuovo stato: ${status}`)
})
```
- Questi Observable non emettono il primo valore!

---
## valueChanges
Puoi ottenere il primo valore del form con l'operatore `startWith`

_**FormControl**_
```ts
this.name.valueChanges.pipe(
  startWith(this.name.value)
)
```
_**FormGroup**_
```ts
this.profile.valueChanges.pipe(
  startWith(this.profile.value)
)
```

---
## valueChanges
Utile per creare **stati derivati**

```ts
name$ = this.name.valueChanges.pipe(
  startWith(this.name.value)
);
```
```ts
pageTitle$ = this.name$.pipe(
  map(name => {
    return name
      ? `Sto creando l'utente ${name}...`
      : `Crea un nuovo utente!`
  })
)
```
```html
<h1>{{ pageTitle$ | async }}</h1>
<form>...</form>
```
---
## Validatori asincroni
Un validatore può essere asincrono e ritornare un Observable o una Promise

```ts
import { ValidationErrors } from '@angular/forms';

function forbiddenValidator(control: AbstractControl): Promise<ValidationErrors | null> {

  return fetch(`/api/check?name=${control.value}`)
  then(res => res.json())
  then(res => {
    return res
        ? null
        : { forbidden: true }
  })
}
```

---
## Validatori asincroni
Un validatore può essere asincrono e ritornare un Observable o una Promise

```ts
@Injectable({ providedIn: 'root' })
export class ForbiddenValidator {
  
  constructor(private http: HttpClient) {}
  
  validate(): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> => {
      return this.http.get<boolean>(`/api/check?name=${control.value}`).pipe(
          map(res => {
            return res
                ? null
                : { forbidden: true }
          })
      )
    }
  }
}
```

---
## Validatori asincroni - utilizzo
Il terzo parametro di un `FormControl` accetta un array di validatori asincroni.

- _Se il validatore è una semplice funzione:_

```ts
name = new FormControl('', [], [forbiddenValidator])
```
- _Se il validatore richiede un servizio:_

```ts
constructor(private forbiddenValidator: ForbiddenValidator) {}
```

```ts
name = new FormControl('', [], [this.forbiddenValidator.validate()])
```
_**Nota**: un validatore asincrono non parte finché tutti i validatori sincroni di quel controllo non danno più errori_

---
## Cross-field validation
Un validatore può essere agganciato a un `FormGroup` per validare più campi alla volta

```ts
function forbiddenValidator(control: AbstractControl) {
  
  const name = control.get('name')?.value;
  const surname = control.get('surname')?.value;
  
  if (name === 'Mario' && surname === 'Rossi') {
    return {
      forbidden: `${name} ${surname} non è il benvenuto ⛔`
    }
  }
}
```
_**Nota**: l'`AbstractControl`, in questo caso, sarà un `FormGroup`_  
_**Nota**: anche in questo caso il validatore può essere asincrono, e parte quando **tutti** i controlli figli sono validi_

---
## Cross-field validation
Puoi specificare dei validatori sincroni o asincroni nel secondo parametro di `FormGroup`:

```ts
profile = new FormGroup({
  name: new FormControl(''),
  surname: new FormControl('')
}, {
* validators: [forbiddenValidator],
* asyncValidators: []
})
```

L'errore `forbidden` sarà applicato al FormGroup, non ai singoli controlli:

```ts
this.profile.hasError('forbidden'); // true / false

this.profile.get('name')!.hasError('forbidden'); // sempre false
```

In questo esempio, _name_ e _surname_ saranno sempre `VALID`
