Il metodo `handler.set()` è una trap per impostare il valore di una property.

```JS
const monster1 = { eyeCount: 4 };

const handler1 = {
  set(obj, prop, value) {
    if ((prop === 'eyeCount') && ((value % 2) !== 0)) {
      console.log('Monsters must have an even number of eyes');
    } else {
      return Reflect.set(...arguments);
    }
  }
};

const proxy1 = new Proxy(monster1, handler1);

proxy1.eyeCount = 1;
// expected output: "Monsters must have an even number of eyes"

console.log(proxy1.eyeCount);
// expected output: 4

proxy1.eyeCount = 2;
console.log(proxy1.eyeCount);
// expected output: 2
```

## Sintassi
```JS
const p = new Proxy(target, {
  set: function(target, property, value, receiver) {
  }
});
```

### Parametri
I seguenti parametri sono passati al metodo `set()`. `this` è legato all'handler.

`target`
L'object destinatario

`property`
Il nome o il Symbol della property da impostare

`value`
Il nuovo valore della property da impostare.

`receiver`
L'object al quale  l'assegnazione  er originariamente diretta. Questo è solitamente il proxy stetto. Ma un `set()` handler può anche essere chiamato indirettamente, attraverso la catena prototipale o varie altre strade.

Ad esempio, supponiamo uno script `obj.name="jen"`, e `obj` non è un proxy, e non ha una propria property `.name`, ma ha un proxy nella sua catena prototipale. Quel proxy `set()` handler sarà chiamato, e `obj` sarà passato come ricevente.

### Valore di Return
Il metodo `set()` deve restituire un valore booleano.
* Return `true` indica che l'assegnazione è riuscita.
*  Se il metodo `set()` restituisce `false`, e l'assegnazione è avvenuta nel codice strict-mode, un [`TypeError`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypeError) sarà lanciato.

## Descrizione
Il metodo `handler.set()` è una trap per impostare il valore di una property-

### Intercettazioni
Questa trap intercetta queste operazioni:
* Assegnazione di un property: `proxy[foo] = bar` e `proxy.foo = bar`
* Addegnazione di una property ereditata `Object.create(proxy)[foo] = bar`
* [`Reflect.set()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/set) 

### Invarianti
Se le seguenti invarianti sono violatr, il proxy lancerà un [`TypeError`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypeError):

* Non è possibile cambiare il valore di una property che è differente dal valore della corrispondente propertu dell'object target se tale property è una data property non-writable, non-configurable.
* Non è possibile settare il valore di una property se la property corrispondente dell'object target è una property accessor non-configurable che un `undefined` come suo attributo `Set`
* Nella modalità Strict, un return false da un `set` handler lancia una eccezione [`TypeError`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypeError)

## Esempi
### Trap che imposta il valore di una property
Il seguente codice intrappola il setting del valore di una property.

```JS
const p = new Proxy({}, {
  set: function(target, prop, value, receiver) {
    target[prop] = value;
    console.log('property set: ' + prop + ' = ' + value);
    return true;
  }
})

console.log('a' in p);  // false

p.a = 10;               // "property set: a = 10"
console.log('a' in p);  // true
console.log(p.a);       // 10

```

[handler.set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/set)
[[Javascript]]
[[Proxy]]
[[handler.get()]]
[[Object.defineProperty()]]