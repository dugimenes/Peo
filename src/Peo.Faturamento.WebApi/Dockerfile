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
COPY ["src/Peo.Faturamento.WebApi/Peo.Faturamento.WebApi.csproj", "src/Peo.Faturamento.WebApi/"]
COPY ["src/Peo.Faturamento.Application/Peo.Faturamento.Application.csproj", "src/Peo.Faturamento.Application/"]
COPY ["src/Peo.Faturamento.Domain/Peo.Faturamento.Domain.csproj", "src/Peo.Faturamento.Domain/"]
COPY ["src/Peo.Faturamento.Infra.Data/Peo.Faturamento.Infra.Data.csproj", "src/Peo.Faturamento.Infra.Data/"]
COPY ["src/Peo.Faturamento.Integrations.Paypal/Peo.Faturamento.Integrations.Paypal.csproj", "src/Peo.Faturamento.Integrations.Paypal/"]
COPY ["src/Peo.Core/Peo.Core.csproj", "src/Peo.Core/"]
COPY ["src/Peo.Core.Infra.Data/Peo.Core.Infra.Data.csproj", "src/Peo.Core.Infra.Data/"]
COPY ["src/Peo.Core.Infra.ServiceBus/Peo.Core.Infra.ServiceBus.csproj", "src/Peo.Core.Infra.ServiceBus/"]
COPY ["src/Peo.Core.Web/Peo.Core.Web.csproj", "src/Peo.Core.Web/"]
COPY ["src/Peo.ServiceDefaults/Peo.ServiceDefaults.csproj", "src/Peo.ServiceDefaults/"]

# Restore dependencies
RUN dotnet restore "src/Peo.Faturamento.WebApi/Peo.Faturamento.WebApi.csproj" \
    --runtime linux-musl-x64 \
    --disable-parallel

# Build stage
FROM restore AS build
ARG BUILD_CONFIGURATION
WORKDIR /src

# Copy all source code
COPY ["src/", "src/"]

# Build the application
WORKDIR "/src/src/Peo.Faturamento.WebApi"
RUN dotnet build "Peo.Faturamento.WebApi.csproj" \
    -c ${BUILD_CONFIGURATION} \
    -o /app/build \
    --no-restore \
    --runtime linux-musl-x64

# Publish stage
FROM restore AS publish
ARG BUILD_CONFIGURATION
WORKDIR /src

# Copy all source code
COPY ["src/", "src/"]

# Publish directly (without separate build step)
WORKDIR "/src/src/Peo.Faturamento.WebApi"
RUN dotnet publish "Peo.Faturamento.WebApi.csproj" \
    -c ${BUILD_CONFIGURATION} \
    -o /app/publish \
    --no-restore \
    --runtime linux-musl-x64 \
    --self-contained false \
    /p:UseAppHost=false

# Final stage
FROM base AS final
ARG APP_VERSION

# Labels for metadata
LABEL maintainer="PEO Platform Team" \
      version="${APP_VERSION}" \
      description="PEO Billing Service - Payment Processing API" \
      org.opencontainers.image.title="peo-faturamento-webapi" \
      org.opencontainers.image.description="Billing and Payment Processing API for PEO Platform" \
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
ENTRYPOINT ["dotnet", "Peo.Faturamento.WebApi.dll"]
