<span id="_Toc308702327" class="anchor"><span id="_Toc188339609" class="anchor"></span></span>Working with Properties
=====================================================================================================================

<span id="_Toc300926414" class="anchor"><span id="_Toc301261998" class="anchor"></span></span>The examples in this section can be run from the scalacheck-basic folder. In a command prompt window, switch to the scalacheck-basic folder and run the command “*mvn scala:console*”.

<span id="_Toc308702051" class="anchor"><span id="_Toc188339610" class="anchor"></span></span>Defining a simple property
------------------------------------------------------------------------------------------------------------------------

The simplest possible property-based test uses the *forAll* object, with a closure/function as the logic for the check, and uses an existing generator to generate random data (two lists of integers in the example below):

import org.scalacheck.Prop.forAll

val propConcatLists = forAll { (l1: List[Int], l2: List[Int]) =\> l1.size + l2.size == (l1 ::: l2).size }

In order to execute a property test, the *check* method can be used:

propConcatLists.check

When run like this, ScalaCheck will use the default generators in scope (more on this later), and run all the possible inputs, up to 100 by default, through our property check and report whether the property held true for all of them or if any of them was falsified.

All this code can be run from within the Scala console (REPL). When running this code, the following should be visible on the screen:

+ OK, passed 100 tests.

*org.scalacheck.Prop.forAll* takes a function as a parameter, and creates a property that can be tested with the *check* method. The function passed to *forAll* should implement the test/check logic, and should return a Boolean value (defining whether the test was successful or not).

The function provided to *forAll* can receive a parameter of any type as long as there’s a suitable *Arbitrary* instance in the function scope. The Arbitrary type will be described in detail later on in this document but in a nutshell, it is a special wrapper type in ScalaCheck that is used to generate random data. ScalaCheck provides built-in generators for standard data types, but it is also possible to build custom data generators for our own data types (e.g., domain objects from our application). See the chapter below on custom generators and arbitrary types.

It is possible to have more than one property to test the same function, and to combine multiple properties into a single one to check a single function.

<span id="_Toc300926415" class="anchor"><span id="_Toc301261999" class="anchor"><span id="_Toc308702052" class="anchor"><span id="_Toc188339611" class="anchor"></span></span></span></span>Grouping properties
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

While it is possible to define multiple properties as immutable values (Scala’s *val*s) or nullary methods (methods with no parameters), it is usually more convenient to logically group related properties in a single class/object extending the *Properties* class. Definition of property checks when using the *Properties* class is done using the *property* method:

```
import org.scalacheck.Properties
import org.scalacheck.Prop.forAll

object BiggerSpecification extends Properties("A bigger test specification") {
property("testList") = forAll { (l1: List[Int], l2: List[Int]) =\>
  l1.size + l2.size == (l1 ::: l2).size
}

property("Check concatenated string") = forAll { (a:String, b:String) =\>
  a.concat(b) == a + b}
}
```

When using the *Properties* class, all the enclosed properties can be run at once with the *Properties.check* or *Properties.check(params)* methods (depending on whether we want to provide testing parameters or run with default parameters)

```
BiggerSpecification.check
```

Additionally, the Properties class provides the include() method that allows the creation of groups of specifications based on other groups of specifications:

```
object AllSpecifications extends Properties("All my specifications") {
  include(BiggerSpecification)
  include(AnotherSpefication)
  include(...)
}
```

<span id="_Toc300926416" class="anchor"><span id="_Toc301262000" class="anchor"><span id="_Toc308702053" class="anchor"><span id="_Toc188339612" class="anchor"></span></span></span></span>Advanced Property Usage
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### <span id="_Toc300926417" class="anchor"><span id="_Toc301262001" class="anchor"><span id="_Toc308702054" class="anchor"><span id="_Toc188339613" class="anchor"></span></span></span></span>Conditional properties

Often, it’s useful to run our properties for a specific range or subset of data. This can be implemented with a generator that only generates the values that we want or, depending on the kind of subset, properties can be defined to select valid data for their check with a filter condition.

For the former, examples of custom generators will be shown later. For the latter usage with filters, we have to provide a Boolean predicate that upon evaluation to true will run the provided property check. The predicate is linked with the property check using the ==\> method:

```
import org.scalacheck.Prop._

val propMakeList = forAll { (n: Int) =>
  (n >= 0 && n <20000) ==> (List.fill(n)("").length == n)
}
```

As usual, the property above can be run with *propMakeList.check*.

It is important to keep in mind not to be too strict when defining the conditions, as ScalaCheck will eventually give up if only a very low amount of test data fulfills the condition and will report that very little data was available for the property. Compare the following property check with the one above:

```
import org.scalacheck.Prop._
val propMakeListPicky = forAll { n: Int =>
  (n >= 0 && n <1) ==> (List.fill(n)("").length == n)
}
```

When running *propMakeListPicky.check*,tit would produce the following output:

```
! Gave up after only 42 passed tests. 500 tests were discarded.
```

<span id="_Toc300926418" class="anchor"><span id="_Toc301262002" class="anchor"></span></span>This was a rather extreme example but there may be real world situations where this may become an issue. Please note that the number of passed tests will vary from execution to execution, due to the random nature of the input data.

### <span id="_Toc308702055" class="anchor"><span id="_Toc188339614" class="anchor"></span></span>Combining properties

ScalaCheck also provides Boolean operators and methods to combine several different properties into one:

| prop1 && prop2              | Returns a new property that holds if and only if both prop1 and prop2 hold. If one of the properties doesn't generate a result, the new property will generate false                                                            |
|-----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| prop1 || prop2              | Returns a new property that holds if either prop1 or prop2 (or both) hold                                                                                                                                                       |
| prop1 == prop2              | Returns a new property that holds if and only if prop1 holds exactly when prop2 holds. In other words, every input which makes prop1 true also makes prop2 true and every input which makes prop1 false also makes prop2 false. |
| all(prop1, prop2, …, propN) | Combines properties into one which is true if and only if all the properties are true                                                                                                                                           |
| atLeastOne(prop1, …, propN) | Combines properties into one which is true if at least one of the properties is true        