# Scala Validation

A small and simple validation framework for Scala.

## Features

* Validates a form of fields.
* Each field can have as many as needed validators in a list.
* Fail-fast per field: Once some of validators of a field failed, the process of form validation proceeds 
  with a validation for a next field in a list, and so on.
* Some basic validators are already implemented (RequiredStringValidator, MinLengthValidator, 
  MaxLengthValidator, SimpleEmailValidator, etc).

## Where I can use it?

When dealing with UI forms like in web applications.  

## Usage

```scala
object Usage extends App {
  
  import com.plameno.validation._
  
  // A data model that we want to validate
  case class Person(
    login: String,
    email: String,
    age: Int
  )

  // Custom validator
  case class MaturePersonValidator(p: Person) extends Validator() {
    override val defaultError = "You should be over 18 years old"
    def isValid = p.age > 18
  }

  // A form that validates our data model Person
  class PersonValidator(p: Person) extends FormValidator {
    val validators = Map(
      "login" -> List(
        RequiredStringValidator(p.login),
        MinLengthValidator(p.login, 2),
        // Below we use a custom error message:
        MaxLengthValidator(p.login, 6, Some("Login is too long"))),
      "email" -> List(
        RequiredStringValidator(p.email), 
        SimpleEmailValidator(p.email)),
      "age" -> List(
        PositiveIntValidator(p.age), 
        MaturePersonValidator(p)))
  }

  val person = Person(login = "123456xx", email = null, age = 17)

  val personForm = new PersonValidator(person)

  // Here the validation performs
  val errors = personForm.validate

  val prettyErrors = errors.map{ case (f, e) => s"  $f: $e;" }.mkString("\n")

  println(s"Validation errors:\n$prettyErrors")
}
```

Output:
```
Validation errors:
  login: Login is too long;
  email: Field is required;
  age: You should be over 18 years old;
```