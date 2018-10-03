# The Pattern Language

## Logic and Decisions

Almost all programs are written to solve some problem that can be expressed as one or more choices that the program must make a decision about.  Decisions are deconstructed in to one or more expressions that ultimately result in a boolean value.  Always forcing the developer to reduce a value all the way down to a single boolean value introduces a lot o complexity and redundancy.  Pattern's expressiveness aims to simplify decision and make the code more readable.

In addtion to the traditional `If`-`Else` statements, Pattern introduces the `Compare`-`With`-`When` statement that allows for direct testing of a pair of like-typed values which provides a grouping of possible actions to be taken based on the logical disposition of the values.

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
            result = MyClass(Args = Arguments).Initialize
        Handle RangeError
            result = "Initialization resulted in a RangeError {LastError.Message}"
        Handle Error
            result = LastError.Message

    Leave Domain

    return LastError.Set As Number

End Application

Object MyObject Type In MyDomain

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
        Other
            Throw Error(Message = "Too many arguments!")
    End Choice

End Object