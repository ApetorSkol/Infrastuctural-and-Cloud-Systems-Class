# Stage 1
# Builder stage insppired by presentation and https://gohugo.io/getting-started/quick-start/ official documentation
FROM alpine:latest AS builder

# Install the Hugo go app.
RUN apk add --update hugo

WORKDIR /app

# Copy Hugo files into container
COPY . .

# Generate HTML using hugo
RUN hugo --minify

# Stage 2
# Deploy stage
FROM nginx:1.25-alpine

# Add OCI annotations as LABELs based on https://github.com/opencontainers/image-spec/blob/main/annotations.md
LABEL org.opencontainers.image.title="PA234 Homework"
LABEL org.opencontainers.image.description="Nginx container for serving Student name and UCO on port 8080"
LABEL org.opencontainers.image.version="1.0.0"
LABEL org.opencontainers.image.created="2025-03-03"
LABEL org.opencontainers.image.authors="555179@mail.muni.cz"
LABEL org.opencontainers.image.url="https://github.com/ApetorSkol/Infrastuctural-and-Cloud-Systems-Class/tree/main/1.HW/quickstart"
LABEL org.opencontainers.image.documentation="https://github.com/yourusername/nginx-unprivileged/README.md"
LABEL org.opencontainers.image.source="https://github.com/yourusername/nginx-unprivileged"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.vendor="MUNI"
LABEL org.opencontainers.image.base.name="pahw"

# Set workdir to the NGINX default directory
WORKDIR /usr/share/nginx/html

# Copy HTML generated HTML into the Workdir
COPY --from=builder /app/public .

# Expose port 80 so we can access data from container
EXPOSE 80/tcp
