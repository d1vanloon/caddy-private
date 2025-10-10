# caddy-private

[![Build and Publish Docker Image](https://github.com/d1vanloon/caddy-private/actions/workflows/docker-publish.yml/badge.svg)](https://github.com/d1vanloon/caddy-private/actions/workflows/docker-publish.yml)

A custom Caddy Docker image with the Cloudflare DNS plugin pre-installed for automated SSL certificate management with Cloudflare DNS challenges.

## Overview

This project provides a Docker image based on the official Caddy web server with the `caddy-dns/cloudflare` plugin built-in. This enables Caddy to automatically obtain and renew SSL certificates using Cloudflare's DNS-01 challenge, which is particularly useful for:

- Wildcard SSL certificates
- Internal services without public HTTP access
- Services behind firewalls or load balancers
- Automated certificate management for multiple domains

## Features

- **Caddy** - Latest stable Caddy web server
- **Cloudflare DNS Plugin** - Built-in support for Cloudflare DNS challenges
- **Multi-architecture** - Supports both `linux/amd64` and `linux/arm64`
- **Automated Builds** - GitHub Actions workflow for continuous integration
- **Version Tagging** - Automatic semantic versioning based on Caddy releases

## Usage

### Pull the Image

```bash
docker pull ghcr.io/d1vanloon/caddy-private:latest
```

### Basic Usage

```bash
docker run -d \
  --name caddy \
  -p 80:80 \
  -p 443:443 \
  -v $(pwd)/Caddyfile:/etc/caddy/Caddyfile \
  -v caddy_data:/data \
  -v caddy_config:/config \
  ghcr.io/d1vanloon/caddy-private:latest
```

### Using with Cloudflare DNS

1. **Get your Cloudflare API token:**
   - Go to [Cloudflare Dashboard](https://dash.cloudflare.com/profile/api-tokens)
   - Create a custom token with `Zone:Read` and `DNS:Edit` permissions
   - Copy the token

2. **Create a Caddyfile:**
   ```caddyfile
   {
     # Cloudflare DNS challenge configuration
     acme_dns cloudflare {
       env CLOUDFLARE_API_TOKEN
     }
   }

   # Example domain with automatic HTTPS
   example.com {
     reverse_proxy localhost:8080
   }

   # Wildcard certificate example
   *.example.com {
     reverse_proxy localhost:8080
   }
   ```

3. **Run with environment variables:**
   ```bash
   docker run -d \
     --name caddy \
     -p 80:80 \
     -p 443:443 \
     -e CLOUDFLARE_API_TOKEN=your_token_here \
     -v $(pwd)/Caddyfile:/etc/caddy/Caddyfile \
     -v caddy_data:/data \
     -v caddy_config:/config \
     ghcr.io/d1vanloon/caddy-private:latest
   ```

### Docker Compose Example

```yaml
version: '3.8'

services:
  caddy:
    image: ghcr.io/d1vanloon/caddy-private:latest
    container_name: caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    environment:
      - CLOUDFLARE_API_TOKEN=your_token_here
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
    networks:
      - caddy

volumes:
  caddy_data:
  caddy_config:

networks:
  caddy:
    external: true
```

## Available Tags

- `latest` - Latest stable build
- `2.10.0` - Specific Caddy version
- `2.10` - Minor version (latest patch)
- `2` - Major version (latest minor)

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `CLOUDFLARE_API_TOKEN` | Cloudflare API token for DNS challenges | Yes (for Cloudflare DNS) |
| `CLOUDFLARE_EMAIL` | Cloudflare account email (legacy) | No |

## Configuration

The image uses the standard Caddy configuration format. Place your `Caddyfile` in the container at `/etc/caddy/Caddyfile` or mount it as a volume.

### Cloudflare DNS Challenge

To use Cloudflare DNS challenges, configure the `acme_dns` directive in your Caddyfile:

```caddyfile
{
  acme_dns cloudflare {
    env CLOUDFLARE_API_TOKEN
  }
}
```

## Building Locally

```bash
# Build the image
docker build -t caddy-private .

# Run locally
docker run -d \
  --name caddy \
  -p 80:80 \
  -p 443:443 \
  -e CLOUDFLARE_API_TOKEN=your_token \
  -v $(pwd)/Caddyfile:/etc/caddy/Caddyfile \
  caddy-private
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## Support

For issues and questions:
- [GitHub Issues](https://github.com/d1vanloon/caddy-private/issues)
- [Caddy Documentation](https://caddyserver.com/docs/)
- [Cloudflare DNS Plugin](https://github.com/caddy-dns/cloudflare)

