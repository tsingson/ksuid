# ksuid [![Go Report Card](https://goreportcard.com/badge/github.com/segmentio/ksuid)](https://goreportcard.com/report/github.com/segmentio/ksuid) [![GoDoc](https://godoc.org/github.com/segmentio/ksuid?status.svg)](https://godoc.org/github.com/segmentio/ksuid) [![Circle CI](https://circleci.com/gh/segmentio/ksuid.svg?style=shield)](https://circleci.com/gh/segmentio/ksuid.svg?style=shield)

ksuid is a Go library that can generate and parse KSUIDs.

# What is a KSUID?

KSUID is for K-Sortable Unique IDentifier. It's a way to generate globally
unique IDs similar to RFC 4122 UUIDs, but contain a time component so they
can be "roughly" sorted by time of creation. The remainder of the KSUID is
randomly generated bytes.

# Why use KSUIDs?

Distributed systems often require unique IDs. There are numerous solutions
out there for doing this, so why KSUID?

## 1. Sortable by Timestamp

Unlike the more common choice of UUIDv4, KSUIDs contain a timestamp component
that allows them to be roughly sorted by generation time. This is obviously not
a strong guarantee as it depends on wall clocks, but is still incredibly useful
in practice.

## 2. No Coordination Required

Snowflake IDs[1] and derivatives require coordination, which significantly
increases the complexity of implementation and creates operations overhead.
While RFC 4122 UUIDv1s do have a time component, there aren't enough bytes of
randomness to provide strong protections against duplicate ID generation.

KSUIDs use 128-bits of pseudorandom data, which provides a 64-times larger
number space than the 122-bits in the well-accepted RFC 4122 UUIDv4 standard.
The additional timestamp component drives down the extremely rare chance of
duplication to the point of near physical infeasibility, even assuming extreme
clock skew (> 24-hours) that would cause other severe anomalies.

1. https://blog.twitter.com/2010/announcing-snowflake

## 3. Lexographically Sortable, Portable Representations

The binary and string representations are lexicographically sortable, which
allows them to be dropped into systems which do not natively support KSUIDs
and retain their k-sortable characteristics.

The string representation is that it is base62-encoded, so that they can "fit"
anywhere alphanumeric strings are accepted.

# How do they work?

KSUIDs are 20-bytes: a 32-bit unsigned integer UTC timestamp and a 128-bit
randomly generated payload. The timestamp uses big-endian encoding, to allow
lexicographic sorting. The timestamp epoch is adjusted to March 5th, 2014,
providing over 100 years of useful life starting at UNIX epoch + 14e8. The
payload uses a cryptographically-strong pseudorandom number generator.

The string representation is fixed at 27-characters encoded using a base62
encoding that also sorts lexicographically.

# Command Line Tool

This package comes with a simple command-line tool `ksuid`. This tool can
generate KSUIDs as well as inspect the internal components for debugging
purposes.

## Usage examples

### Generate 4 KSUID

```sh
$ ./ksuid -n 4
0ujsszwN8NRY24YaXiTIE2VWDTS
0ujsswThIGTUYm2K8FjOOfXtY1K
0ujssxh0cECutqzMgbtXSGnjorm
0ujsszgFvbiEr7CDgE3z8MAUPFt
```

### Inspect the components of a KSUID

Using the inspect formatting on just 1 ksuid:

```sh
$ ./ksuid -f inspect $(./ksuid)

REPRESENTATION:

  String: 0ujtsYcgvSTl8PAuAdqWYSMnLOv
     Raw: 0669F7EFB5A1CD34B5F99D1154FB6853345C9735

COMPONENTS:

       Time: 2017-10-09 21:00:47 -0700 PDT
  Timestamp: 107608047
    Payload: B5A1CD34B5F99D1154FB6853345C9735
```

Using the template formatting on 4 ksuid:

```sh
$ ./ksuid -f template -t '{{ .Time }}: {{ .Payload }}' $(./ksuid -n 4)
2017-10-09 21:05:37 -0700 PDT: 304102BC687E087CC3A811F21D113CCF
2017-10-09 21:05:37 -0700 PDT: EAF0B240A9BFA55E079D887120D962F0
2017-10-09 21:05:37 -0700 PDT: DF0761769909ABB0C7BB9D66F79FC041
2017-10-09 21:05:37 -0700 PDT: 1A8F0E3D0BDEB84A5FAD702876F46543
```

### Generate detailed versions of new KSUID

Generate a new KSUID with the corresponding time using the time formatting:

```sh
$ go run cmd/ksuid/main.go -f time -v
0uk0ava2lavfJwMceJOOEFXEDxl: 2017-10-09 21:56:00 -0700 PDT
```

Generate 4 new KSUID with details using template formatting:

```sh
$ ./ksuid -f template -t '{ "timestamp": "{{ .Timestamp }}", "payload": "{{ .Payload }}", "ksuid": "{{.String}}"}' -n 4
{ "timestamp": "107611700", "payload": "9850EEEC191BF4FF26F99315CE43B0C8", "ksuid": "0uk1Hbc9dQ9pxyTqJ93IUrfhdGq"}
{ "timestamp": "107611700", "payload": "CC55072555316F45B8CA2D2979D3ED0A", "ksuid": "0uk1HdCJ6hUZKDgcxhpJwUl5ZEI"}
{ "timestamp": "107611700", "payload": "BA1C205D6177F0992D15EE606AE32238", "ksuid": "0uk1HcdvF0p8C20KtTfdRSB9XIm"}
{ "timestamp": "107611700", "payload": "67517BA309EA62AE7991B27BB6F2FCAC", "ksuid": "0uk1Ha7hGJ1Q9Xbnkt0yZgNwg3g"}
```

Display the detailed version of a new KSUID:

```sh
$ ./ksuid -f inspect

REPRESENTATION:

  String: 0ujzPyRiIAffKhBux4PvQdDqMHY
     Raw: 066A029C73FC1AA3B2446246D6E89FCD909E8FE8

COMPONENTS:

       Time: 2017-10-09 21:46:20 -0700 PDT
  Timestamp: 107610780
    Payload: 73FC1AA3B2446246D6E89FCD909E8FE8

```
