Units of Measure: Proposal for TypeScript
=========================================

[![experimental](http://badges.github.io/stability-badges/dist/experimental.svg)](http://github.com/badges/stability-badges)

[Read the issue](https://github.com/Microsoft/TypeScript/issues/364)

## Overview

Units of measure is a useful [F# feature](http://msdn.microsoft.com/en-us/library/dd233243.aspx) that provides the optional ability to create tighter constraints on numbers.

TypeScript could benefit from a similar feature that would add **zero runtime overhead**, increase type constraints, and help decrease programmer error when doing mathematical calculations that involve units. The feature should prefer explicity.

## Defining Units of Measure

Units of measure should probably use syntax similar to type aliases (#957). More discussion is needed on this, but for the purpose of this document it will use the following syntax:

```
type <unit-name> [ = unit ];
```

The optional unit part can be used to define a new unit in terms of previously defined units. 

### Example Definitions

```
type m;
type s;
type a = m / s**2;
```

Units of measure can be defined in any order. For example, `a` in the example above could have been defined before `m` or `s`.

### Circular Definitions

Circular definitions are NOT allowed. For example:

```typescript
type a = b;
type b = a; // error
```

## Use with Number

Units of measure can be defined on a number type in any of the following ways:

```
type m;

// 1. Explicitly
let distance: number<m> = 12<m>;
// 2. Implictly
let distance = 12<m>;
// 3. Using Number class
let distance = new Number(10)<s>;
```

TODO: Maybe we shouldn't use the `<m>` syntax because it might conflict with jsx files. Additionally, this current syntax seems weird to be doing `type m` then later `number<m>`. We really need to think this out more.

## Detailed Full Example

```
type m;
type s;
type a = m / s**2;

let acceleration = 12<a>,
    time         = 10<s>;

let distance = 1/2 * acceleration * (time ** 2); // valid -- implicitly typed to number<m>
let avgSpeed = distance / time;                  // valid -- implicitly typed to number<m/s>

time += 5<s>;         // valid
time += 5;            // error -- cannot convert number to number<s>
time += distance;     // error -- cannot convert number<m> to number<s>

// Convert to another unit. The following should be thought out more:
time += (distance as number)<s>;  // valid

acceleration += 12<m / s**2>;         // valid
acceleration += 10<a>;             // valid
acceleration += 12<m / s**2> * 10<s>; // error -- cannot convert number<m/s> to number<a>
```

### Use With Non-Unit of Measure Number Types

Sometimes previously written code or external libraries will return number types without a unit of measure. In these cases, it is useful to allow the programmer to specify the unit like so:

```
type s;

let time = 3<s>;
    
time += MyOldLibrary.getSeconds();    // error -- cannot add number to number<s>
time += MyOldLibrary.getSeconds()<s>; // valid
```

## Dimensionless Unit

A dimensionless unit is a unit of measure defined as `number<1>`.

```
let ratio = 10<s> / 20<s>, // implicitly typed to number<1>
    time: number<s>;

time = 2<s> * ratio;
time *= ratio;
time = 2<s> + ratio; // error, cannot add number<1> to number<s>
time = ratio;        // error, cannot assign number<1> to number<s>
```

## Scope

Works the same way as `type`. 

## External and Internal Modules

Also works the same way as `type`.

TODO: Consider linking multiple units of measure together. For example, if an external library has a definition for meters and another external library has a definition for meters, then consider a way of linking these two definitions together.

## Definition File

Units of measure can be defined in TypeScript definition files ( `.d.ts`) and can be used by any file that references it. Defining units of measure in a definition file is done just the same as defining one in a `.ts` file.

## Compilation

The units of measure feature will not create any runtime overhead. For example:

```
type cm;
type m;

let metersToCentimeters = 100<cm / m>,
    length: number<cm> = 20<m> * metersToCentimeters;
```

Compiles to the following JavaScript:

```javascript
var metersToCentimeters = 100,
    length = 20 * metersToCentimeters;
```

## Math Library

Units of measure should work well with the current existing [Math object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math).

Some examples:

```
Math.min(0<s>, 4<m>); // error, cannot mix number<s> with number<m> -- todo: How would this constraint be defined?

let volume = Math.pow(2<m>, 3)<m**3>;
let length = Math.sqrt(4<m^2>)<m>;
```
