# Running Paperclip in Podman on Windows (WSL2)

This guide provides instructions for running Paperclip using Podman Desktop and the Podman CLI on Windows with WSL2.

## Prerequisites

- **Podman Desktop** installed and running.
- **Podman CLI** available in your terminal (WSL2 or PowerShell).
- **WSL2** enabled and configured as the Podman backend.
- **Podman machine** initialized and started (`podman machine start`).

## Quickstart with Podman Compose

Paperclip provides a `docker-compose.yml` that is compatible with `podman-compose`.

1. **Clone the repository:**
   ```bash
   git clone https://github.com/paperclipai/paperclip.git
   cd paperclip
   ```

2. **Set up environment variables:**
   Create a `.env` file in the root of the project with the following content:
   ```env
   BETTER_AUTH_SECRET=placeholder_secret_1234567890
   HOST=0.0.0.0
   ```
   *Note: Setting `HOST=0.0.0.0` is required for the container to be accessible from your Windows host browser.*

   Alternatively, you can set them in your terminal before running the compose command:

   **Bash (WSL2):**
   ```bash
   export BETTER_AUTH_SECRET=placeholder_secret_1234567890
   export HOST=0.0.0.0
   ```

   **PowerShell:**
   ```powershell
   $env:BETTER_AUTH_SECRET="placeholder_secret_1234567890"
   $env:HOST="0.0.0.0"
   ```

3. **Start the services:**
   Using `podman-compose` (ensure it is installed: `pip install podman-compose` or via your package manager):
   ```bash
   podman-compose up --build
   ```
   *Note: On Windows, Podman handles volume mounts through WSL2. The simplest path is to use relative paths as defined in the compose file. If you encounter permission errors, see the Troubleshooting section.*

4. **Access Paperclip:**
   Open your browser and navigate to `http://localhost:3100`.

## RAM Usage

Based on measurements for a separate container setup (Node.js server + PostgreSQL 17), the minimum RAM usage at start is approximately:

- **Paperclip Server (Node.js):** ~280 MB
- **PostgreSQL Database:** ~140 MB
- **Total Startup RAM:** ~420 MB

To check the live RAM usage of your Paperclip instance in Podman:

1. Open a terminal.
2. Run the following command:
   ```bash
   podman stats
   ```
   This will show a live stream of resource usage for all running containers.

## Troubleshooting Podman on Windows

### Volume Permissions
If you encounter permission issues with the data directory, you can try adding the `:Z` or `:z` flag to the volume mapping in `docker-compose.yml`, or manually adjust the permissions within the WSL2 distribution.

Example:
```yaml
    volumes:
      - ./data/docker-paperclip:/paperclip:Z
```

### Networking
Podman on Windows typically maps `localhost` correctly to the WSL2 VM. If you cannot reach the service, ensure that the Podman machine is correctly configured to forward ports.

## Manual Run (Without Compose)

If you prefer to run the containers manually:

1. **Create a Pod:**
   ```bash
   podman pod create --name paperclip-pod -p 3100:3100 -p 5432:5432
   ```

2. **Run Postgres:**
   ```bash
   podman run -d --pod paperclip-pod \
     --name paperclip-db \
     -e POSTGRES_USER=paperclip \
     -e POSTGRES_PASSWORD=paperclip \
     -e POSTGRES_DB=paperclip \
     postgres:17-alpine
   ```

3. **Build and Run Paperclip:**
   ```bash
   podman build -t paperclip-local .
   podman run -d --pod paperclip-pod \
     --name paperclip-server \
     -e DATABASE_URL=postgres://paperclip:paperclip@localhost:5432/paperclip \
     -e BETTER_AUTH_SECRET=placeholder_secret_1234567890 \
     paperclip-local
   ```
