Units of Measure: Proposal for TypeScript
=========================================

[![experimental](http://badges.github.io/stability-badges/dist/experimental.svg)](http://github.com/badges/stability-badges)

## Overview

Units of measure is a useful [F# feature](http://msdn.microsoft.com/en-us/library/dd233243.aspx) that provides the optional ability to create tighter constraints on floating point and signed integer values.

TypeScript could benefit from a similar units of measure feature. Such a feature would add zero runtime overhead, increase type constraints, and help decrease programmer error.

## Defining Units of Measure

Units of measure are defined as such:

```
unit <unit-name> [ = unit ];
```

The optional unit part can be used to define a new unit in terms of previously defined units. 

The `<unit-name>` is unique so there cannot be any other types or variables defined with the same name within the unit's scope. For example, the following would not be valid:

```
var m = 10;

unit m; // error
```

### Example Definitions

```
unit m;
unit s;
unit a = m/s^2;
```

Note: The caret symbol does not denote a bitwise XOR operator, but rather an exponent. In this case, `m/s^2` is equivalent to `m/s/s`.

### Circular Definitions

Circular definitions are NOT allowed. For example:

```typescript
unit a = b / c;
unit b = a * c; // error
unit c = b / a; // error
```

## Use with Number

Units of measure can be defined on a number type in any of the following ways:

```
unit m;

// 1. Explicitly
var distance : number<m> = 12<m>;
// 2. Implictly
var distance = 12<m>;
// 3. Using Number class (can be done implicitly or explicitly)
var distance = new Number(10)<s>;
```

## Detailed Full Example

```
unit m;
unit s;
unit a = m/s^2;

var acceleration = 12<a>,
    time         = 10<s>;

var distance = 1/2 * acceleration * time * time; // valid -- implicitly typed to number<m>
var avgSpeed = distance / time;                  // valid -- implicitly typed to number<m/s>

time += 5<s>;     // valid
time += 5;        // error -- cannot convert number to number<s>
time += distance; // error -- cannot convert number<m> to number<s>

acceleration += 12<m/s^2>;         // valid
acceleration += 10<a>;             // valid
acceleration += 12<m/s^2> * 10<s>; // error -- cannot convert number<m/s> to number<a>
```

### Use With Non-Unit of Measure Number Types

Sometimes previously written code or external libraries will return number types without a unit of measure. In these cases, it is useful to allow the programmer to specify the unit like so:

```
unit s;

var time = 3<s>;
    
time += MyOldLibrary.getSeconds();    // error -- cannot add number to number<s>
time += MyOldLibrary.getSeconds()<s>; // valid
```

## Dimensionless Unit

A dimensionless unit is a unit of measure defined as `number<1>`.

```
var ratio = 10<s> / 20<s>, // implicitly typed to number<1>
    time : number<s>;

time = 2<s> * ratio;
time *= ratio;
time = 2<s> + ratio; // error, cannot add number<1> to number<s>
time = ratio;        // error, cannot assign number<1> to number<s>
```

## Modules

Units of measure can be defined on the module level and exported like such:

```
module MyModule {
    export unit m;
}
```

Then used in other parts of the application by writing:

```
unit m = MyModule.m;
```

Such a feature would handle conflicts well as a developer can rename a unit of measure by writing:

```
unit meter = MyModule.m;
```

## Definition File

Units of measure can be defined in TypeScript definition files ( `.d.ts`) and can be used by any file that references it. Defining units of measure in a defition file is done just the same as defining one in a `.ts` file.

## Compilation

The units of measure feature will not create any runtime overhead. For example:

```
unit cm;
unit m;

var metersToCentimeters = 100<cm/m>,
    length : number<cm> = 20<m> * metersToCentimeters;
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
Math.min(0<s>, 4<m>); // error, cannot mix number<s> with number<m>

var volume = Math.pow(2<m>, 3)<m^3>;
var length = Math.sqrt(4<m^2>)<m>;
```

## Outstanding Questions

As discussed in [this thread](https://github.com/Microsoft/TypeScript/issues/364#issuecomment-51720786).


### 1. How can a function return a new combination of the units of measure used?

Side note: Maybe this question should be thrown out.

For example, how would the `Math.pow` function be defined if we want it to implicitly want to raise the power of a unit. Such as in the case `var area = Math.pow(2<m>, 2);` where area is implicitly typed to `number<m^2>`. Take note that such a case could only occur when passing in a number literal for the power.

```
pow<u>(base : number<u>, pow : number) : number<u ^ pow> {
    // other code here
}
```

Maybe:

```
// times return type is number<u * t>
times<u, t>(firstNumber : number<u>, secondNumber : number<t>) {
    return firstNumber * secondNumber;
}
```

How would you take the nth root of a unit of measure in this case?
