# multiplex

A binary stream multiplexer. Stream multiple streams of binary data over a single binary stream. Like [mux-demux](https://npmjs.org/package/mux-demux) but faster since it only works with binary streams.

[![NPM](https://nodei.co/npm/multiplex.png)](https://nodei.co/npm/multiplex/)

## api

### `var multiplex = require('multiplex')([options], [onStream])`

Returns a new multiplexer. You can use this to create sub-streams. All data written to sub-streams will be emitted through this. If you pipe a multiplex instance to another multiplex instance all substream data will be multiplexed and demultiplexed on the other end.

`onStream` will be called with `(stream, id)` whenever a new remote sub-stream is created with an id that hasn't already been created with `.createStream`.

Options include:

* `opts.limit` - set the max allowed message size. default is no maximum
* `opts.binaryName` - allow use of Buffers as stream names (ids)
* `opts.objName` - allow use of Objects as stream names (ids). see example below

Any other options set in `options` are used as defaults options when creating sub streams.

### `stream = multiplex.createStream([id], [options])`

Creates a new sub-stream with an optional whole string `id` (default is the stream channel id).

Sub-streams are duplex streams.

Options include:

* `opts.chunked` - enables chunked mode on all streams (message framing not guaranteed)
* `opts.halfOpen` - make channels support half open mode meaning that they can be readable but not writable and vice versa

If multiplex is instantiated with opts.objName set to true then you can:

```
var stream = multiplex.createStream({
  name: 'name_of_stream', // optional, will be generated if not set
  my: 'extra',
  attached: 'metadata
})
```

and the onStream callback will receive the entire object in place of an id. You can use this to send additional meta-data to the receiver when creating streams. Only the .name property will be used as the stream name (if it is set). One caveat: Don't use this to send arbitrary objects over which you have no control. The name property _must_ either be unique or not set.

### `stream = multiplex.receiveStream(id, [options])`

Explicitly receive an incoming stream.

This is useful if you have a function that accepts an instance of multiplex
and you want to receive a substream.

### `stream = multiplex.createSharedStream(id, [options])`

Create a shared stream. If both ends create a shared stream with
the same id, writing data on one end will emit the same data on the other end

## events

### `multiplex.on('error', function (err) {})`

Emitted when the outer stream encounters invalid data

### `stream.on('error', function (err) {})`

Emitted if the inner stream is destroyed with an error

### example

```js
var multiplex = require('multiplex')
var plex1 = multiplex()
var stream1 = plex1.createStream()
var stream2 = plex1.createStream()

var plex2 = multiplex(function onStream(stream, id) {
  stream.on('data', function(c) {
    console.log('data', id, c.toString())
  })
})

plex1.pipe(plex2)

stream1.write(new Buffer('stream one!'))
stream2.write(new Buffer('stream two!'))
```

### contributing

multiplex is an **OPEN Open Source Project**. This means that:

> Individuals making significant and valuable contributions are given commit-access to the project to contribute as they see fit. This project is more like an open wiki than a standard guarded open source project.

See the [CONTRIBUTING.md](contributing.md) file for more details.

### contributors

multiplex is only possible due to the excellent work of the following contributors:

<table><tbody><tr><th align="left">maxogden</th><td><a href="https://github.com/maxogden">GitHub/maxogden</a></td></tr>
<tr><th align="left">1N50MN14</th><td><a href="https://github.com/1N50MN14">GitHub/1N50MN14</a></td></tr>
<tr><th align="left">substack</th><td><a href="https://github.com/substack">GitHub/substack</a></td></tr>
<tr><th align="left">mafintosh</th><td><a href="https://github.com/mafintosh">GitHub/mafintosh</a></td></tr>
</tbody></table>
