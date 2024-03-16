# libp2p-noise secure transport

This crate modify from [go-libp2p](https://github.com/libp2p/go-libp2p/tree/master/p2p/security/noise).

### Why this crate?

libp2p use `libp2p.PrivateNetwork` for private network, but it implement in a strange way.

In fact, the encrypted stream for a private libp2p tcp connection looks like this:

```
tcp stream: conn
    |
    v
private network: salsa20(conn,psk)
    |
    v
secure transport: noise(salsa20(conn,psk))
```

It's double encryption and noise has already provide a psk for handshake, so we can use noise directly.

After modify, the encrypted stream for a private libp2p tcp connection looks like this:

```
# from Noise_XX_DH25519_ChachaPoly_SHA256 to Noise_XXpsk3_DH25519_AESGCM_BLAKE2b
tcp stream: conn
    |
    v
secure transport: noise(conn,psk)
```

### Usage

Use `libp2p.PrivateNetwork`:

```go
transports = libp2p.ChainOptions(
    libp2p.NoTransports,
    libp2p.Transport(tcp.NewTCPTransport),
)

finalOpts := []libp2p.Option{
    libp2p.Identity(hostKey),
    libp2p.ListenAddrs(listenAddrs...),
    libp2p.PrivateNetwork(psk),
    transports,
}

h, err := libp2p.New(
    finalOpts...,
)
if err != nil {
    return nil, err
}
```

After:

```go

import (
    "github.com/Dreamacro/libp2p-noise"
)

transports = libp2p.ChainOptions(
    libp2p.NoTransports,
    libp2p.Transport(tcp.NewTCPTransport),
    libp2p.Security(noise.ID, noise.New(psk)),
)

finalOpts := []libp2p.Option{
    libp2p.Identity(hostKey),
    libp2p.ListenAddrs(listenAddrs...),
    transports,
}

libp2p.New(finalOpts...)
```
