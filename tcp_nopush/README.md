# tcp_nopush and tcp_nodelay

## How to run

1. Run Nginx and attach the strace. [Instructions here](https://github.com/lucasrodcosta/nginx-studies#how-to-run).

2. Make the HTTP and HTTPS requests and check the logs:

    ```bash
    curl -v -s "http://localhost:8080/nopush_off/1K" -o /dev/null

    curl -v -s "http://localhost:8080/nopush_on/1K" -o /dev/null

    curl -v -s "http://localhost:8080/nopush_nodelay_on/1K" -o /dev/null
    ```

## Notes

### tcp_nodelay

Operating systems makes use of the [Nagle's algorithm](https://en.wikipedia.org/wiki/Nagle%27s_algorithm) to improve the efficiency of TCP protocol by reducing the number of packets that need to be sent over the network. In UNIX, the TCP stack waits up to 200ms to avoid to send a packet that would be to small.

The `TCP_NODELAY` option allows Nginx to bypass this mechanism and send the data as soon as possible

### tcp_nopush

The `tcp_nopush` directive activates the `TCP_CORK` option in Linux TCP stack (or the `TCP_NOPUSH` option on FreeBSD). This option blocks the data until the packet reaches the MSS (maximum segment size).

From the logs, we can see the use of `TCP_CORK` on `setsockopt()`:

```
[pid 22] setsockopt(3, SOL_TCP, TCP_CORK, [0], 4) = 0
```

When we're not using the directive `tcp_nopush on;`, we see the following:

```
[pid 22] setsockopt(3, SOL_TCP, TCP_NODELAY, [1], 4) = 0
```

In fact, the TCP manpage explains that `TCP_CORK` and `TCP_NODELAY` are mutually exclusive, but can be combined since Linux 2.5.9.
