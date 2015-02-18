# Mitel-MiTAI
An analysis of the MiTAI network protocol, used for interaction with a Mitel PBX

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
    * [Unknown Attribute #2](#unknown-attribute-2)
    * [Body Length](#body-length)
    * [Message Type](#message-type)
    * [Header End](#header-end)
  * [Body](#body)
* [Actions](#actions)
  * [Connect](#connect)
  * [Heartbeat](#heartbeat)

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

Hex               | UTF-8       | Meaning
------------------|-------------|--------
`F9 F8 F7 F6`     | `....`      | [Message Prefix](#message-prefix)
`55 00 00 00`     | `U...`      | [Message Length](#message-length)
`12`              | `.`         | [Header Length](#header-length)
`43 43`           | `CC`        | [Action](#action)
`20 36 30 31`     | ` 601`      | [Unknown Attribute #1](#unknown-attribute-1)
`20 20 20 31`     | `   1`      | [Unknown Attribute #2](#unknown-attribute-2)
`20 20 20 36 36`  | `   66`     | [Body Length](#body-length)
`20 4E`           | ` N`        | [Message Type](#message-type)
`00`              | `.`         | Header End
`6F 6E 73 74` ... `33 3A 37 44` | `Mons` ... `3:7D` | Body

Hex value `00` fills the blanks and hex `20` is a seperator.

### Header

#### Message Prefix
All messages start with the hex value `F9 F8 F7 F6`.

#### Message Length
The message length is the total amount of characters in the message, minus the [Message Prefix](#message-prefix) and [Message Length](#message-length) parts. The length equals the decimal value of an [UTF-8 character](https://en.wikipedia.org/wiki/UTF-8#Codepage_layout), which will be used.

The message length in the example is `85`, which is the decimal value of the UTF-8 character `U` (capital U).

#### Header Length
The header length is the total amount of characters in the header, minus the [Header End](#header-end). It is shown as an UTF-8 character, just like [Message Length](#message-length).

#### Action
The action is displayed as two characters, like `CC`. The first character has only two values, `C` (command) or `R` (response). `C` is a message sent from the client to the server, and `R` is the response from the server to the client. The second character represents the action. See also the [Actions section](#actions).

Observed values: `*C` (connection), `*B` (after connection), `*H` (heartbeat), `CS` (device action), `RA` (device action)

#### Unknown Attribute #1
Yet unknown...

Observed values: `601` (connection), `0` (all other)

#### Unknown Attribute #2
This field looks like an additional attribute to the action field. The exact function is yet unknown. 

Observed values: `0` (heartbeat), `1` (connection), `3` (device select), `25` (device query), `26` (device alter), 

#### Body Length
The body length is the total amount of characters after the [Header End](#header-end). When there's no body, this value will be `0`. The body length will be shown as a combination of all characters in this field. When the field contains multiple characters, like `   66`, the merged characters form the total length.

#### Message Type
The seems like the message type indicates the structure of the body content, but the exact function is yet unknown. 

Observed values: `N` (connection), `0` (heartbeat), `H` (device action)

### Body
...

## Actions

### Connect

##### Command
It seems that the body always has the same length of 66 characters. It starts with the hostname of the client (`Monster`), directly followed by the process ID of the client application (`_pid.144088`). The message ends with the MAC address of the client (`00:00:36:E1:33:7D`). Between the hostname/process ID and MAC address are fillers (`00`) and seperators (`11`).

```
F9 F8 F7 F6 55 00 00 00   12 43 43 20 36 30 31 20   ....U... .CC 601
20 20 31 20 20 20 36 36   20 4E 00 4D 6F 6E 73 74     1   66  N.Monst
65 72 5F 70 69 64 00 31   34 30 38 38 00 00 00 00   er_pid.1 4088....
00 00 00 00 00 00 00 00   00 11 00 00 00 00 00 00   ........ ........
00 00 00 00 00 00 00 00   00 00 00 11 30 30 3A 30   ........ ....00:0
30 3A 33 36 3A 45 31 3A   33 33 3A 37 44            0:36:E1: 33:7D
```

##### Response
The response body only contains the `pbx_A-cid3`. The returned value might be something like some internal naming (in case of clustering `pbx_A`, `pbx_B`, etc.). The meaning of `cid3` is also unknown, probably a client or connection ID.
```
F9 F8 F7 F6 1F 00 00 00   12 52 43 20 36 30 31 20   ........ .RC 601 
20 20 31 20 20 20 31 32   20 4E 00 70 62 78 5F 41     1   12  N.pbx_A
2D 63 69 64 33 0A 00                                -cid3..
```

### Heartbeat
##### Command
```
F9 F8 F7 F6 13 00 00 00   12 43 48 20 20 20 30 20   ........ .CH   0 
20 20 30 20 20 20 20 30   20 30 00                    0    0  0.
```

##### Response
```
F9 F8 F7 F6 13 00 00 00   12 52 48 20 20 20 30 20   ........ .RH   0 
20 20 30 20 20 20 20 30   20 30 00                    0    0  0.
```
