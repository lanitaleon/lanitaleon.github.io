# Classes

Dart is an object-oriented language with classes and mixin-based inheritance. Every object is an instance of a class, and all classes except Null descend from Object. Mixin-based inheritance means that although every class (except for the top class, Object) has exactly one superclass, a class body can be reused in multiple class hierarchies.

## Using constructors

constant

```dart
void main() {
  var a = const ImmutablePoint(1, 1);
  var b = const ImmutablePoint(1, 1);

  print(identical(a, b)); // T
}

class ImmutablePoint {
  final int x;
  final int y;
  const ImmutablePoint(this.x, this.y);
}

// Lots of const keywords here.
const pointAndLine = const {
  'point': const [const ImmutablePoint(0, 0)],
  'line': const [const ImmutablePoint(1, 10), const ImmutablePoint(-2, 11)],
};

// Only one const, which establishes the constant context.
const pointAndLine = {
  'point': [ImmutablePoint(0, 0)],
  'line': [ImmutablePoint(1, 10), ImmutablePoint(-2, 11)],
};
```

这个const简写只要写一段时间flutter就会缝进肌肉记忆，疯狂报warning;

## Getting an object's type

Use a type test operator rather than runtimeType to test an object’s type. In production environments, the test object is Type is more stable than the test object.runtimeType == Type.

```dart
print('The type of a is ${a.runtimeType}');
```

## Instance variables

All uninitialized instance variables have the value null.

All instance variables generate an implicit getter method. 

Non-final instance variables and late final instance variables without initializers also generate an implicit setter method.

If you initialize a non-late instance variable where it’s declared, the value is set when the instance is created, which is before the constructor and its initializer list execute. As a result, non-late instance variable initializers can’t access this.

```dart
class Point {
  double? x; // Declare instance variable x, initially null.
  double? y; // Declare y, initially null.
}

void main() {
  var point = Point();
  point.x = 4; // Use the setter method for x.
  assert(point.x == 4); // Use the getter method for x.
  assert(point.y == null); // Values default to null.
}
```

注意这个unnamed constuctor, 下一节还会介绍;

```dart
void main() {
  var a =  ProfileMark('a');
  var b = ProfileMark.unnamed();

  print(a.name); 
  print(b.name);
}

class ProfileMark {
  final String name;
  final DateTime start = DateTime.now();

  ProfileMark(this.name);
  ProfileMark.unnamed() : name = '';
}
```

final 成员变量尽量初始化掉，如果非要延迟初始化:

- Use a factory constructor.

- Use late final, but be careful: a late final without an initializer adds a setter to the API.

## Implicit interfaces

菀菀类卿; 可多声明;

```dart
class Point implements Comparable, Location {...}
```

注意下重写 \_name 和 greet();

```dart
// A person. The implicit interface contains greet().
class Person {
  // In the interface, but visible only in this library.
  final String _name;

  // Not in the interface, since this is a constructor.
  Person(this._name);

  // In the interface.
  String greet(String who) => 'Hello, $who. I am $_name.';
}

// An implementation of the Person interface.
class Impostor implements Person {
  String get _name => '';

  String greet(String who) => 'Hi $who. Do you know who I am?';
}

String greetBob(Person person) => person.greet('Bob');

void main() {
  print(greetBob(Person('Kathy')));
  print(greetBob(Impostor()));
}
```

## Class variables and methods

### static

Static variables aren’t initialized until they’re used.

```dart
class Queue {
  static const initialCapacity = 16;
  // ···
}
```

Static methods (class methods) don’t operate on an instance, and thus don’t have access to this. 

```dart
import 'dart:math';

class Point {
  double x, y;
  Point(this.x, this.y);

  static double distanceBetween(Point a, Point b) {
    var dx = a.x - b.x;
    var dy = a.y - b.y;
    return sqrt(dx * dx + dy * dy);
  }
}

void main() {
  var a = Point(2, 2);
  var b = Point(4, 4);
  var distance = Point.distanceBetween(a, b);
  assert(2.8 < distance && distance < 2.9);
  print(distance);
}
```

# Constructors

Use this only when there is a name conflict. Otherwise, Dart style omits the this.

```dart
class Point {
  double x = 0;
  double y = 0;

  // Generative constructor with initializing formal parameters:
  Point(this.x, this.y);
}
```

Subclasses don’t inherit constructors from their superclass. A subclass that declares no constructors has only the default (no argument, no name) constructor.

dart你做得好啊;

## Named constructors

```dart
const double xOrigin = 0;
const double yOrigin = 0;

class Point {
  final double x;
  final double y;

  Point(this.x, this.y);

  // Named constructor
  Point.origin()
      : x = xOrigin,
        y = yOrigin;
}
```

## Invoking a non-default superclass constructor

By default, a constructor in a subclass calls the superclass’s unnamed, no-argument constructor. The superclass’s constructor is called at the beginning of the constructor body. 

成熟的语言会自己调用super();

the order of execution is as follows:

- initializer list

- superclass’s no-arg constructor

- main class’s no-arg constructor

```dart
class Person {
  String? firstName;

  Person.fromJson(Map data) {
    print('in Person');
  }
}

class Employee extends Person {
  // Person does not have a default constructor;
  // you must call super.fromJson().
  Employee.fromJson(super.data) : super.fromJson() {
    print('in Employee');
  }
}

void main() {
  var employee = Employee.fromJson({});
  print(employee);
  // Prints:
  // in Person
  // in Employee
  // Instance of 'Employee'
}
```

## Super parameters

```dart
class Vector2d {
  final double x;
  final double y;

  Vector2d(this.x, this.y);
}

class Vector3d extends Vector2d {
  final double z;

  // Forward the x and y parameters to the default super constructor like:
  // Vector3d(final double x, final double y, this.z) : super(x, y);
  Vector3d(super.x, super.y, this.z);
}
```

Super-initializer parameters cannot be positional if the super-constructor invocation already has positional arguments, but they can always be named:

这个写法也是蛮神奇……

```dart
class Vector2d {
  double x;
  double y;

  Vector2d.named({required this.x, required this.y});
}

class Vector3d extends Vector2d {
  double z;

  // Forward the y parameter to the named super constructor like:
  // Vector3d.yzPlane({required double y, required this.z})
  //       : super.named(x: 0, y: y);
  Vector3d.yzPlane({required super.y, required this.z}) : super.named(x: 0);
}

void main() {
  double a = 1.0;
  double b = 9.0;
  Vector3d cube = Vector3d.yzPlane(y: a, z: b);
  print('x: ${cube.x}, y: ${cube.y}, z: ${cube.z}');
}
```

## Initializer list

The right-hand side of an initializer doesn’t have access to `this`.

```dart
// Initializer list sets instance variables before
// the constructor body runs.
Point.fromJson(Map<String, double> json)
    : x = json['x']!,
      y = json['y']! {
  print('In Point.fromJson(): ($x, $y)');
}

Point.withAssert(this.x, this.y) : assert(x >= 0) {
  print('In Point.withAssert(): ($x, $y)');
}
```

## Redirecting constructors

Sometimes a constructor’s only purpose is to redirect to another constructor in the same class. A redirecting constructor’s body is empty, with the constructor call (using this instead of the class name) appearing after a colon (:).

直接写个默认值构造函数和这种，这种勉强简洁点吧，大差不差;

```dart
class Point {
  double x, y;

  // The main constructor for this class.
  Point(this.x, this.y);

  // Delegates to the main constructor.
  Point.alongXAxis(double x) : this(x, 0);
}
```

## Constant constructors

第一节已经见过了, this 可以省略;

```dart
class ImmutablePoint {
  static const ImmutablePoint origin = ImmutablePoint(0, 0);

  final double x, y;

  const ImmutablePoint(this.x, this.y);
}
```

```dart
class ImmutablePoint {
  static const ImmutablePoint origin = ImmutablePoint(0, 0);

  final double x, y;

  const ImmutablePoint(this.x, this.y);
}
```

## Factory constructors

factory竟然是关键字;

Factory constructors have no access to this.

Use the factory keyword when implementing a constructor that doesn’t always create a new instance of its class. For example, a factory constructor might return an instance from a cache, or it might return an instance of a subtype.

```dart
class Logger {
  final String name;
  bool mute = false;

  // _cache is library-private, thanks to
  // the _ in front of its name.
  static final Map<String, Logger> _cache = <String, Logger>{};

  factory Logger(String name) {
    return _cache.putIfAbsent(name, () => Logger._internal(name));
  }

  factory Logger.fromJson(Map<String, Object> json) {
    return Logger(json['name'].toString());
  }

  Logger._internal(this.name);

  void log(String msg) {
    if (!mute) print(msg);
  }
}

var logger = Logger('UI');
logger.log('Button clicked');

var logMap = {'name': 'UI'};
var loggerJson = Logger.fromJson(logMap);
```

# Methods

You may have noticed that some operators, like !=, aren’t in the list of names. That’s because they’re just syntactic sugar. For example, the expression e1 != e2 is syntactic sugar for !(e1 == e2).

## 运算符重载(?)

竟然支持这个东西;

```dart
class Vector {
  final int x, y;

  Vector(this.x, this.y);

  Vector operator +(Vector v) => Vector(x + v.x, y + v.y);
  Vector operator -(Vector v) => Vector(x - v.x, y - v.y);

  @override
  bool operator ==(Object other) =>
      other is Vector && x == other.x && y == other.y;

  @override
  int get hashCode => Object.hash(x, y);
}

void main() {
  final v = Vector(2, 3);
  final w = Vector(2, 2);

  assert(v + w == Vector(4, 5));
  assert(v - w == Vector(0, 1));
}
```

## Getters and setters

注意自定义的写法; 关键字 get set;

```dart
class Rectangle {
  double left, top, width, height;

  Rectangle(this.left, this.top, this.width, this.height);

  // Define two calculated properties: right and bottom.
  double get right => left + width;
  set right(double value) => left = value - width;
  double get bottom => top + height;
  set bottom(double value) => top = value - height;
}

void main() {
  var rect = Rectangle(3, 4, 20, 15);
  assert(rect.left == 3);
  rect.right = 12;
  assert(rect.left == -8);
}
```

## Abstract methods

Instance, getter, and setter methods can be abstract, defining an interface but leaving its implementation up to other classes.

```dart
abstract class Doer {
  // Define instance variables and methods...

  void doSomething(); // Define an abstract method.
}

class EffectiveDoer extends Doer {
  void doSomething() {
    // Provide an implementation, so the method is not abstract here...
  }
}
```

# Extend a class

继承，大差不差;

covariant

https://dart.dev/guides/language/sound-problems#the-covariant-keyword

Some (rarely used) coding patterns rely on tightening a type by overriding a parameter’s type with a subtype, which is invalid. In this case, you can use the covariant keyword to tell the analyzer that you are doing this intentionally. This removes the static error and instead checks for an invalid argument type at runtime.

```dart
class Animal {
  void chase(Animal x) { ... }
}

class Mouse extends Animal { ... }

class Cat extends Animal {
  @override
  void chase(covariant Mouse x) { ... }
}
```

If you override ==, you should also override Object’s hashCode getter. 

```dart
class Person {
  final String firstName, lastName;

  Person(this.firstName, this.lastName);

  // Override hashCode using the static hashing methods
  // provided by the `Object` class.
  @override
  int get hashCode => Object.hash(firstName, lastName);

  // You should generally implement operator `==` if you
  // override `hashCode`.
  @override
  bool operator ==(Object other) {
    return other is Person &&
        other.firstName == firstName &&
        other.lastName == lastName;
  }
}

void main() {
  var p1 = Person('Bob', 'Smith');
  var p2 = Person('Bob', 'Smith');
  var p3 = 'not a person';
  assert(p1.hashCode == p2.hashCode);
  assert(p1 == p2);
  assert(p1 != p3);
}
```

## noSuchMethod()

To detect or react whenever code attempts to use a non-existent method or instance variable, you can override noSuchMethod():

```dart
class A {
  // Unless you override noSuchMethod, using a
  // non-existent member results in a NoSuchMethodError.
  @override
  void noSuchMethod(Invocation invocation) {
    print('You tried to use a non-existent member: '
        '${invocation.memberName}');
  }
}
```

可以调用未实现的方法的两种场景：

a. 声明为 dynamic

b. 重写过noSuchMethod();

# Mixins

代码重用

```dart
class Musician extends Performer with Musical {
  // ···
}

class Maestro extends Person with Musical, Aggressive, Demented {
  Maestro(String maestroName) {
    name = maestroName;
    canConduct = true;
  }
}

mixin Musical {
  bool canPlayPiano = false;
  bool canCompose = false;
  bool canConduct = false;

  void entertainMe() {
    if (canPlayPiano) {
      print('Playing piano');
    } else if (canConduct) {
      print('Waving hands');
    } else {
      print('Humming to self');
    }
  }
}
```

Mixins and mixin classes cannot have an extends clause, and must not declare any generative constructors.

限制可重用的类范围可以这样：

only classes that extend or implement the Musician class can use the mixin MusicalPerformer.

```dart
class Musician {
  // ...
}
mixin MusicalPerformer on Musician {
  // ...
}
class SingerDancer extends Musician with MusicalPerformer {
  // ...
}
```

## mixin class

The mixin class declaration requires a language version of at least 3.0.

A mixin declaration defines a mixin. A class declaration defines a class. A mixin class declaration defines a class that is usable as both a regular class and a mixin, with the same name and the same type.

Any restrictions that apply to classes or mixins also apply to mixin classes:

- Mixins can’t have extends or with clauses, so neither can a mixin class.

- Classes can’t have an on clause, so neither can a mixin class.

abstract mixin class 可以实现和on限制范围差不多的效果;

```dart
abstract mixin class Musician {
  // No 'on' clause, but an abstract method that other types must define if 
  // they want to use (mix in or extend) Musician: 
  void playInstrument(String instrumentName);

  void playPiano() {
    playInstrument('Piano');
  }
  void playFlute() {
    playInstrument('Flute');
  }
}

class Virtuoso with Musician { // Use Musician as a mixin
  void playInstrument(String instrumentName) {
    print('Plays the $instrumentName beautifully');
  }  
} 

class Novice extends Musician { // Use Musician as a class
  void playInstrument(String instrumentName) {
    print('Plays the $instrumentName poorly');
  }  
} 
```

# Enums

All enums automatically extend the Enum class. They are also sealed, meaning they cannot be subclassed, implemented, mixed in, or otherwise explicitly instantiated.

Abstract classes and mixins can explicitly implement or extend Enum, but unless they are then implemented by or mixed into an enum declaration, no objects can actually implement the type of that class or mixin.

```dart
enum Color { red, green, blue }
```

## Declaring enhanced enums

加强枚举类的特殊需求：

- 实例变量必须是final的;

- 构造函数都必须是const的;

- factory构造函数必须返回固定的已知的enum实例;

- index, hashCode, == 不能被重写;

- 不能自定义字段名 values, enum默认有values getter;

- 至少包含一个实例，且所有实例必须定义在enum声明的开头;

- 默认继承自Enum类，所以不能再继承其他类;

- Enhanced enums require a language version of at least 2.17.

```dart
enum Vehicle implements Comparable<Vehicle> {
  car(tires: 4, passengers: 5, carbonPerKilometer: 400),
  bus(tires: 6, passengers: 50, carbonPerKilometer: 800),
  bicycle(tires: 2, passengers: 1, carbonPerKilometer: 0);

  const Vehicle({
    required this.tires,
    required this.passengers,
    required this.carbonPerKilometer,
  });

  final int tires;
  final int passengers;
  final int carbonPerKilometer;

  int get carbonFootprint => (carbonPerKilometer / passengers).round();

  bool get isTwoWheeled => this == Vehicle.bicycle;

  @override
  int compareTo(Vehicle other) => carbonFootprint - other.carbonFootprint;
}
```

## Using enums

上板子;

```dart
final favoriteColor = Color.blue;
if (favoriteColor == Color.blue) {
  print('Your favorite color is blue!');
}

assert(Color.red.index == 0);
assert(Color.green.index == 1);
assert(Color.blue.index == 2);

List<Color> colors = Color.values;
assert(colors[2] == Color.blue);

var aColor = Color.blue;
switch (aColor) {
  case Color.red:
    print('Red as roses!');
  case Color.green:
    print('Green as grass!');
  default: // Without this, you see a WARNING.
    print(aColor); // 'Color.blue'
}

print(Color.blue.name); // 'blue'

print(Vehicle.car.carbonFootprint);
```

# Extension methods

新语言是不一样哈，扩展像呼吸一样简单;

```dart
// lib/string_apis.dart
extension NumberParsing on String {
  int parseInt() {
    return int.parse(this);
  }
  // ···
}

import 'string_apis.dart';
// ···
print('42'.parseInt()); // Use an extension method.
```

## Using extension methods

上板子;

You can’t invoke extension methods on variables of type dynamic. 

扩展方法是静态解析的，所以用不了dynamic;

```dart
dynamic d = '2';
print(d.parseInt()); // Runtime exception: NoSuchMethodError
```

能找到类型就是有类型;

```dart
var v = '2';
print(v.parseInt()); // Output: 2
```

API 冲突，自由的代价;

```dart
// Defines the String extension method parseInt().
import 'string_apis.dart';

// Also defines parseInt(), but hiding NumberParsing2
// hides that extension method.
import 'string_apis_2.dart' hide NumberParsing2;

// ···
// Uses the parseInt() defined in 'string_apis.dart'.
print('42'.parseInt());


// Both libraries define extensions on String that contain parseInt(),
// and the extensions have different names.
import 'string_apis.dart'; // Contains NumberParsing extension.
import 'string_apis_2.dart'; // Contains NumberParsing2 extension.

// ···
// print('42'.parseInt()); // Doesn't work.
print(NumberParsing('42').parseInt());
print(NumberParsing2('42').parseInt());


// Both libraries define extensions named NumberParsing
// that contain the extension method parseInt(). One NumberParsing
// extension (in 'string_apis_3.dart') also defines parseNum().
import 'string_apis.dart';
import 'string_apis_3.dart' as rad;

// ···
// print('42'.parseInt()); // Doesn't work.

// Use the ParseNumbers extension from string_apis.dart.
print(NumberParsing('42').parseInt());

// Use the ParseNumbers extension from string_apis_3.dart.
print(rad.NumberParsing('42').parseInt());

// Only string_apis_3.dart has parseNum().
print('42'.parseNum());
```

## Implementing extension methods

The members of an extension can be methods, getters, setters, or operators. 

```dart
extension NumberParsing on String {
  int parseInt() {
    return int.parse(this);
  }

  double parseDouble() {
    return double.parse(this);
  }
}

extension on String {
  bool get isBlank => trim().isEmpty;
}

extension MyFancyList<T> on List<T> {
  int get doubleLength => length * 2;
  List<T> operator -() => reversed.toList();
  List<List<T>> split(int at) => [sublist(0, at), sublist(at)];
}
```

# Callable objects

To allow an instance of your Dart class to be called like a function, implement the call() method.

```dart
class WannabeFunction {
  String call(String a, String b, String c) => '$a $b $c!';
}

var wf = WannabeFunction();
var out = wf('Hi', 'there,', 'gang');

void main() => print(out);
```