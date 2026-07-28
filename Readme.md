# Project 2.1 --- Optimizing Container Images using Multi-Stage Docker Builds

## Project Objective

Learn how multi-stage Docker builds reduce image size by separating the
**build environment** from the **runtime environment**.

### Why Multi-Stage Builds?

A traditional Dockerfile keeps build tools (compiler, SDK, package
manager) inside the final image, making it larger and less secure.

A multi-stage Dockerfile: - Compiles the application in one stage. -
Copies only the compiled binary into a minimal runtime image. - Produces
a smaller, faster, and more secure container.

------------------------------------------------------------------------

# Project Architecture

``` text
                 Developer
                     │
                     ▼
             Go Source Code
          (calculator.go)
                     │
                     ▼
        Multi-Stage Dockerfile
                     │
      ┌──────────────┴──────────────┐
      ▼                             ▼
 Build Stage (Ubuntu + Go)     Runtime Stage (scratch)
 Install Go compiler            No compiler/tools
 Compile application            Copy compiled binary
      │                             │
      └──────────────┬──────────────┘
                     ▼
          Lightweight Docker Image
                     │
                     ▼
          Running Calculator Container
```

------------------------------------------------------------------------

# Project Workflow

``` text
Create Go Application
        │
        ▼
Create Traditional Dockerfile
        │
        ▼
Build Traditional Image
        │
        ▼
Create Multi-Stage Dockerfile
        │
        ▼
Compile Application (Build Stage)
        │
        ▼
Copy Binary to scratch Image
        │
        ▼
Build Optimized Image
        │
        ▼
Run Container
        │
        ▼
Compare Image Sizes
```

------------------------------------------------------------------------

# Project Files

``` text
multi-stage-Docker-Build/
│
├── calculator.go
├── Dockerfile                 # Multi-stage Dockerfile
├── nomultistagedockerfile      # Traditional Dockerfile
└── screenshots/
```

------------------------------------------------------------------------

# Traditional Dockerfile

``` dockerfile
FROM ubuntu AS build

RUN apt-get update && apt-get install -y golang-go

ENV GO111MODULE=off

COPY . .

RUN CGO_ENABLED=0 go build -o /app .

ENTRYPOINT ["/app"]
```

### Drawbacks

-   Final image contains Ubuntu.
-   Go compiler remains inside image.
-   Larger image size.
-   Higher attack surface.

------------------------------------------------------------------------

# Multi-Stage Dockerfile

``` dockerfile
FROM ubuntu AS build

RUN apt-get update && apt-get install -y golang-go

ENV GO111MODULE=off

COPY . .

RUN CGO_ENABLED=0 go build -o /app .

FROM scratch

COPY --from=build /app /app

ENTRYPOINT ["/app"]
```

### Benefits

-   Build tools are removed.
-   Only compiled binary is copied.
-   Very small final image.
-   Faster downloads and deployments.

------------------------------------------------------------------------

# Step-by-Step Commands

## 1. Verify project

``` bash
ls
```

Expected:

``` text
Dockerfile
calculator.go
nomultistagedockerfile
screenshots/
```

## 2. Build traditional image

``` bash
docker build -f nomultistagedockerfile -t calculator-normal .
```

## 3. Build multi-stage image

``` bash
docker build -t calculator-multistage .
```

## 4. Compare image sizes

``` bash
docker images
```

Observe that `calculator-multistage` is much smaller.

## 5. Run traditional image

``` bash
docker run --rm calculator-normal
```

## 6. Run multi-stage image

``` bash
docker run --rm calculator-multistage
```

Both should produce the same application output.

## 7. Inspect images

``` bash
docker image inspect calculator-normal
docker image inspect calculator-multistage
```

------------------------------------------------------------------------

# How the Multi-Stage Build Works

1.  Docker starts the **build** stage using Ubuntu.
2.  Go compiler is installed.
3.  `calculator.go` is compiled into `/app`.
4.  A new stage begins using `scratch` (empty image).
5.  Only the compiled `/app` binary is copied.
6.  The container starts by executing `/app`.

------------------------------------------------------------------------

# Important Docker Commands

``` bash
docker build -t calculator-multistage .
docker build -f nomultistagedockerfile -t calculator-normal .
docker images
docker run --rm calculator-multistage
docker run --rm calculator-normal
docker image inspect calculator-multistage
docker history calculator-multistage
docker rmi calculator-normal calculator-multistage
```

------------------------------------------------------------------------

# Comparison

  Feature                      Traditional   Multi-Stage
  ---------------------------- ------------- ----------------
  Go compiler in final image   Yes           No
  Ubuntu in final image        Yes           No (`scratch`)
  Image size                   Large         Small
  Security                     Lower         Higher
  Deployment speed             Slower        Faster

------------------------------------------------------------------------

# Expected Outcome

-   Traditional and multi-stage images are successfully built.
-   Both run the same application.
-   Multi-stage image is significantly smaller.
-   Build tools are excluded from the final image.
-   You understand how build and runtime stages are separated.
