# ECMAScript Proposal: Save Navigation (Elvis) Operator

This proposal introduces a new accessor modifier `?`, preceeding property access via a `.` or `[]`. An example of which would be: `foo?.bar` or `foo?['bar']`

### Introduction

The safe navigation operator, also often known as the null propagation operator or the null-conditional Operator in C#, or the Elvis operator due to it's syntax, allows for the safe access of nested object properties which may be null.

Given the following object:

```.js
let foo = {
  some: {
    nested: {
      property: "bar"
    },
    fun: () => "baz",
    fun2: () => { prop: "zig" }
  }
};
```

The operator itself is used as syntactic sugar for null checking as follows:

```.js
let elvis = foo?.some?.nested?.property;
let elvis2 = foo?.some?.fun();
let elvis3 = foo?.some?.fun().?prop;
let elvis4 = foo?['some']?['nested']?['property'];
let elvis5 = foo?['some']?['fun']()?['prop'];
```

Which is equivalent to:

```.js
let prop = foo && foo.some && foo.some.nested && foo.some.nested.property || null;

let prop2 = foo && foo.some && typeof foo.some.fun === 'function' && foo.some.fun() || null;

let _nv3
let prop3 = (_nv3=foo)
  && (_nv3=_nv3.some)
  && (_nv3=_nv3.fun)
  && typeof _nv3 === 'function'
  && (_nv3=_nv3())
  && _nv3.prop || null;

let prop4 = foo && foo['some'] && foo['some']['nested'] && foo['some']['nested']['property'] || null;

let _nv5
let prop5 = (_nv5=foo)
  && (_nv5=_nv5['some'])
  && (_nv5=_nv5['fun'])
  && typeof _nv5 === 'function'
  && (_nv5=nv5())
  && _nv5['prop'] || null;
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
