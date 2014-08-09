Units of Measure: Proposal for TypeScript
=========================================

[![experimental](http://badges.github.io/stability-badges/dist/experimental.svg)](http://github.com/badges/stability-badges)

## Overview

Units of measure is a useful [F# feature](http://msdn.microsoft.com/en-us/library/dd233243.aspx) that provides the optional ability to create tighter constraints on floating point and signed integer values.

TypeScript could benefit from a similar units of measure feature and expand to include tiny type support for `string`, `boolean`, and `date` objects. Such a feature would add zero runtime overhead, increase type constraints, and help decrease programmer error.

## Defining Type Annotations

Type annotations are defined similarly to ambient types:

```typescript
declare type <unit-name> [ = measure ];
```

The optional measure part can be used to define a new type in terms of previously defined types. Note that this composite type cannot be used with non-`number` types.

**Examples:**

```typescript
declare type m;
declare type s;
declare type a = m/s^2;
declare type email;
```

Note: The caret symbol does not denote a bitwise XOR operator, but rather an exponent. In this case, `m/s^2` is equivalent to `m/s/s`.

## Use with Number

```typescript
declare type m;
declare type s;
declare type a = m/s^2;

var acceleration = 12<a>,
    time         = 10<s>;

var distance = 1/2 * acceleration * time * time; // valid -- implicitly typed to number<m>
var avgSpeed = distance / time;                  // valid -- implicitly typed to number<m/s>

time += 5<s>;     // valid
time += 5;        // compile error -- Cannot convert number to number<s>
time += distance; // compile error -- Cannot convert number<m> to number<s>

acceleration += 12<m/s^2>;         // valid
acceleration += 10<a>;             // valid
acceleration += 12<m/s^2> * 10<s>; // compile error -- Cannot convert number<m/s> to number<a>
```

## Use with String, Boolean, and Date

```typescript
declare type email;

function sendEmail(email: string<email>, message : string) {
    // send the email in here
}

var myEmail = "david@email.com"<email>;
sendEmail(myEmail, "Hello!");           // valid
sendEmail("some string", "Hello!");     // compile error -- Cannot convert string to string<email>
```

The following is invalid with non-number types:

```typescript
declare type m;
declare type s;
declare type a = m/s^2;

var myString : string<a>;   // compile error -- Cannot use composite types with non-number types
```

## Additional Cases

```typescript
declare type m;
declare type email;
declare type startDate;
declare type flag;

var num    = new Number<m>(),
    str    = new String<email>(),
    date   = new Date<startDate>(),
    b      = new Boolean<flag>();
```

## Supported Types

* String
* Number
* Boolean
* Date

## Additional Considerations

It might be beneficial to have a type constraint on these defined types. For example:

```typescript
declare type email : string;
```
