# AIS Tool

[![Maven version][project-badge]](https://search.maven.org/search?q=g:dev.samypr100%20AND%20a:aistool)
[![GitHub Actions][github-actions-badge]](https://github.com/samypr100/aistool/actions/workflows/native.yml)

[project-badge]: http://img.shields.io/maven-central/v/dev.samypr100/aistool.svg?color=%234DC71F
[github-actions-badge]: https://github.com/samypr100/backports.asyncio.runner/actions/workflows/main.yml/badge.svg

A simple and efficient command-line interface for decoding AIS (Automatic
Identification System) messages, built on top of the powerful
[aismessages](https://github.com/tbsalling/aismessages) Java library.

## Features

- Decode AIS messages from strings, files, or stdin
- Supports both JSON (default) and CSV outputs
- Native builds using GraalVM available in the
  [releases](https://github.com/samypr100/aistool/releases)

## Building

Build native image using Maven:

```bash
./mvnw clean package -P native
```

This will produce a native executable (e.g. `aistool`) in the `target/`
directory, which you can run directly:

```bash
cd target/
./aistool --help
```

Profiling and Native Agent:

To update `META-INF/native-image` with appropriate reflect-config, ensure you
exercise all possible cases needed at runtime that involve reflection. To start
the agent, you can use the below command.

```bash
./mvnw clean package -Dagent=true -DskipNativeBuild=true
```

## Usage

```
Usage: aistool [-f <file>] [-i <string>] [-o <format>] [-v]
 -f,--file <file>       Decode AIS inputs from a given file.
 -i,--input <string>    Decode AIS input from a string or from stdin using '-'.
 -o,--output <format>   Output format: 'json' (default) or 'csv'.
 -v,--verbose           Enable verbose output.
```

## Examples

### Decode a single AIS message from a string

```bash
aistool -i '!AIVDM,1,1,,A,18UG;P0012G?Uq4EdHa=c;7@051@,0*53' -o json | jq
```

### Decode multiple messages from a file

```bash
aistool -f ais_messages.txt -o csv
```

### Pipe from stdin

```bash
echo '!AIVDM,1,1,,A,18UG;P0012G?Uq4EdHa=c;7@051@,0*53' | aistool -i - -o json
```

### Enable verbose output

```bash
aistool -i '!AIVDM,...' -o json -v
```

## Output Formats

### JSON

Each decoded message is printed as a structured JSON object, including metadata
when available such as:

- Message type (`PositionReportClassAScheduled`, etc.)
- Source MMSI
- Vessel location, speed, heading, status
- NMEA message fields
- Decoder metadata

Example output (pretty-printed via `jq`):

```json5
[
  {
    messageType: "PositionReportClassAScheduled",
    sourceMmsi: { mmsi: 576048000 },
    latitude: 37.912167,
    longitude: -122.42299,
    // ...
  },
]
```

### CSV

Each message is flattened into a CSV row with relevant fields extracted.

## Internals

This tool is a GraalVM compatible setup around:

- [`aismessages`](https://github.com/tbsalling/aismessages)
- [`aiscli`](https://github.com/tbsalling/aiscli)

## Credits

Credits to Thomas Borg Salling for `aismessages` and `aiscli`.
