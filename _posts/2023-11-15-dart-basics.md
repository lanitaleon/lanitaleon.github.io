# variables

## normal

var, null, late;

```dart
var name = 'Bob';
Object name = 'Bob';

String name = 'Bob';
String? name = 'Bob';

int? lineCount;
assert(lineCount == null);
int lineCount2 = 0;

late String description;
void main() {
  description = 'Feijoada!';
  print(description);
}

// This is the program's only call to readThermometer().
late String temperature = readThermometer();
```

## final & const

Although a final object cannot be modified, its fields can be changed. In comparison, a const object and its fields cannot be changed: they’re immutable.

```dart
final name = 'Bob'; // Without a type annotation
final String nickname = 'Bobby';

const Object i = 3; // Where i is a const Object with an int value...
const list = [i as int]; // Use a typecast.
const map = {if (i is int) i: 'int'}; // Use is and collection if.
const set = {if (list is List<int>) ...list}; // ...and a spread.
```

# operators

整除 ~/

```dart
assert(2 + 3 == 5);
assert(2 - 3 == -1);
assert(2 * 3 == 6);
assert(5 / 2 == 2.5); // Result is a double
assert(5 ~/ 2 == 2); // Result is an int
assert(5 % 2 == 1); // Remainder

assert('5/2 = ${5 ~/ 2} r ${5 % 2}' == '5/2 = 2 r 1');
```

as, is, is!

```dart
(employee as Person).firstName = 'Bob';
if (employee is Person) {
  // Type check
  employee.firstName = 'Bob';
}
```

nullable

```dart
// Assign value to a
a = value;
// Assign value to b if b is null; otherwise, b stays the same
b ??= value;

var visibility = isPublic ? 'public' : 'private';
String playerName(String? name) => name ?? 'Guest';
```

cascade notation

```dart
var paint = Paint()
  ..color = Colors.black
  ..strokeCap = StrokeCap.round
  ..strokeWidth = 5.0;

var paint = Paint();
paint.color = Colors.black;
paint.strokeCap = StrokeCap.round;
paint.strokeWidth = 5.0;

querySelector('#confirm') // Get an object.
  ?..text = 'Confirm' // Use its members.
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'))
  ..scrollIntoView();

var button = querySelector('#confirm');
button?.text = 'Confirm';
button?.classes.add('important');
button?.onClick.listen((e) => window.alert('Confirmed!'));
button?.scrollIntoView(); 
```

如果函数返回值是void就不能用级联；

```dart
var sb = StringBuffer();
sb.write('foo')
  ..write('bar'); // Error: The sb.write() call returns void, and you can’t construct a cascade on void.
```

# comments

same as Java

```dart
void main() {
  // TODO: refactor into an AbstractLlamaGreetingFactory?
  print('Welcome to my Llama farm!');
}

void main() {
  /*
   * This is a lot of work. Consider raising chickens.

  Llama larry = Llama();
  larry.feed();
  larry.exercise();
  larry.clean();
   */
}
```

doc, [] can link to class

```dart
/// A domesticated South American camelid (Lama glama).
///
/// Andean cultures have used llamas as meat and pack
/// animals since pre-Hispanic times.
///
/// Just like any other animal, llamas need to eat,
/// so don't forget to [feed] them some [Food].
class Llama {
  String? name;

  /// Feeds your llama [food].
  ///
  /// The typical llama eats one bale of hay per week.
  void feed(Food food) {
    // ...
  }

  /// Exercises your llama with an [activity] for
  /// [timeLimit] minutes.
  void exercise(Activity activity, int timeLimit) {
    // ...
  }
}
```

To parse Dart code and generate HTML documentation, you can use Dart’s documentation generation tool. 

https://dart.dev/tools/dart-doc

```shell
cd my_app
dart pub get
dart doc .
Documenting my_app...
...
Success! Docs generated into /Users/me/projects/my_app/doc/api
```

By default, the documentation files are static HTML files, placed in the doc/api directory. You can create the files in a different directory with the --output flag.

```shell
dart help doc
```

dart sdk文档，没咋看。

https://api.dart.dev/stable/3.1.5/index.html

# metadata

@Deprecated, @deprecated, @override, @pragma

You can use @deprecated if you don’t want to specify a message. However, we recommend always specifying a message with @Deprecated.

自定义注解

```dart
class Todo {
  final String who;
  final String what;

  const Todo(this.who, this.what);
}

@Todo('Dash', 'Implement this function')
void doSomething() {
  print('Do something');
}
```

看上去作用不是很大，读不到注解对象的参数；

# libraries & imports

```dart
import 'package:lib1/lib1.dart';
import 'package:lib2/lib2.dart' as lib2;

// Uses Element from lib1.
Element element1 = Element();

// Uses Element from lib2.
lib2.Element element2 = lib2.Element();

// Import only foo.
import 'package:lib1/lib1.dart' show foo;

// Import all names EXCEPT foo.
import 'package:lib2/lib2.dart' hide foo;
```

懒加载

```dart
import 'package:greetings/hello.dart' deferred as hello;

Future<void> greet() async {
  await hello.loadLibrary();
  hello.printGreeting();
}
```

# keywords

跟Java大差不差

https://dart.dev/language/keywords
