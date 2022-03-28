# Proxy
L'object `Proxy` permette di creare un proxy per un altro object, che può intercettare e ridefinire le operazioni fondamentali per quel object.

## Descrizione
L'object `Proxy` permette di creare un object che può essere usato al posto di un object originale, ma che può ridefinire le operazioni fondamentali dell'object come getting, setting e definizione delle properties. Gli object Proxy sono comunemente usati per registrare gli accessi alle property, formattare, o sanitizzare gl input e così via.

Puoi creare un `Proxy` con due parametri:
* `target`: l'object originale che vuoi delegare
* `handler`: un object che definisce che operazioni possono essere intercettate e come ridefinire le operazioni intercettate.

Per esempio, questo codice definisce un semplice target con solo due properties, e un ancora più semplice handler con nessuna proprietà.

```JS
const target = {
  message1: "hello",
  message2: "everyone"
};

const handler1 = {};

const proxy1 = new Proxy(target, handler1);

```

Poiché l'handler è vuoto, questo proxy si comporta come il target originale:

```JS
console.log(proxy1.message1); // hello
console.log(proxy1.message2); // everyone
```

Per personalizzare il proxy, definiamo funzioni sull'object handler:

```JS
const target = {
  message1: "hello",
  message2: "everyone"
};

const handler2 = {
  get(target, prop, receiver) {
    return "world";
  }
};

const proxy2 = new Proxy(target, handler2);

```

Qui abbiamo fornito una implementazione di un handler [`get()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/get), che intercetta i tentativi di accesso all properties del target.

Le funzioni handler sono chiamate a volte *traps*, pesumibilmente perché cattirano le chiamate al target object.
La trap molto semplice nell'`handler2` seguente ridefinisce tutte le property accessors:
```JS
console.log(proxy2.message1); // world
console.log(proxy2.message2); // world
```

Con l'aiuto della classe [`Reflect`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect) possiamo dare ad alcuni accessors il comportamento originale e ridefinirne altri.

```JS
const target = {
  message1: "hello",
  message2: "everyone"
};

const handler3 = {
  get(target, prop, receiver) {
    if (prop === "message2") {
      return "world";
    }
    return Reflect.get(...arguments);
  },
};

const proxy3 = new Proxy(target, handler3);

console.log(proxy3.message1); // hello
console.log(proxy3.message2); // world

```

## Costruttore
[`Proxy()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy)
Crea un nuovo object.

## Metodi Statici
[`Proxy.revocable()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/revocable)

## Esempi
### Esempio base
In questo semplice esempio, il numero `37` viene restituito come un valore di default quando la property name non è un object. Esso usa l'handler `get()`.

```JS
const handler = {
  get(obj, prop) {
    return prop in obj ?
      obj[prop] :
      37;
  }
};

const p = new Proxy({}, handler);
p.a = 1;
p.b = undefined;

console.log(p.a, p.b);
//  1, undefined

console.log('c' in p, p.c);
//  false, 37

```

### Proxy di inoltro no-op
In questo esempio, usiamo un object nativo JS al quale il nostro proxy inoltrerà tutte le operazione che sono applicati ad esso.

```JS
const target = {};
const p = new Proxy(target, {});

p.a = 37;
//  operation forwarded to the target

console.log(target.a);
//  37
//  (The operation has been properly forwarded!)

```

Nota che mentre questo "no-op" lavora per l'object JS, non funziona per gli objects nativi browser, come gli elementi DOM.

### Validazione
Con un `Proxy`, puoi facilmente validare i valori passati ad un object. Questi esempi usano l'handler [`set()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/set)

```JS
let validator = {
  set(obj, prop, value) {
    if (prop === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError('The age is not an integer');
      }
      if (value > 200) {
        throw new RangeError('The age seems invalid');
      }
    }

    // The default behavior to store the value
    obj[prop] = value;

    // Indicate success
    return true;
  }
};

const person = new Proxy({}, validator);

person.age = 100;
console.log(person.age); // 100
person.age = 'young';    // Throws an exception
person.age = 300;        // Throws an exception

```

### Estendere un costruttore
Una funzione proxy può facilmente estendere un costruttore con un nuovo costruttore. 

Questo esempio usa gli handlers [`construct()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/construct) e [`apply()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/apply)

```JS
function extend(sup, base) {
  base.prototype = Object.create(sup.prototype);
  base.prototype.constructor = new Proxy(base, {
    construct: function(target, args) {
      var obj = Object.create(base.prototype);
      this.apply(target, obj, args);
      return obj;
    },
    apply: function(target, that, args) {
      sup.apply(that, args);
      base.apply(that, args);
    }
  });
  return base.prototype.constructor;
}

var Person = function(name) {
  this.name = name;
};

var Boy = extend(Person, function(name, age) {
  this.age = age;
});

Boy.prototype.gender = 'M';

var Peter = new Boy('Peter', 13);

console.log(Peter.gender);  // "M"
console.log(Peter.name);    // "Peter"
console.log(Peter.age);     // 13

```


### Manipolazione dei nodi DOM
In questo esempio usiamo `Proxy` per alternare un attributo di due differenti elementi: così quando impostiamo l'attributo su un elemento, l'attributo è unset sull'altro.

Creiamo un object `view` che è un proxy per un object con una property `selected`. L'handler del proxy definisce l'handler `set()`.

Quando assegniamo un elemento HTML a `view.selected`, l'attributo `aria-selected` dell'elemento è impostato su `true`.
Se poi assegniamo un elemento differente a `view.selected`, l'attributo `aria-selected` di questo elemento è impostato su `true` e quello dell'elemento precedente automaticamente su `false`.

```JS
const view = new Proxy({
  selected: null
},
{
  set(obj, prop, newval) {
    let oldval = obj[prop];

    if (prop === 'selected') {
      if (oldval) {
        oldval.setAttribute('aria-selected', 'false');
      }
      if (newval) {
        newval.setAttribute('aria-selected', 'true');
      }
    }

    // The default behavior to store the value
    obj[prop] = newval;

    // Indicate success
    return true;
  }
});

const item1 = document.getElementById('item-1');
const item2 = document.getElementById('item-2');

// select item1:
view.selected = item1;

console.log(`item1: ${item1.getAttribute('aria-selected')}`);
// item1: true

// selecting item2 de-selects item1:
view.selected = item2;

console.log(`item1: ${item1.getAttribute('aria-selected')}`);
// item1: false

console.log(`item2: ${item2.getAttribute('aria-selected')}`);
// item2: true

```

### Correzione di un value e di una property extra
L'object proxy `products` valuta il valore passato e lo converte in un array se necessario. L'object supporta inoltre una property extra chiamata `latestBrowser` sia come getter che setter.

```JS
let products = new Proxy({
  browsers: ['Internet Explorer', 'Netscape']
},
{
  get(obj, prop) {
    // An extra property
    if (prop === 'latestBrowser') {
      return obj.browsers[obj.browsers.length - 1];
    }

    // The default behavior to return the value
    return obj[prop];
  },
  set(obj, prop, value) {
    // An extra property
    if (prop === 'latestBrowser') {
      obj.browsers.push(value);
      return true;
    }

    // Convert the value if it is not an array
    if (typeof value === 'string') {
      value = [value];
    }

    // The default behavior to store the value
    obj[prop] = value;

    // Indicate success
    return true;
  }
});

console.log(products.browsers);
//  ['Internet Explorer', 'Netscape']

products.browsers = 'Firefox';
//  pass a string (by mistake)

console.log(products.browsers);
//  ['Firefox'] <- no problem, the value is an array

products.latestBrowser = 'Chrome';

console.log(products.browsers);
//  ['Firefox', 'Chrome']

console.log(products.latestBrowser);
//  'Chrome'

```

###  Trovare un array item object dalla sua property
Questo proxy estende un array con alcune utility features.
Come puoi vedere, puoi "definire" in maniera flessibile le properties senza usare `Object.defineProperties()`.
Questo esempio può essere adattato per trovare una riga di una tabella tramite la sua cella.
In questo caso, il target sarà `table.rows`.

```JS
let products = new Proxy([
  { name: 'Firefox', type: 'browser' },
  { name: 'SeaMonkey', type: 'browser' },
  { name: 'Thunderbird', type: 'mailer' }
],
{
  get(obj, prop) {
    // The default behavior to return the value; prop is usually an integer
    if (prop in obj) {
      return obj[prop];
    }

    // Get the number of products; an alias of products.length
    if (prop === 'number') {
      return obj.length;
    }

    let result, types = {};

    for (let product of obj) {
      if (product.name === prop) {
        result = product;
      }
      if (types[product.type]) {
        types[product.type].push(product);
      } else {
        types[product.type] = [product];
      }
    }

    // Get a product by name
    if (result) {
      return result;
    }

    // Get products by type
    if (prop in types) {
      return types[prop];
    }

    // Get product types
    if (prop === 'types') {
      return Object.keys(types);
    }

    return undefined;
  }
});

console.log(products[0]);          // { name: 'Firefox', type: 'browser' }
console.log(products['Firefox']);  // { name: 'Firefox', type: 'browser' }
console.log(products['Chrome']);   // undefined
console.log(products.browser);     // [{ name: 'Firefox', type: 'browser' }, { name: 'SeaMonkey', type: 'browser' }]
console.log(products.types);       // ['browser', 'mailer']
console.log(products.number);      // 3

```

### Un esempio di `traps` list completo

Ora per creare un esempio completo di `traps` list, per scopi didattici, proveremo a creare un proxy per un object non-native che è particolarmente adatto a queste operazioni: il `docCookies` global object è creato da  [a simple cookie framework](https://reference.codeproject.com/dom/document/cookie/simple_document.cookie_framework).

```JS
/*
  var docCookies = ... get the "docCookies" object here:
  https://reference.codeproject.com/dom/document/cookie/simple_document.cookie_framework
*/

var docCookies = new Proxy(docCookies, {
  get (oTarget, sKey) {
    return oTarget[sKey] || oTarget.getItem(sKey) || undefined;
  },
  set: function (oTarget, sKey, vValue) {
    if (sKey in oTarget) { return false; }
    return oTarget.setItem(sKey, vValue);
  },
  deleteProperty: function (oTarget, sKey) {
    if (!sKey in oTarget) { return false; }
    return oTarget.removeItem(sKey);
  },
  ownKeys: function (oTarget, sKey) {
    return oTarget.keys();
  },
  has: function (oTarget, sKey) {
    return sKey in oTarget || oTarget.hasItem(sKey);
  },
  defineProperty: function (oTarget, sKey, oDesc) {
    if (oDesc && 'value' in oDesc) { oTarget.setItem(sKey, oDesc.value); }
    return oTarget;
  },
  getOwnPropertyDescriptor: function (oTarget, sKey) {
    var vValue = oTarget.getItem(sKey);
    return vValue ? {
      value: vValue,
      writable: true,
      enumerable: true,
      configurable: false
    } : undefined;
  },
});

/* Cookies test */

console.log(docCookies.my_cookie1 = 'First value');
console.log(docCookies.getItem('my_cookie1'));

docCookies.setItem('my_cookie1', 'Changed value');
console.log(docCookies.my_cookie1);

```


[[Javascript]]
[Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy#no-op_forwarding_proxy)
[[Object.defineProperty()]]