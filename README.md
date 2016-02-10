# ECMAScript Proposal: Save Navigation (Elvis) Operator

This proposal introduces a new accessor modifier `?`, preceeding property access via a `.` or `[]`. An example of which would be: `foo?.bar` or `foo?['bar']`

### Introduction

The safe navigation operator, also often known as the null propagation operator, or the null-conditional Operator in C#, or the Elvis operator due to it's syntax, allows for the safe access of nested object properties which may be null.

Using the operator in the manner: `foo?.bar` ensures that:

* the parent object is truthy
* if it is no

Given the following object:

```.js
let foo = {
  bar: {
    baz: {
      qux: "bar"
    },
    fun: () => "baz",
    fun2: () => { baz: "qux" }
  },
  baz: 0
};
```

The operator itself is used as syntactic sugar for null checking as follows:

Currently the idiomatic way of doing this in ECMAScript is as follows:

```.js
let prop = null;

if (foo) {
  prop = foo.bar
}
```

Specifically, if foo exists and is truthy, we pull it's property `bar`

Using the new safe navigation operator, it now looks like this:

```.js
let prop = foo?.bar;
```

Which desugars to:

```.js
let prop = foo ? foo.bar : null;
```

### More Examples:

```.js
let prop2 = foo?.bar?.baz?.qux;
let prop3 = foo?.bar?.fun();
let prop4 = foo?.bar?.fun2().?baz;
let prop5 = foo?['bar']?['baz']?['qux'];
let prop6 = foo?['bar']?['fun']()?['baz'];
```

Which desugar to:

```.js
let prop = foo && foo.bar && foo.bar.baz && foo.bar.baz.qux || null;

let prop2 = foo && foo.bar && typeof foo.bar.fun === 'function' && foo.bar.fun() || null;

let _nv3
let prop3 = (_nv3=foo)
  && (_nv3=_nv3.bar)
  && (_nv3=_nv3.fun)
  && typeof _nv3 === 'function'
  && (_nv3=_nv3())
  && _nv3.baz || null;

let prop4 = foo && foo['bar'] && foo['bar']['baz'] && foo['bar']['baz']['qux'] || null;

let _nv5
let prop5 = (_nv5=foo)
  && (_nv5=_nv5['bar'])
  && (_nv5=_nv5['fun'])
  && typeof _nv5 === 'function'
  && (_nv5=nv5())
  && _nv5['baz'] || null;
```

This works the same way with bracket property accessors:


#### Assignment

The save navigation operator *could* also be used for assignment, but would only work for plain object literals

```.js
let myVar = {};

myVar?.some?.property = 5;
// myVar is { some: { property: 5 } }
```

Would desugar to:

```.js
let myVar = {};

let _nv;
(_nv=myVar=myVar||{}) && (_nv=_nv.some=_nv.some||{}) && (_nv.property=5);
```
