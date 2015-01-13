<span id="_Toc300926419" class="anchor"><span id="_Toc188339615" class="anchor"></span></span>Generators
========================================================================================================

<span id="_Toc300926420" class="anchor"><span id="_Toc301262003" class="anchor"></span></span>The samples in this section can be run using the contents of the scalacheck-basic, folder, but the content needs to be compiled. In a command window, switch to the scalacheck-basic folder and then first type ```mvn compile``` and then ```mvn scala:console```.

<span id="_Toc308702056" class="anchor"><span id="_Toc188339616" class="anchor"></span></span>Built-in generators
-----------------------------------------------------------------------------------------------------------------

ScalaCheck provides built-in generators for common data types: Boolean, Int, Long, Double, String, Char, Byte, Float, containers with those data types (Lists, Arrays), and other commonly used objects such as Date, Throwable, Scala’s Option and Either, Tuples, and Function objects. All these built-in generators are in class ```org.scalacheck.Arbitrary```.

These generators can be used as-is in our own property checks, or can be combined to create more complex generators and arbitrary objects for our own property checks.

<span id="_Toc308702057" class="anchor"><span id="_Toc188339617" class="anchor"></span></span>Custom generators
---------------------------------------------------------------------------------------------------------------

Often, when testing our own domain objects and functionality, it is be necessary to create our own custom generators to act as random data sources for our property checks. This allows us to define property checks with functions that take domain objects as their parameters.

Custom generators in ScalaCheck are implemented using class ```org.scalacheck.Gen```.

In order to present custom generators, the Rectangle class will be used as our domain object. This is a simple case class that encapsulates the (simple) logic for handling a rectangle with height and width, with the following constructor signature:

```scala
case class Rectangle(val width: Double, val height: Double) {
  // note that there’s a bug in the function below!
  lazy val area = if(width % 11 ==0) (width * 1.0001 * height) else (width * height)
  
  // correct version of the area method
  lazy val areaCorrect = (width * height)

  lazy val perimeter = (2*width) + (2*height)
  
  def biggerThan(r:Rectangle) = (area > r.area)
}
```

Please note that in the snippet above the ```Rectangle.area``` method contains a small bug in it in order to force it to fail for certain values. The correct area method is called ```areaCorrect```.

This class also provides methods ```area```, ```perimeter``` and ```biggerThan```. The full source code for this class is available as part of this document

At their simplest level, generators are functions that optionally take some input data (where this input data is used to customize the kind of random data that is generated) and generate output for ScalaCheck to use as input in a property check.

In the case of the Rectangle class, we’ll create a generator function that returns a Rectangle object as well as the random height and the width values that were used to create the object. The purpose for this generator is to create a property check to verify that the internal calculation for the area and the width is correct.

Generators return Option objects, which means that they do not necessarily have to return an actual value if the input data is not suitable to generate output values.

In our example, the generator will be defined as a function that returns a tuple of 3 elements of type ```(Rectangle, Double, Double)```, where the ```Double``` values represent the height and the width that were used to create the ```Rectangle``` object:

```scala
import org.scalacheck.Gen

val rectangleGen: Gen[(Rectangle, Double, Double)] = for {
  height <- Gen.choose(0,9999)
  width <- Gen.choose(0,9999)
} yield((Rectangle(width, height), width, height))
```

It is common to implement generators using a for comprehension in order to keep the code short and concise, but as generators are normal functions, they can be implemented in any way as long as they return a value of the required type (or no value, see below)

Now that we’ve created the generator, it can also be run as a standalone object using its ```apply``` method in Scala’s console; the ```apply``` method returns an ```Option[T]``` object (so that generators can also return no value using the None object, if needed) where ```T``` is the type of the generator, in this case ```(Rectangle, Double, Double)```. The ```apply``` method also needs a ```Params``` object in order to run, which is used to encapsulate all the parameters needed for data generation:

```scala
rectangleGen(new org.scalacheck.Gen.Params)
```

Alternatively, the parameterless ```sample``` method can be used instead to generate sample values from an existing generator.

In the console, the result of running either one of the methods above will produce something like this:

```scala
Option[(Rectangle, Double, Double)] = Some((Rectangle(9714.0,7002.0),9714.0,7002.0))
```

Now our property check can be written using our new custom generator by providing it as a parameter to the ```forAll``` method. Once the generator is in place, the same data type that is produced by the generator must be used as the type for the input parameter(s) of the property check function. In this case, the tuple ```(Rectangle, Double, Double)``` is used as the return type by the generator and the type for the ```input``` parameter. If the types do not match, the Scala compiler will complain.

The code for the property check using the generator is:

```scala
import org.scalacheck.Prop._
import org.scalacheck.{Arbitrary, Properties, Gen}

object RectangleSpecification extends Properties("Rectangle specification") {
  property("Test area") = forAll(rectangleGen) { (input: (Rectangle,Double,Double)) =>
    input match {
      case(r, width, height) => r.area == width * height
  }}
}
```

In the example code above, pattern matching is used so that we can extract named parameters from the tuple; this is not necessary and was only implemented for clarity reasons, as we could have also referred to the values within the tuple using their indexes (```input._1```, ```input._2``` and ```input._3```)

To run this property check, use the ```RectangleSpecification.check``` method. As we’ve used incorrect logic to calculate the area, the output should be failure:

```
! Rectangle specification.Test area: Falsified after 15 passed tests.

> ARG_0: (Rectangle(4169.0,1988.0),4169.0,1988.0)
```

<span id="_Toc300926421" class="anchor"><span id="_Toc301262004" class="anchor"><span id="_Toc308702058" class="anchor"><span id="_Toc188339618" class="anchor"></span></span></span></span>The Gen companion object
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The Gen companion object provides some commodity methods and implicit methods to generate all sorts of data. Two of the most useful ones are the ```Gen.choose``` method, which is able to generate random data of any type (as long as there’s an implicit Choose[T] object in scope for the type, but this is always the case for basic data types) within the given limits and ```Gen.oneOf```, which returns an element from the given set.

<span id="_Toc300926422" class="anchor"><span id="_Toc301262005" class="anchor"><span id="_Toc308702059" class="anchor"><span id="_Toc188339619" class="anchor"></span></span></span></span>The Arbitrary generator
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The Arbitrary generator is a special generator in ScalaCheck. Arbitrary generators are built on existing generators.

The Arbitrary generator allows us to simplify our properties because the data generation logic is implemented as an implicit function that is brought into the scope of the property check. This means that when using Arbitrary objects, ScalaCheck can automatically generate test data for any custom type without the need to provide an explicit reference to the generator(s) required in the call to ```forAll```.

In this example, we’ll use the same ```Rectangle``` case class but we’ll create an implicit value which wraps the ```Rectangle``` generator and returns an ```Arbitrary[Rectangle]``` object every time it is called.

First of all, we need a generator of Rectangle objects. Note that this time we’re only returning Rectangle objects:

```scala
val arbRectangleGen: Gen[Rectangle] = for {
  height<- Gen.choose(0,9999)
  width<- Gen.choose(0,9999)
} yield(Rectangle(width, height))
```

Once the generator is in place, defining the arbitrary generator is a rather straightforward affair using the Arbitrary object and providing our generator as a parameter to its ```apply``` method:

```scala
import org.scalacheck.Arbitrary
implicit val arbRectangle: Arbitrary[Rectangle] = Arbitrary(arbRectangleGen)
```

In some cases, the Scala compiler will not be able to infer the type of our arbitrary object, therefore the explicit type may need to be provided as part of the val definition (as we did above).

Finally, we create our new property that checks the correctness of the ```Rectangle.biggerThan``` method:

```scala
object ArbitraryRectangleSpecification extends Properties("Arbitrary Rectangle spec") {
  property("Test biggerThan") = forAll{ (r1: Rectangle, r2: Rectangle) =>
    (r1 biggerThan r2) == (r1.area > r2.area)
  }
}
```

Note how the ```forAll``` method is not provided an explicit reference to an existing generator, because it’s using the implicit Arbitrary generator ```arbRectangle``` that we just defined, which is conveniently in the scope of the property check. This means that our new property check is not bound to any specific generator but only to the one that is currently in scope. It also means that the same property check can be transparently reused with different generators only by importing different implicit Arbitrary generators.

Compare the property above with the same version of the property check but without an arbitrary generator:

```scala
object ... extends Properties("...") {
  property("Test biggerThan") = forAll(**rectangleGen, rectangleGen**){ (r1: Rectangle, r2: Rectangle) =>
    (r1 biggerThan r2) == (r1.area > r2.area)
  }
}
```

The main difference is that now two custom generators have been provided, one for each one of the parameters of the function provided to ```forAll```. In order to keep our code focused on the actual testing logic and remove all the typing boilerplate, it is recommended to create the implicit Arbitrary generator.

<span id="_Toc300926423" class="anchor"><span id="_Toc301262006" class="anchor"><span id="_Toc308702060" class="anchor"><span id="_Toc188339620" class="anchor"></span></span></span></span>More generators
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

In addition to the topics described above, ScalaCheck also provides more specialized generators such as sized generators, conditional generators and generators of containers (such as lists and arrays). Please refer to the ScalaCheck user guide as well as to the ScalaCheck online API documentation for more information.
