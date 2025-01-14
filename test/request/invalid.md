Invalid requests
================

### ICE protocol and GET method

<!-- meta={"type": "request"} -->
```http
GET /music/sweet/music ICE/1.0
Host: example.com


```

```log
off=0 message begin
off=4 len=18 span[url]="/music/sweet/music"
off=27 error code=8 reason="Expected SOURCE method for ICE/x.x request"
```

### ICE protocol, but not really

<!-- meta={"type": "request"} -->
```http
GET /music/sweet/music IHTTP/1.0
Host: example.com


```

```log
off=0 message begin
off=4 len=18 span[url]="/music/sweet/music"
off=24 error code=8 reason="Expected HTTP/"
```

### Headers separated by CR

<!-- meta={"type": "request"} -->
```http
GET / HTTP/1.1
Foo: 1\rBar: 2


```

```log
off=0 message begin
off=4 len=1 span[url]="/"
off=16 len=3 span[header_field]="Foo"
off=21 len=1 span[header_value]="1"
off=23 error code=3 reason="Missing expected LF after header value"
```

### Headers separated by LF

<!-- meta={"type": "request"} -->
```http
POST / HTTP/1.1
Host: localhost:5000
x:x\nTransfer-Encoding: chunked

1
A
0

```

```log
off=0 message begin
off=5 len=1 span[url]="/"
off=17 len=4 span[header_field]="Host"
off=23 len=14 span[header_value]="localhost:5000"
off=39 len=1 span[header_field]="x"
off=41 len=1 span[header_value]="x"
off=42 error code=10 reason="Invalid header value char"
```

### Empty headers separated by CR

<!-- meta={"type": "request" } -->
```http
POST / HTTP/1.1
Connection: Close
Host: localhost:5000
x:\rTransfer-Encoding: chunked

1
A
0

```

```log
off=0 message begin
off=5 len=1 span[url]="/"
off=7 url complete
off=17 len=10 span[header_field]="Connection"
off=28 header_field complete
off=29 len=5 span[header_value]="Close"
off=36 header_value complete
off=36 len=4 span[header_field]="Host"
off=41 header_field complete
off=42 len=14 span[header_value]="localhost:5000"
off=58 header_value complete
off=58 len=1 span[header_field]="x"
off=60 header_field complete
off=61 error code=2 reason="Expected LF after CR"
```

### Empty headers separated by LF

<!-- meta={"type": "request"} -->
```http
POST / HTTP/1.1
Host: localhost:5000
x:\nTransfer-Encoding: chunked

1
A
0

```

```log
off=0 message begin
off=5 len=1 span[url]="/"
off=17 len=4 span[header_field]="Host"
off=23 len=14 span[header_value]="localhost:5000"
off=39 len=1 span[header_field]="x"
off=42 error code=10 reason="Invalid header value char"
```

### Invalid header token #1

<!-- meta={"type": "request", "noScan": true} -->
```http
GET / HTTP/1.1
Fo@: Failure


```

```log
off=0 message begin
off=4 len=1 span[url]="/"
off=18 error code=10 reason="Invalid header token"
```

### Invalid header token #2

<!-- meta={"type": "request", "noScan": true} -->
```http
GET / HTTP/1.1
Foo\01\test: Bar


```

```log
off=0 message begin
off=4 len=1 span[url]="/"
off=19 error code=10 reason="Invalid header token"
```

### Invalid method

<!-- meta={"type": "request"} -->
```http
MKCOLA / HTTP/1.1


```

```log
off=0 message begin
off=5 error code=6 reason="Expected space after method"
```

### Illegal header field name line folding

<!-- meta={"type": "request", "noScan": true} -->
```http
GET / HTTP/1.1
name
 : value


```

```log
off=0 message begin
off=4 len=1 span[url]="/"
off=20 error code=10 reason="Invalid header token"
```

### Corrupted Connection header

<!-- meta={"type": "request", "noScan": true} -->
```http
GET / HTTP/1.1
Host: www.example.com
Connection\r\033\065\325eep-Alive
Accept-Encoding: gzip


```

```log
off=0 message begin
off=4 len=1 span[url]="/"
off=16 len=4 span[header_field]="Host"
off=22 len=15 span[header_value]="www.example.com"
off=49 error code=10 reason="Invalid header token"
```

### Corrupted header name

<!-- meta={"type": "request", "noScan": true} -->
```http
GET / HTTP/1.1
Host: www.example.com
X-Some-Header\r\033\065\325eep-Alive
Accept-Encoding: gzip


```

```log
off=0 message begin
off=4 len=1 span[url]="/"
off=16 len=4 span[header_field]="Host"
off=22 len=15 span[header_value]="www.example.com"
off=52 error code=10 reason="Invalid header token"
```

### Missing CR between headers

<!-- meta={"type": "request", "noScan": true} -->

```http
GET / HTTP/1.1
Host: localhost
Dummy: x\nContent-Length: 23
GET / HTTP/1.1
Dummy: GET /admin HTTP/1.1
Host: localhost
```

```log
off=0 message begin
off=4 len=1 span[url]="/"
off=16 len=4 span[header_field]="Host"
off=22 len=9 span[header_value]="localhost"
off=33 len=5 span[header_field]="Dummy"
off=40 len=1 span[header_value]="x"
off=41 error code=10 reason="Invalid header value char"
```

### Spaces before headers

<!-- meta={ "type": "request" } -->

```http
POST /hello HTTP/1.1
Host: localhost
Foo: bar
 Content-Length: 38

GET /bye HTTP/1.1
Host: localhost


```

```log
off=0 message begin
off=0 len=4 span[method]="POST"
off=4 method complete
off=5 len=6 span[url]="/hello"
off=12 url complete
off=17 len=3 span[version]="1.1"
off=20 version complete
off=22 len=4 span[header_field]="Host"
off=27 header_field complete
off=28 len=9 span[header_value]="localhost"
off=39 header_value complete
off=39 len=3 span[header_field]="Foo"
off=43 header_field complete
off=44 len=3 span[header_value]="bar"
off=49 error code=10 reason="Unexpected whitespace after header value"
```

### Spaces before headers (lenient)

<!-- meta={ "type": "request-lenient-headers" } -->

```http
POST /hello HTTP/1.1
Host: localhost
Foo: bar
 Content-Length: 38

GET /bye HTTP/1.1
Host: localhost


```

```log
off=0 message begin
off=0 len=4 span[method]="POST"
off=4 method complete
off=5 len=6 span[url]="/hello"
off=12 url complete
off=17 len=3 span[version]="1.1"
off=20 version complete
off=22 len=4 span[header_field]="Host"
off=27 header_field complete
off=28 len=9 span[header_value]="localhost"
off=39 header_value complete
off=39 len=3 span[header_field]="Foo"
off=43 header_field complete
off=44 len=3 span[header_value]="bar"
off=49 len=19 span[header_value]=" Content-Length: 38"
off=70 header_value complete
off=72 headers complete method=3 v=1/1 flags=0 content_length=0
off=72 message complete
off=72 reset
off=72 message begin
off=72 len=3 span[method]="GET"
off=75 method complete
off=76 len=4 span[url]="/bye"
off=81 url complete
off=86 len=3 span[version]="1.1"
off=89 version complete
off=91 len=4 span[header_field]="Host"
off=96 header_field complete
off=97 len=9 span[header_value]="localhost"
off=108 header_value complete
off=110 headers complete method=1 v=1/1 flags=0 content_length=0
off=110 message complete
```
