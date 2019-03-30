# Kotlin

## Summary
Kotlin is designed to be pragmatic / practical to solve real problems in a practical manner. The design is based on observed successful and unsuccessful solutions. It's not a "research language", they aren't trying to solve / explore new ideas, this reduces the language's complexity. The language is made with tooling in mind, enabling IDEs to easily analyse the code and make improvement suggestions. Kotlin has the same performance Java Open-Source!

Sample code
```kotlin
data class Person(val name: String, val age: Int? = null)

fun main(args: Array<String) {
    val persons = listOf(Person("Alice"), Person("Bob", age = 29))

    val oldest = persons.maxBy { it.age ?: 0 }
    println("The oldest is:; $oldest")
}
```
Kotlin is statically typed yet it can infer the type from the following assignment `val x = 1`. Static typing pros: performance, reliability, maintainability, tool support. Also supports functional programming: First-class functions, immutability, no side effects. Functional program often leads to more "elegant" code, and is also safer in terms of multithreading.
## Compiling Kotlin code
Kotlin uses `kotlinc` to compile the Kotlin-files to Java byte code.The kotlin-runtime library must be distributed with the Kotlin code.
```
kotlinc <source file> -include-runtime -d <jar name>
java -jar <jar name>
```
### Generated Java class-files
Since Kotlin compiles to Java byte code, the genrerated code must adhere to Java standards. Classes defined in Kotlin will be compiled to a file named `<ClassName>.class`. Package level functions / variables however don't easily "translate" to a class, therefore Kotlin compiles them into a class based on the file-name and postfixing it with `Kt` like so `<FileNameKt.kt>`. The file name can be overriden by prepending the file content with `@file.JvmName("WantedClassName")`.
```kotlin
//filename: join.kt
package strings
fun joinToString(...): String { ... }
```
The above will be compiled into the following, based on the kotlin filename:
```java
//filename: join.class
package strings;
public class JoinKt {
    public static String joinToString(...) { ... }
}
```
# Primitives and other basic types
Kotlin doesn't have primitive types like Java does, which is why they have method-calls. A Kotlin "primitve" that is nullable will be converted to Java's wrapper-class for that primitive type.
Only properties that are of primitive types can be marked `const`. 
# Functions
Kotlin's functions support both named parameters and default parameters. Since we can name our parameters both during function-decalaration and usage, we can reorder the parameters of the function call. Functions can be declared on the package level, and inside other functions.
```kotlin
listOf(1,2).joinToString(prefix = "[", postfix = "]")   //Same as below
listOf(1,2).joinToString(postfix = "]",prefix = "[")    //Same as above
listOf(1,2).joinToString() // Uses all th default parameter values defined in the function declaration
//collection instantiation
val set = hashSetOf(1, 7, 53)
val list = arrayListOf(1, 7, 53)
val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")  //to is not a special construct, just a normal infix function
```
Since Kotlin supports default values we don't need to overload functions and constructors. Java however needs these overloads, in order to generate them use the `@JvmOverloads` annotation on the function declaration. Storing parameter names in .class files is supported as an optional feature only starting with Java 8, and Kotlin maintains compatibility with Java 6. As a result, the compiler can't recognize the parameter names used in your call and match them against the method definition.
## Function types
The examples below illustrates how to declare function types. We put the parameter types in paranthesis, followed by an arrow and then the return type of the function.
```kotlin
val sum: (xValue: Int, yValue: Int) -> Int = { xValue, yValue -> xValue + yValue }  //We can specify parameter names if we want to
val action: () -> Unit = { println(42) }
val funOrNull: ((Int, Int) -> Int)? = null
```
```kotlin
Parameter types     Return type
   /   \                 |
(Int, String)    ->    Unit //When declaring a "void" function type, the Unit type must be used.
```
We can also return functions, like so:
```kotlin
fun returnTransformingFunction(str: String) : () -> String = { "$str : ${str.toLowerCase()}" }
println(returnTransformingFunction("MEH")()) // MEH : meh
```
## infix functions
We can declare `infix` functions so we wont need the dot-notation and "()" method-invocation indicator:
```kotlin
infix fun Int.pow(power: Int): Int {
     var result = 1
     for (i in 1..power) {
         result *= this
     }
     return result
}
val number = 10 pow 2   // 100, infix function call
```
## Extension functions
All compiled Java-class can be extended with Kotlin's extension functions. Extension functions don't have access to private or protected members of the "receiver". Extension functions are compiled the same way as package lever functions. Extension functions are very useful when the extended functionality is only relevant for a small part of the code, that way we avoid bloating the Class with unnecessary methods used only by a small subset of the application.
```kotlin
//filename: StringHelpers
"Receiver type"              "Receiver object"
        |                       /       |
fun String.lastChar(): Char = this.get(this.length - 1) //We can omit `this` if we want to, it's implicit
```
When compiled into a java class it will look something like this:
```java
public class StringHelpersKt {
    public char lastChar(String $receiverString) {
        return receiverString.charAt(receiverString.length - 1);
    }
}
```
And be callable in Java using this:
```java
char c = StringHelpersKt.charAt("Java");
```
Since it's compiled into a static method with the receiver object as its first parameter, no adapter is created or runtime overhead "generated".
## Extension properties
An extenstion property can't have any state, so extension properties are most useful to make it simpler to acces "properties" of an object just like a normal property. Both `get()` and `set(value: Char)` can be implemented. When accessing extension properties from Java, the properties should be accessed by invoking its getter explicity: `StringUtilKt.getLastChar("Java");`
```kotlin
val String.lastChar: Char
    get() = get(length - 1)

println("yolo".lastChar)
```
## Any, Unit, and Nothing
In Kotlin, `Any` is a supertype of all types, including primitive types such as Int. Under the hood, the `Any` type corresponds to Java's java.lang.Object. The `Unit` type in Kotlin serves the same purpose as `void` in java, it can be used as the return type a function that has nothing interesting to return. Unlike Java's `void`, Kotlin's `Unit` can be used as a type argument, since it actually is a type implemented as a "singleton" value. The `Nothing` type indicates that "this function never returns" or terminates normally, for instance in an infinite while-loop.
```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```
`Nothing` functions can be used on the right side of the "elvis operator".

# Class stuff
All classes in Kotlin are "closed" by default and therefore not open for inheritance. If it's supposed to be inheritable mark it with the `open` keyword. Kotlin supports the delegation pattern natively using the `by` keyword. `sealed` restricts the possible subclasses of a class. As with Java, Kotlin classes is limited to subclass/extend only one class. The `override` keyword is required in Kotlin.

Due to kotlins support for Java 6 (which doesnt support default methods for interfaces), interfaces with method implmentations are converted into an interface defining all the method and a java class with a static implementation of the methods defined in the interface.

The `init{...}` method inside a class can be used to define special initalization logic, lik Java's constructor. If all constructor parameters have default values, kotlin will generate an additional constructor without parameters that uses all the default values, this makes it easier to use kotlin with libraries that instantiate classes via parameterless constructors.

A nested inner class in Kotlin is converted to a static inner class in Java classes by default, to make it into an inner instance class in Java add the `inner` keyword.
| Class A declared within another class B | In Java | In Kotlin |
| --- | --- | --- |
| Nested class (doesn't store a reference to an outer class) | static class A | class A |
| Inner class (stores a reference to an outer class) | class A | inner class A |
To reference an outer class in kotlin we must write `this@OuterClassName` from the inner class.
```kotlin
class Outer {
    inner class Inner {
        fun getOuterReference(): Outer = this@Outer
    }
}
```
## Visibility
| Modifier | Class member | Top-level declaration | Conserved when compiled to Java? |
| --- | --- | --- | --- |
| public (default) | Visible everywhere | Yes |
| internal | Visible in a module | Visible in a module | No, becomes  public |
| protected | Visible in subclasses | -, different than java | Yes |
| private | Visible in a class | - | Yes, a java class is compiled to a package-private declaration |
The names of `internal` members of a class are mangled in order to avoid unexpected name clashes. In Kotlin, an outer class cannot see the private members of an inner class. In the class constuctor the fields / parameters can be prefixed with a visibility modifier, the default is `public`.
### Open, final and abstract modifiers, final by default
Methods that should be overridable must be marked with the `open` keyword. the overriden open method is itself open for being overriden by another subclass, this can be avoided by using `final override fun funname()`. Smart casts can only be used with variables marked `val`.

## Implementing properties declared in interfaces
In Kotlin, an interface can contain abstract property declarations for both `val` and `var`. These interfaces are converted to normal interfaces with getter and setter methods i Java byte code. The kotlin class implementing the interface must `override` the property.
```kotlin
interface Person {
    var name: String
}
```
```java
interface Person {
    String getName();
    void setName(String name);
}
```
## Equals, == and ===
In Java we always use `obj1.equals(obj2)`, in Kotlin however we do `obj1 == obj2` because it is always converted to `obj1.equals(obj2)` when compiled to Java. Use `===` if a reference check is what we are after.
## Data class
The data class provides us with a default implementation for `toString`, `hashCode`, `equals`, and `copy`. Just prefix the class with `data` and we're all set, note that all constructor arguments in the primary constructor must be either `val` or `var`.

## Class delegation: using the "by" keyword
Kotlin makes using the delegation pattern a breeze. Whenever we're implementing an interface, we can say that we're __delegating__ the implementation of the interface to another obejct, using the `by` keyword. See the example below for an illustration of the difference between the Kotlin code and Java code.
```kotlin
interface SoundMaker {
    fun makeSound(): String
}

class Engine(): SoundMaker {
    override fun makeSound(): String = "Vroom"
}

class Car(engine: Engine): SoundMaker by engine //That is it

fun main(args: Array<String>) {
    println(Car(Engine()).makeSound())
    //Vroom
}
```
```java
//Assume Engine and SoundMaker is the same as above
class Car implements SoundMaker {
    Engine engine;
    public Car(Engine engine) {
        this.engine = engine;
    }
    public String makeSound() { //More work
        return engine.makeSound();
    }
}
```

### `object` keyword
The `object` keyword declares a class and also creates an instance of it, it expresses __singleton objects__, __companion objects__ and what's known as __anonymous classes__ in Java. 
#### Kotlin singleton 
To create a singleton object in kotlin we use  `object` keyword. Just like a class, and object declaration can contain properties, methods, initializer blocks and so on, and also inherit from both classes and interfaces. Constructors  however aren't allowed.
```kotlin
// The following is a singleton Payroll instance.
object CarFactory {    //The object keyword isn't assigned to a var/val, instead it's a "different way" of creating a class
    fun createCar() = Car(/** whatever constructor**/)
}

//The car factory singleton is used in this way
val car = CarFactory.createCar()
```
An object declaration in Kotlin is compiled to a class with a static field holding the single instance named INSTANCE. To use this instance from Java code do `CarFactory.INSTANCE.createCar()`.
### Companion objects - static methods on classes in Kotlin
Kotlin __can't__ have `static` methods / variables, instead kotlin relies on package level functions / variables and object declarations. If we absolutely want static methods on classes when interoperating with Java we can provide the class with a `companion object`. We are allowed to defined extension functions on companion like so: `fun Person.Companion.fromJson(..)`
```kotlin
class Person(name: String) {
    companion object {  //We can optionally include a "class"-name for the companion object like so: companion object Creator
        fun someStaticallyMeaningfulMethod() = Person
    }
}
//Usage of companion objects in Kotlin
val subscribingUser = Person.someStaticallyMeaningfulMethod() 
//Usage of companion objects in Java
Person.Companion.someStaticallyMeaningfulMethod();
```
#### Anonymous class instantiation
An `object` can be defined like a Java anonymous inner class by omitting a class-name when instantiating the `object`. If we wan't to reuse this object expression we can assign it to a val or var. Unlike object declarations, anonymous objects aren't singletons. Every time an object expression is executed, a new instance is created. An anonymous object can inherit from multiple interfaces. The anonymous classes can access variables defined in the enclosing function, these variables don't need to be final like in Java.
```kotlin
val squakSound = object : SoundMaker() {
    override fun makeSound() : String = "Squak squak"
}
```

# Nullability
Kotlin is null-safe, meaning that we must explicitly declare that a variable can be null, this in turn must be handled when using the variable. We can handle this in three ways, normal Java-style nullcheck, __safe call__ operator `?`, or __not null assertion__ operator `!!`.
```kotlin
fun nullable(nullableStr: String?) {    // nullableStr = "HELLO"
    println(nullableStr.toLowerCase())  // Compile error since we don't handle the null case
    println(nullableStr?.toLowerCase()) // hello
    println(nullableStr!!.toLowerCase()) // hello
    println(nullableStr?.toLowerCase() ?: "default")    //prints default if nullableStr is null. Elvis-operator ?: provides a fallback value
    if (nullableStr != null) {  // Smart check
        println(nullableStr.toLowerCase())  //No longer a need to use either ? or !!
    }
}
```
## __safe call__ `?` and __elvis operator__ `?:`
Tries to access the variable / method if the value isn't null. If the value is null, it will short-circuit and return null. If we want to provide a different default value when the value is null, use the __elvis operator__: `nullableStr?.toLowerCase() ?: "default value"` 
## __not null assertion__ !!
Be careful with `!!` as it can lead to KotlinNullPointerException when the `!!` is used, not later. It's useful if we for some reason know that it can't be null but the compiler is unable to verify this itself, e.g a custom validator called before the variable is accessed using `!!`.
## Extensions for nullables types
We can define extension functions for null-values! Since they can be called with a null-value as the receiver object, we don't need to perform null-checks.
```kotlin
fun String?.isNullOrBlank(): Boolean = this == null || this.isBlank()
fun verifyUserInput(input: String?) {
    if (input.isNullOrBlank()) {
        // do something
    }
}
```
## Nullability and Java, Platform types
Sometimes Java code contains information about nullabitlity, expressed using annotations. @Nullable -> String? and @NotNull -> String in Kotlin. Kotlin recognizes nullability annotations from the following packages: __android.support.annotation, javax.annotation, org.jetbrains.annotations__. If we don't have these annotations, the Java type becomes a `platform type` in Kotlin. A `platform type` is a type for which Kotlin doesn't have nullability information. Platform types were introduced so that we won't need to perform null-checks for all calls on Java-defined classes. Kotlin denotes these platform types by postfixing an exclamation mark to the type, e.g `String!`.
## Smart casts
In Kotlin, casting is achieved using the `as` keyword. It can be combined with both the __safe call__ and __elvis operator__. Kotlin also has smart casts like shown below:
```kotlin
class Person(val name: String) {
    override fun equals(o: Any?): Boolean {
        val otherPerson = o as? Person ?: return false
        //Due to the smart case above, we don't need to cast it again below
        return otherPerson.name == name
    }
}
```

# Lambdas
Below is the syntax for a lambda expression. Always curly braces, no paranthesis around arguments. Lambdas are assignable to var / val.
```kotlin
//           Parameters          Body
//            /      \            |
val sum = { x: Int, y: Int  ->  x + y }  //Must always be in curly braces, never parantheses.


{ println(42) }()  //Executes the lambdaexpression immediately, output: 42
//The above should not be done, if the block should be executed immediately, use run { ... }
run { println(42) }
```
We can omit definig the arguments if there is only argument, its default name is `it`, i.e `people.maxBy { it.age }`. Avoid using `it` when nesting lambdas, this can lead to confusion. If the last parameter of a method is a lambda, the lambda expression can be moved outside the method's "()".

```kotlin
list.maxBy({ it.number })   //Java way, also legal in Kotlin
list.maxBy() { it.number }  //Kotlin way, but since the are no other parameters the () can be omitted
list.maxBy { it.number }    //Kotlin way way
```
Evry lambda expression is converted into an anonymous object i the compiled Java code unless its an `inline` lambda. This object is reused if it doesn't access any variables from the function where it's defined.
## Lambdas with receivers
The `with` and `apply` functions are examples of lambdas with receivers. Lambdas with receivers aren't available in Java, the ability to call methods of a different object in the body of a lambda without any additional qualifiers. The `with` function takes two parameters, the instance it works on, and the lambda expression to process with the instance. The `with` function converts its first argumen into a __receiver__ of the lambda that's passed as a second argument, the receiver can be accessed using `this`. If the outer class has a method that matches the with method's we need to specify which methods should be used like so: `this@OuterClass.methodname()`. The `apply` function is declared as an extension function and works like `with`, but `apply` returns the receiver object. This ise useful when we want to create an instance of a class and immediately initalize some values.
```kotlin
fun alphabet() = with(StringBuilder()) {
    for (letter in 'A'..'Z') { append(letter) }
    append("\nNow I know the alphabet")
    toString()
}

fun alphabet2() = StringBuilder().apply {   //Kotlin has created a function, buildString, that does this
    for (letter in 'A'..'Z') { append(letter) }
    append("\nNow I know the alphabet")
}.toString()

//Instance creation example with init values
fun createViewWithCustomAttributes(context: Context) = TextView(context).apply {
    text = "Sample text"
    textSize = 20.0
    setPadding(10, 0, 0, 0)
}
```
## Return statements in lambdas: return from an enclosing function
If we use the `return` keyword in a lambda, it **returns froms the function in which we called the lambda**, not just from the lambda istelf. Such a return statement i called a **non-local return**, because it returns from a larger block than the block containinug the return statement.
```kotlin
fun lookForAlice(people: List<Person>) {
    peopele.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return  //This will actually cause the entire lookForAlice-function to return due to non-local-return
        }
    }
    println("Alice is not found")
}
```
The return from the outer function is possibly __only__ if the function that takes the lambda as argument is inlined. We can use **local-return** as well so that the enclosing function continues after the lambda has finsihed, we do this be adding a `label`.
```kotlin
fun lookForAlice(people: List<Person>) {
    peopele.forEach label@{ //We can give the label a custom name if want, for instance check@{...}, 
        if (it.name == "Alice") {
            return@label  
            //return@forEach    could also have been used if we don't want to use label
        }
    }
    println("Alice might be somewhere")
}
```
An alternative to using labels is to write anonymous functions instead. Anonymous functions only return from the anonymous function, not the enclosing one:
```kotlin
fun lookForAlice(people: List<Person>) {
    peopele.forEach(fun (person) {  //No parameter types for anonymous functions
        if (it.name == "Alice") {
            return  //This will actually cause the entire lookForAlice-function to return due to non-local-return
        }
        println("Alice is not found")
    })
}
```
The return rule is quite simple: `return` returns from the closest function declared using the `fun` keyword. Lambda expressions don't use the `fun` keyword, so a `return` in a lambda return from the outer function.
## Member references
A member reference works the same as in Java like so: `ClassName::memberName`, e.g `Person::age`. They also work for extension functions, constructors and package-level functions.
```kotlin
val getAge = { person: Person -> person.age }
//Rewritten using member references
val getAge2 = Person::age

fun toppy() = println("toppy!")
//>>> run(::toppy)
//toppy!

val createPerson = ::Person
val p = createPerson("Alice", 29)

val p = Person("Magga", 28)
val maggasAge = p::age
println(maggasAge())
//28
```
# Kotlin stdlib
## The `let` function
The `let` function makes it easier to deal with nullable expressions. Toghether with a safe-call operator, it allows you to evaluate an expression, check the result for null, and store result in a variable, all in a single, concise expression. One of its most common uses is handling a nullable argument that should be passed to a function that expects a non-null parameter.
```kotlin
var email : String? = ""

//Old Java-ish way
if (email != null) {
    sendEmail(email)
}
// Fancy Kotlin .let-way
email?.let { sendEmailTo(it) }
```
We can use nested `let`-calls if we need to, but this isn't recommended, becomes verbose and hard to read.

## The `use` extension function
There is a useful extension function called `use` that works with autocloseable and closeable resources, much like Java's try-with-resources.

## Late-initialized properties
Many frameworks initiliaze objects in a separate method after the constructor has been called, for example as part of "inversion of control", this means that all of these properties must be nullable and must be called with `!!` or `?.` all the time. To combat this, we can use the `lateinit` keyword. A `lateinit` variable must always be a `var` and not a `val`. If we try to access the property before it's been initialized, we get an exception "lateinit property my-Service has not been initialized". `lateinit` is often used with dependency injection, and is useful when using Spring.
```kotlin
class MyService  {
    fun performAction(): String = "foo"
}

//Nasty non-lateinit way
class MyTestNasty {
    private var myService: MyService? = null
    @Before fun setUp() { 
        myService = MyService()
    }
    @Test fun testAction() {
        Assert.assertEquals("foo", myService!!.performAction())
    }
}
//Nice lateinit-way
class MyTestNice {
    private lateinit var myService: MyService
    @Before fun setUp() {
        myService = MyService()
    }
    @Test fun testAction() {
        Assert.assertEquals("foo", myService.performAction())
    }
}
```
# Operator overloading
In Kotlin we can overload mathematical operators like plus, minus, and also comparators, equals, and get/set and in. 
```kotlin
operator fun String.minus(other: String): String = replace(other, "")
```
Commutativity is not automatically supported in Kotlin, this must be manually implemented, i.e:
```kotlin
operator fun Int.minus(other: Long): Long = toLong() - other
operator fun Long.minus(ohter: Int): Long = this - other.toLong()
```

# Generics 
`reified` type parameters allow us to refer at runtime to the specific types used as type arguments in an inline function call. For normal classes or functions, this isn't possible, because type arguments are erased at runtime. To create an upper bound for the type use `fun <T : Number>`. In effect, a type parameter without an upper bound will have the upper bound `Any?`. Therefore, if we want to guarantee that the value can't be null it must specifially have a non-null upper bound, for instance `Any`, like this: `<T : Any>`. If we need to specifiy multiple constraints things look a little different: 
```kotlin
fun <T> ensureTrailingPeriod(seq: T) where T : CharSequence, T : Appendable {
    // do something...
}
```
## Covariance:
Class covariant: `Producer<A>` is a subtype of `Producer<B>` if A is a subtype of B.
```kotlin
interface Producer<out T> {
    fun produce(): T
}
class Herd<out T : Animal> {
    ...
}

fun takeCareOfCats(cats: Herd<Cat>) {
    for (i in 0 until cats.size) {
        cats[i].cleanLitter()
    }
    feedAll(cats)
}
```

Marking a type parameter of a class as covariant makes it possible to pass values of that class as function arguments and return values when the type arguments don't exactly match the ones in the function definition. `out` is used when the meaning is that a class can produce values of type T but not consume them.
```kotlin
interface Transformer<T> {
    fun transform(t: T)  :  T
//                   |      |
//         "in" position   "out" position
}
```
Contravariance works the exact opposite way of covariance. A `Consumer<A>` is a subtype of `Consumer<B>` if B is a subtype of A. To make this easier, we can say that covariances are "producers" and contravariants are "consumers". For a covariant type `Producer<T>`, the subtyping is preserved, but for a contravariant type `Consumer<T>`, the subtyping is reversed.
```kotlin
//Animal      Producer<Animal>        Consumer<Animal>
// / \               / \                      |
//  |                 |                       |
//  |                 |                      \ /
// Cat          Producer<Cat>           Consumer<Cat>
```
The `in` keyword means values of the corresponding type are `passed in` to methods of this class and consumed by this class.
# Other nice features
## Importing and type alias
Kotlin supports named-imports and typealiasing. Named imports works for everything that is imported, like so `import strings.lastChar as last` and is useful when name-clashes occurs.
## Destructuring
Destructuring allows us to destructure an object into several `var` / `val` in a single expression like so: 
```kotlin
val (name, age) = Person("Magnus", 28)`
```
Achieved by either manually generating `operator fun component1() = name` or automatically when using a `data class`. The standard library allows us to access the first five elements of a container. Destructuring can also be used with loops.
## Triple quoted strings
When using triple qouted strings, characters don't need escaping. These string literals can also contain line breaks. All indentations are included in a triple quoted string, to trim the margin on the left, use `trimMargin()`. String templates can be used with triple qouted strings.
```kotlin
val kotlinLogo = """| //
                   .|//
                   .|/ \ """
println(kotlinlogo.trimMargin("."))
| //
|//
|/ \
```
## Accessing backing fields
The backing field can be accessed using `field`. The accessor visibility can be changed by setting the visiblity modifier in front of `set` or `get`
```kotlin
class User(val name: String) {
    var address: String = "unspecified"
        set(value: String) {
            println("""Address was changed for $name: "$field" -> $value".""".trimIndent())
            field = value
        }
}
```
## Sequences
When performing successive list-manipulation, it can become inefficient with several filter and map calls when the list contains many elements. To improve this, we can use sequences. When using sequences, no intermediate collections are created to store the temporary elements. Sequences are exactly the same as Java 8's streams. As a rule of thumb, only use sequnces for large collections. We distinguish between `intermediate` and `terminal` operations. The __intermediate__ operations won't be performed unless a __terminal__ operation is part of the chain, and are therefore performed lazily. For sequences, all operations in a chain are performed on an element before doing the same operations on the next element.
```kotlin
listOf(1,2,3,4,5).asSequence()
    .filter {
        println("filter $it")
        it % 2 == 0
    }.map {
        println("map: ${it / 2}")
        it / 2
    }.toList()
// Prints the following
//filter 1
//filter 2
//map: 1
//filter 3
//filter 4
//map: 2
//filter 5

listOf(1,2,3,4,5)   //Not a sequence
    .filter {
        println("filter $it")
        it % 2 == 0
    }.map {
        println("map: ${it / 2}")
        it / 2
    }.toList()
// Prints the following
//filter 1
//filter 2
//filter 3
//filter 4
//filter 5
//map: 1
//map: 2
```