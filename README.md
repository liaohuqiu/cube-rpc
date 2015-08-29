# Cube RPC

This RPC protocal is a half-duplex protocal, it is simple and easy to develop, easy to debug.

It is also high performance.

There are already some applications in this real world. For example: [Android-Gems](http://www.android-gems.com/)

### Serialize protocal

This protocal choose [BinPack](http://binpack.liaohuqiu.net/) as the serialize protocal. It is simple and very fast.

# Communication mechanism

### Messages

There are four kinds of message:

*   Query   (From client to server)
*   Answer  (From server to client)
*   Welcome (From server to client)
*   Close   (client to server, or server to client)


### Call process

* It is half-duplex.

    ```
                
                    +--------+      +--------+
                    | Server |      | Client |
                    +--------+      +--------+
                         |               |
                         | <-------------|
                         |               |
                         |  Welcome      |
                         | ------------> |
                         |               |
                         |  Query        |
                         | <-------------|
                         |               |
                         |  Answer       |
                         | ------------> |
                         |               |
                         | <-------------|
                         | ------------> |
                         | <-------------|
                         | ------------> |
                         |               |
                         |  Closed       |
                         | <-------------|
                         |               |
                         |               |
                         | ---- X -------|
                         |               |
    ```

When the server receive a new connection, it must send the `Welcome` message to the client.

Only after the `Welcome` message has been recieved, the client can sent the `Query`.

### Disconnect gracefully

If the Server or the Client want to close the connection gracefully. It must send the `Close` message.

The other side will close the socket connection after recieve the `Close` message.

The sponser close the connection after detect the socket has been disconnected.

### Proxy

* From the Server side, it looks like Client.

* From the Client side, it looks like Server.

# Message structure

All the messages contain a header, but only `Query` and `Answer` contain a body.

### Message header

The header is 4 or 8 bytes long.

* Data structure

    ```c
    struct Header {
        /**
         * magic: CB
         */
        union {
            char bytes[2];
            uint16_t u16;
        } magic;
        uint8_t version;
        uint8_t msgType;
        /**
         * only Query and Answer has this field, little endian byte order
         */
        uint32_t bodySize;
    };
    ```

* Plain text

    ```
    byte    "C"
    byte    "B"
    byte    version
    byte    message_type

    int32   body_size;               // little endian byte order, not network byte order
    ```

    The header of `Welcome` and `Close` is 4 bytes long, they do not have body_size field.

* Message Type

    ```
    MESSAGE_TYPE_WELCOME        = 0x01
    MESSAGE_TYPE_CLOSE          = 0x02
    MESSAGE_TYPE_QUERY          = 0x03
    MESSAGE_TYPE_ANSWER         = 0x04
    ```

### Message Body

#### Query

It is a list:

```
integer qid;
string  service;
string  method;
dict    params;
```

#### Answer

It is a list:

```
integer qid;
integer status;
dict    data;
```

#### Exception

If the answer is a normal response, status is 0, else status is none-zero and data must contain the following data:

|Key|Value type|description|
|---|---|---|
|exception    | string           | exception name|
|code         | int           | |
|message      | string           | |
|raiser       | string           | method*service @proto:host:port|
|detail       | dict      | |


# Implements

* Python:  https://github.com/liaohuqiu/cube-rpc-python
* C++: Will open source soon.
* JAVA: Will open source soon.

Welcome to implement for more language.
