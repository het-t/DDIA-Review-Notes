- Evolvability and compatibility
- Forward compatible and backward compatible
- Encoding, serialization and marshaling
- Decoding, deserrialization and unmarshaling
- json, xml and csv formats
- Human readable textual formats
- Ambiguity about handling of numbers
- No support for binary strings

- Binary encoding
- Encodes everything into binary 

# Thrift protocol
## Binary protocol
- Uses field tags instead of field names
- Length when necessary
## Compact protocol
- Packs field type and tag number into a single byte
- Variable length integers, rather than ful 8 bytes uses 2 bytes
- Top bit of each byte indicates whether there are still more bytes to come

# Protocol buffer
- Same as compact protocol but does bit packing differently
- **Backward compatibility** new code reads old data 
- **Forward compatibility** old code reads new data

## Apache Avro
- Has 2 schema languages - for human editing and one based on JSON more machine readable
- Byte sequence have nothing to identify fields or their datatypes.
- Avro allows schema evolution by using reader and writer's schema
- When encoding data it uses any schema that is available to it - writer's schema
- When reading data, it expects data to be in some schema - reader's schema
- Reader and writer schema dont have to be exactly same - they just have to be compatible
- Code reading the data may have been generated from reader's schema during build process
- Avro resolved the differences by looking at the writer's schema and reader's schema side by side and translating the data from ther writer's schema into reader's schema
- Old reader schema new writer schema - Forward compatiblity
- New reader schema old writer schema - Backward compatiblity
- Code reading the encoded data must have access to writer's schema
  1. Large files with lots of records can contain schema with it
  2. Individually written records in database can store schema version no. All versions of schema is stored in database. Reader extracts version no from record and fetch particular schema from database.
  3. When two processes are communicating with each other, they can negotiate the schema version over network and then use them for lifecylce of connection.
