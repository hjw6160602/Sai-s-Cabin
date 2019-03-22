# 以太坊NAT解析

```go
type discoverTable interface {
	Self() *discover.Node
	Close()
	Resolve(target discover.NodeID) *discover.Node
	Lookup(target discover.NodeID) []*discover.Node
	ReadRandomNodes([]*discover.Node) int
}
```

### 拨号状态 `dialsate类` 的定义

```go
// dialstate schedules dials and discovery lookups.
// it get's a chance to compute new tasks on every iteration
// of the main loop in Server.run.
type dialstate struct {
	maxDynDials int
	ntab        discoverTable
	netrestrict *netutil.Netlist

	lookupRunning bool
	dialing       map[discover.NodeID]connFlag
	lookupBuf     []*discover.Node // current discovery lookup results
	randomNodes   []*discover.Node // filled from Table
	static        map[discover.NodeID]*dialTask
	hist          *dialHistory

	start     time.Time        // time when the dialer was first used
	bootnodes []*discover.Node // default dials when there are no peers
}

```


### 持续监听方法`listenLoop()` 这是一个死循环的goroutine 会监听端口并接收外部的请求。

```go
// listenLoop runs in its own goroutine and accepts
// inbound connections.
func (srv *Server) listenLoop() {
	defer srv.loopWG.Done()
	log.Info("RLPx listener up", "self", srv.makeSelf(srv.listener, srv.ntab))

	// This channel acts as a semaphore limiting
	// active inbound connections that are lingering pre-handshake.
	// If all slots are taken, no further connections are accepted.
	tokens := maxAcceptConns
	if srv.MaxPendingPeers > 0 {
		tokens = srv.MaxPendingPeers
	}
	//创建maxAcceptConns个槽位。 我们只同时处理这么多连接。 多了也不要。
	slots := make(chan struct{}, tokens)
	//把槽位填满。
	for i := 0; i < tokens; i++ {
		slots <- struct{}{}
	}

	for {
		// Wait for a handshake slot before accepting.
		<-slots

		var (
			fd  net.Conn
			err error
		)
		for {
			fd, err = srv.listener.Accept()
			if tempErr, ok := err.(tempError); ok && tempErr.Temporary() {
				log.Debug("Temporary read error", "err", err)
				continue
			} else if err != nil {
				log.Debug("Read error", "err", err)
				return
			}
			break
		}

		// Reject connections that do not match NetRestrict.
		// 白名单。 如果不在白名单里面。那么关闭连接。
		if srv.NetRestrict != nil {
			if tcp, ok := fd.RemoteAddr().(*net.TCPAddr); ok && !srv.NetRestrict.Contains(tcp.IP) {
				log.Debug("Rejected conn (not whitelisted in NetRestrict)", "addr", fd.RemoteAddr())
				fd.Close()
				slots <- struct{}{}
				continue
			}
		}

		fd = newMeteredConn(fd, true)
		log.Trace("Accepted connection", "addr", fd.RemoteAddr())

		// Spawn the handler. It will give the slot back when the connection
		// has been established.
		go func() {
			//看来只要连接建立完成之后。 槽位就会归还。 SetupConn这个函数我们记得再dialTask.Do里面也有调用， 这个函数主要是执行连接的几次握手。
			srv.SetupConn(fd, inboundConn, nil)
			slots <- struct{}{}
		}()
	}
}
```

### 以太坊自定义的连接类 `conn` 遵守 `transport` 协议

```go
// conn wraps a network connection with information gathered
// during the two handshakes.
type conn struct {
	fd net.Conn
	transport
	flags connFlag
	cont  chan error      // The run loop uses cont to signal errors to SetupConn.
	id    discover.NodeID // valid after the encryption handshake
	caps  []Cap           // valid after the protocol handshake
	name  string          // valid after the protocol handshake
}
```
###  `transport` 协议继承自 `MsgReadWriter` 协议
```go
type transport interface {
	// The two handshakes.
	doEncHandshake(prv *ecdsa.PrivateKey, dialDest *discover.Node) (discover.NodeID, error)
	doProtoHandshake(our *protoHandshake) (*protoHandshake, error)
	// The MsgReadWriter can only be used after the encryption
	// handshake has completed. The code uses conn.id to track this
	// by setting it to a non-nil value after the encryption handshake.
	MsgReadWriter
	// transports must provide Close because we use MsgPipe in some of
	// the tests. Closing the actual network connection doesn't do
	// anything in those tests because NsgPipe doesn't use it.
	close(err error)
}
```
###  这里 `MsgReadWriter` 协议同事遵守两个协议 `MsgReader` 和 `MsgWriter` 下面的内容来自于 `p2p/message.go` 文件
```go
// MsgReadWriter provides reading and writing of encoded messages.
// Implementations should ensure that ReadMsg and WriteMsg can be
// called simultaneously from multiple goroutines.
type MsgReadWriter interface {
	MsgReader
	MsgWriter
}

type MsgReader interface {
	ReadMsg() (Msg, error)
}

type MsgWriter interface {
	// WriteMsg sends a message. It will block until the message's
	// Payload has been consumed by the other end.
	//
	// Note that messages can be sent only once because their
	// payload reader is drained.
	WriteMsg(Msg) error
}
```
### 也就是说 `conn` 是可以 `ReadMsg` 和 `WriteMsg` 的

```go
// TCPDialer implements the NodeDialer interface by using a net.Dialer to
// create TCP connections to nodes in the network
type TCPDialer struct {
	*net.Dialer
}

// Dial creates a TCP connection to the node
func (t TCPDialer) Dial(dest *discover.Node) (net.Conn, error) {
	addr := &net.TCPAddr{IP: dest.IP, Port: int(dest.TCP)}
	return t.Dialer.Dial("tcp", addr.String())
}
```



