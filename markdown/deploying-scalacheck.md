<span id="_Toc308702376" class="anchor"><span id="_Toc188339652" class="anchor"></span></span>Appendix 2: DEPLOYING SCALACHECK
==============================================================================================================================

<span id="_Toc308702093" class="anchor"><span id="_Toc188339653" class="anchor"></span></span>Using JAR files
-------------------------------------------------------------------------------------------------------------

The easiest way to get started with ScalaCheck or to get ScalaCheck incorporated into an existing code base is to deploy the required JAR file. If using a build tool such as Ant, pre-compiled JAR files for different versions of ScalaCheck are available from the Downloads section in the project’s page at Google Code ([link](http://code.google.com/p/scalacheck/downloads/list)). Only one JAR file is required, as ScalaCheck has no external dependencies.

<span id="_Toc308702094" class="anchor"><span id="_Toc188339654" class="anchor"></span></span>Maven
---------------------------------------------------------------------------------------------------

We will assume that Maven 2.x or 3.x is installed and that a suitable pom.xml file has already been created, and that it has been configured to use the Scala compiler plugin to compile Scala code in the project. If not, please refer to the [Scala Maven documentation](http://www.scala-lang.org/node/345) or use one of the existing [Maven archetype templates](http://docs.codehaus.org/display/MAVENUSER/Archetypes+List) for Scala development to get the project started.

If not already present, the following repository needs to be added to our project’s pom.xml file in the repositories section:

\<repository\>

\<id\>scala-tools.org\</id\>

\<name\>Scala-Tools Maven2 Repository\</name\>

\<url\>http://scala-tools.org/repo-releases\</url\>

\</repository\>

Then, the following dependency needs to be declared:

\<dependency\>

\<groupId\>org.scala-tools.testing\</groupId\>

\<artifactId\>scalacheck\_2.9.0-1\</artifactId\>

\<version\>1.9\</version\>

\</dependency\>

Bear in mind that ScalaCheck is cross-compiled for multiple releases of Scala, so it is important to know which version of Scala we are currently using, as it is defined as part of the dependency name (2.9.0-1 in the example above).

<span id="_Toc308702095" class="anchor"><span id="_Toc188339655" class="anchor"></span></span>Simple Build Tool (0.11.x)
------------------------------------------------------------------------------------------------------------------------

SBT is the preferred tool for Scala development for “pure” Scala projects. The instructions below assume that SBT 0.11.x has been installed, that the sbt startup script is in the path and that a suitable SBT project exists in the current folder. Please note that these instructions are not applicable when using SBT 0.7.x, as there have been extensive changes in the way that SBT handles project definitions starting with version 0.10.

To get ScalaCheck as a dependency in our SBT project the following lines of code need to be added to our project’s *build.sbt* file:

libraryDependencies += "org.scala-tools.testing" %% "scalacheck" % "1.9" % "test"

Get into the SBT console by running “*sbt*” in your project folder, and then run the “*reload*” and “*update*” commands to get SBT to refresh the project file (in case SBT did not do it automatically) and to update its dependencies.

When defining a new dependency in *build.sbt*, please note that there’s no need to specify the current version of Scala in use as part of the ScalaCheck dependency, since by using “%%” SBT will figure it out automatically for us.