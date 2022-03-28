# Object.defineProperty()
Il metodo statico `Object.defineProperty()` definisce direttamente una nuova proprietà  su un object, o modifica una proprietà esistente su un object, e ritorna un object.

```JS
const object1 = {};

Object.defineProperty(object1, 'property1', {
  value: 42,
  writable: false //non modificabile
});

object1.property1 = 77;
// throws an error in strict mode 

console.log(object1.property1);
// expected output: 42
```

## Sintassi
`Object.defineProperty(obj, prop, descriptor)
`

### Parameter
`obj`
l'object sul quale definire la proprietà

`prop`
Il nome o il [Symbol]( https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol) della proprietà da definire o modificare.

`descriptor`
Il descriptor per la propiet' da definire o modificare

### Return Value
L'object che è stato passato alla funzione
 
### Descrizione
 
Questo metodo consente una aggiunta precisa o modifica ad una proprietà di un object. La normale aggiunta di una proprietà attraverso l'assegnazione crea delle proprietà che saranno mostrate durante l'enumerazione(`for...in loop` o `Object.keys`), questi valori possono essere cambiati e possono essere cancellati. Questo metodo ci consegnte questi dettagli extra da cambiare dai loro default. Per default, i valori aggiunti con `Object.defineProperty()` sono immutabili e non enumerabili.

I property descriptors presenti in un object sono di due tipi: data descriptors e accessor descriptors.
Un **data descriptor** è una proprietà che ha un valore, che può essere o non essere writable. 
Un **accessor descriptor** è una proprietà descritta da una coppia di funzioni getter-setter.
Un descriptor deve essere di uno di questi due tipi, non può essere di entrambi.

Sia i data che gli accessor descriptors sono degli objecys. 
Condividono le seguenti postional keys (da notare: i default menzionati qui si riferiscono alle proprietà create con Object.defineProperty())

`configurable`
`true` se il tipo di property descriptor può essere modificato e se può essere cancellato dall'object corrispondente. **Il default è `false`**

`enumerable`
`true` se e solo se questa proprietà si mostra durante l'enumerazione delle proprietà dell'object corrispondente. **Default `false`**

Un **data descriptor** inoltre ha le seguenti optiona keys:

`value`
Il valore associato con la property. Può essere un qualunque valore valido di JS (numero, object, functuon...) **Il default è `undefined`**

`writable`
`true` se il valore associato con la property può essere modificato con un operatore di assegnazione. **Il default è `false`**

Un **accessor descriptor** ha anche le seguenti optiona keys:

`get`
Una funzione che serve da getter per la proprietà, o undefined se non c'è un getter.
Quando la  property è acceduta, questa funzione è chiamata senza argomenti e con `this` impostato come object attraverso il quale la property è stata acceduta (questo può non essere l'object sul quale la proprietà è definita per ereditarietà). Il valore di ritorno che sarà usato è il valore della property.
**Il default è `undefined`**

`set`
Una funzione che serve da setter per la proprietà, o `undefined` se non ci sono dei setter. Quando la proprietà è assegnata, questa funzione è chiamata con un argomento (il valore passato alla property) e con `this` impostato sull'object alla quale la proprietà è assegnata. **Il default è `undefined`**

Se un descriptor non ha le key `value`, `writable`, `get` e `set`, è trattato come data descriptor. Se ha entrambe le keys [value o writable] e [get o set], una eccezione viene lanciata. 

Teniamo in mente che questi attributi non sono necessaiamente le proprietà del descriptor. Le proprietà ereditate saranno considerate a loro volta. Per assicurare che questi defaults siano preservati, è possibile freezare l'object upfront, specificare tutte le opzioni come esplicite, o puntare a `null` con `Object.create(null)`.

```JS
// using __proto__
var obj = {};
var descriptor = Object.create(null); // no inherited properties
descriptor.value = 'static';

// not enumerable, not configurable, not writable as defaults
Object.defineProperty(obj, 'key', descriptor);

// being explicit
Object.defineProperty(obj, 'key', {
  enumerable: false,
  configurable: false,
  writable: false,
  value: 'static'
});

// recycling same object
function withValue(value) {
  var d = withValue.d || (
    withValue.d = {
      enumerable: false,
      writable: false,
      configurable: false,
      value: value
    }
  );

  // avoiding duplicate operation for assigning value
  if (d.value !== value) d.value = value;

  return d;
}
// ... and ...
Object.defineProperty(obj, 'key', withValue('static'));

// if freeze is available, prevents adding or
// removing the object prototype properties
// (value, get, set, enumerable, writable, configurable)
(Object.freeze || Object)(Object.prototype);

```

## Esempi
### Creare una proprietà

Quando una specifica proprietà non esiste nell'object, `Object.defineProperty()` crea una nuova property come descritta. I campi per il descriptor possono essere omessi, e i valori di default questi campi sono impostati.

```JS
var o = {}; // Creates a new object

// Example of an object property added
// with defineProperty with a data property descriptor
Object.defineProperty(o, 'a', {
  value: 37,
  writable: true,
  enumerable: true,
  configurable: true
});
// 'a' property exists in the o object and its value is 37

// Example of an object property added
// with defineProperty with an accessor property descriptor
var bValue = 38;
Object.defineProperty(o, 'b', {
  // Using shorthand method names (ES2015 feature).
  // This is equivalent to:
  // get: function() { return bValue; },
  // set: function(newValue) { bValue = newValue; },
  get() { return bValue; },
  set(newValue) { bValue = newValue; },
  enumerable: true,
  configurable: true
});
o.b; // 38
// 'b' property exists in the o object and its value is 38
// The value of o.b is now always identical to bValue,
// unless o.b is redefined

// You cannot try to mix both:
Object.defineProperty(o, 'conflict', {
  value: 0x9f91102,
  get() { return 0xdeadbeef; }
});
// throws a TypeError: value appears
// only in data descriptors,
// get appears only in accessor descriptors

```

### Modificare una property
Quando una property esiste già,  `Object.defineProperty()` cerca di modificare la property in accordo con il valore del descriptor e della configurazione dell'object corrente. Se il vecchio descriptor avesse il suo attributo configurable impostato a false la proprietà direbbe di essere "non configurable".
Non è possibile cambiare alcun valore di una accesso property non-configurable.
Per le data properties che sono configurabili, è possibile modificare il valore se la proprietà è writable, ed è possibile cambiare l'attributo `writable` da `true` a `false`.
Non è possibile passare tra i tipi di property data e accessor quando la property è non-configurable.

Un [`TypeError`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypeError) è lanciato quando si cerca di cambiare un attributo non-configurable (eccetto `value` e `writable`, se persesso) a meno che il corrente e il nuovo valore siano uguali.

#### Writable attribute
Quando la property `writable` è impostato su `false`, la property si dice "non-writable".
Non può essere riassegnata
```JS

var o = {}; // Creates a new object

Object.defineProperty(o, 'a', {
  value: 37,
  writable: false
});

console.log(o.a); // logs 37
o.a = 25; // No error thrown
// (it would throw in strict mode,
// even if the value had been the same)
console.log(o.a); // logs 37. The assignment didn't work.

// strict mode
(function() {
  'use strict';
  var o = {};
  Object.defineProperty(o, 'b', {
    value: 2,
    writable: false
  });
  o.b = 3; // throws TypeError: "b" is read-only
  return o.b; // returns 2 without the line above
}());

```

Come visto nell'esempio, provare a riscrivere una property non-writable non la cambia ma non lancia neppure una eccezione.

#### Enumerable attribute
L'attributo della property `enumerable` definisce se la property viene selezionata da [`Object.assign()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) o dallo [`spread`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) operator.
Per le property non-[`Symbol`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol) è anche definito se si mostra in un [`for...in`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in) loop e [`Object.keys()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys) oppure no.

```JS
var o = {};
Object.defineProperty(o, 'a', {
  value: 1,
  enumerable: true
});
Object.defineProperty(o, 'b', {
  value: 2,
  enumerable: false
});
Object.defineProperty(o, 'c', {
  value: 3
}); // enumerable defaults to false
o.d = 4; // enumerable defaults to true
         // when creating a property by setting it
Object.defineProperty(o, Symbol.for('e'), {
  value: 5,
  enumerable: true
});
Object.defineProperty(o, Symbol.for('f'), {
  value: 6,
  enumerable: false
});

for (var i in o) {
  console.log(i);
}
// logs 'a' and 'd' (in undefined order)

Object.keys(o); // ['a', 'd']

o.propertyIsEnumerable('a'); // true
o.propertyIsEnumerable('b'); // false
o.propertyIsEnumerable('c'); // false
o.propertyIsEnumerable('d'); // true
o.propertyIsEnumerable(Symbol.for('e')); // true
o.propertyIsEnumerable(Symbol.for('f')); // false

var p = { ...o }
p.a // 1
p.b // undefined
p.c // undefined
p.d // 4
p[Symbol.for('e')] // 5
p[Symbol.for('f')] // undefined

```

#### Configurable attribute
L'attributo `configurable` controlla allo stesso tempo se una property può essere cacellata da un object e se i suoi attributi (oltre a `value` e `writable`) possono essere cambiati.

```JS
var o = {};
Object.defineProperty(o, 'a', {
  get() { return 1; },
  configurable: false
});

Object.defineProperty(o, 'a', {
  configurable: true
}); // throws a TypeError
Object.defineProperty(o, 'a', {
  enumerable: true
}); // throws a TypeError
Object.defineProperty(o, 'a', {
  set() {}
}); // throws a TypeError (set was undefined previously)
Object.defineProperty(o, 'a', {
  get() { return 1; }
}); // throws a TypeError
// (even though the new get does exactly the same thing)
Object.defineProperty(o, 'a', {
  value: 12
}); // throws a TypeError // ('value' can be changed when 'configurable' is false but not in this case due to 'get' accessor)

console.log(o.a); // logs 1
delete o.a; // Nothing happens
console.log(o.a); // logs 1

```

Se l'attributo `configurable` di `o.a` fosse stato `true`, nessuno degli errori sarebbe lanciato e la property sarebbe cancellata alla fine.

#### Aggiungere property e default valuse
È importante considerare il modo in cui i value degli attributi sono applicati.
C'è spesso una differeza tra usare la dot notation per assegnare un valore e usare `Object.defineProperty()`, come mostrato nell'esempio seguente.

```JS
var o = {};

o.a = 1;
// is equivalent to:
Object.defineProperty(o, 'a', {
  value: 1,
  writable: true,
  configurable: true,
  enumerable: true
});

// On the other hand,
Object.defineProperty(o, 'a', { value: 1 });
// is equivalent to:
Object.defineProperty(o, 'a', {
  value: 1,
  writable: false,
  configurable: false,
  enumerable: false
});

```

#### Custom Setters e Getters
L'esempio seguente mostra come implementare un object self-archiving. Quando la `temperature` property è impostata, l'array `archive` ottiene una voce di registro.

```JS
function Archiver() {
  var temperature = null;
  var archive = [];

  Object.defineProperty(this, 'temperature', {
    get() {
      console.log('get!');
      return temperature;
    },
    set(value) {
      temperature = value;
      archive.push({ val: temperature });
    }
  });

  this.getArchive = function() { return archive; };
}

var arc = new Archiver();
arc.temperature; // 'get!'
arc.temperature = 11;
arc.temperature = 13;
arc.getArchive(); // [{ val: 11 }, { val: 13 }]

```

In questo esempio, un getter restituisce sempre lo stesso valore.
```JS
var pattern = {
    get() {
        return 'I always return this string, ' +
               'whatever you have assigned';
    },
    set() {
        this.myname = 'this is my name string';
    }
};

function TestDefineSetAndGet() {
    Object.defineProperty(this, 'myproperty', pattern);
}

var instance = new TestDefineSetAndGet();
instance.myproperty = 'test';
console.log(instance.myproperty);
// I always return this string, whatever you have assigned

console.log(instance.myname); // this is my name string

```

#### Ereditarietà delle properties
Se una accessor property è ereditata, i suoi metodi `get` e `set` saranno chiamati quando la property è acceduta e modificata sugli objects discendenti. Se questi metodi usano una variabile per conservare un valore, questo valore sarà condiviso con altri objects.

```JS
function myclass() {
}

var value;
Object.defineProperty(myclass.prototype, "x", {
  get() {
    return value;
  },
  set(x) {
    value = x;
  }
});

var a = new myclass();
var b = new myclass();
a.x = 1;
console.log(b.x); // 1

```

Questo può essere sistemato salvando il valore in un'altra property. Nei metodi `get` e `set`, `this` punta all'object che è usato per accedere o modificare la property.

```JS
function myclass() {
}

Object.defineProperty(myclass.prototype, "x", {
  get() {
    return this.stored_x;
  },
  set(x) {
    this.stored_x = x;
  }
});

var a = new myclass();
var b = new myclass();
a.x = 1;
console.log(b.x); // undefined

```

 A differenza delle accessor properties, le value properties sono sempre impostate sull'object stesso, non sul prototipo. Tuttavia, se una value property non-writable è ereditato, impedisce comunque di modificare la property dell'object.  
 
 ```JS
function myclass() {
}

myclass.prototype.x = 1;
Object.defineProperty(myclass.prototype, "y", {
  writable: false,
  value: 1
});

var a = new myclass();
a.x = 2;
console.log(a.x); // 2
console.log(myclass.prototype.x); // 1
a.y = 2; // Ignored, throws in strict mode
console.log(a.y); // 1
console.log(myclass.prototype.y); // 1

 ```
 
 [[javascript]]
[Object.defineProperty()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

[[Proxy]]