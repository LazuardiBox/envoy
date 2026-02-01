# Envoy & Caddy Proxy Configuration

This repository contains the configuration for a local reverse proxy setup using **Envoy** and **Caddy**.

## Overview

The setup uses **Envoy** as the edge gateway to handle TLS termination and request routing, and **Caddy** as a high-performance static file server for the frontend applications.

### Architecture

```mermaid
graph TD
    User[User] -->|HTTPS :443| Envoy[Envoy Proxy]
    
    subgraph "Envoy Routing"
        Envoy -->|*.toefl.wiki| TOEFL_Group
        Envoy -->|*.cpns.app| CPNS_Group
    end

    subgraph "Services (Localhost)"
        TOEFL_Group -->|auth-server| Port4001[Auth API :4001]
        TOEFL_Group -->|auth-static| Port4002[Auth Static (Caddy) :4002]
        TOEFL_Group -->|platform-server| Port4003[Platform API :4003]
        TOEFL_Group -->|platform-static| Port4004[Platform Static (Caddy) :4004]

        CPNS_Group -->|auth-server| Port5001[Auth API :5001]
        CPNS_Group -->|auth-static| Port5002[Auth Static (Caddy) :5002]
        CPNS_Group -->|platform-server| Port5003[Platform API :5003]
        CPNS_Group -->|platform-static| Port5004[Platform Static (Caddy) :5004]
    end
```

## Directory Structure

- **`envoy/`**: Contains Envoy configurations and certificates.
  - `config/envoy.yaml`: Main Envoy configuration file.
  - `cert/`: TLS certificates for domains.
- **`caddy/`**: Contains Caddy configurations.
  - `Caddyfile`: Caddy configuration for serving static files.

## Domains & Ports Configuration

### TOEFL Wiki (`*.toefl.wiki`)

| Domain | Type | Cluster/Backend | Port | Description |
|--------|------|-----------------|------|-------------|
| `auth-server.toefl.wiki` | API | `auth_server_toefl_backend` | `4001` | Auth Service API |
| `auth.toefl.wiki` | Static | `auth_static_toefl_backend` | `4002` | Auth Frontend (served by Caddy) |
| `platform-server.toefl.wiki` | API | `platform_server_toefl_backend` | `4003` | Platform Service API |
| `platform.toefl.wiki` | Static | `platform_static_toefl_backend` | `4004` | Platform Frontend (served by Caddy) |

### CPNS App (`*.cpns.app`)

| Domain | Type | Cluster/Backend | Port | Description |
|--------|------|-----------------|------|-------------|
| `auth-server.cpns.app` | API | `auth_server_cpns_backend` | `5001` | Auth Service API |
| `auth.cpns.app` | Static | `auth_static_cpns_backend` | `5002` | Auth Frontend (served by Caddy) |
| `platform-server.cpns.app` | API | `platform_server_cpns_backend` | `5003` | Platform Service API |
| `platform.cpns.app` | Static | `platform_static_cpns_backend` | `5004` | Platform Frontend (served by Caddy) |

## Prerequisites

- **Envoy**: Must be installed and accessible.
- **Caddy**: Must be installed and accessible.
- **Local DNS**: Ensure the domains (e.g., `auth.toefl.wiki`) point to `127.0.0.1` in your `/etc/hosts` file.

## Configuration Notes

- **Certificates**: The `envoy.yaml` is configured to look for certificates in `/etc/envoy/cert/`. If running locally, you may need to adjust the paths in `envoy/config/envoy.yaml` to point to the `envoy/cert/` directory in this repository, or mount the directory correctly if using Docker.

## Running the Services

1. **Start Caddy**:
   ```bash
   cd caddy
   caddy run
   ```

2. **Start Envoy**:
   ```bash
   envoy -c envoy/config/envoy.yaml
   ```
