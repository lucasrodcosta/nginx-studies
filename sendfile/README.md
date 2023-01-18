# sendfile

- [Requirements to run locally](#Requirements)
- [How to run locally](#how-to-run)
- [Notes and conclusions](#notes)

## Requirements

An OpenResty docker image with debug flag and [strace](https://strace.io/).

1. Clone the [docker-openresty repo](https://hub.docker.com/r/openresty/openresty)

2. Add the flag `--with-debug` inside the `RESTY_CONFIG_OPTIONS` in file `alpine/Dockerfile`:

    ```
    ARG RESTY_CONFIG_OPTIONS="\
    --with-debug \
    --with-compat \
    // other lines
    ```

3. Add the instruction `RUN apk add strace` before the copy of Nginx conf files in `alpine/Dockerfile`:

    ```
    # Add additional binaries into PATH for convenience
    ENV PATH=$PATH:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin

    RUN apk add strace

    # Copy nginx configuration files
    COPY nginx.conf /usr/local/openresty/nginx/conf/nginx.conf
    ```

4. Build the Nginx:

    ```
    docker build -t openresty-with-debug:latest -f alpine/Dockerfile .
    ```

## How to run

1. Run Nginx:

    ```
    make run
    ```

2. In another terminal, attach the strace process to Nginx:

    ```
    make strace
    ```

3. In another terminal, reload the Nginx:

    ```
    make reload
    ```

4. Make the HTTP and HTTPS requests and check the logs:

    ```
    # HTTP
    curl -v -s "http://localhost:8080/1K" -o /dev/null

    # HTTPS
    curl -v -s -k "https://localhost:443/1K" -o /dev/null
    ```

## Notes

### Nginx phases

From the logs it's possible to check the Nginx execution phases (rewrite > post rewrite > access > post access > content > log):

![Nginx phases](/sendfile/img/nginx_phases.png)

It's interesting to see the existence of a cache to Lua codes:

![Lua cache](/sendfile/img/lua_cache.png)

### read/write when `sendfile off`

![Nginx read/write](/sendfile/img/nginx_read_write.png)

From the strace logs, it's possible to see how it occurs at operational system:

![strace read](/sendfile/img/strace_read.png)

![strace write](/sendfile/img/strace_write.png)

:point_right: The same happens in HTTP and HTTPS

### sendfile with HTTP replaces read/write

![Nginx sendfile](/sendfile/img/nginx_sendfile.png)

![strace sendfile](/sendfile/img/strace_sendfile.png)

:point_right: It's also possible to check that less buffers are used

### sendfile with HTTPS don't works due to SSL cryptography

![Nginx HTTPS](/sendfile/img/nginx_https.png)

From the strace logs we can see the read and write operations in operational system

![strace read HTTPS](/sendfile/img/strace_read_https.png)
![strace write HTTPS](/sendfile/img/strace_write_https.png)

### sendfile with proxy_pass?

It doesn't works. The `sendfile` only works for static files from a storage. Check the logs in [logs/upstream](https://github.com/lucasrodcosta/nginx-studies/tree/main/sendfile/logs/upstream).

**BUT** you can use some kind of micro caching on tmpfs. In this case, the `sendfile` will be useful.
