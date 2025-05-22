# SOCKS5代理与TCP转发实现详解

本笔记将详细介绍如何用Go语言实现一个SOCKS5代理，完成TCP流量的转发，并解释每一步的设计原因。关键步骤配有代码示例，便于理解。

---

## 一、SOCKS5代理与TCP转发简介

SOCKS5代理是一种通用的代理协议，支持多种网络协议的转发，常用于突破网络限制、隐藏客户端真实IP等场景。其核心功能是将客户端的TCP流量转发到目标服务器。

TCP转发则是指将接收到的TCP连接的数据原封不动地转发到另一个目标地址，实现数据的中继。

---

## 二、实现步骤

### 1. 监听本地端口，接收客户端连接

**设计原因：** 代理服务器需要作为中间人，首先监听一个本地端口，等待客户端连接。

```go
ln, err := net.Listen("tcp", ":1080")
if err != nil {
    log.Fatal(err)
}
for {
    conn, err := ln.Accept()
    if err != nil {
        log.Println(err)
        continue
    }
    go handleConnection(conn)
}
```

### 2. 进行SOCKS5握手

**设计原因：** SOCKS5协议要求客户端与代理服务器之间先进行握手，协商认证方式。

- 读取客户端发来的认证方法列表
- 选择支持的认证方式（通常为无认证）
- 返回协商结果

```go
// 读取客户端发来的认证方法
buf := make([]byte, 258)
n, err := conn.Read(buf)
if err != nil { return }
// 响应无认证
conn.Write([]byte{0x05, 0x00})
```

### 3. 解析客户端请求，获取目标地址

**设计原因：** 客户端会发送一个请求，告知代理服务器要连接的目标地址和端口。

- 读取请求报文，解析目标地址类型（IPv4/域名/IPv6）
- 提取目标主机和端口

```go
// 读取请求
n, err = conn.Read(buf)
if err != nil { return }
// 解析目标地址
// 省略详细解析代码，可用第三方库如 github.com/armon/go-socks5
```

### 4. 连接目标服务器

**设计原因：** 代理服务器需要作为客户端，主动连接目标服务器。

```go
targetConn, err := net.Dial("tcp", targetAddr)
if err != nil {
    // 返回连接失败响应
    return
}
```

### 5. 通知客户端连接已建立

**设计原因：** 按SOCKS5协议，代理需回复客户端，表示连接目标服务器成功。

```go
// 回复客户端连接成功
conn.Write([]byte{0x05, 0x00, 0x00, 0x01, 0,0,0,0, 0,0})
```

### 6. 转发数据（TCP转发）

**设计原因：** 建立连接后，需实现客户端与目标服务器之间的双向数据转发。

```go
// 双向转发
func relay(src, dst net.Conn) {
    io.Copy(src, dst)
}
go relay(conn, targetConn)
go relay(targetConn, conn)
```

---

## 三、为何这样设计

1. **协议兼容性**：SOCKS5协议标准化，兼容多种客户端和应用。
2. **安全性**：通过握手和认证机制，可扩展支持多种认证方式。
3. **灵活性**：支持多种地址类型（IPv4/IPv6/域名），适应不同网络环境。
4. **高效转发**：使用`io.Copy`实现高效的TCP流量转发，简洁且性能优良。

---

## 四、完整代码参考

可参考 [github.com/armon/go-socks5](https://github.com/armon/go-socks5) 或 [Go官方x/net/proxy](https://pkg.go.dev/golang.org/x/net/proxy) 实现更完整的SOCKS5代理。

---

如需更详细的代码实现或扩展功能（如UDP转发、认证等），可进一步补充。
