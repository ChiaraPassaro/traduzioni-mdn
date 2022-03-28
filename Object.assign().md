# Object.assign()

Il metodo `Object.assign()` copia tutte le proprie properties enumerabili da un o più object origine ad un object di destinazione. restituisce l'object destinazione modificato.

```JS
const target = { a: 1, b: 2 };
const source = { b: 4, c: 5 };

const returnedTarget = Object.assign(target, source);

console.log(target);
// expected output: Object { a: 1, b: 4, c: 5 }

console.log(returnedTarget);
// expected output: Object { a: 1, b: 4, c: 5 }

```

## Sintassi
`Object.assign(target, ...sources)`

### Parametri
`target`
L'object target - quello sul quale applicare le properties dell'object origine, che è restituito dopo la modifica.

`sources`
Object o objects di origine - gli objects che contengono le properties che vuoi applicare.

### Valore di return
L'object target

## Descrizione
Le properties nell'object target sono sovrascritte dalle properties nelle sorgenti se hanno la stessa key.
Le ultime property delle sorgenti sovrascrivono quelle delle prime.

Il metodo  `Object.assign()` copia solo le properties enumerabili e proprie dell'object sorgente nell'object target. Usa `Get` sulla sorgente e `Set` sul target, così invoca [getters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get) e [setters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set)

Così assegna le properties, oppure le copia o definisce nuove properties. Questo può renderlo inadatto per unire nuove properties in un prototipo se l'unione delle origini contiene dei getters.

Per copiare le definizioni delle property (includendo la loro enumerabilità) nei prototypes, usa invece [`Object.getOwnPropertyDescriptor()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor) and [`Object.defineProperty()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) .

Sia le properties `String` che `Symbol` vengono copiate.

In caso di errore, per esempio se una property non è writable, un [`TypeError`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypeError) è sollevato, e l'object `target` è cambiato se nessuna properties è stata aggiunta rima dell'errore.

> Nota: `Object.assign()` non può essere lanciato su origini `null` o `undefined`.

## Esempi
### Clonare un object
```JS
const obj = { a: 1 };
const copy = Object.assign({}, obj);
console.log(copy); // { a: 1 }
```

### Avviso per il Deep Clone
Per il [deep cloning](https://developer.mozilla.org/en-US/docs/Glossary/Deep_copy), abbiamo la necessità di usare delle alternative, perché `Object.assign()` copria i valori delle property.
Se l'origine è la referenza ad un objcet, copia solo il valore della referenza.

```JS
function test() {
  'use strict';

  let obj1 = { a: 0 , b: { c: 0}};
  let obj2 = Object.assign({}, obj1);
  console.log(JSON.stringify(obj2)); // { "a": 0, "b": { "c": 0}}

  obj1.a = 1;
  console.log(JSON.stringify(obj1)); // { "a": 1, "b": { "c": 0}}
  console.log(JSON.stringify(obj2)); // { "a": 0, "b": { "c": 0}}

  obj2.a = 2;
  console.log(JSON.stringify(obj1)); // { "a": 1, "b": { "c": 0}}
  console.log(JSON.stringify(obj2)); // { "a": 2, "b": { "c": 0}}

  obj2.b.c = 3;
  console.log(JSON.stringify(obj1)); // { "a": 1, "b": { "c": 3}}
  console.log(JSON.stringify(obj2)); // { "a": 2, "b": { "c": 3}}

  // Deep Clone
  obj1 = { a: 0 , b: { c: 0}};
  let obj3 = JSON.parse(JSON.stringify(obj1));
  obj1.a = 4;
  obj1.b.c = 4;
  console.log(JSON.stringify(obj3)); // { "a": 0, "b": { "c": 0}}
}

test();

```

### Unire objects
```JS
const o1 = { a: 1 };
const o2 = { b: 2 };
const o3 = { c: 3 };

const obj = Object.assign(o1, o2, o3);
console.log(obj); // { a: 1, b: 2, c: 3 }
console.log(o1);  // { a: 1, b: 2, c: 3 }, target object itself is changed.

```

### Unire con le stesse properties
```JS
const o1 = { a: 1, b: 1, c: 1 };
const o2 = { b: 2, c: 2 };
const o3 = { c: 3 };

const obj = Object.assign({}, o1, o2, o3);
console.log(obj); // { a: 1, b: 2, c: 3 }
```

Le properties sono sovrascritte dagli altri objects contenenti le stesse properties che vengono dopo nell'ordine dei parametri.

### Copiare le properties symbol-typed
```JS
const o1 = { a: 1 };
const o2 = { [Symbol('foo')]: 2 };

const obj = Object.assign({}, o1, o2);
console.log(obj); // { a : 1, [Symbol("foo")]: 2 } (cf. bug 1207182 on Firefox)
Object.getOwnPropertySymbols(obj); // [Symbol(foo)]
```

### Properties sulla catena protitipale e non-enumerable properties non possono essere copiate

```JS
const obj = Object.create({ foo: 1 }, { // foo is on obj's prototype chain.
  bar: {
    value: 2  // bar is a non-enumerable property.
  },
  baz: {
    value: 3,
    enumerable: true  // baz is an own enumerable property.
  }
});

const copy = Object.assign({}, obj);
console.log(copy); // { baz: 3 }
```

### Primitivi saranno wrappati in objects.

```JS
const v1 = 'abc';
const v2 = true;
const v3 = 10;
const v4 = Symbol('foo');

const obj = Object.assign({}, v1, null, v2, undefined, v3, v4);
// Primitives will be wrapped, null and undefined will be ignored.
// Note, only string wrappers can have own enumerable properties.
console.log(obj); // { "0": "a", "1": "b", "2": "c" }

```

### Le eccezioni interromperano il task di copia in corso
```JS
const target = Object.defineProperty({}, 'foo', {
  value: 1,
  writable: false
}); // target.foo is a read-only property

Object.assign(target, { bar: 2 }, { foo2: 3, foo: 3, foo3: 3 }, { baz: 4 });
// TypeError: "foo" is read-only
// The Exception is thrown when assigning target.foo

console.log(target.bar);  // 2, the first source was copied successfully.
console.log(target.foo2); // 3, the first property of the second source was copied successfully.
console.log(target.foo);  // 1, exception is thrown here.
console.log(target.foo3); // undefined, assign method has finished, foo3 will not be copied.
console.log(target.baz);  // undefined, the third source will not be copied either.

```

### Copiare gli accessors
```JS
const obj = {
  foo: 1,
  get bar() {
    return 2;
  }
};

let copy = Object.assign({}, obj);
console.log(copy);
// { foo: 1, bar: 2 }
// The value of copy.bar is obj.bar's getter's return value.

// This is an assign function that copies full descriptors
function completeAssign(target, ...sources) {
  sources.forEach(source => {
    let descriptors = Object.keys(source).reduce((descriptors, key) => {
      descriptors[key] = Object.getOwnPropertyDescriptor(source, key);
      return descriptors;
    }, {});

    // By default, Object.assign copies enumerable Symbols, too
    Object.getOwnPropertySymbols(source).forEach(sym => {
      let descriptor = Object.getOwnPropertyDescriptor(source, sym);
      if (descriptor.enumerable) {
        descriptors[sym] = descriptor;
      }
    });
    Object.defineProperties(target, descriptors);
  });
  return target;
}

copy = completeAssign({}, obj);
console.log(copy);
// { foo:1, get bar() { return 2 } }

```


[Object.assign()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign#primitives_will_be_wrapped_to_objects)
[[Javascript]]
[[Proxy]]
[[Object.defineProperty()]]