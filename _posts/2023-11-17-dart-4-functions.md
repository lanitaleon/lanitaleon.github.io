Dart is a true object-oriented language, so even functions are objects and have a type, Function.

This means that functions can be assigned to variables or passed as arguments to other functions.

```dart
bool isNoble(int atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}

isNoble(atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}

bool isNoble(int atomicNumber) => _nobleGases[atomicNumber] != null;
```

# Parameters

A function can have any number of required positional parameters. These can be followed either by named parameters or by optional positional parameters (but not both).

Named parameters are optional unless they’re explicitly marked as required.

Wrapping a set of function parameters in [] marks them as optional positional parameters. If you don’t provide a default value, their types must be nullable as their default value will be null:

```dart
const Scrollbar({super.key, required Widget child});

void enableFlags({bool? bold, required bool hidden}) {...}
```

可选参数, 默认值;

```dart
String say(String from, String msg, [String? device]) {
  var result = '$from says $msg';
  if (device != null) {
    result = '$result with a $device';
  }
  return result;
}
assert(say('Bob', 'Howdy') == 'Bob says Howdy');
assert(say('Bob', 'Howdy', 'smoke signal') == 'Bob says Howdy with a smoke signal');


String say(String from, String msg, [String device = 'carrier pigeon']) {
  var result = '$from says $msg with a $device';
  return result;
}
assert(say('Bob', 'Howdy') == 'Bob says Howdy with a carrier pigeon');
```

main

```dart
void main() {
  print('Hello, World!');
}

// Run the app like this: dart args.dart 1 test
void main(List<String> arguments) {
  print(arguments);

  assert(arguments.length == 2);
  assert(int.parse(arguments[0]) == 1);
  assert(arguments[1] == 'test');
}
```

function as parameter

```dart
void printElement(int element) {
  print(element);
}

var list = [1, 2, 3];

// Pass printElement as a parameter.
list.forEach(printElement);

var loudify = (msg) => '!!! ${msg.toUpperCase()} !!!';
assert(loudify('hello') == '!!! HELLO !!!');
```

# 匿名函数

```dart
([[Type] param1[, …]]) {
  codeBlock;
};

const list = ['apples', 'bananas', 'oranges'];
list.map((item) {
  return item.toUpperCase();
}).forEach((item) {
  print('$item: ${item.length}');
});

list
    .map((item) => item.toUpperCase())
    .forEach((item) => print('$item: ${item.length}'));
```

# lexical scope

跟着括号走;

```dart
bool topLevel = true;

void main() {
  var insideMain = true;

  void myFunction() {
    var insideFunction = true;

    void nestedFunction() {
      var insideNestedFunction = true;

      assert(topLevel);
      assert(insideMain);
      assert(insideFunction);
      assert(insideNestedFunction);
    }
  }
}
```

# 闭包

add2 = (int i) => 2 + i;

```dart
/// Returns a function that adds [addBy] to the
/// function's argument.
Function makeAdder(int addBy) {
  return (int i) => addBy + i;
}

void main() {
  // Create a function that adds 2.
  var add2 = makeAdder(2);

  // Create a function that adds 4.
  var add4 = makeAdder(4);

  assert(add2(3) == 5);
  assert(add4(3) == 7);
}
```

引用比较

```dart
void foo() {} // A top-level function

class A {
  static void bar() {} // A static method
  void baz() {} // An instance method
}

void main() {
  Function x;

  // Comparing top-level functions.
  x = foo;
  assert(foo == x);

  // Comparing static methods.
  x = A.bar;
  assert(A.bar == x);

  // Comparing instance methods.
  var v = A(); // Instance #1 of A
  var w = A(); // Instance #2 of A
  var y = w;
  x = w.baz;

  // These closures refer to the same instance (#2),
  // so they're equal.
  assert(y.baz == x);

  // These closures refer to different instances,
  // so they're unequal.
  assert(v.baz != w.baz);
}
```

# Return values

All functions return a value. If no return value is specified, the statement return null; is implicitly appended to the function body.

```dart
foo() {}
assert(foo() == null);

(String, int) foo() {
  return ('something', 42);
}
```

# Generators

```dart
Iterable<int> naturalsTo(int n) sync* {
  int k = 0;
  while (k < n) yield k++;
}

Stream<int> asynchronousNaturalsTo(int n) async* {
  int k = 0;
  while (k < n) yield k++;
}

Iterable<int> naturalsDownFrom(int n) sync* {
  if (n > 0) {
    yield n;
    yield* naturalsDownFrom(n - 1);
  }
}
```

关于yield

https://stackoverflow.com/questions/55776041/what-does-yield-keyword-do-in-flutter

I think the correct answer for the yield\* is, delegating to another generator rather than call a function. yield* simply delegates to the another generator which means the current generator stops, another generator takes the job until it stops producing. After that one stops producing values, the main generator resumes producing its own values.