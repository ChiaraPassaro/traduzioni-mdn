# handler.get()

Il metodo `handler.get()` è una trap per prendere il valore di una property.

```JS
const monster1 = {
  secret: 'easily scared',
  eyeCount: 4
};

const handler1 = {
  get: function(target, prop, receiver) {
    if (prop === 'secret') {
      return `${target.secret.substr(0, 4)} ... shhhh!`;
    }
    return Reflect.get(...arguments);
  }
};

const proxy1 = new Proxy(monster1, handler1);

console.log(proxy1.eyeCount);
// expected output: 4

console.log(proxy1.secret);
// expected output: "easi ... shhhh!"
```

## Sintassi
```JS
const p = new Proxy(target, {
  get: function(target, property, receiver) {
  }
});
```

### Parametri
I seguenti parametri sono passati al metodo `get()`. `this` è legato all'handler.

`target`
L'object destinazione

`property`
Il nome o il Symbol della property da prendere

`receiver`
Sia il proxy che un object che eredita dal proxy.

### Valore di return
Il metodo `get()` restituisce qualunque valore.

## Descrizione
L'`handler.get()` è una trap per prendere il valore di una property.

### Intercettazioni
Questa trap può intercettare queste operazioni:
* Accesso alla property: `proxy[foo]`e `proxy.bar`
* Accesso alla property ereditata: `Object.create(proxy)[foo]`
* -   [`Reflect.get()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/get)

### Invarianti
Se le seguenti invarianti vengono violate, il proxy lancerà un [`TypeError`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypeError):

* Il valore riportato per una property deve essere lo stesso della corrispondente property nell'object target se la property dell'object target è un non-writable, non-configurable propria property data
* Il valore riportato per una property deve essere undefided se la property corrispondente nell'object target è un proprio accessor non-configurable che ha `undefined` come suo attributo `Get`.

## Esempi
### Trap per prelevare una property value
```JS
const p = new Proxy({}, {
  get: function(target, property, receiver) {
    console.log('called: ' + property);
    return 10;
  }
});

console.log(p.a); // "called: a"
                  // 10
```

Il seguente codice viola una invariante

```JS
const obj = {};
Object.defineProperty(obj, 'a', {
  configurable: false,
  enumerable: false,
  value: 10,
  writable: false
});

const p = new Proxy(obj, {
  get: function(target, property) {
    return 20;
  }
});

p.a; // TypeError is thrown
```


[Handler.get()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/get)
[[Javascript]]
[[Proxy]]
[[Object.defineProperty()]]