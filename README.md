# ECMAScript Proposal: Save Navigation (Elvis) Operator

This proposal introduces a new accessor modifier `?`, preceeding property access via a `.` or `[]`. An example of which would be: `foo?.bar` or `foo?['bar']`

### Introduction

The safe navigation operator, also often known as the null propagation operator, or the null-conditional Operator in C#, or the Elvis operator due to it's syntax, allows for the safe access of nested object properties which may be null.

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

Note the use of `foo != null` rather than `foo !== null && foo !== undefined`

Using the new safe navigation operator, the access now looks like this:

```.js
let prop = foo?.bar?.baz; // { qux: "rar" }
```

Which desugars to:

```.js
let prop = foo != null && foo.bar != null ? foo.bar.baz : null;
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
let prop4 = foo?['bar']?['baz']?['qux'];
//> prop4 = "rar"
let prop5 = foo?['bar']?['fun2']()?['baz'];
//> prop5 = "qux"
```

Which desugar to the following, respectively:

```.js
let prop = foo != null
  && foo.bar != null
  && foo.bar.baz != null
  ? foo.bar.baz.qux
  : null;

let prop2 = foo != null
  && foo.bar != null
  ? foo.bar.fun()
  : null;

let _nv3
let prop3 = (_nv3=foo) != null
  && (_nv3=_nv3.bar) != null
  && (_nv3=_nv3.fun) != null
  && (_nv3=_nv3()) != null
  ? _nv3.baz
  : null;

let prop4 = foo != null
  && foo['bar'] != null
  && foo['bar']['baz'] != null
  ? foo['bar']['baz']['qux']
  : null;

let _nv5
let prop5 = (_nv5=foo) != null
  && (_nv5=_nv5['bar']) != null
  && (_nv5=_nv5['fun2']) != null
  && (_nv5=nv5()) != null
  ? _nv5['baz']
  : null;
```

This works the same way with bracket property accessors:


#### Assignment

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

Because the assignment scenario makes an assumption about that shape and type of the object being modified, it may not be a good candidate for inclusion.
