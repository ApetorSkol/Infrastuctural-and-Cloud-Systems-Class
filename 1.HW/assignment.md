Homework 2 - Multi-stage container build
What is a multi-stage build and why is it useful?
Docker Build Process
In a traditional Docker build process, a single Dockerfile defines all the steps needed to create a container image. This includes installing dependencies, compiling source code, and setting up the runtime environment. The issue with this approach is that all build-time dependencies and tools remain in the final image, leading to large, bloated, and potentially insecure images.
 
Example: Regular Build for a Go Application
```
FROM golang:1.21 

WORKDIR /app 

COPY . . 

RUN go build -o myapp main.go 

CMD ["./myapp"] 
```
Problems with this approach:

The final image contains Go compilers, build tools, and unnecessary dependencies, increasing size.

The attack surface is larger, making the image less secure.

Every time the source code changes, Docker rebuilds everything, making builds inefficient.

Multi-Stage Builds: A Solution to Bloated Images
Docker multi-stage builds solve these problems by allowing multiple FROM instructions in a Dockerfile. Each FROM starts a new independent build stage, allowing selective copying of required artifacts (e.g., compiled binaries) into the final image. This way, the final image contains only what is necessary for runtime.

Example: Multi-Stage Build for the Same Go Application
```
# First stage: Build
FROM golang:1.21 AS builder

WORKDIR /app

COPY . .

RUN go build -o myapp main.go

# Second stage: Minimal runtime image
FROM alpine:latest

WORKDIR /root/

COPY --from=builder /app/myapp .

RUN chmod +x myapp

CMD ["./myapp"]
```
First Stage (builder)

Uses a full golang image to compile the application.

Copies the source code, runs the build process, and produces a binary.

Second Stage (runtime)

Uses a lightweight alpine image (only 5MB).

Copies only the built binary (myapp) from builder.

The final image excludes the Go compiler, making it much smaller and more secure.

Assignment
Your task is to create a simple web application using the Hugo static site generator, incorporating non-trivial modifications that are not available in the upstream source. This assignment is designed to be artificially didactic, requiring you to build Hugo from source and integrate your changes before generating the static website.

Your task is to create a multi-stage Docker build that:

Builds the image from Hugo source code.

Uses the built Hugo binary to generate the static website.

Ensure that the generated website includes your name and UCO, as the UCO will be automatically checked to verify the solution (i.e. on http://localhost/about) by curl and grep.

Serves the static files on port 8080 using an unprivileged Nginx image.

Add basic annotations from OCI specification in Dockerfile to your resulting container image (e.g. author name, email, version, ...)
Publishes the Dockerfile and the resulting container to the e-INFRA CZ Container Registry (see docs). Login to cerit.io using e-INFRA CZ account and you will see container image repository under your username. To push into this repository you will have to tag the container as:

cerit.io/<your_e-infra-cz_login>/website:hw-submission


Make the resulting image the smallest possible. Try to beat less than 10 MB. The size of your final Docker image will be scored and published, allowing you to compare your results with your colleagues. The one with smallest container image will be immediately signed up for Datacentre excursion.
