Getting Started
===============

<span id="_Toc308702048" class="anchor"><span id="_Toc188339606" class="anchor"><span id="_Toc300926413" class="anchor"></span></span></span>Properties and Generators
----------------------------------------------------------------------------------------------------------------------------------------------------------------------

ScalaCheck is based around the concept of Properties and Generators. According to the official ScalaCheck documentation, a property is “*the testable unit in ScalaCheck”,* while a Generator is defined as *“responsible for generating test data in ScalaCheck”.*

Properties are used to test the output of a specific function while generators are used by properties to generate random data for the logic checked by the property, and the property will confirm whether the property holds true based on the random test data given by the Generator, or if it is falsified.

<span id="_Toc308702049" class="anchor"><span id="_Toc188339607" class="anchor"></span></span>Creating tests with ScalaCheck
----------------------------------------------------------------------------------------------------------------------------

Creating property checks with ScalaCheck requires that we:

-   define the logic for the property check

-   define the data requirements; are the checks going to be performed using basic data types, or will they be performed using custom objects/domain objects from our application?

It is important to know that ScalaCheck properties are evaluated in terms of Boolean logic: either the property holds or the property is falsified (i.e. the code must return to true or false as its result). ScalaCheck does not provide support for assertions so all property checks are generally validated using the equality == operator. The only exception is exceptions (no pun intended) as ScalaCheck provides support to validate a property based on whether it throws an exception or not.

Once the logic for the check has been defined, we need to define the kind of data to be used.

ScalaCheck’s main feature is that it relies on automatic test data generation to ensure that we don’t miss any case when writing our unit tests. In order to do that, it provides mechanisms out-of-the-box to generate random data for basic data types (strings, integers, double-precision numbers, amongst other basic types), as well as containers for those types (lists, sequences, options, amongst others).

If our property checks do not use basic data types, we will have to write our own custom data generators before we can write the property checks. Fortunately, writing custom generators is not a very complex task (and in most cases it is an essential one).

The next two sections will walk us through the process of creating our first ScalaCheck properties.

<span id="_Toc308702050" class="anchor"><span id="_Toc188339608" class="anchor"></span></span>Running the sample snippets
-------------------------------------------------------------------------------------------------------------------------

In addition to the accompanying package with code examples, throughout the content below there are a lot of code snippets that show concepts as they are introduced to the reader. The code example package (see link in Appendix 1) should be deployed to a directory on the local machine to facilitate execution of the snippets below.

Most of the snippets are executable when typed directly in a Scala console. The beginning of each main section or subsection will point to a folder with the relevant source code and other related files.

Please refer to Appendix 1 for more details on the tools required.