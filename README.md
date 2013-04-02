# Continuable

This library is a replacement for many of the core node.js libraries but using continuable style instead of node callback style.

What is continuable style you say?  Why a continuable function is an async function that returns a continuation.

For example, consider a continuable version of setTimeout, named sleep:

```js
function sleep(ms) {
  return function (callback) {
    setTimeout(callback, ms);
  }
}
```

Here, the sleep function returns a new function that accepts the callback.  That returned function is the continuation.

All continuations must accept only a callback and must call that callback with the typical node.js style `(err, args...)` pattern.

## continuable/stream

A stream in this system is an extremely simple interface at the minimum.  A readable stream is simply any object with a `.read()(callback)` continuation style method.  A writable stream is any object with a `.write(chunk)(callback)` method.

The `.read()(callback)` function will callback with `(null, chunk)` for data events.  It will call `(err)` for errors, and `()` for the end of the stream.

Likewise, you can tell a writable stream to end by passing calling `.write(null)`.

For convenience, this library comes with a `Stream` class that implements `.read()` and `.write(chunk`) where writes to the object can be read out of the object.  There is also internal buffering with high-water/low-water marks for proper flow-control.

```
var Stream = require('continuable/stream');
var stream = new Stream();

// await is a utility that suspends the current fiber and waits for the continuation to resolve.
await(stream.write("Hello"))
var greeting = await(stream.read());
```

## continuable/fs

If you want to interface with the filesystem using continuable APIs, this is your module.  It supports file streams and basic fs primitives as continuable style functions.

```js
// Copy a file using continuable and fibers
var Fiber = require('fibers');
var fs = require('continuable/fs');


Fiber(function () {
  // Create a readable stream
  var fd = await(fs.open("myfile.txt", "r"));
  var readable = new fs.ReadStream(fd);
  
  // Create a writable stream
  fd = await(fs.open("copy.txt", "w"));
  var writable = new fs.WriteStream(fd);
  
  // Pipe one stream to the other using a regular do..while loop.
  do {
    var chunk = await(readable.read());
    await(writable.write(chunk));
  } while (chunk);

}).run();
```
