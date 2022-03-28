# Reflect
`Reflect` è un object incorporato che fornisce metodi per le operazioni JS intercettabili. I metodi sono gli stessi di quelli degli handler proxy. `Reflect` non è un object function, quindi non è costruibile.

## Descrizione
A differenza di molti degli object globali, `Feflect` non è un costruttore. Non puoi usarlo con l'operatore [`new` ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new) o invocando l'object `Reflect` come un object function. Tutte le properties e i metodi di `Reflect` sono statici (priprio come l'object `Math`).

L'object `Reflect` fornisce le seguenti funzioni statiche che hanno lo stesso nome dei metodi handler dei proxy.

Alcuni di questi metodi sono gli stessi dei corrispondenti metodi su un Object, anche se ci somo [alcune sottili differenze tre essi](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/Comparing_Reflect_and_Object_methods)

## Metodi statici
[`Reflect.apply(target, thisArgument, argumentsList)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/apply)
Chiama una funzione `target` con gli argomenti come specificato dal parametro `argumentList`. Guarda anche [`Function.prototype.apply()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply).

[`Reflect.construct(target, argumentsList[, newTarget])`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/construct)
L'operatore `new` come una funzione. Equivale a chiamare `new target(...argumentsList)`. Inoltre fornisce l'opzione di specificare un prototipo differente.

[`Reflect.defineProperty(target, propertyKey, attributes)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/defineProperty)
Simile a [`Object.defineProperty()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty). Restituisce un booleano che è `true` se la property è stata definita con successo.

[`Reflect.deleteProperty(target, propertyKey)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/deleteProperty)
L'operatore `delete` come una funzione. Equivale a chiamare `delete target[propertyKey]`

[`Reflect.get(target, propertyKey[, receiver])`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/get)
Restituisce il valore di una property. Lavora come una funzione che prende una property da un object `target[propertyKey]` 

[`Reflect.getOwnPropertyDescriptor(target, propertyKey)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/getOwnPropertyDescriptor)
Simile a [`Object.getOwnPropertyDescriptor()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor). Restituisce una property descriptor che restituisce una property se esiste in quell'object, altrimenti `undefined`.

[`Reflect.getPrototypeOf(target)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/getPrototypeOf)
Simile a [`Object.getPrototypeOf()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf).

[`Reflect.has(target, propertyKey)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/has)
Restituisce un booleano indicando se il target ha una property. Sia propria che ereditata. Funziona come l'operatore `in` come una funzione.

[`Reflect.isExtensible(target)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/isExtensible)
La stessa cosa di [`Object.isExtensible()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/isExtensible). Restituisce un booleano che è `true` se il target è estensibile.

[`Reflect.ownKeys(target)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/ownKeys)
Restituisce un array delle property key proprie dell'object target.

[`Reflect.preventExtensions(target)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/preventExtensions)
Simile a  [`Object.preventExtensions()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/preventExtensions)  Restituisce un booleano che è `true` se l'update è avvenuto con successo.

[`Reflect.set(target, propertyKey, value[, receiver])`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/set) Una funzione che assegna valori alle properties. Restituisce un booleano che è `true` se l'update è avvenuto con successo.

[`Reflect.setPrototypeOf(target, prototype)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/setPrototypeOf) Una funzione che imposta il prototipo di un object.
Restituisce un booleano che è `true` se l'update è avvenuto con successo.

## Esempi
### Intercettare se un object contiene una certa property

```JS
const duck = {
  name: 'Maurice',
  color: 'white',
  greeting: function() {
    console.log(`Quaaaack! My name is ${this.name}`);
  }
}

Reflect.has(duck, 'color');
// true
Reflect.has(duck, 'haircut');
// false

```

### Restituire le key proprie di un object

```JS
Reflect.ownKeys(duck);
// [ "name", "color", "greeting" ]

```

### Aggiungere una nuova property all'object
```JS
Reflect.set(duck, 'eyes', 'black');
// returns "true" if successful
// "duck" now contains the property "eyes: 'black'"

```

[Reflect](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect)

[[Javascript]]
[[Proxy]]
[[Handler.set]]
[[handler.get()]]
