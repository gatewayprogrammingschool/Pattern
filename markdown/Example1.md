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
