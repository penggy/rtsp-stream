# rtsp-stream

Encode/Decode an RTSP stream.

This project aims for 100% compliance with [RFC
2326](https://tools.ietf.org/html/rfc2326). If you find something
missing, please [open an
issue](https://github.com/watson/rtsp-stream/issues).

[![Build status](https://travis-ci.org/watson/rtsp-stream.svg?branch=master)](https://travis-ci.org/watson/rtsp-stream)
[![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat)](https://github.com/feross/standard)

## Installation

```
npm install rtsp-stream
```

## Usage

Let's set up a TCP server, listen on port 5000 and have it respond to
RTSP requests:

```js
var net = require('net')
var rtsp = require('rtsp-stream')

var server = net.createServer(function (socket) {
  var decoder = new rtsp.Decoder()
  var encoder = new rtsp.Encoder()

  decoder.on('request', function (req) {
    console.log(req.method, req.uri)

    // output the request body
    req.pipe(process.stdout)

    req.on('end', function () {
      // prepare a new response to the client
      var res = encoder.response()

      res.setHeader('CSeq', req.headers['cseq'])
      res.end('Hello World!')
    })
  })

  // pipe the data from the client into the decoder
  socket.pipe(decoder)

  // ...and pipe the response back to the client from the encoder
  encoder.pipe(socket)
})

server.listen(5000)
```

## API

The rtsp-stream module exposes the following:

- `STATUS_CODES` - List of valid RTSP status codes
- `Decoder` - The decoder object
- `Encoder` - The encoder object
- `Request` - A readable stream representing an incoming RTSP request. Given as
  the first argument to the `Decoder` request event
- `Response` - A writable stream representing a response to the client.
  Generated by `encoder.response()`

### `Decoder`

The `Decoder` object is a writable stream. It emits a `request` event
for every request in the written data.

#### Event: request

Emitted every time a new request header is found. The event listener is
called with a single arguemnt:

- `req` - An instance of `rtspStream.Request`

### `Encoder`

A readable stream. Outputs valid RTSP responses.

### `Request`

Exposes the body of the incoming request by implementing a readable
stream interface.

Also exposes the RTSP request header using the following properties:

#### `Request.rtspVersion`

The RTSP protocol version used in the request. By all intents and
purposes you can expect this to always be `1.0`.

#### `Request.method`

The RTSP request method used in the request. The following are
standardized in [RFC 2326](https://tools.ietf.org/html/rfc2326), but
others are also used in the wild:

- `DESCRIBE`
- `ANNOUNCE`
- `GET_PARAMETER`
- `OPTIONS`
- `PAUSE`
- `PLAY`
- `RECORD`
- `SETUP`
- `SET_PARAMETER`
- `TEARDOWN`

#### `Request.uri`

The RTSP request URI used.

#### `Request.headers`

An object containing the headers used on the RTSP request. Object keys
represent header fields and the object values the header values.

All header field names are lowercased for your convenience.

Values from repeating header fields are joined in an array.

### `Response`

#### `Response.statusCode`

The status code used in the response. Defaults to `200`. For alist of
valid status codes see the `rtspStream.STATUS_CODES` object.

#### `Response.headersSent`

A boolean. `true` if the response headers have flushed.

#### `Response.setHeader(name, value)`

Set a header to be sent along with the RTSP response body. Throws an
error if called after the headers have been flushed.

#### `Response.getHeader(name)`

Get a header value. Case insensitive.

#### `Response.removeHeader(name)`

Remove a header so it is not sent to the client. Case insensitive.
Throws an error if called after the headers have been flushed.

#### `Response.writeHead([statusCode[, statusMessage][, headers]])`

Force writing of the RTSP response headers to the client. Will be called
automatically on the first to call to either `response.write()` or
`response.end()`.

Throws an error if called after the headers have been flushed.

Arguments:

- `statusCode` - Set a custom status code (overrides the
  `response.statusCode` property)
- `statusMessage` - Set a custom status message related to the status
  code (e.g. the default status message for the status code `200` is
  `OK`)
- `headers` - A key/value headers object used to set extra headers to be
  sent along with the RTSP response. This will augment the headers
  already set using the `response.setHeader()` function

## License

MIT
