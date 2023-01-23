# Nginx studies

A set of studies about how Nginx works and how it interacts with the operating system under the hoods.

Each directory is a study, with a README file containing the instructions to reproduce it, some logs and
notes.

- [sendfile](https://github.com/lucasrodcosta/nginx-studies/tree/main/sendfile)
- [tcp_nopush](https://github.com/lucasrodcosta/nginx-studies/tree/main/tcp_nopush)
- TODO: kTLS
- TODO: http/2
- TODO: http/3

## Requirements

All studies requires an OpenResty docker image with debug flag and [strace](https://strace.io/).

1. Clone the [docker-openresty repo](https://hub.docker.com/r/openresty/openresty)

2. Add the flag `--with-debug` inside the `RESTY_CONFIG_OPTIONS` in file `alpine/Dockerfile`:

    ```Dockerfile
    ARG RESTY_CONFIG_OPTIONS="\
    --with-debug \
    --with-compat \
    // other lines
    ```

3. Add the instruction `RUN apk add strace` before the copy of Nginx conf files in `alpine/Dockerfile`:

    ```Dockerfile
    # Add additional binaries into PATH for convenience
    ENV PATH=$PATH:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin

    RUN apk add strace

    # Copy nginx configuration files
    COPY nginx.conf /usr/local/openresty/nginx/conf/nginx.conf
    ```

4. Build the Nginx:

    ```bash
    docker build -t openresty-with-debug:latest -f alpine/Dockerfile .
    ```

## How to run the images of the studies

1. Run Nginx:

    ```bash
    make run
    ```

2. In another terminal, attach the strace process to Nginx:

    ```bash
    make strace
    ```

3. In another terminal, reload the Nginx:

    ```bash
    make reload
    ```

4. Make the requests. Each study has its own instructions about this step.
