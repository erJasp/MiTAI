# MiTAI
An analysis of the MiTAI network protocol, used for interaction with, for example, a Mitel PBX.

## Table of Contents
* [Connections](#connections)
* [Message Structure](#message-structure)
  * [Message Example](#message-example) 
  * [Header](#header)
    * [Message Prefix](#message-prefix)
    * [Message Length](#message-length)
    * [Header Length](#header-length)
    * [Action](#action)
    * [Unknown Attribute #1](#unknown-attribute-1)
    * [Parameter](#parameter)
    * [Body Length](#body-length)
    * [Body Type](#body-type)
* [Actions](#actions)
  * [Command](#command)
    * [Connect](#cc-connect)
    * [Keepalive](#ch-keepalive)
  * [Response](#response)
    * [Connect](#rc-connect)
    * [Keepalive](#rh-keepalive)

## Connections
The default `TCP` ports are `8000` and `8001` (SSL).

## Message Structure
The messages can be saparated in a header and a body. All messages contain a header, the body is optional. When there's no body, the message will end with the *Header End*.

### Message Example

```
F9 F8 F7 F6 55 00 00 00   12 43 43 20 36 30 31 20   ....U... .CC 601
20 20 31 20 20 20 36 36   20 4E 00 4D 6F 6E 73 74     1   66  N.Monst
65 72 5F 70 69 64 00 31   34 30 38 38 00 00 00 00   er_pid.1 4088....
00 00 00 00 00 00 00 00   00 11 00 00 00 00 00 00   ........ ........
00 00 00 00 00 00 00 00   00 00 00 11 30 30 3A 30   ........ ....00:0
30 3A 33 36 3A 45 31 3A   33 33 3A 37 44            0:36:E1: 33:7D
```

Hex               | ASCII       | Meaning
------------------|-------------|--------
`F9 F8 F7 F6`     | `....`      | [Message Prefix](#message-prefix)
`55 00 00 00`     | `U...`      | [Message Length](#message-length)
`12`              | `.`         | [Header Length](#header-length)
`43 43`           | `CC`        | [Action](#action)
`20 36 30 31`     | ` 601`      | [Unknown Attribute #1](#unknown-attribute-1)
`20 20 20 31`     | `   1`      | [Parameter](#parameter)
`20 20 20 36 36`  | `   66`     | [Body Length](#body-length)
`20 4E`           | ` N`        | [Body Type](#body-type)
`00`              | `.`         | Header End
`6F 6E 73 74` ... `33 3A 37 44` | `Mons` ... `3:7D` | Body

Hex value `00` fills the blanks and hex `20` is a seperator.

### Header

#### Message Prefix
All messages start with the hex value `F9 F8 F7 F6`.

#### Message Length
The message length is the total amount of characters in the message, minus the *Message Prefix* and *Message Length* parts. The length equals the decimal value of an [ASCII character](https://en.wikipedia.org/wiki/ASCII#ASCII_printable_code_chart), which will be used.

The message length in the example is `85`, which is the decimal value of the ASCII character `U` (capital U).

#### Header Length
The header length is the total amount of characters in the header, minus the *Header End*. It is shown as an ASCII character, just like *Message Length*.

#### Action
The action is displayed as two characters, like `CC`. The first character has only two values, `C` (command) or `R` (response). `C` is a message sent from the client to the server, and `R` is the response from the server to the client. The second character represents the action. See also the [Actions section](#actions).

#### Unknown Attribute #1
This might be a client version number. This parameter only has a value on the connect action.

Parameter | Action
----------|------
601       | Connect
0         | Everything else

#### Parameter
When an action allows multiple operations, the parameters specifies the exact operation.

Parameter | Action
----------|-------
0         | Keepalive
1         | Connect
3         | Select device (before 25 and 26)
25        | Alter device
26        | Query device

#### Body Length
The body length is the total amount of characters after the *Header End*. When there's no body, this value will be `0`. The body length will be shown as a combination of all characters in this field. When the field contains multiple characters, like `   66`, the merged characters form the total length.

#### Body Type
This fields indicates the structure of content in the body. 

Parameter | Meaning
----------|--------
`0`       | No body
`N`       | Values divided with a separator and filled with blanks
`H`       | Values structured using the [ASN.1 BER/DER standard](https://en.wikipedia.org/wiki/Abstract_Syntax_Notation_One)

## Actions

### Command

#### `CC` Connect
**Message**
```
F9 F8 F7 F6 55 00 00 00   12 43 43 20 36 30 31 20   ....U... .CC 601
20 20 31 20 20 20 36 36   20 4E 00 4D 6F 6E 73 74     1   66  N.Monst
65 72 5F 70 69 64 00 31   34 30 38 38 00 00 00 00   er_pid.1 4088....
00 00 00 00 00 00 00 00   00 11 00 00 00 00 00 00   ........ ........
00 00 00 00 00 00 00 00   00 00 00 11 30 30 3A 30   ........ ....00:0
30 3A 33 36 3A 45 31 3A   33 33 3A 37 44            0:36:E1: 33:7D
```

**Header**

Description | Attr #1 | Parameter  | Body Type | Response Action |
------------|---------|------------|-----------|-----------------|
Connect     | 601     | 1          | N         | RC              |

**Body**

Hex                             | ASCII               | Meaning
--------------------------------|---------------------|--------
`4D 6F 6E 73` ... `34 30 38 38` | `Monster_pid.14088` | Client hostname and process ID
`11`                            | `.`                 | Field separator hostname/PID and MAC
`30 30 3A 30` ... `33 3A 37 44` | `00:0` ... `3:7D`   | Client MAC address

#### `CH` Keepalive
**Message**
```
F9 F8 F7 F6 13 00 00 00   12 43 48 20 20 20 30 20   ........ .CH   0 
20 20 30 20 20 20 20 30   20 30 00                    0    0  0.
```

**Header**

Description | Attr #1 | Parameter  | Body Type | Response Action |
------------|---------|------------|-----------|-----------------|
Keepalive   | 0       | 0          | 0         | RH              |

### Response

#### `RC` Connect
**Message**

```
F9 F8 F7 F6 1F 00 00 00   12 52 43 20 36 30 31 20   ........ .RC 601 
20 20 31 20 20 20 31 32   20 4E 00 70 62 78 5F 41     1   12  N.pbx_A
2D 63 69 64 33 0A 00                                -cid3..
```

**Header**

Description | Attr #1 | Parameter  | Body Type | Command action  |
------------|---------|------------|-----------|-----------------|
Connect     | 601     | 1          | N         | CC              |

**Body**

Hex                             | ASCII               | Meaning
--------------------------------|---------------------|--------
`70 62 78 5F` ... `63 69 64 33` | `pbx_` ... `cid3`   | Server and client/connection ID (?)

#### `RH` Keepalive
**Message**

```
F9 F8 F7 F6 13 00 00 00   12 52 48 20 20 20 30 20   ........ .RH   0 
20 20 30 20 20 20 20 30   20 30 00                    0    0  0.
```

**Header**

Description | Attr #1 | Parameter  | Body Type | Command action  |
------------|---------|------------|-----------|-----------------|
Keepalive   | 0       | 0          | 0         | CH              |
