# Overview & usage

Class modifiers, besides abstract, require a language version of at least 3.0.

abstract, base, final, interface, sealed, mixin;

## abstract

既可以extend，又可以implement;

https://stackoverflow.com/questions/70824365/dart-implements-extends-for-abstract-class

```dart
// Library a.dart
abstract class Vehicle {
  void moveForward(int meters);
}

// Library b.dart
import 'a.dart';

// Error: Cannot be constructed
Vehicle myVehicle = Vehicle();

// Can be extended
class Car extends Vehicle {
  int passengers = 4;
  // ···
}

// Can be implemented
class MockVehicle implements Vehicle {
  @override
  void moveForward(int meters) {
    // ...
  }
}
```

## base

只能extend;

You must mark any class which implements or extends a base class as base, final, or sealed. This prevents outside libraries from breaking the base class guarantees.

```dart
// Library a.dart
base class Vehicle {
  void moveForward(int meters) {
    // ...
  }
}

// Library b.dart
import 'a.dart';

// Can be constructed
Vehicle myVehicle = Vehicle();

// Can be extended
base class Car extends Vehicle {
  int passengers = 4;
  // ...
}

// ERROR: Cannot be implemented
base class MockVehicle implements Vehicle {
  @override
  void moveForward() {
    // ...
  }
}
```

## interface

只能implement;

Other libraries can’t override methods that the interface class’s own methods might later call in unexpected ways.

https://en.wikipedia.org/wiki/Fragile_base_class

```dart
// Library a.dart
interface class Vehicle {
  void moveForward(int meters) {
    // ...
  }
}

// Library b.dart
import 'a.dart';

// Can be constructed
Vehicle myVehicle = Vehicle();

// ERROR: Cannot be inherited
class Car extends Vehicle {
  int passengers = 4;
  // ...
}

// Can be implemented
class MockVehicle implements Vehicle {
  @override
  void moveForward(int meters) {
    // ...
  }
}
```

## abstract interface

The most common use for the interface modifier is to define a pure interface.

Like an interface class, other libraries can implement, but cannot inherit, a pure interface. Like an abstract class, a pure interface can have abstract members.

## sealed

用来干嘛呢，定义一种enum模式的继承？

To create a known, enumerable set of subtypes, use the sealed modifier.

The compiler is aware of any possible direct subtypes because they can only exist in the same library. This allows the compiler to alert you when a switch does not exhaustively handle all possible subtypes in its cases:

```dart
sealed class Vehicle {}

class Car extends Vehicle {}

class Truck implements Vehicle {}

class Bicycle extends Vehicle {}

// ERROR: Cannot be instantiated
Vehicle myVehicle = Vehicle();

// Subclasses can be instantiated
Vehicle myCar = Car();

String getVehicleSound(Vehicle vehicle) {
  // ERROR: The switch is missing the Bicycle subtype or a default case.
  return switch (vehicle) {
    Car() => 'vroom',
    Truck() => 'VROOOOMM',
  };
}
```

## Combining modifiers

[abstract] + [base, interface, final, sealed] + [mixin] + class;

A sealed class is always implicitly abstract.

sealed不需要abstract前缀;

[interface, final, sealed] 不能和 mixin 一起;

# Class modifiers for API maintainers

这一节主要讲 dart3.0 的变动对自开发的库造成的影响以及如何升级，随便看看;

## mixin

Dart 3.0 no longer allows classes to be used as mixins by default.

## sealed

It exists primarily to enable exhaustiveness checking in pattern matching. 

这里switch加了case编译不通过;

```dart
sealed class Amigo {
}
class Lucky extends Amigo {}
class Dusty extends Amigo {}
class Ned extends Amigo {}

String lastName(Amigo amigo) =>
    switch (amigo) {
       Lucky _ => 'Day',
       Dusty _ => 'Bottoms',
       Ned _   => 'Nederlander',
    };

void main() {
  Amigo l = Lucky();
  print(lastName(l));
}
```

The sealed class can’t itself be directly constructible.

保证子类可完全穷举;

Every direct subtype of the sealed type must be in the same library where the sealed type is declared.

不然编译器找不到……

# Reference

https://dart.dev/language/modifier-reference

特性对比表