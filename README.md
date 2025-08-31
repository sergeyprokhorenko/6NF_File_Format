# 6NF File Format

**Filename Extension**: `.6nf`  

## 1. Introduction  
[6NF](https://en.wikipedia.org/wiki/Sixth_normal_form) File Format is a new bitemporal, sixth-normal-form (6NF)-inspired data exchange format designed for DWH and for reporting. It replaces complex hierarchical formats like XBRL, XML, JSON, and YAML.

## 2. Design Principles  
- **Database Friendly Flat Structure**: No nested objects or arrays. No need for parsing
- **6NF Compatibility**: Direct mapping to 6NF database tables. No need for normalization
- **Bitemporal Database Compatibility**: All data includes valid_from and recorded_at timestamps
- **UTC Time Standard**: All timestamps must be in UTC format, denoted by the 'Z' suffix (e.g., 2023-01-01T12:00:00Z)
- **Struct Grouping**: Multiple attributes with shared temporal context
- **Compactness**: Uses [Crockford’s Base32](https://www.crockford.com/base32.html) - encoded [UUIDv7](https://datatracker.ietf.org/doc/html/rfc9562#name-uuid-version-7) for identifiers
- **Readability**: Clean syntax with minimal punctuation
- **PostgreSQL Style**: Uses snake_case notation of identifiers (names)
- **Case Sensitivity**: Keywords are UPPERCASE and case-sensitive. Identifiers (names) are lowercase and case-sensitive
- **UTF-8 Encoding**: Files use UTF-8 encoding

## 3. Syntax (EBNF)  
```ebnf
6nf             = [version] { entity | reference | attribute | attribute_ref | struct | relationship } ;
version         = "VERSION" number "\n" ;
entity          = "ENTITY" entity_name entity_id "\n" ;
reference       = "REFERENCE" name reference_id value "\n" ;
attribute       = "ATTRIBUTE_OF" entity_name entity_id name value valid_from recorded_at "\n" ;
attribute_ref   = "ATTRIBUTE_REF_OF" entity_name entity_id name reference_id valid_from recorded_at "\n" ;
struct          = "STRUCT_OF" entity_name entity_id name valid_from recorded_at "\n" 
                 { name (value | reference_id) "\n" } ;
relationship    = "RELATIONSHIP" name relationship_id valid_from recorded_at "\n" 
                 { name ( entity_id | reference_id ) "\n" } ;
value           = string | number | iso8601 | "true" | "false" ;
string          = "\"" { character } "\"" ;
number          = [ "-" ] digit { digit } [ "." digit { digit } ] ;
valid_from      = iso8601 ;
recorded_at     = iso8601 ;
entity_name     = ( letter | "_" ) { letter | digit | "_" } ;
name            = ( letter | "_" ) { letter | digit | "_" } ;
entity_id       = 26 * base32_char ;
reference_id    = 26 * base32_char ;
relationship_id = 26 * base32_char ;
base32_char     = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" 
                | "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "J" | "K" 
                | "M" | "N" | "P" | "Q" | "R" | "S" | "T" | "V" | "W" | "X" | "Y" | "Z" ;
```

## 4. Example  
```
VERSION 7
ENTITY bank 01K3Y0690AJCRFEJ2J49X6ZECY
REFERENCE country_code 01K3Y07Z94DGJWVMB0JG4YSDBV "US"
ATTRIBUTE_OF bank 01K3Y0690AJCRFEJ2J49X6ZECY bank_name "Bank Alpha" 2023-01-01T00:00:00Z 2023-01-01T12:00:00Z
ATTRIBUTE_REF_OF bank 01K3Y0690AJCRFEJ2J49X6ZECY country_code 01K3Y07Z94DGJWVMB0JG4YSDBV 2023-01-01T00:00:00Z 2023-01-01T12:00:00Z
STRUCT_OF bank 01K3Y0690AJCRFEJ2J49X6ZECY bank_address 2023-01-01T00:00:00Z 2023-01-01T12:00:00Z
  country_code 01K3Y07Z94DGJWVMB0JG4YSDBV
  street "123 Main St"
  city "New York"
  zip "10001"
ENTITY account 01K3Y0G45CP4GMGE94BYQ09DFM
ATTRIBUTE_OF account 01K3Y0G45CP4GMGE94BYQ09DFM account_balance 100000.50 2023-01-01T00:00:00Z 2023-01-01T12:00:00Z
ATTRIBUTE_OF account 01K3Y0G45CP4GMGE94BYQ09DFM account_expiration 2025-12-31T23:59:59Z 2023-01-01T00:00:00Z 2023-01-01T12:00:00Z
RELATIONSHIP bank_x_account 01K3Y0NR1Q3KTA9A6J9KYPK6YB 2023-01-01T00:00:00Z 2023-01-01T12:00:00Z
  bank 01K3Y0690AJCRFEJ2J49X6ZECY
  account 01K3Y0G45CP4GMGE94BYQ09DFM
```

