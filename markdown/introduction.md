Introduction
============

<span id="_Toc301261996" class="anchor"><span id="_Toc308702044" class="anchor"><span id="_Toc188339600" class="anchor"></span></span></span>Overview
-----------------------------------------------------------------------------------------------------------------------------------------------------

The purpose of this document is to introduce Scala and Java developers to ScalaCheck, how to build test cases and how to integrate ScalaCheck cases with existing unit test case suites.

Some of the examples can be run from Scala’s console by simply copying and pasting the code, but all of the code used throughout this document is available as a separate package (Maven 2.x or 3.x, and Simple Build Tool 0.7.x are required to build the examples).

<span id="_Toc308702045" class="anchor"><span id="_Toc188339601" class="anchor"></span></span>Knowledge Requirements
--------------------------------------------------------------------------------------------------------------------

In order to maximize the value of the content in this deliverable and to be able to create property tests and custom generators with ScalaCheck, it is essential to understand the following Scala concepts:

-   Case classes

-   Implicit conversions and parameters

-   Generic typing

<span id="_Toc300926411" class="anchor"><span id="_Toc188339602" class="anchor"></span></span>What is ScalaCheck?
=================================================================================================================

ScalaCheck is a unit testing library that provides unit testing of code using property specifications with random automatic test data generation.

ScalaCheck is currently hosted in <http://code.google.com/p/scalacheck/>, including documentation and readymade JAR files, and the source code is currently available from github.com: <https://github.com/rickynils/scalacheck>.

Other features provided by ScalaCheck include:

-   Test case minimization (i.e. what is the shortest test case the fails?)

-   Support for custom data generators

-   Collection of statistical information about the input data

<span id="_Toc308702046" class="anchor"><span id="_Toc188339603" class="anchor"></span></span>Benefits of ScalaCheck compared to traditional unit testing
---------------------------------------------------------------------------------------------------------------------------------------------------------

ScalaCheck is not meant to replace traditional unit testing frameworks (JUnit, Specs, ScalaTest), but to extend them and provide some advantages for new and existing unit testing suites:

-   Automatic generation of data allows developers to focus on defining the purpose of the actual test case, rather than spending time and energy looking for corner cases by themselves

-   Property-based tests provide a lot more testing for a lot less code than assertion-based tests

-   Can help in scenarios where whole classes of test cases have to be tested and it is not feasible to write tests for all distinct test scenarios

<span id="_Toc308702047" class="anchor"><span id="_Toc188339604" class="anchor"></span></span>Use Cases
-------------------------------------------------------------------------------------------------------

The following are use cases and scenarios where ScalaCheck could prove itself useful for existing projects:

-   Testing of code that expects any kind of input values within the boundaries of a specific data type or domain type, i.e. code that works on strings or code that works on domain classes, where ScalaCheck can create any kind of string or any kind of random domain object for validating purposes

-   Application logic that maintains mutable state, where ScalaCheck generates random commands/input that affect the internal state that gets validated at the end of the property check

-   State machines, where ScalaCheck can be used to generate random input and to verify the correctness of the end state

-   Parsers

-   Data processors

-   Hadoop’s mappers and reducers

-   Validators (i.e. validators of web form fields in Spring or equivalent )