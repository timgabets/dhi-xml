## DHI XML
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Build Status](https://travis-ci.org/timgabets/dhi-xml.svg?branch=master)](https://travis-ci.org/timgabets/dhi-xml)

Rust community library for serializaing/deserializing TSYS DHI (Device Host Interface) XML messages.

### Usage

```rust
use dhi_xml::{DHIRequest, DHIResponse};
```

DHI requests:
```rust
// JSON payload (for example, from HTTP request)
let iso_data = r#"{
    "i000": "0100",
    "i002": "555544******0895",
    "i007": "Transmission date & time ",
    "i011": "STAN",
    "i012": "hhmmss",
    "i013": "MMDD",
    "i037": "Retrieval Reference Number"
}"#;

// Deserializing request from the given JSON payload
let r: DHIRequest = DHIRequest::new(serde_json::from_str(&iso_data).unwrap());

// The data may now be accessed with
assert_eq!(r.iso_fields["i002"], "555544******0895");

// Serialization
let msg = r.serialize().unwrap();

// The message may be sent through the TCP stream
let s = TcpStream::connect(host);
s.write_all(&msg.as_bytes());
```

DHI response:
```rust

let s = r##"
    <?xml version="1.0"?>
    <RequestResponse>
        <Header/>
        <Result><Code>0</Code><Description>OK</Description></Result>
     	<ISO8583-87><i000>0110</i000><i002>555544******0961</i002><i003>300000</i003><i004>000000000000</i004><i007>2804114717</i007></ISO8583-87>"
    </RequestResponse>
"##;

// Deserialization from the XML payload
let resp: DHIResponse = from_reader(s.as_bytes()).unwrap();

// Accessing data
assert_eq!(resp.res.code, 0);
assert_eq!(resp.res.description, "OK");
assert_eq!(resp.iso_fields["i000"], "0110");
assert_eq!(resp.iso_fields["i002"], "555544******0961");

// Serializing response to JSON
let msg = resp.serialize().unwrap();

// Sending as payload in HTTP response
Ok(HttpResponse::Ok()
    .content_type("application/json")
    .header("X-Hdr", "sample")
    .body(msg)),
```

