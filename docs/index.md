# Pattern

Pattern is a Dotnet Core based language to provide a modern approach to system development and architecture based on standard design patterns.

## Philosophy

for decades, Object Oriented design principles have provided the strongest foundation for designing large applications.  Object Oriented provides the ability to not only [encapsulate](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming)) code, but also extend code in ways that provide for [strong cohesion](https://en.wikipedia.org/wiki/Cohesion_(computer_science)) of objects and domains while [loosely coupling](https://en.wikipedia.org/wiki/Loose_coupling) implementations from their sibling implementations.  Pattern strives to provide an expressive language that standardizes how many OOP (Object Oriented Programming) patterns are defined and _implemented_.

## Modeled Patterns

The design patterns to be structured within the Pattern language include (but will not be limited to)...

* [Singleton](https://en.wikipedia.org/wiki/Singleton_pattern)
* [MVC (Model-View-Controller)](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)
* [Facade](https://en.wikipedia.org/wiki/Facade_pattern)
* [Adapter](https://en.wikipedia.org/wiki/Adapter_pattern)

<!-- include (language-logic.md) -->

## Logic and Decisions

Almost all programs are written to solve some problem that can be expressed as one or more choices that the program must make a decision about.  Decisions are deconstructed in to one or more expressions that ultimately result in a boolean value.  Always forcing the developer to reduce a value all the way down to a single boolean value introduces a lot o complexity and redundancy.  Pattern's expressiveness aims to simplify decision and make the code more readable.

In addition to the traditional `If`-`Else` statements, Pattern introduces the `Compare`-`With`-`When` statement that allows for direct testing of a pair of like-typed values which provides a grouping of possible actions to be taken based on the logical disposition of the values.

Finally, Pattern provides `Choice`-`When` statement which works by testing a value against a possible set of values and allowing a specific action to be taken if the condition is met.

```pattern
// Defines a public method named DescribeRelationship that
// determines if the parameter y is in a boundary,
// returning a String expressing the relationship.
Public String DescribeRelationship y Of Number
    // Local variable to hold our result.
    Set msg Of String

    // Local variables defining the Range
    Set x Of Number = 10
    Set z Of Number = 15

    // Gets the logical relationship between y and the bounding
    // pair of x and z.
    Compare y With x To z
        When ContainsInclusive
            msg = "y is contained between {x} and {z}"
        When LessThan
            msg = "y is less than the lower boundary {x}"
        When GreaterThan
            msg = "y is greater than the upper boundary {z}"
        Other
            msg = "No other comparisons matched."
        Error
            msg = "Error: {LastError} occured."
            // Error block are good candidates to escape the scope
            // by rethrowing them.
            Rethrow
    End Compare

    return msg
```

In the `Compare`-`With`-`To` constraint, the first boundary must be less than or equal to the second boundary.  If not then an error condition exists and should be handled.  When an `Error` is Thrown or Rethrown, it must be handled or it will short circuit each caller until it is handled by an `Error Scope`.

```pattern

Application MyApp

Public Number EntryPoint Arguments Of Array(String)

    Set result Of String

    In Domain MyDomain

        Error Scope
            result = ApplicationState.Initialize
        Handle RangeError
            result = "Initialization resulted in a RangeError {LastError.Message}"
        Handle Error
            result = LastError.Message

    Leave Domain

    return LastError.Set As Number


ApplicationState

Set _args Of Array(String)

Public Property Args To SetArgs

Private Me SetArgs args Of Array(string)
    _args = args
    Return

Public String Initialize
    Choice _args.Length
        When 0
            Throw RangeError(Message = "Args cannot be empty.")
        When 1
            // Do something
            Return _args(0)
        Otherwise
            Throw Error(Message = "Too many arguments!")
    End Choice

End ApplicationState

End Application<!-- /include -->
<!-- include (language-types.md) -->
# The Pattern Language

Pattern presents an opportunity to design a language from the ground up that implements some key principles to aid the developer in rapid software engineering tasks.  Pattern is an expressive language which is intended to reduce the burden of documentation and ease testing of artifacts.

## Types

All data of Pattern is type-safe. This provides the developer the safety and assurance that the compiled code creates artifacts that are well known within the saystem and cannot be forced into places where they don't belong. Pattern provides for implementing problem domains that fence types to a specific related section of the application architecture.

### Primitive Types

All primitives of Pattern are immutable by default.  The `mutable` operator allows for the creation of mutable primitive instances.  All primitives are also real values, which means that the `Number` type does not contain floating point errors as the IEEE floating point standard is not used.

#### Defined Primitives and Primitive Structures

* `Type` - the definition of data.  Types contain a bit specifying if a value is immutable (default is immutable), another bit indicating if the type is a `Reference`, and a string representing the fully qualified notation that describes the complete type.  All types have a Default value which is a constant stored in memory.
* `Reference` - an extension to `Type` that provides an immutable 64-bit link to some other data whereas the value is the logical location of the type in RAM.  `Reference` also contains a `Bit` that indicates whether the reference (and the type data it is linked with) has been requested to be garbage collected.  A reference can be transferred such that the lifecycle of the linked value is relocated to a different scope.
* `Node` - A collection of four `Reference` values representing a Parent, Left sibling and Right sibling, and the data linked to the `Node`.
* `Pair` - An extension of the `Node` type that adds a `Reference` value that points to a key.  A `Bag` of `Pair` objects allows for a direct movement of the Current index through searches on the Key values in the `Bag`.  Duplicate Keys are allowed and consecutive searches for a key value result in the movement of the Current index to the next `Pair` with that key.
* `Array` - A linear collection of `Reference` instances of the same `Type`.  Arrays are bounded with the first 64-bits of data representing the length of the Array.  When an array value is requested via a numbered index, this value is compared to the length to ensure that the `Array` cannot be referenced outside its range.  Arrays may be grown by a specified factor and accessed like a Stack or Queue in addition to direct indexing.
* `Bag` - A linear collection of `Pair` values.  A `Bag` is not bounded and is structured as a tree with a specific Root, Leftmost, Rightmost, and Current index.  A `Bag` may be referenced by Key (64-bit value) which does not move the Current index.  `Bag` exposes an enumerable `Sequential` interface which allows the data to be traversed in either direction and provides for skipping data nodes during traversal. Enumerating a `Bag` begins at the left-most `Node` and travels forwards and backward by walking the tree.
* `Flag` - A one or zero.  `Flag` values can have normal bitwise operations applied to them.
* `Number` - All numeric values.  Numbers are defined by a pair of 64-bit integers and a single `Flag` representing the signed-ness of the value.  The first 64-bit integer represents the whole portion and the second represents the fractional portion.  Numbers can have a range of valid values assigned.  Attempts to set the value outside of the bounds of the range results in an `Error`.
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
  * ContainsExclusive
  * DoesNotContainExclusive
* `Character` - A single UTF-16 character.
* `String` - The representation of textual (`Character`) data. Text is stored as an `Array` of `Character` values.  Text can be accessed via a zero terminated sequence, a length-specified sequence, or a raw array of characters.
* `Blob` - The representation of an `Array` of 8-bit values.
* `Uri` - The representation of a Universal Resource Identifier.  A Uri can be constructed to represent the path to any data within an Application in addition to defining access to data outside of the Application.
* `Date` - The representation of an near-exact point in time.  This type is constructed with the same structure as a number where the whole number represents the number of seconds since January 1 of year zero.  The second 64-bits represent fractional seconds, giving Pattern the ability to represent a very fine point in time.  All values stored in the `Date` type are UTC.
* `Place` - The representation of a physical location.  The `Place` type provides the following values...
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

* `Domain` - A type that represents the boundary that an object lives within.  This is separate from scope as a `Domain` prevents domain bounded objects from interacting with objects of another Domain.  All objects may be instantiated within a specific `Domain`.
* `Singleton` - A state-full type containing a 'Bag' of 'Bag' objects that is globally scoped.  Derivations of `Singleton` may have methods and properties added.  The methods of a Singleton are always static.
* `Facade` - A stateless type that contains one or more static methods for a specified `Domain`.  The parameters and return types of a method of a `Facade` must be immutable.
* `Model` - A collection of named properties, each representing a specific simple type, another `Model`, or an Array of `Model` instances.  `Model` instances must be scoped to a `Domain`.
* `View` - A definition of interaction between an actor and a `Controller`.  A `View` may only expose a state `Bag` of `Pair` instances and named properties representing a `Model` or `Bridge`.  `View` types must also define `Triggers` that can be watched by `Controller` instances.  View instances must be scoped to a `Domain`.
* `Controller` - A instantiatable type that contains a `Facade` that provides private static methods that that are invoked by triggers defined on a `View`.  The `Controller` only holds instance data that watches the `Triggers` of a `View`.  A `Contoller` must be scoped to a specific `Domain`.  All data passed between a View and a Controller must be a Model.  Controllers are responsible for implementing the business rules that translate View data to and from Documents.
* `Bridge` - An abstract immutable instance type that bridges Models between two `Domains`.  `Bridge` instances provide a map that defines the relationship of properties between at least one `Model` of each Domain.
* `Adapter` - An abstract stateless type that exposes `Triggers` that are watched by one or more `DataBridge` instances that are the passive side of the `Adapter`. Triggers may be exposed directly or called from within static methods exposed to the active side of the `Adapter`.  An `Adapter` must be defined for a specific actor `Domain` and watcher `Domain`.  All `Model` instances returned from a `Bridge` are deep copies of the original `Model` of the watcher `Domain`.  Models returning to the original `Domain` are mediated in the `Bridge`.
* `Actor` - A representation of a single user, service or other actor performing an operation within the application.  `Actor` definitions specify the application-specific properties of actors that may be read and/or set within the application.
* `Permission Group` - A named collection of `Actor` definitions that can be used within methods of the Application to allow or deny actions for the members of the `Permission Group`.
* `Document` - A representation of a single collection of related data.  A `Document` must have a list of one or more `Data Row` definitions.  Each `Data Row` may have zero or more instances within a document.  Documents containing more than one Data Row definition must have one or more `Data Relationship` definition leading to other Data Row definitions such that instances of the Data Rows can be manipulated relationally.  A Document must have a `Document Key` definition that is used to locate instances of a `Document`.  An instance of a `Document` can only represent one instance of the top-most data in the `Document`.  Collections of Documents are accessed through the `Document Provider`.  Each Document has a `Document Number` property of type Guid that uniquely identifies the specific instance of the `Document` in memory.  Additionally, each `Document` may contain zero or more `Data Index` definitions containing `Data Field` definitions that represent the specific instance of the data in the `Document`.  Upon submitting changes to a document, each change is tracked by pushing a time-stamped collection of field state changes to a stack.  Changes can be undone by popping the changes from the stack and applying the data back to the `Document`.  This allows for transactional behavior as well as point-in-time state retrieval, which creates a copy of the document (with a new `Document Number`) representing the data at the requested previous point-in-time.  Any change that would violate the security for any data of the `Document` will be rejected with an error specifying all violations.
* `Document Store` - An abstract stateful type that holds one or more `Document` definitions and their instances arranged in an `Array` for each `Document` definition.  Indexes for each `Array` of `Document` instances are stored in `Pair` `Bag` instances that are balanced as a sorted binary tree for the fastest retrieval of data from the `Document Store`.  `Document` instances leaving the `Document Store` are deep copies of the requested `Document`.  When a `Document` enters the `Document Store`, any existing `Document` instances are merged with the changed data which is time-tracked in the incoming `Document` instance.
* `Document Provider` - A stateful abstract type that encapsulates one or more `Document Store` instances and exposes methods that can store, retrieve or delete data within those `Document Store` instances. The `Document Provider` exposes `Triggers` that are watched by `Data Bridge` instances that perform the CRUD operations of the data source.  The `Document Provider` receives all requests for `Document` instances and is responsible for the retrieval, updating and deletion of `Document` instances within `Document Store`.  When the `Document Store` does not contain a requested `Document` instance then the `Document Provider` makes a request via `Trigger` to retrieve an existing `Document` or optionally create a new `Document` instance.  Whenever the `Document Provider` updates any data within the `Data Store` it calls one or more `Trigger` definitions to persist the affected `Document` instances.
* `Data Field` - The definition of a single element of data.  A Data Field specifies the Type of the data (`Flag`, `Number`, `Guid`, `String`, `Date` or `Blob`). A Data Field has properties specifying its default value, range and a flag specifying whether or not the Data Field must remain encrypted when provided as a result of a `Service` call.  Data security is applied at the `Data Field` level.  Each `Data Field` must define at least one `Permission Group` with rights to perform Create, Read, Update and Delete activities on the specified field within a `Document`.
* `Data Record` - The collection of one or more `Data Field` and `Data Index` definitions that define a single `Document` of data.
* `Data Index` - The collection of one or more `Data Field` definitions that define a queryable set of data within the `Document`.
* `Data Relationship` - The definition of the relationship between two `Data Record` definitions.  A `Data Relationship` specifies one or more `Data Field` definitions common to the parent and child `Data Record` definitions specifying the relationship.  A `Data Relationship` may contain zero or more Trigger definitions that are used by the Document Provider to perform operations when specific actions
* `Data Adapter` - A stateless abstract type that exposes basic CRUD methods to manage persistence of the `Document` instances passed to it from `Data Bridge` `Trigger` listeners.
* `Data Bridge` - A stateless abstract type that provides static methods that listen for Trigger calls to perform data access. Each `Data Bridge` is not constrained to a specific `Domain` and represents the only place in an `Application` where `Data Adapter` instances are accessed.
* `Client` - A type representing a stateless proxy to a single outside method such as a microservice, REST call, or SOAP service call.  The output (if any) of a `Client` must be a `Document Store` instance that may be merged into another `Document Store`.
* `Service` - A state-full type that extends a `View` that listens for requests for data manipulation and persistence.
* `Application` - A stateful representation of the data, data manipulation, and exposure of data to actors (input) and receivers (output).  An application contains an ApplicationState `Singleton` which exposes the public entry points (one or more `Service` instances) into the application.  The `Application` exposes a single constructor that exposes the traditional __main__ entry point for an `Application`.

Non-Primitive Types in Pattern must extend a Patterned Type and must follow the Domain constraints of the base type.  All data in motion or at rest is stored in instance of a Model.  Data is encrypted in memory and an authorization token must be presented to the `Document Provider` on any access to data.  transit with the only exception being when the receiver data is incapable of interpreting the encryption.  In the case , the data must be returned in a serialized form such as BSON, JSON or XML.

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
<!-- /include -->

<!-- include (Example1.md) -->
# Example 1

```pattern

Application Scope My::App

Needs Pattern

Application MyApp
  Register PeopleSet In Default

  Start
    Begin PeopleSet
  End Start

End Application

Injection Set PeopleSet
  Register PersonController In Injection Domain
  Register PeopleService Controlled By PersonController
End PeopleSet

Model Person
  Constant PersonID Of Guid
  Mutable FirstName Of String
  Mutable LastName Of String

  Property Name
    From
      Return "{LastName}, {FirstName}"
    End From
  End Name
End Person

View PersonEditorDef
  Trigger GetPerson Of Person
    Parameter personID Of Guid
  End GetPerson

  Trigger GetPersons Of Array(Person)
    Parameter criteria Of String
  End GetPersons

  Trigger SavePersons Of Number
    Parameter persons Of Array(Person)
  End SavePersons

  Method SendStatus Of String
    Parameter status Of String
  End SendStatus
End PersonEditorDef

Service PersonService Of PersonEditorDef
  Default Of Array(Person) Via Get
    Parameter searchCriteria Of String
    Return GetPersons searchCriteria
  End Default

  GetPerson Of Person Via Get
    Parameter personUid Of Guid
    Return GetPerson personUid
  End GetPerson

  UpdatePersons Of Number Via Post
    Parameter persons Of Array(Person)
    Return SavePersons persons
  End UpdatePersons

  Outbound SendStatus Of String
    Parameter status Of String
    Return "Controller indicates {status}"
  End SendStatus
End PersonEditor

Controller PersonController
  Add View PEF Of PersonEditorDef

  PEF.GetPersons
    Parameter view Of PersonEditorDef
    Parameter search Of String

    Constant status Of String

    Error Scope
      Constant result Of Array(Person) =
        0: (ID = Guid, FirstName = "John", LastName = "Snow")
        1: (ID = Guid, FirstName = "Jaimie", LastName = "Lanister")
        2: (ID = Guid, FirstName = "Arya", LastName = "Stark")
      End result

      status = "GoT Characters Loaded."

      Return result
    Handle Error
      status = Error.Message
      Return Array(Person)
    Always
      view.SendStatus status
  End PEF.GetPersons
End PersonController
```
<!-- /include -->
