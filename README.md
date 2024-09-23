# tcp-proxy

A small TCP proxy written in Go, forked from https://github.com/jpillora/go-tcp-proxy

This project was intended for debugging text-based protocols. The next version will address binary protocols.

## Install

**Source**

``` sh
$ go get -v github.com/dtimm/go-tcp-proxy/cmd/tcp-proxy
```

## Usage

```
$ tcp-proxy --help
Usage of tcp-proxy:
  -c    output ansi colors
  -d duration
        delay time as golang duration
  -h    output hex
  -l string
        local address (default ":9999")
  -match string
        match regex (in the form 'regex')
  -n    disable nagles algorithm
  -r string
        remote address (default "localhost:80")
  -replace string
        replace regex (in the form 'regex~replacer')
  -unwrap-tls
        remote connection with TLS exposed unencrypted locally
  -v    display server actions
  -vv
        display server actions and all tcp data
```

*Note: Regex match and replace*
**only works on text strings**
*and does NOT work across packet boundaries*

### Simple Example

Since HTTP runs over TCP, we can also use `tcp-proxy` as a primitive HTTP proxy:

```
$ tcp-proxy -r echo.jpillora.com:80
Proxying from localhost:9999 to echo.jpillora.com:80
```

Then test with `curl`:

```
$ curl -H 'Host: echo.jpillora.com' localhost:9999/foo
{
  "method": "GET",
  "url": "/foo"
  ...
}
```

### Delay Example

```
$ tcp-proxy -r echo.jpillora.com:80 -d 200ms
Proxying from localhost:9999 to echo.jpillora.com:80
```

Test with `curl` and it will add 200ms to each end of the request.

### Match Example

```
$ tcp-proxy -r echo.jpillora.com:80 -match 'Host: (.+)'
Proxying from localhost:9999 to echo.jpillora.com:80
Matching Host: (.+)

#run curl again...

Connection #001 Match #1: Host: echo.jpillora.com
```

### Replace Example

```
$ tcp-proxy -r echo.jpillora.com:80 -replace '"ip": "([^"]+)"~"ip": "REDACTED"'
Proxying from localhost:9999 to echo.jpillora.com:80
Replacing "ip": "([^"]+)" with "ip": "REDACTED"
```

```
#run curl again...
{
  "ip": "REDACTED",
  ...
```

*Note: The `-replace` option is in the form `regex~replacer`. Where `replacer` may contain `$N` to substitute in group `N`.*

### Todo

* Implement `tcpproxy.Conn` which provides accounting and hooks into the underlying `net.Conn`
* Verify wire protocols by providing `encoding.BinaryUnmarshaler` to a `tcpproxy.Conn`
* Modify wire protocols by also providing a map function
* Implement [SOCKS v5](https://www.ietf.org/rfc/rfc1928.txt) to allow for user-decided remote addresses