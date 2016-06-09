Units of Measure: Proposal for TypeScript
=========================================

[![experimental](http://badges.github.io/stability-badges/dist/experimental.svg)](http://github.com/badges/stability-badges)

[Read the issue](https://github.com/Microsoft/TypeScript/issues/364)

## Overview

Units of measure is a useful [F# feature](http://msdn.microsoft.com/en-us/library/dd233243.aspx) that provides the optional ability to create tighter constraints on numbers.

TypeScript could benefit from a similar feature that would add **zero runtime overhead**, increase type constraints, and help decrease programmer error when doing mathematical calculations that involve units. The feature should prefer explicity.

## Defining Units of Measure

Units of measure should probably use syntax similar to type aliases (#957). More discussion is needed on this, but for the purpose of this document it will use the following syntax:

```typescript
type measure <name> [ = unit ];
```

The optional unit part can be used to define a new unit in terms of previously defined units. 

### Example Definitions

```typescript
type measure m;
type measure s;
type measure a = m / s**2;
```

Units of measure can be defined in any order. For example, `a` in the example above could have been defined before `m` or `s`.

### Circular Definitions

Circular definitions are NOT allowed. For example:

```typescript
type measure a = b;
type measure b = a; // error
```

## Use with Number

Units of measure can be defined on a number type in any of the following ways:

```typescript
type measure m;

// 1. Explicitly
let distance: number<m> = 12<m>;
// 2. Implictly
let distance = 12<m>;
// 3. Using Number class
let distance = new Number(10)<s>;
```

TODO: Maybe we shouldn't use the `<m>` syntax because it might conflict with jsx files.

## Detailed Full Example

```typescript
type measure m;
type measure s;
type measure a = m / s**2;

let acceleration = 12<a>,
    time         = 10<s>;

let distance = 1/2 * acceleration * (time ** 2); // valid -- implicitly typed to number<m>
let avgSpeed = distance / time;                  // valid -- implicitly typed to number<m/s>

time += 5<s>;         // valid
time += 5;            // error -- cannot convert number to number<s>
time += distance;     // error -- cannot convert number<m> to number<s>

// converting to another unit requires asserting to number then the measure
time += (distance as number)<s>; // valid

acceleration += 12<m / s**2>;         // valid
acceleration += 10<a>;                // valid
acceleration += 12<m / s**2> * 10<s>; // error -- cannot convert number<m/s> to number<a>
```

### Use With Non-Unit of Measure Number Types

Sometimes previously written code or external libraries will return number types without a unit of measure. In these cases, it is useful to allow the programmer to specify the unit like so:

```typescript
type measure s;

let time = 3<s>;
    
time += MyOldLibrary.getSeconds();    // error -- type 'number' is not assignable to type 'number<s>'
time += MyOldLibrary.getSeconds()<s>; // valid
```

## Dimensionless Unit

A dimensionless unit is a unit of measure defined as `number<1>`.

```typescript
let ratio = 10<s> / 20<s>; // implicitly typed to number<1>
let time: number<s>;

time = 2<s> * ratio;         // valid
time = time / ratio;         // valid
time = (ratio as number)<s>; // valid
time = 2<s> + ratio;         // error, cannot assign number<1> to number<s>
time = ratio;                // error, cannot assign number<1> to number<s>
time = ratio<s>;             // error, cannot assert number<1> to number<s>
```

## Scope

Works the same way as `type`. 

## External and Internal Modules

Also works the same way as `type`.

In addition, if an external library has a definition for meters and another external library has a definition for meters then they should be able to be linked together by doing:

```typescript
import {m as mathLibraryMeterType} from "my-math-library";
import {m as mathOtherLibraryMeterType} from "my-other-math-library";
    
type measure m = mathLibraryMeterType | mathOtherLibraryMeterType;
```
    
TODO: The above needs more thought though.

## Definition File

Units of measure can be defined in TypeScript definition files ( `.d.ts`) and can be used by any file that references it. Defining units of measure in a definition file is done just the same as defining one in a `.ts` file.

## Compilation

The units of measure feature will not create any runtime overhead. For example:

```typescript
type measure cm;
type measure m;

let metersToCentimeters = 100<cm / m>;
let length: number<cm> = 20<m> * metersToCentimeters;
```

Compiles to the following JavaScript:

```typescript
var metersToCentimeters = 100;
var length = 20 * metersToCentimeters;
```

## Math Library

Units of measure should work well with the current existing [Math object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math).

Some examples:

```typescript
Math.min(0<s>, 4<m>); // error, cannot mix number<s> with number<m> -- todo: How would this constraint be defined?

let volume = Math.pow(2<m>, 3)<m**3>;
let length = Math.sqrt(4<m^2>)<m>;
```
