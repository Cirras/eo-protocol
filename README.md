# EO Protocol

An XML-based specification of the Endless Online network protocol and data files.

## Motivation

Most of this information has been available for some time. It's been scattered through different projects, only partially documented, or living in the heads of various community members.

The idea is to put all of that collective knowledge in one place.

## Use cases

- Documentation
- Code generation

## Projects

Community projects based on this specification.

### Libraries

1. **[eolib-go](https://github.com/ethanmoffat/eolib-go)** ([@ethanmoffat](https://github.com/ethanmoffat))
    * Core Golang library for writing Endless Online applications.
2. **[eolib-java](https://github.com/cirras/eolib-java)** ([@cirras](https://github.com/cirras))
    * Core Java library for writing Endless Online applications.
3. **[eolib-python](https://github.com/cirras/eolib-python)** ([@cirras](https://github.com/cirras))
    * Core Python library for writing Endless Online applications.
4. **[eolib-rs](https://github.com/sorokya/eolib-rs)** ([@sorokya](https://github.com/sorokya))
    * Core Rust library for writing Endless Online applications.
5. **[eolib-ts](https://github.com/cirras/eolib-ts)** ([@cirras](https://github.com/cirras))
    * Core Typescript library for writing Endless Online applications.

### Documentation

1. **[eo-protocol-web](https://eoserv.net/eo-protocol-web/)** ([@tehsausage](https://github.com/tehsausage))
    * Generated protocol documentation site.

## Acknowledgements

- Julian Smythe
  - The `eo_protocol.txt` and `pub_protocol.txt` documentation that this project is based on, which was [released](https://eoserv.net/forum/topic/23799) in 2017 on the EOSERV forums.
  - The [EOSERV](https://github.com/eoserv/eoserv) project, which has been a valuable reference point.
  - Answering questions that none of us even knew we had about the protocol and official packet reader.
- Richard Leek
  - Improvements and corrections to the aforementioned `eo_protocol.txt` and `pub_protocol.txt` documentation. (See [eo_protocol](https://github.com/sorokya/eo_protocol))