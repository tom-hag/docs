

From https://github.com/GoogleContainerTools/distroless:


> "Distroless" images contain only your application and its runtime dependencies. They do not contain package managers, shells or any other programs you would expect to find in a standard Linux distribution.
> ...
> Restricting what's in your runtime container to precisely what's necessary for your app is a best practice employed by Google and other tech giants that have used containers in production for many years. It improves the signal to noise of scanners (e.g. CVE) and reduces the burden of establishing provenance to just what you need.
> 
> Distroless images are very small. The smallest distroless image, gcr.io/distroless/static-debian12, is around 2 MiB. That's about 50% of the size of alpine (~5 MiB), and less than 2% of the size of debian (124 MiB).

>Note that distroless images by default do not contain a shell. That means the Dockerfile `ENTRYPOINT` command, when defined, must be specified in `vector` form, to avoid the container runtime prefixing with a shell.

>This works:

```dockerfile
ENTRYPOINT ["myapp"]
```

>But this does not work:

```dockerfile
ENTRYPOINT "myapp"
```

>For the same reasons, if the entrypoint is set to the empty vector, the CMD command should be specified in `vector` form (see examples below). Note that by default static, base and cc images have the empty vector entrypoint. Images with an included language runtime have a language specific default (see: [java](https://github.com/GoogleContainerTools/distroless/blob/main/java/README.md#usage), [nodejs](https://github.com/GoogleContainerTools/distroless/blob/main/nodejs/README.md#usage), [python3](https://github.com/GoogleContainerTools/distroless/blob/main/python3/README.md#usage)).

For an example rust api (version `1.8.5`) you can use this Dockerfile:

```Dockerfile
# Build stage
FROM rust:1.85 as builder
WORKDIR /app
ADD . /app
RUN cargo build --release

# Prod stage
FROM gcr.io/distroless/cc
EXPOSE 8080
COPY --from=builder /app/target/release/api /
ENTRYPOINT ["./api"]      
```
 
 and yields the following image size:
 ```console
➜  api main ✗ docker images api --format "table {{.Repository}}\t{{.Size}}"
REPOSITORY              SIZE
playlearn-api-backend   29.7MB


```

which is much smaller than the alpine rust image
```console
➜  api main ✗ docker images rust --format "table {{.Repository}}\t{{.Size}}"
REPOSITORY   SIZE
rust         874MB
```