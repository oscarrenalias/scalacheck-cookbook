Real World Scenario: Testing a Hadoop MAp-Reduce Job with ScalaCheck
====================================================================

<span id="_Toc308702073" class="anchor"><span id="_Toc188339632" class="anchor"></span></span>Hadoop
----------------------------------------------------------------------------------------------------

For those not familiar with Hadoop, this scenario may require a basic introduction.

Hadoop is built around the concepts of mappers and reducers. Following are the definitions from the original MapReduce paper *(J. Dean and S. Ghemawat. MapReduce: Simplified Data Processing on Large Clusters. In OSDI’04, 6th Symposium on Operating Systems Design and Implementation, Sponsored by USENIX, in cooperation with ACM SIGOPS, pages 137–150, 2004.)*:

> > “The computation takes a set of input key/value pairs, and produces a set of output key/value pairs. The user of the MapReduce library expresses the computation as two functions: map and reduce. Map, written by the user, takes an input pair and produces a set of intermediate key/value pairs. The MapReduce library groups together all intermediate values associated with the same intermediate key I and passes them to the reduce function.
> >
> > The reduce function, also written by the user, accepts an intermediate key and a set of values for that key. It merges together these values to form a possibly smaller set of values. Typically just zero or one output value is produced per reduce invocation. The intermediate values are supplied to the user’s reduce function via an iterator. This allows us to handle lists of values that are too large to fit in memory.”

Hadoop is a MapReduce library. As a Java library, Hadoop represents map functions as Mapper classes and reduce functions as Reducer classes. Naturally, in a distributed environment there will be many mapper instances and many reducer instances running as concurrent tasks. Hadoop ensures that each mapper task receives a manageable chunk of the input data, and will dynamically decide how many reducer tasks are needed and which data to feed into each one of them. When the reducer tasks have run, Hadoop collects their output and makes it available in the file system.

Between the map and the reduce computation phases, Hadoop will group and sort all the keys and values from the mapper tasks, so that the input for a reducer task is a single key and a list of values that mapper tasks generated for that key.

Regarding input and output, Hadoop can read any kind of file format from the file system (either Hadoop’s own Hadoop File System – HFS, or the local file system when testing) but generally the input is based on text files; mappers usually read one line at a time from a file and produce keys and values based on specific logic. Reducers process this data and write it back to temporary output, which is consolidated by Hadoop at the end of the job.

<span id="_Toc308702074" class="anchor"><span id="_Toc188339633" class="anchor"></span></span>The scenario
----------------------------------------------------------------------------------------------------------

This scenario describes how to use ScalaCheck to test a Hadoop MapReduce job which is written in Java.

In this scenario we’ll create a couple of property checks to verify the correctness of the map and reduce functions used in Hadoop’s WordCount example (<http://wiki.apache.org/hadoop/WordCount>). WordCount is a job that is used to count the number of times that a word appears in the file(s) provided to the job. WordCount is a simple job, but the concepts for creating suitable ScalaCheck property checks remain applicable in more complex Hadoop scenarios.

ScalaCheck is ideal for these use cases as usually both mappers and reducers in a Hadoop job will work on arbitrary data, and ScalaCheck’s random data generation features can be used to their full extent to generate a suitable range of inputs for the jobs.

### <span id="_Toc308702075" class="anchor"><span id="_Toc188339634" class="anchor"></span></span>WordCount mapper

In WordCount, the input for the mapper is, as previously described, a key-value pair; the key is the position of the line within the file (offset in bytes), while the value is a line of text.

This is the code for the mapper:

public static class Map extends Mapper\<LongWritable, Text, Text, IntWritable\> {

private final static IntWritable one = new IntWritable(1);

private Text word = new Text();

public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

String line = value.toString();

StringTokenizer tokenizer = new StringTokenizer(line);

while (tokenizer.hasMoreTokens()) {

word.set(tokenizer.nextToken());

context.write(word, one);

}

}

}

In WordCount, the mapper splits incoming lines so that the output keys are the words from the line while the output values are always the value “1” (each word instance counts as one). The mapper ignores the input key because in this case, it is not needed for our logic.

### <span id="_Toc314170210" class="anchor"><span id="_Toc308702076" class="anchor"><span id="_Toc308702358" class="anchor"><span id="_Toc308702077" class="anchor"><span id="_Toc188339635" class="anchor"></span></span></span></span></span>WordCount reducer

Later on the reducer will simply sump up all the “1” values for a given word (key), to determine how many times a particular word appeared in the text. Since Hadoop will group all the same keys into one single key with its values being a list of occurrences (a list of number 1s), all the mapper needs to do is add up the list and generate as its output, the same word as the key and the total sum value as the value:

public static class Reduce extends Reducer\<Text, IntWritable, Text, IntWritable\> {

public void reduce(Text key, Iterable\<IntWritable\> values, Context context) throws IOException, InterruptedException {

if(key.getLength() == 0) // do nothing for empty keys

return;

int sum = 0;

for (IntWritable value : values) // otherwise sump up the values

sum += value.get();

context.write(key, new IntWritable(sum));

}

}

### <span id="_Toc308702078" class="anchor"><span id="_Toc188339636" class="anchor"></span></span>Input and output

The input for the job will be taken from a text file (or files) stored in the local filesystem. The contents of the file(s) will be plain text.

The output of the job will be another text file where each line will contain a word from the source file as well as the number of times that the same word occurred throughout the text.

### <span id="_Toc314170213" class="anchor"><span id="_Toc314170214" class="anchor"><span id="_Toc308702079" class="anchor"><span id="_Toc188339637" class="anchor"></span></span></span></span>Other requirements

In this particular example, we’ll use Cloudera’s MrUnit library to help us take care of the internal Hadoop plumbing for unit testing (initialization of input and output data types, initialization of contexts, job execution, etc.). The correct library version and configuration is provided in the Maven project needed to run these test cases.

<span id="_Toc308702080" class="anchor"><span id="_Toc188339638" class="anchor"></span></span>Defining the test cases
---------------------------------------------------------------------------------------------------------------------

In this scenario, the following test cases will be executed:

1.  Ensure that the mapper can handle lines with one single word. The input for this property check will be a key (with type LongWritable) and single word (with type Text). Only basic generators will be required for this check.

2.  Ensure that the mapper can handle lines with an arbitrary number of words. The input for this check will be a key (with type LongWritable) and a tuple containing a line of text with the pre-calculated number of words in that line. A more complex custom generator will be required to pre-calculate the number of words.

3.  Verify that the reducer can correctly process any input. The input for this check will be a key (with type Text) and a pair with a list of values representing the number of appearances of the given word in the input file given to the Hadoop job, as well as the pre-calculated sum of those values (which is the same value that the reducer should generate). As in the previous test case, a more complex custom generator will be required to pre-calculate the expected output values from the mapper.

The splitting of input files into key-value (offset-line) pairs precedes the map phase of processing and is outside the scope of our tests below. Similarly, the generation of the output file following the reduce phase is outside the scope of our tests.

<span id="_Toc314170217" class="anchor"><span id="_Toc308702081" class="anchor"><span id="_Toc188339639" class="anchor"></span></span></span>Building the generators
--------------------------------------------------------------------------------------------------------------------------------------------------------------------

In order to build property checks for the scenario above, we’re going to need some specific data generators for the Hadoop data types as, unfortunately for us, Hadoop internally only works with its own wrapper types that are optimized for size when serialized.

The code snippets presented below can be run from within the Scala console, but must be added in the same sequence as presented in this document. In order to start the Scala console, switch to the **java-hadoop-scalacheck** folder and run “*mvn scala:console*”.

### <span id="_Toc308702082" class="anchor"><span id="_Toc188339640" class="anchor"></span></span>Building the basic generators

We’ll start by building some basic generators to create random strings, words, blank spaces (random amounts of them), and then we’ll put those in lines, lists and “bags” (please see below). These generators are generic and do not depend on Hadoop types.

All these generators are in the GenText object.

First, generators to create random strings either from a list of characters, or from a generator of random characters:

import org.scalacheck.Gen

import org.scalacheck.Gen.\_

def genString(genc: Gen[Char]): Gen[String] = for {

lst \<- Gen.listOf(genc)

} yield {

lst.foldLeft(new StringBuilder)(\_+=\_).toString()

}

def genString(chars: Seq[Char]): Gen[String] = genString(Gen.oneOf(chars))

The first generator generates a randomly long list of characters, as selected from the random character generator. The random character generator can be initialized with an arbitrary and unlimited number of characters. The list of character is converted to a string using StringBuilder.

The second generator builds on the first one, and the list of characters can be provided ad-hoc rather than through another character generator.

The next set of generators used the one we have just defined to create strings that satisfy the property that are never empty:

def genNonemptyString(genc: Gen[Char]): Gen[String] =

genc.combine(genString(genc))((x,y) =\> Some(x.get + y.get))

def genNonemptyString(chars: Seq[Char]): Gen[String] =

genNonemptyString(Gen.oneOf(chars))

Just like before, the first generator contains the generation logic; it uses another random character generator to create the characters for the non-empty string while the second generator allows providing a pre-defined static list of characters that will be used as part of the string.

implicit def seqToChar(coll: Seq[Int]): Seq[Char] = for {

c \<- coll

} yield(c.toChar)

val spaceChars: Seq[Char] = Vector(9, 32)

val nonWhitespaceChars: Seq[Char] = (33 to 126)

val genWord = genNonemptyString(nonWhitespaceChars)

val genWhitespace = genNonemptyString(spaceChars)

These generators build non-empty words, or strings which contain characters considered blank spaces (in the ASCII range 9 to 32). They are used by other generators and property checks below.

val genLineWithList = for {

lst \<- Gen.listOf(genWord)

} yield (lst.foldLeft(new StringBuilder)(\_++=\_ + genWhitespace.sample.get).toString.trim(), lst)

val genLineWithBag = for {

lst \<- Gen.listOf(genWord)

} yield (lst.foldLeft(new StringBuilder)(\_++=\_ + genWhitespace.sample.get).toString.trim(), bagOf(lst))

Both generators above generate tuples, where the first item is a line of text and the second is either the list or “bag” of the words in the line.

Finally, we can create some arbitrary generators for later usage:

val arbGenWord = Arbitrary(genWord)

var arbGenLineWithList = Arbitrary(genLineWithList)

val arbGenLineWithBag = Arbitrary(genLineWithBag)

Now we can move onto creating generators of Hadoop-specific data.

### Hadoop generators

The following are all generators specific for Hadoop data types:

import org.apache.hadoop.io.{Text, LongWritable, IntWritable}

def genLongWritable(upperRange:Int) = for {

num \<- Gen.choose(0, upperRange)

} yield(new LongWritable(num))

val genIntWritable = for {

num \<- Gen.choose(0, 9999)

} yield(new IntWritable(num))

val genIntWritableList = Gen.listOf(genIntWritable)

def genLongWritableList(upperRange:Int) = Gen.listOf(genLongWritable(upperRange))

Based on the knowledge acquired so far, the first four generators are rather straightforward: generators for single values of IntWritable and LongWritable, and generators of lists of those two types using Gen.listOf.

All of Hadoop’s own types are simple wrappers on Java’s basic types so can use the default generators in the Gen object to generate random values of the basic type, and then wrap those values using Hadoop’s own type. In the case of the *LongWritable* generator, we’re even parameterizing it by allowing to specify and upper value range for the generated values (may not always be needed, but it was included here for the sake of showing a parameterized generator)

The last generator is used to create a tuple where the first element is a list of IntWritable objects (created with the previous generator) and the second is the sum of all the values in the first list. This will be required in a specific test scenario later on:

import com.company.hadoop.tests.HadoopImplicits.\_

val genIntWritableListWithTotal: Gen[(List[IntWritable], Int)] = for {

l \<- genIntWritableList

} yield((l, l.foldLeft(0)((total, x) =\> x + total)))

In this case we require the support of the HadoopImplicits object, that contains a handful of useful implicit conversions between Hadoop’s types and the basic Scala data types (e.g. IntWritable converted to and from Integer, LongWritable to Long, and so on). These implicit conversions are more heavily used in the actual property checks, and there’s some more detailed information about them below.

### Testing the Generators

We’ve built quite a few custom generators so that we can build property checks for <span id="_Toc308702083" class="anchor"><span id="_Toc308702365" class="anchor"></span></span>our selected scenarios. But how do we know that our generators are generating the data that they say will generate? In other words, how can we be sure that they generate valid data?

To ensure the correctness of our generators we’re going to build a few simple ScalaCheck property checks for generators; property checks that check custom generators used in other property checks:

object GeneratorSpecification extends Properties("Generator tests") {

import GenText.\_

import HadoopGenerators.\_

property("genNonemptyString property") = forAll(genNonemptyString(nonWhitespaceChars)) {(s:String) =\>

s != "" &&

s.forall(\_ != ' ')

}

property("genWord property") = forAll(genWord) { (w:String) =\>

w != "" &&

w.forall(\_ != ' ')

}

property("genLineWithList property") = forAll(genLineWithList) { case (line, list) =\>

list.forall(w =\> w.forall(\_ != ' ')) &&

list.filterNot(w =\> line.indexOf(w) \> 0).size == 0

}

property("genIntWritableListWithTotal property") = forAll(genIntWritableListWithTotal) {

case(list, total) =\>

list.foldLeft(0)((x, sum) =\> x + sum) == total

}

}

<span id="_Toc308702085" class="anchor"></span>The property checks consist of simple logic that validates that the data that they produce is according to their definition, e.g. words generated by *genWord* are always non-empty, that the list of words produced by *genLineWithList* does not contain more words than the line of text, and so on.

We’ve created more generators than the four tested above, but they are not tested because they are either used as part of the ones tested above (*genString*, *genWhitespace*), or they are so basic that they do not need testing (*genIntWritable*, *genIntWritableList*, etc).

Building the property checks
----------------------------

Now that we’ve got all the generators that we’re going to need and we’ve ensured that the key ones generate correct data, we can proceed to build the actual property checks. Since the generators are doing most of the work now by pre-calculating the correct expected values, the logic in the property checks will mostly consists of running the mapper/reducer and then comparing their output values with the expected values.

### <span id="_Toc308702086" class="anchor"><span id="_Toc188339644" class="anchor"></span></span>Some implicits to make our life easier

Implicit conversions are a very useful feature in the Scala language toolset, that provide Scala with a dynamic language flavor even though it’s statically typed, by providing transparent conversion between different types through implicit functions/methods.

In our example, implicits are very handy because they allow our code to transparently use Scala native types in place of Hadoop’s own wrapper types, e.g. use Int instead of IntWritable in a Hadoop method call. This simplifies our code because we do not have to write all those conversions manually.

For our scenarios we have defined two-way conversions between Int and IntWritable, Long and LongWritable, Text and String:

object HadoopImplicits {

import org.apache.hadoop.mrunit.types.Pair

implicit def IntWritable2Int(x:IntWritable) = x.get

implicit def Int2WritableInt(x:Int) = new IntWritable(x)

implicit def LongWritable2Long(x:LongWritable) = x.get

implicit def Long2LongWritable(x:Long) = new LongWritable(x)

implicit def Text2String(x:Text) = x.toString

implicit def String2Text(x:String) = new Text(x)

implicit def Pair2Tuple[U,T](p:Pair[U,T]):Tuple2[U,T] = (p.getFirst, p.getSecond)

}

Hadoop has a few more wrapper types but only the ones above are required here.

The last implicit conversion between the Pair type in MrReduce and Scala’s built-in tuple type is used when dealing with the results of the MapDriver and ReduceDriver classes, used later on for unit testing mappers and reducers.

### <span id="_Toc308702087" class="anchor"><span id="_Toc188339645" class="anchor"></span></span>Handling of single words by the mapper

The first property check consists of ensuring that the mapper can handle lines with one word only; the expected output is a single pair where the key is the given word (which was provided as a parameter as the key) and the value is 1 (represented here by the *one* static object):

import org.scalacheck.Properties

object WordCountMapperSingleWordProperty extends Properties("Mapper property") {

import org.scalacheck.Prop.\_

import com.company.hadoop.tests.GenText.\_

import com.company.hadoop.tests.HadoopGenerators.\_

import com.company.hadoop.WordCount.\_

import scala.collection.JavaConversions.\_

import org.apache.hadoop.mrunit.mapreduce.MapDriver

import com.company.hadoop.tests.HadoopImplicits.\_

import org.apache.hadoop.io.{IntWritable, LongWritable}

val mapper = new Map

val one = new IntWritable(1)

property("The mapper correctly maps single words") = {

forAll(genLongWritable(99999), genWord) {(key:LongWritable, value:String) =\>

val driver = new MapDriver(mapper)

val results = driver.withInput(key, value).run

results.headOption.map(pair =\>

pair.\_1.toString == value && pair.\_2 == one).get

}

}

}

First of all, this property requires a handful of import statements to make sure that we’ve got all needed classes in the scope. This includes our own class with implicit conversions (since we’ll be transparently converting between Scala types and Hadoop wrapper types) as well as Scala’s *scala.collection.JavaConversions*, imported into the scope so that we can easily convert to and from Scala and Java lists and types (makes it easier to deal with Java’s Array and Iterable types as they’re transparently converted to their Scala equivalent)

*MapDriver* is part of the MrUnit framework and takes care of setting up Hadoop’s internal structures that allow us to run map and reduce jobs for testing purposes. The job is actually run when calling the *run* method in the *driver* object, after providing some test input data key and input value.

In this example, the test key is coming from the *longWritableGen* generator that simply generates a random *LongWritable* object representing an offset from an input file just like Hadoop would do. The value is a random word generated by the *genWord* generator.

The output of the mapper will be a list (a Java Iterable, actually) of *Pair* objects. *Pair* objects are produced by MrUnit and are not part of the Hadoop data types. The validation will consist of getting the first one from the list (there should only be one, as there was only one word as the input), and we do that by using *Iterable.headOption*, which will return the first element wrapped in a Scala *Some* (as an *Option* type).

Once we’ve got that, we can use *Option.map* to execute the validation without extracting the actual value from the *Some* wrapper in a very functional way. There’s an implicit conversion between the *Pair* type and Scala’s *Tuple* type (defined in class *com.company.hadoop.tests. HadoopImplicits*), and that allows us to access the contents of the pair variable using the \_1 and \_2 notation as with tuples. In this case, pair.\_1 refers to the key produced by the mapper (which should be the same word that was provided as the mapper’s input as the value) and the value, accessed as pair.\_2, should be 1. If the comparison is fulfilled, then the property check is validated.

### <span id="_Toc314170225" class="anchor"><span id="_Toc308702088" class="anchor"><span id="_Toc188339646" class="anchor"></span></span></span>Mapping longer lines

The next property check extends the first one to ensure that the mapper can process lines of with arbitrary numbers of words, including empty lines (where the key is still present but the value is an empty line).

In this property check the *LongWritable* is again provided by the *genLongWritable* generator while the second parameter is a tuple where the first element is a line of text and the second value is the list of words in the line, split in the same way as the mapper should do it. Please note that the line of text may contain arbitrary numbers of blank spaces between words.

The generator responsible for this is *genLineWithList*:

import org.scalacheck.Properties

object WordCountMapperLinesProperty extends Properties("Mapping lines of text") {

import com.company.hadoop.WordCount.\_

import scala.collection.JavaConversions.\_

import org.apache.hadoop.mrunit.mapreduce.MapDriver

import org.scalacheck.Prop.\_

import com.company.hadoop.tests.GenText.\_

import com.company.hadoop.tests.HadoopGenerators.\_

import com.company.hadoop.tests.HadoopImplicits.\_

import org.apache.hadoop.io.{IntWritable, LongWritable}

val mapper = new Map

val one = new IntWritable(1)

property("The mapper correctly maps lines with multiple words") =

forAll(genLongWritable(99999), genLineWithList) {

case (key, (line, list)) =\>

val driver = new MapDriver(mapper)

val results = driver.withInput(key, line).run

(results.forall(one == \_.\_2) && results.map(\_.\_1.toString).sameElements(list))

}

}

After setting up the MrUnit driver, the property check consists of two checks: ensuring that all words (keys) produced by the mapper have a value of 1, and that the mapper produced as many keys as indicated by the generator (which by itself already knew how many words were there in the string).

The first check is rather straightforward and implemented by processing all *pairs* in the *results* iterable using *results.forall* (returns true if all items in the iterable match the given Boolean check),

For the second check, method *sameElements* in Scala’s List type comes very handy as it checks if the given list has the exact same elements as the current list, and in the same order; here were are assuming that the mapper will generate its results in the same sequence as the words appeared in the line, which is a valid assumption.

Please note that before comparing the two lists, we have to convert the values in the list with results that was generated by the mapper from Hadoop’s *Text* type to *String* (essentially, the list variable is of type List[Text], not List[String]). Comparing a Text object with a String object, even if the content is the same, will always yield Boolean false. We do the conversion using the *map* method in the List object, to generate another list of Strings.

### <span id="_Toc308702089" class="anchor"><span id="_Toc188339647" class="anchor"></span></span>Reducing data

The last property check is focused on the reducer, and is used to ensure that given a word (key) a number representing the number of times it appears in the input string, the reducer is able to correctly produce an output where the keyis the same word but the value is the sum of the input values.

The *textGen* generator is used to generate a random word, while *intWritableListWithSum* is a generator that produces a tuple where one element is the list of occurrences for the word while the second element is the sum of the values that the reducer is expected to produce.

import org.scalacheck.Properties

object WordCountReducerCheck extends Properties("Reducer spec") {

import com.company.hadoop.WordCount.\_

import scala.collection.JavaConversions.\_

import org.apache.hadoop.mrunit.mapreduce.ReduceDriver

import org.scalacheck.Prop.\_

import com.company.hadoop.tests.GenText.\_

import com.company.hadoop.tests.HadoopGenerators.\_

import com.company.hadoop.tests.HadoopImplicits.\_

val reducer = new Reduce

property("The reducer correctly aggregates data") =

forAll(genWord, genIntWritableListWithTotal) {

case(key, (list, total)) =\> {

val driver = new ReduceDriver(reducer)

val results = driver.withInput(key, list).run

results.headOption.map(\_.\_2.get == total).get == true

}

}

}

After setting up MrData and running the job, the results will be provided as an iterable object where there should only be one element; reducers are only provided one key at a time and we know that the reducer in our sample code should only generate one output key and value.

The pattern is the same as before: obtain the only element using *Iterable.headOption* and then use the *Option.map* method to process the wrapped value to make sure that the *IntWritable* value representing the sum of the input values is the same that as pre-calculated by the generator. If it is, then the property check holds true.

<span id="_Toc308702090" class="anchor"><span id="_Toc188339648" class="anchor"></span></span>Putting it all together
---------------------------------------------------------------------------------------------------------------------

With the property checks in place, the only missing thing is to implement that calls the properties’ *check* method so that we can execute the tests. We could also integrate the code with the ScalaCheck JUnit runner but since we’ve only got three scenarios, we can do it manually this time.

When using the sample code provided with this deliverable, the code can now be easily run with maven:

mvn scala:run -Dlauncher=test

And it should generate the following output:

+ Mapper and reducer tests.The mapper correctly maps single words: OK, passed 100 tests.

+ Mapper and reducer tests.The mapper correctly maps lines with multiple words: OK, passed 100 tests.

+ Mapper and reducer tests.The reducer correctly aggregates data: OK, passed 100 tests.

Please note that the file that implements these property checks in the sample code package does it by grouping all three properties in the same Properties class, instead of having three separate Properties classes. In any case, the final outcome is the same.