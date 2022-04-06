# TCP/IP

### 监听 TCP 端口

以下代码会监听指定的 TCP 端口，并接收一条外部进入的 TCP 连接，然后将读取到的一条信息输出到标准输出( `println!` )。

```rust,editable
use std::net::{SocketAddrV4, Ipv4Addr, TcpListener};
use std::io::{Read, Error};

fn main() -> Result<(), Error> {
    let loopback = Ipv4Addr::new(127, 0, 0, 1);
    let socket = SocketAddrV4::new(loopback, 0);
    let listener = TcpListener::bind(socket)?;
    let port = listener.local_addr()?;
    println!("Listening on {}, access this port to end the program", port);
    let (mut tcp_stream, addr) = listener.accept()?; //block  until requested
    println!("Connection received! {:?} is sending data.", addr);
    let mut input = String::new();
    let _ = tcp_stream.read_to_string(&mut input)?;
    println!("{:?} says {}", addr, input);
    Ok(())
}
```


### 循环接收进入的 TCP 连接
@todo