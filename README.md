## ECMAScript Proposal: Safe Navigation (Elvis) Operator

This proposal introduces a new accessor modifier `?.`, preceeding property access via a `.` or `[]`. An example of which would be: `foo?.bar`, or `foo?.[bar]`, or `foo?.()`

### Introduction

The safe navigation operator, also known as the null-conditional Operator in C#, or the Elvis operator due to it's syntax, allows for the safe access of nested object properties which may be `null` or `undefined`.

Using the operator in the manner: `foo?.bar` ensures that:

* The parent object is able to have properties (I.E. not `null` or `undefined`)
* That's it!

It's a very straightforward concept that is also very powerful.

### Example

Given the following object:

```.js
let foo = {
  bar: {
    baz: {
      qux: "rar"
    },
    fun: () => "baz",
    fun2: () => { baz: "qux" }
  },
  baz: 0
};
```

The operator itself is used as syntactic sugar for null checking as follows:

Currently the idiomatic way of safely accessing `foo.bar.baz` doing this in ECMAScript is as follows:

```.js
let prop = null;

if (foo && foo.bar) {
  prop = foo.bar.baz
}
```

Specifically, if `foo` exists and is truthy, we pull it's property `bar`. Though this is fairly idiomatic and widely used, it is incorrect and fails in the case of falsy values such as `0` and `''` which can have properties. Really what should be happening is this:

```.js
let prop = null;

if (foo != null && foo.bar != null) {
  prop = foo.bar.baz
}
```

Note the use of `foo != null` rather than `foo !== null && foo !== undefined`.

Actually, this still isn't correct, as each of those properties may be an accessor method or even a function call, so they each need to be cached:

```.js
let prop = null;

let _nv;
if ((_nv = foo) != null
  && (_nv = _nv.bar) != null) {
  prop = _nv.baz;
}
```

Now it's doing the right thing -- isn't that simple and easy to do? No. No it isn't, and this is precisely why the safe navigation operator is useful.

Using the safe navigation operator, the access now looks like this:

```.js
let prop = foo?.bar?.baz; // { qux: "rar" }
```

Which desugars to:

```.js
let _nv;
let prop = (_nv = foo) != null && (_nv = _nv.bar) != null ? _nv.baz : null;
```

### More Examples:

```.js
// dot property access
let prop = foo?.bar?.baz?.qux;
//> prop = "rar"

let prop2 = foo?.bar?.fun();
//> prop2 = "bar"

let prop3 = foo?.bar?.fun2().?baz;
//> prop3 = "qux"

// bracket property access
let prop4 = foo?.['bar']?.['baz']?.['qux'];
//> prop4 = "rar"

let prop5 = foo?.['bar']?.['fun2']?.()?.['baz'];
//> prop5 = "qux"
```

Which desugar to the following, respectively:

```.js
let _nv;
let prop = (_nv = foo) != null
  && (_nv=_nv.bar) != null
  && (_nv=_nv.baz) != null
  ? _nv.qux
  : null;

let _nv2;
let prop2 = ((_nv2=foo) != null
  && (_nv2=_nv2.bar) != null
  ? _nv2.fun
  : null)();

let _nv3;
let prop3 = (_nv3 = ((_nv3=foo) != null
  && (_nv3=_nv3.bar) != null
  ? nv3.fun
  : null)()) != null
  ? _nv3.baz
  : null;

let _nv4;
let prop4 = (_nv4=foo) != null
  && (_nv4=_nv4['bar']) != null
  && (_nv4=_nv4['baz']) != null
  ? _nv4['qux']
  : null;

let _nv5;
let prop5 = (_nv5=foo) != null
  && (_nv5=_nv5['bar']) != null
  && (_nv5=_nv5['fun2']) != null
  && typeof _nv5 === 'function'
  && (_nv5=nv5()) != null
  ? _nv5['baz']
  : null;
```


#### Assignment (Not part of the proposal)

The save navigation operator *could* also be used for assignment, but would only work for plain object literals

```.js
let myVar = {};

myVar?.some?.property = 5;
//> myVar = { some: { property: 5 } }
```

Would desugar to:

```.js
let myVar = {};

myVar = myVar != null ? myVar : {};
myVar.some = myVar.some != null ? myVar.some : {};
myVar.some.property = 5;
```

Because the assignment scenario makes an assumption about that shape and type of the object being modified, it isn't a good candidate for inclusion. A good example of why is that typos would create new parts of the object, rather than throwing an exception. It is being mentioned here mainly to show it isn't viable.

### Syntax

The syntax `?.` was chosen for consistency.

The following syntax does not work :

Syntax: `?[`

Example Use:
`foo?['bar']`

Example issue:

```.js
conditional?['foo']?['map']:fallback;
```

This is ambiguous -- does it mean

```.js
(conditional?['foo']) ? ['map'] : fallback
```

or

```.js
conditional ? (['foo']?['map']) : fallback
```

### Further discussion

This github repo is a summary/amalgamation of discussions that have taken place in multiple locations.  Here is a list of the primary sources:

* [esdiscuss](https://esdiscuss.org/topic/existential-operator-null-propagation-operator)
