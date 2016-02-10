# ECMAScript Proposal: Save Navigation (Elvis) Operator

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

let prop2 = foo && foo.some && typeof foo.some.fun === 'function' && foo.some.fun() || null;

let prop4 = foo && foo['some'] && foo['some']['nested'] && foo['some']['nested']['property'] || null;
```

This works the same way with bracket property accessors:
