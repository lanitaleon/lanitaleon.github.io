跟Java大差不差，随便看看可以了;

# Loops

for, while, do while, break, continue;

```dart
var message = StringBuffer('Dart is fun');
for (var i = 0; i < 5; i++) {
  message.write('!');
}

for (final candidate in candidates) {
  candidate.interview();
}

for (final Candidate(:name, :yearsExperience) in candidates) {
  print('$name has $yearsExperience of experience.');
}

var collection = [1, 2, 3];
collection.forEach(print); // 1 2 3

do {
  printLine();
} while (!atEndOfPage());

while (true) {
  if (shutDownRequested()) break;
  processIncomingRequests();
}

for (int i = 0; i < candidates.length; i++) {
  var candidate = candidates[i];
  if (candidate.yearsExperience < 5) {
    continue;
  }
  candidate.interview();
}

candidates
    .where((c) => c.yearsExperience >= 5)
    .forEach((c) => c.interview());
```

闭包的处理和js不太一样惹;

```dart
var callbacks = [];
for (var i = 0; i < 2; i++) {
  callbacks.add(() => print(i));
}

for (final c in callbacks) {
  c();
}
```

The output is 0 and then 1, as expected. In contrast, the example would print 2 and then 2 in JavaScript.

# Branches

if, if-case, switch, try catch, throw;

```dart
if (pair case [int x, int y]) {
  print('Was coordinate array $x,$y');
} else {
  throw FormatException('Invalid coordinates.');
}
```

## switch

When the value matches a case’s pattern, the case body executes. Non-empty case clauses jump to the end of the switch after completion. They do not require a break statement. Other valid ways to end a non-empty case clause are a continue, throw, or return statement.

```dart
var command = 'OPEN';
switch (command) {
  case 'CLOSED':
    executeClosed();
  case 'PENDING':
    executePending();
  default:
    executeUnknown();
}

var command = 'OPEN';
switch (command) {
  case 'OPEN':
    executeOpen();
    continue newCase; // Continues executing at the newCase label.

  case 'DENIED': // Empty case falls through.
  case 'CLOSED':
    executeClosed(); // Runs for both DENIED and CLOSED,

  newCase:
  case 'PENDING':
    executeNowClosed(); // Runs for both OPEN and PENDING.
}
```

注意上面这个 continue:

command = 'OPEN' 会执行 executeOpen() 和  executeNowClosed();

command = 'PENDING' 会执行 executeNowClosed();

## switch expression

新语言是有点优雅;

```dart
// Where slash, star, comma, semicolon, etc., are constant variables...
switch (charCode) {
  case slash || star || plus || minus: // Logical-or pattern
    token = operator(charCode);
  case comma || semicolon: // Logical-or pattern
    token = punctuation(charCode);
  case >= digit0 && <= digit9: // Relational and logical-and patterns
    token = number();
  default:
    throw FormatException('Invalid');
}

token = switch (charCode) {
  slash || star || plus || minus => operator(charCode),
  comma || semicolon => punctuation(charCode),
  >= digit0 && <= digit9 => number(),
  _ => throw FormatException('Invalid')
};
```

## nullable

A default case \(default or \_\) covers all possible values that can flow through a switch. This makes a switch on any type exhaustive.

```dart
// Non-exhaustive switch on bool?, missing case to match null possibility:
switch (nullableBool) {
  case true:
    print('yes');
  case false:
    print('no');
}


switch (nullableBool) {
  case true:
    print('yes');
  case false:
    print('no');
  case _:
    print('null');
}


sealed class Shape {}

class Square implements Shape {
  final double length;
  Square(this.length);
}

class Circle implements Shape {
  final double radius;
  Circle(this.radius);
}

double calculateArea(Shape shape) => switch (shape) {
      Square(length: var l) => l * l,
      Circle(radius: var r) => math.pi * r * r
    };
```

If anyone were to add a new subclass of Shape, this switch expression would be incomplete. Exhaustiveness checking would inform you of the missing subtype. This allows you to use Dart in a somewhat functional algebraic datatype style.

因为有这种检查，可以放心使用这种结构;

## Guard clause

Guards evaluate an arbitrary boolean expression after matching. This allows you to add further constraints on whether a case body should execute. When the guard clause evaluates to false, execution proceeds to the next case rather than exiting the entire switch.

```dart
// Switch statement:
switch (something) {
  case somePattern when some || boolean || expression:
    //             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Guard clause.
    body;
}

// Switch expression:
var value = switch (something) {
  somePattern when some || boolean || expression => body,
  //               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Guard clause.
}

// If-case statement:
if (something case somePattern when some || boolean || expression) {
  //                           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Guard clause.
  body;
}
```
