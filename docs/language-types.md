# The Pattern Language

Pattern presents an opportunity to design a language from the ground up that implements some key principles to aid the developer in rapid software engineering tasks.  Pattern is an expressive language which is intended to reduce the burden of documentation and ease testing of artifacts.

## Types

All data of Pattern is type-safe. This provides the developer the safety and assurance that the compiled code creates artifacts that are well known within the saystem and cannot be forced into places where they don't belong. Pattern provides for implementing problem domains that fence types to a specific related section of the application architecture.

### Primitive Types

All primitives of Pattern are immutable by default.  The `mutable` operator allows for the creation of mutable primitive instances.  All primitives are also real values, which means that the `Number` type does not contain floating point errors as the IEEE floating point standard is not used.

#### Defined Primitives and Primitive Structures

* `Type` - the definition of data.  Types contain a bit specifying if a value is immutable (default is immutable), another bit indicating if the type is a `Reference`, and a string representing the fully qualified notation that desribes the complete type.  All types have a Default value which is a constant stored in memory.
* `Reference` - an extention to `Type` that provides an immutable 64-bit link to some other data whereas the value is the logical location of the type in RAM.  `Reference` also contains a `Bit` that indicates whether the reference (and the type data it is linked with) has been requested to be garbage collected.  A reference can be transferred such that the lifecycle of the linked value is relocated to a different scope.
* `Node` - A collection of four `Reference` values representing a Parent, Left sibling and Right sibling, and the data linked to the `Node`.
* `Pair` - An extension of the `Node` type that provides the addion of a `Reference` value that points to a key.  A `Bag` of `Pair` objects allows for a direct movement of the Current index through searches on the Key values in the `Bag`.  Duplicate Keys are allowed and consecutive searches for a key value result in the movement of the Current index to the next `Pair` with that key.
* `Array` - A linear collection of primitives of the same `Type`.  Arrays are bounded with the first 64-bits of data representing the length of the Array.  When an array value is requested via a numbered index, this value is compared to the length to ensure that the `Array` cannot be referenced outside its range.
* `Bag` - A linear collection of `Pair` values.  A `Bag` is not bounded and is structured as a tree with a specific Root, Leftmost, Rightmost, and Current index.  A `Bag` may be referenced by Key (64-bit value) which does not move the Current index.  `Bag` exposes an enumerable `Sequential` interface which allows the data to be traversed in either direction and provides for skipping data nodes during traversal. Enumerating a `Bag` begins at the left-most `Node` and travels forwards and backward by walking the tree.  
* `Flag` - A one or zero.  `Flag` values can have normal bitwise operations applied to them.
* `Number` - All numeric values.  Numbers are defined by a pair of 64-bit integers and a single `Flag` representing the signedness of the value.  The first 64-bit integer represents the whole portion and the second represents the fractional portion.
* `GUID` - A 128-bit number that may not have mathematical operations and can be compared to a `String` representation of a Guid.
* `Logical` - Decision values.  Boolean logic may be boiled down to true and false, but this is not very expressive and makes for convoluted analysis of data comparisons.  The `Logical` type represents any of the following values...
  * True
  * False
  * Equals
  * NotEquals
  * LessThan
  * NotLessThan
  * GreaterThan
  * NotGreaterThan
  * LessOrEqualThan
  * NotLessOrEqualThan
  * GreaterOrEqualThan
  * NotGreaterOrEqualThan
  * ContainsInclusive
  * DoesNotContainInclusive
  * ContainsExlusive
  * DoesNotContainExclusive
* `Character` - A single UTF-16 character.
* `String` - The representation of textual (`Character`) data. Text is stored as an `Array` of `Character` values.  Text can be acessed via a zero terminated sequence, a length-specified sequence, or a raw array of characters.
* `Date` - The representation of an near-exact point in time.  This type is constructed with the same structure as a number where the whole number represents the number of seconds since January 1 of year zero.  The second 64-bits represent fractional seconds, giving Pattern the ability to represent a very fine point in time.  All values stored in the `Date` type are UTC.
* `Place` - The representaion of a physical location.  The `Place` type provides the following values...
  * Local Time
  * Timezone
  * Planet
  * Continent
  * Country (as defined by [ISO-3166-1](https://en.wikipedia.org/wiki/ISO_3166-1) )
  * EarthLatitude
  * EarthLongitude
  * GpsLocation
* `Culture` - An extension of `Place` that represents the language and cultural representation of `Number` and `Date` values.
* `Enumeration` - A static immutable `Bag` of immutable `EnumNode` values where the key of the Pair is the name of the value being represented by the `Enumeration`.
* `EnumNode` - An extension of `Type` that defines a Key and Value where the Key is always a constant String and the Value may be a constant of any type.

### Patterned Types

Patterned types are types specified by the Pattern language that provide compiler enforced usage of the patterned type.  The types are grouped by their respective design pattern.

* `Domain` - A type that represents the boundary that an object lives within.  This is separate from scope as a `Domain` prevents domain bounded objects from interacting with objects of another Domain.  All objects may be instantiated with a specific `Domain`.
* `Singleton` - A statefull type containing a 'Bag' of 'Bag' objects that is globally scoped.  Derivations of `Singleton` may have methods and properties added.  The methods of a Singleton are always static.
* `Facade` - A stateless type that contains one or more static methods for a specified `Domain`.  The parameters and return types of a method of a `Facade` must be immutable.
* `Model` - A collection of named properties, each representing a specific type.  `Model` instances must be scoped to a `Domain`.
* `View` - A definition of interaction between an actor and a `Controller`.  A `View` may only expose a state `Bag` of `Pair` instances and named properties representing a `Model` or `Bridge`.  `View` types must also define `Triggers` that can be watched by `Controller` instances.  View instances must be scoped to a `Domain`.
* `Controller` - A instantiatable type that contains a `Facade` that provides private static methods that that are invoked by triggers defined on a `View`.  The `Controller` only holds instance data that watches the `Triggers` of a `View`.  A `Contoller` must be scoped to a specific `Domain`.
* `Bridge` - An abstract immutable instance type that bridges data between two `Domains`.  `Bridge` instances provide an provide map that defines the relationship of properties between at least one `Model` of each Domain.
* `Adapter` - An abstract stateless type that exposes `Triggers` that are watched by one or more `DataBridge` instances that are the passive side of the `Adapter`. Triggers may be exposed directly or called from within static methods exposed to the active side of the `Adapter`.  An `Adapter` must be defined for a specific actor `Domain` and watcher `Domain`.
* `Data Store` - An abstract stateful type the holds the data for a specific `Domain`.  The data is stored in an infinitely nestable `Bag` of `Node` instances where the value of each `Node` references either another `Bag` or a `Model` instance.
* `Data Provider` - A stateful abstract type that encapsulates one or more `Data Store` instances and exposes methods that can store, retrieve or delete data within the `Data Store`. The `Data Provider` exposes `Triggers` that are watched by `Data Adapter` object that perform the CRUD operations of the data source.  
* `Data Adapter` - A stateless abstract type that exposes basic CRUD methods to manage persistance of the `Model` instances passed to it.
* `Data Bridge` - A stateless abstract type that provides static methods, each of which are not constrained to a specific `Domain` and represent the only place in an application where `Adapter` instances are accessed.
* `Remote` - A type representing a stateless proxy to a single outside method such as a microservice, REST call, or SOAP service call.  The output (if any) of a `Foriegn Method` must be a `Data Store` instance that may be merged into another `Data Store`.
* `Service` - A statefull type that extends a `View` that listens for requests for data manipulation and persistance.
* `Application` - A stateful representation of the data, data manipulation, and exposure of data to actors (input) and receivers (output).  An application contains an ApplicationState `Singleton` which exposes the public entry points (one or more `Service` instances) into the application.  The `Application` exposes a static methed named `Entry Point` that exposes the traditional __main__ entry point for an `Application`.

Non-Primitive Types in Pattern must extend a Patterned Type and must follow the Domain constraints of the base type.

## Syntax

Pattern is an expressive language which makes documentation and testing very straight forwad.  The syntax is not case sensitive, though a compiler flag may be used to force proper case of keywords.

### Variables

Variables are by default immutable, but may be decalred as mutable using the `Mutable` keyword.

#### Comments

Comments are defined with the standard C syntax of `/*  */` for block comment and `//` for singe-line comments.  Single-line comments may be entered anywhere in a line, but all other text after the beginning of the comment to the end of the line are not compiled.  Block comment may be inserted anywhere and only affect the contents delimited by the block comment tags.

#### Variable Keywords

* `Set` - Declares an immutable variable.
* `Mutable` - Declares a mutable variable.
* `Of` - Specifies the `Type` of the variable.
* `=` - Optionally assigns an initial value to the variable.  If no initial value is defined then the variable's value is default until the value is assigned.  Immutable variables may only have their value set once.
* `Bounded`, `To` - Optionally creates bounds on a set of data.

##### Variable Examples

```Pattern
// Define a number named counter and initialized with 0.
Set position Of Number = 0

// Variables may be initialized by an instance of another variable of the same type.
Set node Of Pair = SomeBag.Current

// Define a mutable number index that is bounded by the numbers from 10 to 20 and initialized with the number 10
Mutable index Of Number Bounded 10 To 20 = 10

/*
Accessing the values of an `Numeration` requires specifying the Enumeration as the Type and the key is inferred from the available keys of the Enumeration.  The actual type of the value is a `EnumNode`.
*/
Mutable action Of SomeEnumeration = FirstValue
```

### Expressions

An expression is a block of any code that generates a single Type instance.  Expressions may be constrained as a method or property accessor, passed a lambda to another expression.  Expressions may not cause side-effects though their execution.

```Pattern
// Define mutable variable of type Numer with its default value.
Mutable x Of Number
// The expression x + 1 is assigned to the mutable variable x
x = x + 1

// Define an immutable Place where the country is US (standard code for the United States)
Set location Of Place (Country = US)
/*
Defines an immutable string who's value is the result of the expression "Hello, " + location.Country.Full
*/
Set message Of String = "Hello, " + location.Country.Full

/*
Expressions may be chained.  Here the instantiation of an immutable Place with the country code for Canada is chained by an expression retrieving the short-code of the country which is then evaluated as the concatenation of that expression's value to "Hello, " resulting in message having been set to "Hello, CA"
*/
Set message of String = "Hello, " + Place (Country = CA).Country.Short
```

### Operators

Being an expressive language, Pattern allows for operators that have two keywords each, one short for and the our descriptive.

|Boolean Operation|Short Operator|Long Operator|Result Type|
|-----------------|:------------:|:-----------:|-----------|
|Equality|==|Is|Flag|
|Logical And|&&|And|Flag|
|Logical Or|\|\||Or|Flag|
|Logical Not|!|Not|Flag|
|Exclusive Or|^|Xor|Flag|

|Assignment Operation|Short Operator|Long Operator|Result Type|
|--------------------|:------------:|:-----------:|-----------|
|Direct Assignment|=|From|Type|
|And|&|n/a|Number, EnumerationNode|
|Or|\|n/a|Number, EnumerationNode|

|Math Operation|Short Operator|Long Operator|Result Type|
|--------------|:------------:|:-----------:|-----------|
|Addition|+|Sum(##,##[,##]+)|Number|
|Subtraction|-|Minus|Number|
|Multiplication|*|Multiply(##,##[,##])|Number|
|Division|/|DivideBy|Number|
|Remainder|%|Remainder|Number|
|Power|^|PowerOf|Number|
|Root|#|Root|Number|

|String Operation|Short Operator|Long Operator|Result Type|
|----------------|:------------:|:-----------:|-----------|
|Concatenation|++|Concat-With[-With]|String|
|Remova All|--|Remove All|String|
|Remove First|-<|Remove First|String|
|Remove Last|->|Remove Last|String|
|Remove Set|-(Array(String))|Remove Set|Array|
|Replace All|!!|Replace All|String|
|Replace First|!<|Replace First|String|
|Replace Last|!>|Replace Last|String|
|Replace Set|!(Array(String))|Replace Set|Array|
|Substring|((#[,#]))|Substring(#[,#])|String|
|First Index Of|<<|First|Number|
|Last Index Of|>>|Last|Number|
|All Indexes Of|><|Indexes|Array|

### Properties

Objects may store and expose zero or more scalar values. The values typically will return an internal value or expression, but may also expose a lambda to a method of the object.  Properties are defined with the `Property` and `To` keywords. Properties may not expose mutable data.

```Pattern
// Exposes the field _x for reading.
Property X To Me._x

// Exposes a lambda MyMethod
Property L To MyMethod

// Exposes the invocation of MyOtherMethod which must be parameter-less and non-Public
Property R To MyOtherMethod()
```

### Methods

Methods create a specific scope of work to be completed and must return a value.  If the work does not produce a result then the returned value is the object being operated on.  The body of a method must be indented by a minimum of 4 spaces.  The first line of code that is not indented (including comments) ends the method's scope. If then `Return` keyword is not present then the compiler will generate an error.

```Pattern
// Declares MyMethod which accepts a Number and Strin and returns a String.
String MyMethod param1 Of string, param2 Of String
    Return param1 ReplaceFirst param2 ReplaceLast param2

// Declares a parameterless method named Bump that returns itself.
Me Bump
    _x = _x Add 1
    Return
```

### Accessibility

Each Method and Property may be `Public`, `Private`, or `Visible`.

#### Public

Exposes an Method, or Property to all code in the defined domain.

```pattern
// Defines a public method NextValue that returns a Number
Public Number NextValue
    Return _myBag.Next.Value

// Defines a public property X that returns the value of _x
Public Property X To _x
```

#### Visible

Exposes a Property or Method only to sub-types of the object.

```pattern
// Defines a property Y that provides the value of the expression _z Multiply 2
Visible Property Y To _z Multiply 2

// Defines a method named BumpBy that accepts a Number and returns a Number
Visible Number BumpBy Factor To Number
    _x = _x Add Factor
    Return _x
```

#### Private

Hides a Property or Method from outside code.

```pattern

// Declares a private property X that returns the result of _a + _offset.
Private Property X to _a Add _offset

// Declares a private method Bump that returns a Number
Private Number Bump
    _x = _x + 1
    Return _x
```

### Objects

Objects in Pattern are specializations of any Type, including Primitive Types, for a specific `Domain`. and are declared using the `In`, `Object` and `End Object` keywords.

```pattern

// Defines an implementation of Node in MyDomain named MyObject.
Object MyObject Of Node In MyDomain

// Declares a Field named _x of type Number Initialized to 1
Mutable _x Of Number = 1

// Defines method Bump that returns this instance
Public Me Bump
    _x = _x + 1
    Return

// Defines a Visible method that resets the _x field to 1.
Visible Me Reset
    _x = 1
    Return

Visible Me SetX Val as Number
    _x = Val
    Return

// Defines a Public Property Current thar returns the value of _x
Public Property Current To _x

End Object

```

### Abstractions

All Patterned Types are intended to be implemented directly, but those implementations can be further specialized Abstractions.

Abstractions can `Specialize` `Public` or `Visible` methods or properties of the base class using the `Specialize` keyword.  Specialized Methods or Properties may not be more accessible than their base-class's definition.  

Abstractions can `Replaces` `Visible` Methods or Properties of the base class using the `Replace` keyword which makes them Public.

```pattern

Object MyAbstraction Of MyObject

    Replaces Reset
      SetX 1
      Return

End Object

```