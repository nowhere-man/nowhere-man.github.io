---
layout: post
title: QUIC协议 - 2 - QUIC报文格式
categories: [网络传输]
tags: [QUIC]
---

一个 QUIC 数据包由一个或多个报头（Header）和有效载荷（Payload）组成。有效载荷中包含了一系列的数据帧（Frame）。QUIC 的报文格式是高度动态化的，可以根据实际情况进行调整，没有固定的报文结构。

## 报文头

QUIC 有两种主要类型的报头：**长报头（Long Header）** 和 **短报头（Short Header）**。

+   **长报头 (Long Header):** 用于**连接建立**过程中的握手阶段，如 `Initial`、`0-RTT`、`Handshake` 和 `Retry` 数据包。它包含版本号和完整的连接 ID。
+   **短报头 (Short Header):** 用于已建立连接的**数据传输**阶段。它的设计非常紧凑，以减少开销。

### Long Header

所有长报头的数据包，其第一个字节的最高位都为 `1`。

```
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+
     |1|   Type (7)  |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                         Version (32)                          |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     | DCID Len (8)  |         Destination Connection ID (?)       ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     | SCID Len (8)  |         Source Connection ID (?)            ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                   Token Length (i)                          ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                         Token (?)                           ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                        Length (i)                           ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                    Packet Number (8/16/24/32)               ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                      Packet Payload (*)                     ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

     Long Header Packet {
          Header Form (1) = 1,
          Version-Specific Bits (7),
          Version (32),
          Destination Connection ID Length (8),
          Destination Connection ID (0..2040),
          Source Connection ID Length (8),
          Source Connection ID (0..2040),
          Version-Specific Data (..),
    }
```

**字段解释:**

+   **Header Form (1 bit):** 第一个比特固定为 `1`，表示这是一个长报头。
+   **Type (7 bits):** 报文类型，不同的值表示不同的握手阶段：
    +   `0x00`: Initial (初始包)
    +   `0x01`: 0-RTT
    +   `0x02`: Handshake (握手包)
    +   `0x03`: Retry (重试包)
+   **Version (32 bits):** QUIC 协议的版本号。`0x00000001` 代表 QUIC v1。对于 Initial 包，这个字段是客户端期望使用的版本。
+   **DCID Len (8 bits):** 目标连接 ID（Destination Connection ID）的长度，单位是字节。
+   **Destination Connection ID (0-2040 bits):** 目标连接 ID。由数据包的接收者选择，用于路由到正确的 QUIC 连接。
+   **SCID Len (8 bits):** 源连接 ID（Source Connection ID）的长度，单位是字节。
+   **Source Connection ID (0-2040 bits):** 源连接 ID。由数据包的发送者选择。
+   **Token Length (Variable):** `Retry` 包中携带的 `Token` 的长度。对于 `Initial` 包，这个字段存在且长度可变。对于其他类型的长报头包，此字段不存在。
+   **Token (Variable):** `Retry` 包中由服务器生成并发送给客户端的令牌，客户端在后续的 `Initial` 包中必须回显此令牌。
+   **Length (Variable):** 有效载荷（Packet Payload）的长度，包括 Packet Number 字段和经过验证的加密部分。长度使用变长整数编码。
+   **Packet Number (8, 16, 24, or 32 bits):** 数据包编号。这是一个单调递增的数字，用于检测丢包和消息重排。它的长度是可变的（1-4字节），实际长度由第一个字节的最低两位决定。
+   **Packet Payload (\*):** 数据包的有效载荷，包含一系列帧（Frames）。这部分是经过加密的。

### Short Header

```
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+
     |0|S|R|R|K|P P|
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                Destination Connection ID (0/32..160)        ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                     Packet Number (8/16/24/32)              ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                      Packet Payload (*)                     ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     Short Header Packet {
          Header Form (1) = 0,
          Version-Specific Bits (7),
          Destination Connection ID (..),
          Version-Specific Data (..),
     }
```

**字段解释:**

+   **Header Form (1 bit):** 第一个比特固定为 `0`，表示这是一个短报头。
+   **Spin Bit (S, 1 bit):** 可选的自旋位，用于被动地测量往返时间（RTT）。
+   **Reserved (R, 2 bits):** 保留位，必须设置为 `0`。
+   **Key Phase (K, 1 bit):** 用于指示密钥更新的状态。`0` 表示旧密钥，`1` 表示新密钥。
+   **Packet Number Length (PP, 2 bits):** Packet Number 字段的长度减一。例如 `00` 表示长度为1字节，`11` 表示长度为4字节。
+   **Destination Connection ID (0-160 bits):** 目标连接 ID。在连接建立后，两端会协商是否省略此字段。
+   **Packet Number (8, 16, 24, or 32 bits):** 数据包编号，编码方式同长报头。
+   **Packet Payload (\*):** 加密的有效载荷，包含各种帧。

### Version Negotiation

QUIC终端收到一个带长包头的数据包及一个其不能理解或不支持的版本时，就可能回复一个版本协商包。短包头数据包不会触发版本协商。

```c++
Version Negotiation Packet {
  Header Form (1) = 1,
  Unused (7),
  Version (32) = 0,
  Destination Connection ID Length (8),
  Destination Connection ID (0..2040),
  Source Connection ID Length (8),
  Source Connection ID (0..2040),
  Supported Version (32) ...,
}
```

## 数据帧

QUIC 的有效载荷由一个或多个帧组成。每个帧都有一个类型字段，后面跟着该类型特定的数据。

| 帧类型 | 名称 | 作用 |
| - | - | - |
| 0x00 | PADDING | 纯填充，无语义；用于对齐/保密长度/PMTU 探测。 |
| 0x01 | PING | 促使对端发送 ACK；保活/可达性探测。 |
| 0x02 | ACK | 确认接收的数据包范围；含 ACK 延迟与范围列表。 |
| 0x03 | ACK_ECN | 同 ACK，并携带 ECN 计数。 |
| 0x04 | RESET_STREAM | 终止单个发送方向的流，给出错误码与最终偏移。 |
| 0x05 | STOP_SENDING | 请求对端停止向该流发送数据并复位，含错误码。 |
| 0x06 | CRYPTO | 携带握手明文（随加密级别变化：Initial/Handshake/1-RTT）。 |
| 0x07 | NEW_TOKEN | 服务器发放地址验证令牌（便于后续 0-RTT/重连）。 |
| 0x08–0x0f | STREAM(变体) | 携带应用数据；变体位标记 FIN/LEN/OFF（是否含长度/偏移/结束）。 |
| 0x10 | MAX_DATA | 连接级接收窗口上限更新（整体流量控制）。 |
| 0x11 | MAX_STREAM_DATA | 指定流的接收窗口上限更新（单流流量控制）。 |
| 0x12 | MAX_STREAMS (BIDI) | 允许的最大双向流数量更新。 |
| 0x13 | MAX_STREAMS (UNI) | 允许的最大单向流数量更新。 |
| 0x14 | DATA_BLOCKED | 连接级因流控受限而被阻塞的告知。 |
| 0x15 | STREAM_DATA_BLOCKED | 指定流因流控受限而被阻塞的告知。 |
| 0x16 | STREAMS_BLOCKED (BIDI) | 因双向流数上限受限而被阻塞的告知。 |
| 0x17 | STREAMS_BLOCKED (UNI) | 因单向流数上限受限而被阻塞的告知。 |
| 0x18 | NEW_CONNECTION_ID | 发布新连接 ID（序号、Retire Prior To、重置令牌等）。 |
| 0x19 | RETIRE_CONNECTION_ID | 通知对端弃用某序号的连接 ID。 |
| 0x1a | PATH_CHALLENGE | 路径验证挑战（携带随机数据）。 |
| 0x1b | PATH_RESPONSE | 对 PATH_CHALLENGE 的响应（回显数据）。 |
| 0x1c | CONNECTION_CLOSE (transport) | 传输层错误导致的连接关闭；含错误码与相关帧类型。 |
| 0x1d | CONNECTION_CLOSE (application) | 应用层关闭连接；含应用错误码与原因。 |
| 0x1e | HANDSHAKE_DONE | 服务器通知握手完成，客户端可丢弃握手状态。 |

**数据传输与握手（核心）**

- CRYPTO: 传 TLS 握手消息（完成密钥协商与认证），没有它就没有连接安全性。
- STREAM: 承载上层应用数据（HTTP/3 等），最常见、最重要的数据承载帧。
- ACK: 确认收到哪些包，驱动丢包检测与拥塞控制核心逻辑，整个可靠传输的关键。
- CONNECTION_CLOSE: 终止连接并携带错误码与原因，优雅/非优雅关闭都依赖它。

流量控制
- MAX_DATA: 提高连接级别可接收数据总量上限（连接级流控）。
- MAX_STREAM_DATA: 提高单条流的可接收数据上限（流级流控）。
- DATA_BLOCKED/STREAM_DATA_BLOCKED: 发送端告知被对端的流控限制卡住了，便于对端放宽窗口（较次要但有用）。

流的生命周期控制
- RESET_STREAM: 发送端中止某个流的发送方向并告知错误码，快速失败/取消。
- STOP_SENDING: 接收端不再需要该流数据，要求对端停止发送，节省带宽。
- STREAMS_BLOCKED/MAX_STREAMS: 控制/提升可并发流的数量上限（协调并发度）。

连接与路径管理
- NEW_CONNECTION_ID/RETIRE_CONNECTION_ID: 管理连接 ID 的轮换与回收，支持无状态重用与路径迁移。
- PATH_CHALLENGE/PATH_RESPONSE: 探测并验证新路径可达性，做连接迁移时非常关键。

健壮性与保活
- PING: 无负载保活/探测可达性/触发对端 ACK。
- PADDING: 填充到目标长度（抗流量分析/PMTU 探测/对齐等）。

完成握手与 0-RTT 配套
- HANDSHAKE_DONE: 服务器通知客户端握手完成（允许清理状态/进入稳定阶段）。
- NEW_TOKEN: 发给客户端的地址验证 Token，优化后续 0-RTT/减少握手开销。

可选但常见扩展
- DATAGRAM: 无可靠、无顺序的消息通道（如 WebTransport/MASQUE 等低延迟场景）。

***

最核心：

- 可靠安全传输三件套：CRYPTO + STREAM + ACK
- 连接/关闭：CONNECTION_CLOSE
- 流控关键：MAX_DATA + MAX_STREAM_DATA
- 流管理：RESET_STREAM + STOP_SENDING
- 迁移相关：NEW_CONNECTION_ID + PATH_CHALLENGE/PATH_RESPONSE

按场景：
- HTTP/3 主路径：CRYPTO, STREAM, ACK, MAX_*, CONNECTION_CLOSE
- 主动取消/早停：STOP_SENDING, RESET_STREAM
- 连接迁移/多路径探索：NEW_CONNECTION_ID, PATH_*
- 低延迟不可靠消息：DATAGRAM
- 保活/测可达：PING



这里介绍两种最重要的帧，除此之外的帧类型参考[RFC 9000 Chapter 19](https://www.rfc-editor.org/rfc/rfc9000.html)

### STREAM 帧

QUIC 的核心，用于传输应用数据。

```
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     | Frame Type (8)|
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                        Stream ID (i)                        ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                         [Offset (i)]                        ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                         [Length (i)]                        ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                        Stream Data (*)                      ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

+   **Frame Type (8 bits):** 范围从 `0x08` 到 `0x0f`。这是一个比特掩码 `0b00001xxx`。
    +   `...1` (OFF bit, `0x04`): 如果为 `1`，则 Offset 字段存在。
    +   `..1.` (LEN bit, `0x02`): 如果为 `1`，则 Length 字段存在。如果为 `0`，则 Stream Data 会一直延伸到数据包末尾。
    +   `.1..` (FIN bit, `0x01`): 如果为 `1`，表示这是该流的最后一部分数据。
+   **Stream ID (Variable):** 数据所属的流的 ID。
+   **Offset (Variable, Optional):** 数据在流中的偏移量。如果 OFF bit 为 `0`，则此字段不存在，偏移量默认为 `0`。
+   **Length (Variable, Optional):** Stream Data 的长度。如果 LEN bit 为 `0`，则此字段不存在。
+   **Stream Data (\*):** 实际的应用数据。

### ACK 帧

用于告知对端哪些数据包已被成功接收。

```
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |            0x02 or 0x03 (8)           |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                     Largest Acknowledged (i)                ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                          ACK Delay (i)                        ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                       ACK Range Count (i)                     ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                      First ACK Range (i)                      ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                       ACK Range (i)                         ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                          ECN Counts (?)                     ...
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     ACK Frame {
          Type (i) = 0x02..0x03,
          Largest Acknowledged (i),
          ACK Delay (i),
          ACK Range Count (i),
          First ACK Range (i),
          ACK Range (..) ...,
          [ECN Counts (..)],
     }
```

+   **Frame Type:** `0x02` (不带 ECN) 或 `0x03` (带 ECN)。
+   **Largest Acknowledged (Variable):** 已确认的最大数据包编号。
+   **ACK Delay (Variable):** 从接收到最大确认包到发送此 ACK 帧之间的延迟时间，单位是微秒。
+   **ACK Range Count (Variable):** ACK Range 块的数量减一。
+   **First ACK Range (Variable):** 第一个 ACK 范围。表示从 `Largest Acknowledged` 开始，有多少个连续的数据包被确认。
+   **ACK Range (Variable):** `Gap` + `ACK Range Length` 的组合。`Gap` 表示与前一个范围的不连续包数量，`ACK Range Length` 表示该范围内的连续包数量。这个字段会重复 `ACK Range Count` 次。
+   **ECN Counts (Optional):** 如果类型是 `0x03`，则包含 ECN (显式拥塞通知) 计数。

## 参考

+ [RFC 8999](https://autumnquiche.github.io/RFC8999_Chinese_Simplified/#RFC8999_QUIC)