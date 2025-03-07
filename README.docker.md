# Docker Setup for Mira Converse

This document provides instructions for running Mira Converse in Docker containers.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- NVIDIA GPU with [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) (for server)

## Configuration

1. Create a `.env` file based on the template:

```bash
cp .env.template .env
```

2. Edit the `.env` file to set your configuration:

```env
# Server Configuration
WEBSOCKET_PORT=8765
WEBSOCKET_API_SECRET_KEY=your_secret_key

# Model Paths (absolute paths to model directories)
WHISPER_MODEL_PATH=/path/to/whisper/model
KOKORO_MODEL_PATH=/path/to/kokoro/model

# TTS Configuration
KOKORO_VOICE_NAME=af_heart

# LLM Configuration
MODEL_NAME=gpt-3.5-turbo
API_SECRET_KEY=your_openai_api_key
API_BASE=https://api.openai.com/v1
```

## Building and Running

The Docker setup for MiraConverse is split into separate containers for the server and client components. You'll need to build and run each container separately.

### Server:

```bash
cd server
docker compose build
docker compose up
```

### Client:

```bash
cd client
docker compose build
docker compose up
```

Note: Make sure to set the `WEBSOCKET_HOST` environment variable in the client's .env file to point to the server's address. If running both containers on the same machine, you can use the Docker host's IP address or hostname.

## Environment Variables

You can override environment variables from the command line:

```bash
WEBSOCKET_API_SECRET_KEY=mysecret WHISPER_MODEL_PATH=/models/whisper docker compose up
```

## Volume Mounts

The Docker setup mounts the following volumes:

- Whisper model directory: `${WHISPER_MODEL_PATH}:/models/whisper`
- Kokoro model directory: `${KOKORO_MODEL_PATH}:/models/kokoro`

## Device Access

The client container has access to:

- Sound devices: `/dev/snd:/dev/snd`

**Note:** The client runs in headless mode (`--no-gui` flag) when using Docker, so no display access is required.

The server container has access to:

- NVIDIA GPU (via NVIDIA Container Toolkit)

## Troubleshooting

### Audio Issues

If you encounter audio issues:

1. Make sure the host audio devices are properly configured
2. Check that the container has proper permissions to access `/dev/snd`
3. Try running the client with `network_mode: "host"` (already configured)

### GPU Access

To verify NVIDIA Container Toolkit is working:

```bash
docker run --rm --gpus all nvidia/cuda:12.5.0-base-ubuntu22.04 nvidia-smi
```

If this command fails, follow the [NVIDIA Container Toolkit installation guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html).

Note: The server container uses CUDA 12.5.0, which is compatible with host systems running CUDA 12.5.0 or later.

### Network Issues

If the client cannot connect to the server:

1. Check that both containers are running:
```bash
docker compose ps
```

2. Verify the WebSocket port is correctly exposed:
```bash
docker compose logs server | grep "WebSocket server"
```

## Running in Production

For production deployments:

1. Use a proper secret management solution
2. Consider using Docker Swarm or Kubernetes for orchestration
3. Set up proper logging and monitoring
4. Use a reverse proxy for WebSocket connections if needed

## Development Workflow

For development:

1. Mount source code as volumes to enable live code changes:

```yaml
volumes:
  - ./server:/app  # For server
  - ./client:/app  # For client
```

2. Use environment-specific .env files:

```bash
# Development
docker compose --env-file .env.development up

# Production
docker compose --env-file .env.production up
```
