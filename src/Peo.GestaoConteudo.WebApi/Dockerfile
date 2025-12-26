# syntax=docker/dockerfile:1

# Build arguments
ARG NET_VERSION=9.0
ARG BUILD_CONFIGURATION=Release
ARG APP_VERSION=1.0.0

# Base stage - Runtime environment
FROM mcr.microsoft.com/dotnet/aspnet:${NET_VERSION}-alpine AS base
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

# Install required dependencies for health checks and diagnostics
RUN apk add --no-cache curl

# Create non-root user
RUN addgroup -g 1000 appuser && \
    adduser -u 1000 -G appuser -s /bin/sh -D appuser && \
    chown -R appuser:appuser /app

# Restore stage - Dependencies
FROM mcr.microsoft.com/dotnet/sdk:${NET_VERSION}-alpine AS restore
ARG BUILD_CONFIGURATION
WORKDIR /src

# Copy solution file
COPY ["Peo.sln", "./"]

# Copy all project files first to maximize docker cache for restore
COPY ["src/Peo.GestaoConteudo.WebApi/Peo.GestaoConteudo.WebApi.csproj", "src/Peo.GestaoConteudo.WebApi/"]
COPY ["src/Peo.GestaoConteudo.Application/Peo.GestaoConteudo.Application.csproj", "src/Peo.GestaoConteudo.Application/"]
COPY ["src/Peo.GestaoConteudo.Domain/Peo.GestaoConteudo.Domain.csproj", "src/Peo.GestaoConteudo.Domain/"]
COPY ["src/Peo.GestaoConteudo.Infra.Data/Peo.GestaoConteudo.Infra.Data.csproj", "src/Peo.GestaoConteudo.Infra.Data/"]
COPY ["src/Peo.Core/Peo.Core.csproj", "src/Peo.Core/"]
COPY ["src/Peo.Core.Infra.Data/Peo.Core.Infra.Data.csproj", "src/Peo.Core.Infra.Data/"]
COPY ["src/Peo.Core.Infra.ServiceBus/Peo.Core.Infra.ServiceBus.csproj", "src/Peo.Core.Infra.ServiceBus/"]
COPY ["src/Peo.Core.Web/Peo.Core.Web.csproj", "src/Peo.Core.Web/"]
COPY ["src/Peo.ServiceDefaults/Peo.ServiceDefaults.csproj", "src/Peo.ServiceDefaults/"]

# Restore dependencies
RUN dotnet restore "src/Peo.GestaoConteudo.WebApi/Peo.GestaoConteudo.WebApi.csproj" \
    --runtime linux-musl-x64 \
    --disable-parallel

# Build stage
FROM restore AS build
ARG BUILD_CONFIGURATION
WORKDIR /src

# Copy all source code
COPY ["src/", "src/"]

# Build the application
WORKDIR "/src/src/Peo.GestaoConteudo.WebApi"
RUN dotnet build "Peo.GestaoConteudo.WebApi.csproj" \
    -c ${BUILD_CONFIGURATION} \
    -o /app/build \
    --no-restore \
    --runtime linux-musl-x64

# Publish stage
FROM restore AS publish
ARG BUILD_CONFIGURATION
ARG APP_VERSION
WORKDIR /src

# Copy all source code
COPY ["src/", "src/"]

# Publish directly (without separate build step)
WORKDIR "/src/src/Peo.GestaoConteudo.WebApi"
RUN dotnet publish "Peo.GestaoConteudo.WebApi.csproj" \
    -c ${BUILD_CONFIGURATION} \
    -o /app/publish \
    --no-restore \
    --runtime linux-musl-x64 \
    --self-contained false \
    /p:UseAppHost=false \
    /p:Version=${APP_VERSION}

# Final stage
FROM base AS final
ARG APP_VERSION

# Labels for metadata
LABEL maintainer="PEO Platform Team" \
      version="${APP_VERSION}" \
      description="PEO Content Management Service - Courses and Lessons API" \
      org.opencontainers.image.title="peo-gestao-conteudo-webapi" \
      org.opencontainers.image.description="Content Management API for PEO Platform" \
      org.opencontainers.image.version="${APP_VERSION}" \
      org.opencontainers.image.vendor="PEO Platform" \
      org.opencontainers.image.source="https://github.com/desenvolvedor-io/peo-platform"

# Environment variables
ENV ASPNETCORE_ENVIRONMENT=Production \
    ASPNETCORE_URLS=http://+:8080 \
    DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false \
    DOTNET_RUNNING_IN_CONTAINER=true \
    TZ=America/Sao_Paulo

WORKDIR /app

# Copy published application from publish stage
COPY --from=publish --chown=appuser:appuser /app/publish .

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

# Entry point
ENTRYPOINT ["dotnet", "Peo.GestaoConteudo.WebApi.dll"]
