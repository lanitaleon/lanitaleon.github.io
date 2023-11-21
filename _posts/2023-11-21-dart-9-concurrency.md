# Asynchronous support

The await expression makes execution pause until that object is available.

https://dart.dev/codelabs/async-await

```dart
Future<void> checkVersion() async {
  var version = await lookUpVersion();
  // Do something with version
}

try {
  version = await lookUpVersion();
} catch (e) {
  // React to inability to look up the version
}

var entrypoint = await findEntryPoint();
var exitCode = await runExecutable(entrypoint, args);
await flushThenExit(exitCode);
```

If you get a compile-time error when using await, make sure await is in an async function. 

```dart
void main() async {
  checkVersion();
  print('In main: version is ${await lookUpVersion()}');
}
```

If your function doesn’t return a useful value, make its return type `Future<void>`.

## Handling Streams

Before using await for, be sure that it makes the code clearer and that you really do want to wait for all of the stream’s results. For example, you usually should not use await for for UI event listeners, because UI frameworks send endless streams of events.

```dart
void main() async {
  // ...
  await for (final request in requestServer) {
    handleRequest(request);
  }
  // ...
}
```

# Isolates - Concurrency in Dart

异步场景：IO，HTTP request，通信等;

```dart
const String filename = 'with_keys.json';

void main() {
  // Read some data.
  final fileData = _readFileSync();
  final jsonData = jsonDecode(fileData);

  // Use that data.
  print('Number of JSON keys: ${jsonData.length}');
}

String _readFileSync() {
  final file = File(filename);
  final contents = file.readAsStringSync();
  return contents.trim();
}


const String filename = 'with_keys.json';

void main() async {
  // Read some data.
  final fileData = await _readFileAsync();
  final jsonData = jsonDecode(fileData);

  // Use that data.
  print('Number of JSON keys: ${jsonData.length}');
}

Future<String> _readFileAsync() async {
  final file = File(filename);
  final contents = await file.readAsString();
  return contents.trim();
}
```

## How isolates work

通常为了提高多核CPU利用率，并发场景会使用共享内存的多线程实现;

但是共享状态并发很容易引发错误，也可能导致代码复杂度提升;

https://en.wikipedia.org/wiki/Race_condition#In_software

Dart 没有用多线程，而是给每个运行一个独立的堆栈;

Instead of threads, all Dart code runs inside of isolates.

Each isolate has its own memory heap, ensuring that none of the state in an isolate is accessible from any other isolate. 

Isolates are like threads or processes, but each isolate has its own memory and a single thread running an event loop.

## Event handling

事件按照先进先出的队列顺序执行;

## Background workers

耗时的任务可以用后台任务做，完成后会返回一个消息;

A worker isolate can perform I/O (reading and writing files, for example), set timers, and more. It has its own memory and doesn’t share any state with the main isolate.

## Code examples

If you’re using Flutter, you can use Flutter’s compute function instead of Isolate.run(). 

```dart
const String filename = 'with_keys.json';

void main() async {
  // Read some data.
  final jsonData = await Isolate.run(_readAndParseJson);

  // Use that data.
  print('Number of JSON keys: ${jsonData.length}');
}

Future<Map<String, dynamic>> _readAndParseJson() async {
  final fileData = await File(filename).readAsString();
  final jsonData = jsonDecode(fileData) as Map<String, dynamic>;
  return jsonData;
}
```

https://github.com/dart-lang/samples/blob/main/isolates/bin/send_and_receive.dart

closure 无处不在;

```dart
const String filename = 'with_keys.json';

void main() async {
  // Read some data.
  final jsonData = await Isolate.run(() async {
    final fileData = await File(filename).readAsString();
    final jsonData = jsonDecode(fileData) as Map<String, dynamic>;
    return jsonData;
  });

  // Use that data.
  print('Number of JSON keys: ${jsonData.length}');
}
```

## Sending multiple messages between isolates

Isolate.spawn();

Isolate.exit();

ReceivePort, SendPort;

https://github.com/dart-lang/samples/blob/main/isolates/bin/long_running_isolate.dart

## Performance and isolate groups

When an isolate calls Isolate.spawn(), the two isolates have the same executable code and are in the same isolate group. 

共享代码;

Isolate.spawnUri() 可以拷贝一份代码，但是两个isolate不在一个组里;

Isolate.spawnUri() 比 spawn() 慢;

不同组之间传消息比在一个组里慢;

Flutter doesn’t support Isolate.spawnUri().

web 版限制好多;

The Dart web platform, however, does not support isolates. 

Dart web apps can use web workers to run scripts in background threads similar to isolates.

Starting a web worker is similar to using Isolate.spawnUri to start an isolate.

Web workers don’t have an equivalent API.