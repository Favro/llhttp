Transfer-Encoding header
========================

## Trailing space on chunked body

<!-- meta={"type": "response"} -->
```http
HTTP/1.1 200 OK
Content-Type: text/plain
Transfer-Encoding: chunked

25  \r\n\
This is the data in the first chunk

1C
and this is the second one

0  \r\n\


```

```log
off=0 message begin
off=13 len=2 span[status]="OK"
off=17 len=12 span[header_field]="Content-Type"
off=31 len=10 span[header_value]="text/plain"
off=43 len=17 span[header_field]="Transfer-Encoding"
off=62 len=7 span[header_value]="chunked"
off=73 headers complete status=200 v=1/1 flags=208 content_length=0
off=75 error code=12 reason="Invalid character in chunk size"
```

### `chunked` before other transfer-encoding

<!-- meta={"type": "response"} -->
```http
HTTP/1.1 200 OK
Accept: */*
Transfer-Encoding: chunked, deflate

World
```

```log
off=0 message begin
off=13 len=2 span[status]="OK"
off=17 len=6 span[header_field]="Accept"
off=25 len=3 span[header_value]="*/*"
off=30 len=17 span[header_field]="Transfer-Encoding"
off=49 len=16 span[header_value]="chunked, deflate"
off=69 headers complete status=200 v=1/1 flags=200 content_length=0
off=69 len=5 span[body]="World"
```

### multiple transfer-encoding where chunked is not the last one

<!-- meta={"type": "response"} -->
```http
HTTP/1.1 200 OK
Accept: */*
Transfer-Encoding: chunked
Transfer-Encoding: identity

World
```

```log
off=0 message begin
off=13 len=2 span[status]="OK"
off=17 len=6 span[header_field]="Accept"
off=25 len=3 span[header_value]="*/*"
off=30 len=17 span[header_field]="Transfer-Encoding"
off=49 len=7 span[header_value]="chunked"
off=58 len=17 span[header_field]="Transfer-Encoding"
off=77 len=8 span[header_value]="identity"
off=89 headers complete status=200 v=1/1 flags=200 content_length=0
off=89 len=5 span[body]="World"
```

### `chunkedchunked` transfer-encoding does not enable chunked enconding

This check that the word `chunked` repeat more than once (with or without spaces) does not mistakenly enables chunked encoding.

<!-- meta={"type": "response"} -->
```http
HTTP/1.1 200 OK
Accept: */*
Transfer-Encoding: chunkedchunked

2
OK
0


```

```log
off=0 message begin
off=13 len=2 span[status]="OK"
off=17 len=6 span[header_field]="Accept"
off=25 len=3 span[header_value]="*/*"
off=30 len=17 span[header_field]="Transfer-Encoding"
off=49 len=14 span[header_value]="chunkedchunked"
off=67 headers complete status=200 v=1/1 flags=200 content_length=0
off=67 len=1 span[body]="2"
off=68 len=1 span[body]=cr
off=69 len=1 span[body]=lf
off=70 len=2 span[body]="OK"
off=72 len=1 span[body]=cr
off=73 len=1 span[body]=lf
off=74 len=1 span[body]="0"
off=75 len=1 span[body]=cr
off=76 len=1 span[body]=lf
off=77 len=1 span[body]=cr
off=78 len=1 span[body]=lf
```

## Invalid OBS fold after chunked value

<!-- meta={"type": "response-lenient-headers" } -->
```http
HTTP/1.1 200 OK
Transfer-Encoding: chunked
  abc

5
World
0


```

```log
off=0 message begin
off=13 len=2 span[status]="OK"
off=17 len=17 span[header_field]="Transfer-Encoding"
off=36 len=7 span[header_value]="chunked"
off=45 len=5 span[header_value]="  abc"
off=54 headers complete status=200 v=1/1 flags=200 content_length=0
off=54 len=1 span[body]="5"
off=55 len=1 span[body]=cr
off=56 len=1 span[body]=lf
off=57 len=5 span[body]="World"
off=62 len=1 span[body]=cr
off=63 len=1 span[body]=lf
off=64 len=1 span[body]="0"
off=65 len=1 span[body]=cr
off=66 len=1 span[body]=lf
off=67 len=1 span[body]=cr
off=68 len=1 span[body]=lf
```

