Appendix 1: References
======================

<span id="_Toc308702091" class="anchor"><span id="_Toc188339650" class="anchor"></span></span>Sample code
---------------------------------------------------------------------------------------------------------

All the code used in this document is currently available in the Innersource repository: <https://innersource.accenture.com/scalacheck/scalacheck-examples>.

The code is structured in folders based on the same structure used throughout this document. Examples where Java and Scala code are mixed (scalacheck-basic, java-scalacheck, scalacheck-integration-junit) require Maven 2.x or 3.x to build and run. Pure Scala examples (scalacheck-integration-specs, scalacheck-integration-scalatest) require Simple Build Tool version 0.11 or greater.

There is no top-level project at the root folder above the sub-folders, so each project is an independent unit by itself.

| Folder                           | Contents                                                                                                                                                   | How to run                                 |
|----------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------|
| scalacheck-basic                 | Basic ScalaCheck examples, showing most of its basic features (basic property checks, data grouping, conditional properties)                               | Switch to the project’s subfolder and run: 
                                                                                                                                                                                                                                             
                                                                                                                                                                                                 mvn scala:run -Dlauncher=test               |
| java-scalacheck                  | Integration of ScalaCheck property checks with Java code                                                                                                   | Switch to the project’s subfolder and run: 
                                                                                                                                                                                                                                             
                                                                                                                                                                                                 mvn scala:run -Dlauncher=test               |
| scalacheck-integration-scalatest | Examples of integration of ScalaTest with with ScalaCheck. Requires SBT.                                                                                   | Switch to the project’s subfolder and run: 
                                                                                                                                                                                                                                             
                                                                                                                                                                                                 sbt reload test                             |
| scalacheck-integration-specs     | Examples of integration of Specs with ScalaCheck. Requires SBT.                                                                                            | Switch to the project’s subfolder and run: 
                                                                                                                                                                                                                                             
                                                                                                                                                                                                 sbt reload test                             |
| scalacheck-integration-junit     | Examples of integration of ScalaCheck with JUnit, as well as JUnit support traits and a JUnit 4 runner for ScalaCheck tests written as Properties classes. 
                                                                                                                                                                                                
                                    It uses the Maven JUnit test runner (Surefire plugin) to run.                                                                                               | Switch to the project’s subfolder and run: 
                                                                                                                                                                                                                                             
                                                                                                                                                                                                 mvn test                                    |
| java-hadoop-scalacheck           | Code and ScalaCheck test cases that test Hadoop’s WordCount example                                                                                        | Switch to the project’s subfolder and run: 
                                                                                                                                                                                                                                             
                                                                                                                                                                                                 mvn scala:run -Dlauncher=test               |

<span id="_Toc308702092" class="anchor"><span id="_Toc188339651" class="anchor"></span></span>Links
---------------------------------------------------------------------------------------------------

The following are some useful links and references on ScalaCheck:

-   ScalaCheck wiki documentation: <http://scalacheck.googlecode.com/svn/artifacts/1.9/doc/api/index.html>

-   ScalaCheck JAR files for download: <http://code.google.com/p/scalacheck/downloads/list>

-   ScalaCheck source code: <https://github.com/rickynils/scalacheck>

-   ScalaCheck API documentation (Scaladoc): <http://scalacheck.googlecode.com/svn/artifacts/1.9/doc/api/index.html#package>

-   Better unit tests with ScalaCheck (and specs): <http://etorreborre.blogspot.com/2008/01/better-unit-tests-with-scalacheck-and.html>

-   ScalaCheck generators for JSON: <http://etorreborre.blogspot.com/2011/02/scalacheck-generator-for-json.html>

-   Specs2 user guide: <http://etorreborre.github.com/specs2/guide/org.specs2.UserGuide.html>

-   Specs2 ScalaCheck integration example: <https://github.com/etorreborre/specs2/blob/1.5/src/test/scala/org/specs2/examples/ScalaCheckExamplesSpec.scala>

-   ScalaTest: <http://www.scalatest.org/>

-   JUnit: <http://www.junit.org/>