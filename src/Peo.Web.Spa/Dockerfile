# syntax=docker/dockerfile:1

# Build arguments
ARG NET_VERSION=9.0
ARG BUILD_CONFIGURATION=Release
ARG APP_VERSION=1.0.0

# Restore stage - Dependencies
FROM mcr.microsoft.com/dotnet/sdk:${NET_VERSION}-alpine AS restore
ARG BUILD_CONFIGURATION
WORKDIR /src

# Copy solution file
COPY ["Peo.sln", "./"]

# Copy all project files first to maximize docker cache for restore
COPY ["src/Peo.Web.Spa/Peo.Web.Spa.csproj", "src/Peo.Web.Spa/"]

# Restore dependencies
RUN dotnet restore "src/Peo.Web.Spa/Peo.Web.Spa.csproj" \
    --runtime browser-wasm \
    --disable-parallel

# Build stage
FROM restore AS build
ARG BUILD_CONFIGURATION
WORKDIR /src

# Copy all source code
COPY ["src/Peo.Web.Spa/", "src/Peo.Web.Spa/"]

# Build the application
WORKDIR "/src/src/Peo.Web.Spa"
RUN dotnet build "Peo.Web.Spa.csproj" \
    -c ${BUILD_CONFIGURATION} \
    -o /app/build \
    --no-restore

# Publish stage - Blazor WASM
FROM restore AS publish
ARG BUILD_CONFIGURATION
ARG APP_VERSION
WORKDIR /src

# Copy all source code
COPY ["src/Peo.Web.Spa/", "src/Peo.Web.Spa/"]

# Publish directly (without separate build step)
WORKDIR "/src/src/Peo.Web.Spa"
RUN dotnet publish "Peo.Web.Spa.csproj" \
    -c ${BUILD_CONFIGURATION} \
    -o /app/publish \
    --no-restore \
    /p:Version=${APP_VERSION}

# Final stage - Nginx for static hosting
FROM nginx:alpine AS final
ARG APP_VERSION

# Labels for metadata
LABEL maintainer="PEO Platform Team" \
      version="${APP_VERSION}" \
      description="PEO Frontend - Blazor WebAssembly SPA" \
      org.opencontainers.image.title="peo-web-spa" \
      org.opencontainers.image.description="Frontend SPA for PEO Platform" \
      org.opencontainers.image.version="${APP_VERSION}" \
      org.opencontainers.image.vendor="PEO Platform" \
      org.opencontainers.image.source="https://github.com/desenvolvedor-io/peo-platform"

# Install curl for health checks
RUN apk add --no-cache curl

# Remove default nginx config
RUN rm -rf /usr/share/nginx/html/*

# Copy published Blazor WASM files
COPY --from=publish /app/publish/wwwroot /usr/share/nginx/html

# Create custom nginx configuration for SPA routing
RUN echo 'server { \n\
    listen 80; \n\
    server_name localhost; \n\
    root /usr/share/nginx/html; \n\
    index index.html; \n\
    \n\
    location / { \n\
        try_files $uri $uri/ /index.html; \n\
    } \n\
    \n\
    location ~* \\.(?:css|js|jpg|jpeg|gif|png|ico|svg|woff|woff2|ttf|eot)$ { \n\
        expires 1y; \n\
        add_header Cache-Control "public, immutable"; \n\
    } \n\
    \n\
    gzip on; \n\
    gzip_vary on; \n\
    gzip_min_length 1024; \n\
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json; \n\
}' > /etc/nginx/conf.d/default.conf

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost/ || exit 1

EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
