# Stage 1
# Builder stage insppired by presentation and https://gohugo.io/getting-started/quick-start/ official documentation
FROM alpine:latest AS builder

# Install the Hugo go app.
RUN apk add --update hugo

WORKDIR /app

# Create hugo website
RUN hugo new site quickstart
WORKDIR quickstart
RUN hugo new theme mato
RUN echo "theme = 'mato'" >> hugo.toml
RUN echo "Matej Slivka 555179" > content/about.md
RUN hugo


# Stage 2
# Deploy stage
# Idea from this repo https://hub.docker.com/r/nginxinc/nginx-unprivileged
FROM nginxinc/nginx-unprivileged:1.25-alpine

# Add OCI annotations as LABELs based on https://github.com/opencontainers/image-spec/blob/main/annotations.md
LABEL org.opencontainers.image.title="PA234 Homework"
LABEL org.opencontainers.image.description="Nginx HUGO container for serving Student name and UCO on port 8080"
LABEL org.opencontainers.image.version="1.0.0"
LABEL org.opencontainers.image.created="2025-03-03"
LABEL org.opencontainers.image.authors="555179@mail.muni.cz"
LABEL org.opencontainers.image.url="https://github.com/ApetorSkol/Infrastuctural-and-Cloud-Systems-Class/tree/main/1.HW/quickstart"
LABEL org.opencontainers.image.documentation="https://github.com/ApetorSkol/Infrastuctural-and-Cloud-Systems-Class/blob/main/1.HW/quickstart/Readme.md"
LABEL org.opencontainers.image.source="https://github.com/ApetorSkol/Infrastuctural-and-Cloud-Systems-Class/tree/main/1.HW/quickstart"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.vendor="MUNI"
LABEL org.opencontainers.image.base.name="pa234hw1"

# Set workdir to the NGINX default directory
WORKDIR /usr/share/nginx/html

# Copy HTML generated HTML into the Workdir
# Set the owner as rootless nginx
COPY --chown=nginx:nginx --from=builder /app/quickstart/public .

# Expose port 8080 so we can access data from container
EXPOSE 8080
