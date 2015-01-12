Common problem scenarios
====================================

The following are some common issues that may appear during the development and execution of ScalaCheck properties.

<span id="_Toc308702096" class="anchor"><span id="_Toc188339657" class="anchor"></span></span>Property check function does not evaluate to true or false
--------------------------------------------------------------------------------------------------------------------------------------------------------

This error will manifest during compile time with the following error message:

[ERROR] /.../src/test/scala/TestFile.scala:27: error: No implicit view available from Unit =\>org.scalacheck.Prop.

The actual type causing the error may differ (it could be Long, String, etc =\>org.scalacheck.Prop.), but what the error message indicates is that our property check is not evaluating in terms of Boolean logic. Property checks must always return true or false and, if they do not comply with that requirement, the compiler will emit this error. In the specific example used above, the function that was being tested produced no result, hence Unit (Scala’s equivalent of *void*) was being shown as the type.

<span id="_Toc308702097" class="anchor"><span id="_Toc188339658" class="anchor"></span></span>Current Scala version and ScalaCheck’s version do not match
---------------------------------------------------------------------------------------------------------------------------------------------------------

If in our preferred build tool we have mismatched the version of ScalaCheck with our version of Scala, the compiler may generate strange and unrelated error messages. Especially when using Maven for building, it is important that we provide the correct name for the ScalaCheck dependency including the version of Scala. This is not an issue when using SBT as it is capable of figuring out the right package version.

<span id="_Toc308702098" class="anchor"><span id="_Toc188339659" class="anchor"></span></span>The condition for the property is too strict
------------------------------------------------------------------------------------------------------------------------------------------

During property execution, if we have defined a condition for the execution of our property check with the ==\> operator and the condition forces ScalaCheck to discard too much data, it will eventually give up and mark the property check as incomplete.

Such cases will be visible in the console as part of ScalaCheck’s output:

! Gave up after only 42 passed tests. 500 tests were discarded.

In these scenarios, the only possibility is to redesign our property check so that we can use a less strict condition, or create a tailored generator that will only provide the data that we really need.