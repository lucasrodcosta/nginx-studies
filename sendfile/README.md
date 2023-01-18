# sendfile

## How to run

1. Run Nginx and attach the strace. [Instructions here](https://github.com/lucasrodcosta/nginx-studies#how-to-run).

2. Make the HTTP and HTTPS requests and check the logs:

    ```bash
    # HTTP
    curl -v -s "http://localhost:8080/1K" -o /dev/null

    # HTTPS
    curl -v -s -k "https://localhost:443/1K" -o /dev/null
    ```

## Notes

### Nginx phases

From logs it's possible to check the Nginx execution phases (rewrite > post rewrite > access > post access > content > log):

![Nginx phases](/sendfile/img/nginx_phases.png)

It's interesting to see the existence of a cache to Lua codes:

![Lua cache](/sendfile/img/lua_cache.png)

### read+write when `sendfile off`

![Nginx read+write](/sendfile/img/nginx_read_write.png)

From the strace logs, it's possible to see how it occurs at operational system:

![strace read](/sendfile/img/strace_read.png)

![strace write](/sendfile/img/strace_write.png)

:point_right: The same happens in HTTP and HTTPS

### sendfile with HTTP replaces read+write

![Nginx sendfile](/sendfile/img/nginx_sendfile.png)

![strace sendfile](/sendfile/img/strace_sendfile.png)

:point_right: It's also possible to check that less buffers are used

### sendfile with HTTPS doesn't work due to SSL cryptography

![Nginx HTTPS](/sendfile/img/nginx_https.png)

From the strace logs we can see the read and write operations in operational system

![strace read HTTPS](/sendfile/img/strace_read_https.png)
![strace write HTTPS](/sendfile/img/strace_write_https.png)

### sendfile with proxy_pass?

The `sendfile` only works for static files from a storage.

But if you use some kind of caching, the `sendfile` will work like a charm.

Check the logs in [logs/upstream](https://github.com/lucasrodcosta/nginx-studies/tree/main/sendfile/logs/upstream).
