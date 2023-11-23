# Hello world

## Variables

```kotlin
val popcorn = 5    // There are 5 boxes of popcorn
val hotdog = 7     // There are 7 hotdogs
var customers = 10 // There are 10 customers in the queue

// Some customers leave the queue
customers = 8
println(customers)
```

## String templates

```kotlin
val customers = 10
println("There are $customers customers")
// There are 10 customers

println("There are ${customers + 1} customers")
// There are 11 customers
```

# Basic types

Integers: Byte, Short, Int, Long;

Unsigned integers: UByte, UShort, UInt, ULong;

Floating-point numbers: Float, Double;

Booleans: Boolean;

Characters: Char;

Strings: String;

```kotlin
// Variable declared without initialization
val d: Int
// Variable initialized
d = 3

// Variable explicitly typed and initialized
val e: String = "hello"

// Variables can be read because they have been initialized
println(d) // 3
println(e) // hello
```

# Collections

Lists, Sets, Maps;

## List

listOf 和 mutableListOf;

```kotlin
// Read only list
val readOnlyShapes = listOf("triangle", "square", "circle")
println(readOnlyShapes)
// [triangle, square, circle]
println("The first item in the list is: ${readOnlyShapes[0]}")
// The first item in the list is: triangle
println("The first item in the list is: ${readOnlyShapes.first()}")
// The first item in the list is: triangle
println("This list has ${readOnlyShapes.count()} items")
// This list has 3 items
println("circle" in readOnlyShapes)

// Mutable list with explicit type declaration
val shapes: MutableList<String> = mutableListOf("triangle", "square", "circle")
println(shapes)
// [triangle, square, circle]

// Add "pentagon" to the list
shapes.add("pentagon") 
println(shapes)  
// [triangle, square, circle, pentagon]

// Remove the first "pentagon" from the list
shapes.remove("pentagon") 
println(shapes)  
// [triangle, square, circle]
```

## Set

setOf 和 mutableSetOf;

```kotlin
// Read-only set
val readOnlyFruit = setOf("apple", "banana", "cherry", "cherry")
println(readOnlyFruit)
// [apple, banana, cherry]
println("This set has ${readOnlyFruit.count()} items")
// true

// Mutable set with explicit type declaration
val fruit: MutableSet<String> = mutableSetOf("apple", "banana", "cherry", "cherry")

fruit.add("dragonfruit")    // Add "dragonfruit" to the set
println(fruit)              // [apple, banana, cherry, dragonfruit]

fruit.remove("dragonfruit") // Remove "dragonfruit" from the set
println(fruit)              // [apple, banana, cherry]
```

## Map

mapOf 和 mutableMapOf;

```kotlin
// Read-only map
val readOnlyJuiceMenu = mapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
println(readOnlyJuiceMenu)
// {apple=100, kiwi=190, orange=100}

println("The value of apple juice is: ${readOnlyJuiceMenu["apple"]}")
// The value of apple juice is: 100

println("This map has ${readOnlyJuiceMenu.count()} key-value pairs")
// This map has 3 key-value pairs

println(readOnlyJuiceMenu.containsKey("kiwi"))

println(readOnlyJuiceMenu.keys)
// [apple, kiwi, orange]
println(readOnlyJuiceMenu.values)
// [100, 190, 100]

// Mutable map with explicit type declaration
val juiceMenu: MutableMap<String, Int> = mutableMapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
println(juiceMenu)
// {apple=100, kiwi=190, orange=100}

juiceMenu.put("coconut", 150) // Add key "coconut" with value 150 to the map
println(juiceMenu)
// {apple=100, kiwi=190, orange=100, coconut=150}

juiceMenu.remove("orange")    // Remove key "orange" from the map
println(juiceMenu)
// {apple=100, kiwi=190, coconut=150}
```

# Control flow

## If

注意三目运算符不支持, 但是可以用 if 省略大括号;

这不就是Java本来的特性...

```kotlin
val d: Int
val check = true

if (check) {
    d = 1
} else {
    d = 2
}

println(d)
// 1

val a = 1
val b = 2

println(if (a > b) a else b) // Returns a value: 2
```

## When

就是 switch;

```kotlin
val obj = "Hello"

when (obj) {
    // Checks whether obj equals to "1"
    "1" -> println("One")
    // Checks whether obj equals to "Hello"
    "Hello" -> println("Greeting")
    // Default statement
    else -> println("Unknown")     
}
// Greeting
```

支持表达式

```kotlin
val temp = 18

val description = when {
    // If temp < 0 is true, sets description to "very cold"
    temp < 0 -> "very cold"
    // If temp < 10 is true, sets description to "a bit cold"
    temp < 10 -> "a bit cold"
    // If temp < 20 is true, sets description to "warm"
    temp < 20 -> "warm"
    // Sets description to "hot" if no previous condition is satisfied
    else -> "hot"             
}
println(description)
// warm
```

## Ranges

感觉是一种有序对象的省略写法;

```kotlin
1..4 is equivalent to 1, 2, 3, 4
1..<4 is equivalent to 1, 2, 3
4 downTo 1 is equivalent to 4, 3, 2, 1
1..5 step 2 is equivalent to 1, 3, 5

'a'..'d' is equivalent to 'a', 'b', 'c', 'd'
'z' downTo 's' step 2 is equivalent to 'z', 'x', 'v', 't'
```

## Loops

这是真的一模一样;

### for

```kotlin
for (number in 1..5) { 
    // number is the iterator and 1..5 is the range
    print(number)
}
// 12345

val cakes = listOf("carrot", "cheese", "chocolate")

for (cake in cakes) {
    println("Yummy, it's a $cake cake!")
}
// Yummy, it's a carrot cake!
// Yummy, it's a cheese cake!
// Yummy, it's a chocolate cake!
```

### while

```kotlin
var cakesEaten = 0
while (cakesEaten < 3) {
    println("Eat a cake")
    cakesEaten++
}
// Eat a cake
// Eat a cake
// Eat a cake

var cakesEaten = 0
var cakesBaked = 0
while (cakesEaten < 3) {
    println("Eat a cake")
    cakesEaten++
}
do {
    println("Bake a cake")
    cakesBaked++
} while (cakesBaked < cakesEaten)
// Eat a cake
// Eat a cake
// Eat a cake
// Bake a cake
// Bake a cake
// Bake a cake
```

# Functions

关键字 fun;

```kotlin
fun hello() {
    return println("Hello, world!")
}

fun main() {
    hello()
    // Hello, world!
}
```

参数和返回值;

```kotlin
fun sum(x: Int, y: Int): Int {
    return x + y
}

fun main() {
    println(sum(1, 2))
    // 3
}
```

## Named arguments

```kotlin
fun printMessageWithPrefix(message: String, prefix: String) {
    println("[$prefix] $message")
}

fun main() {
    // Uses named arguments with swapped parameter order
    printMessageWithPrefix(prefix = "Log", message = "Hello")
    // [Log] Hello
}
```

## Default parameter values

```kotlin
fun printMessageWithPrefix(message: String, prefix: String = "Info") {
    println("[$prefix] $message")
}

fun main() {
    // Function called with both parameters
    printMessageWithPrefix("Hello", "Log") 
    // [Log] Hello
    
    // Function called only with message parameter
    printMessageWithPrefix("Hello")        
    // [Info] Hello
    
    printMessageWithPrefix(prefix = "Log", message = "Hello")
    // [Log] Hello
}
```

## Functions without return

```kotlin
fun printMessage(message: String) {
    println(message)
    // `return Unit` or `return` is optional
}

fun main() {
    printMessage("Hello")
    // Hello
}
```

## Single-expression functions

```kotlin
fun sum(x: Int, y: Int): Int {
    return x + y
}

fun main() {
    println(sum(1, 2))
    // 3
}

fun sum(x: Int, y: Int) = x + y

fun main() {
    println(sum(1, 2))
    // 3
}
```

## Lambda expressions

```kotlin
fun uppercaseString(string: String): String {
    return string.uppercase()
}
fun main() {
    println(uppercaseString("hello"))
    // HELLO
}

fun main() {
    println({ string: String -> string.uppercase() }("hello"))
    // HELLO
}

fun main() {
    val upperCaseString = { string: String -> string.uppercase() }
    println(upperCaseString("hello"))
    // HELLO
}
```

作为参数传递;

```kotlin
val numbers = listOf(1, -2, 3, -4, 5, -6)
val positives = numbers.filter { x -> x > 0 }
val negatives = numbers.filter { x -> x < 0 }
println(positives)
// [1, 3, 5]
println(negatives)
// [-2, -4, -6]
```

函数的参数类型;

```kotlin
val upperCaseString: (String) -> String = { string -> string.uppercase() }

fun main() {
    println(upperCaseString("hello"))
    // HELLO
}
```

函数返回值;

```kotlin
fun toSeconds(time: String): (Int) -> Int = when (time) {
    "hour" -> { value -> value * 60 * 60 }
    "minute" -> { value -> value * 60 }
    "second" -> { value -> value }
    else -> { value -> value }
}

fun main() {
    val timesInMinutes = listOf(2, 10, 15, 1)
    val min2sec = toSeconds("minute")
    val totalTimeInSeconds = timesInMinutes.map(min2sec).sum()
    println("Total time is $totalTimeInSeconds secs")
    // Total time is 1680 secs
}
```

自执行;

```kotlin
println({ string: String -> string.uppercase() }("hello"))
// HELLO
```

trailing lambdas

如果只有一个lambda参数或者lambda是最后一个参数可以写成:

```kotlin
// The initial value is zero. 
// The operation sums the initial value with every item in the list cumulatively.
println(listOf(1, 2, 3).fold(0, { x, item -> x + item })) // 6

// Alternatively, in the form of a trailing lambda
println(listOf(1, 2, 3).fold(0) { x, item -> x + item })  // 6
```

# Classes

```kotlin
class Contact(val id: Int, var email: String)

class Contact(val id: Int, var email: String) {
    val category: String = ""
}

class Contact(val id: Int, var email: String = "example@gmail.com") {
    val category: String = "work"
}
```

## 创建实例

```kotlin
class Contact(val id: Int, var email: String)

fun main() {
    val contact = Contact(1, "mary@gmail.com")
    // Prints the value of the property: email
    println(contact.email)           
    // mary@gmail.com

    // Updates the value of the property: email
    contact.email = "jane@gmail.com"
    
    // Prints the new value of the property: email
    println(contact.email)           
    // jane@gmail.com
}
```

## 成员函数

```kotlin
class Contact(val id: Int, var email: String) {
    fun printId() {
        println(id)
    }
}

fun main() {
    val contact = Contact(1, "mary@gmail.com")
    // Calls member function printId()
    contact.printId()           
    // 1
}
```

## Data classes

DTO? 提供了一些简化的读数据的方式;

toString

```kotlin
data class User(val name: String, val id: Int)

// Automatically uses toString() function so that output is easy to read
println(user)            
// User(name=Alex, id=1)
```

compare

```kotlin
val user = User("Alex", 1)
val secondUser = User("Alex", 1)
val thirdUser = User("Max", 2)

// Compares user to second user
println("user == secondUser: ${user == secondUser}") 
// user == secondUser: true

// Compares user to third user
println("user == thirdUser: ${user == thirdUser}")   
// user == thirdUser: false
```

copy instance

```kotlin
val user = User("Alex", 1)
val secondUser = User("Alex", 1)
val thirdUser = User("Max", 2)

// Creates an exact copy of user
println(user.copy())       
// User(name=Alex, id=1)

// Creates a copy of user with name: "Max"
println(user.copy("Max"))  
// User(name=Max, id=1)

// Creates a copy of user with id: 3
println(user.copy(id = 3)) 
// User(name=Alex, id=3)
```

# Null safety

和 dart 一样，致力于在编译阶段就解决掉 null 检查;

```kotlin
fun main() {
    // neverNull has String type
    var neverNull: String = "This can't be null"

    // Throws a compiler error
    neverNull = null

    // nullable has nullable String type
    var nullable: String? = "You can keep a null here"

    // This is OK  
    nullable = null

    // By default, null values aren't accepted
    var inferredNonNull = "The compiler assumes non-nullable"

    // Throws a compiler error
    inferredNonNull = null

    // notNull doesn't accept null values
    fun strLength(notNull: String): Int {                 
        return notNull.length
    }

    println(strLength(neverNull)) // 18
    println(strLength(nullable))  // Throws a compiler error
}
```

## Use safe calls

接收null或者执行结果;

```kotlin
fun lengthString(maybeString: String?): Int? = maybeString?.length

fun main() { 
    var nullString: String? = null
    println(lengthString(nullString))
    // null
}

fun main() {
    var nullString: String? = null
    println(nullString?.uppercase())
    // null
}
```