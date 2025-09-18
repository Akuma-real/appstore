# Certimate

## Product Overview
Certimate is a self-hosted SSL/TLS certificate automation platform that streamlines issuance, renewal, and delivery. With visual workflows and rich integrations, it embeds certificate lifecycle management into existing operations and reduces the effort of maintaining multiple domains or environments.

## Default Accounts
- Default administrator username: `admin@certimate.fun`
- Default administrator password: `1234567890`
- Change the password after the first login and configure alert channels such as email or Webhook

## Key Features
- Automates certificate issuance and renewal with DNS-01 and HTTP-01 challenges
- Supports single, multi-domain, and wildcard certificates with RSA or ECC algorithms
- Integrates with 40+ DNS providers and 100+ deployment targets across major clouds and infrastructures
- Provides workflow engine, notification channels, and certificate format conversion (PEM/PFX/JKS)
- Fully self-hosted deployment keeps data on your servers for security and compliance

## Notes
- Use the Docker image tag that matches the app version; review release notes before upgrading
- Prepare credentials for DNS providers or deployment targets that Certimate needs to access
- Application data resides in the mounted `./data` directory; plan regular backups and retention
- When exposed to the public internet, protect the dashboard with HTTPS and additional access controls

## Setup Steps
1. Import the Certimate app in 1Panel and confirm the HTTP port mapping (default 8090)
2. After deployment, visit `http://<host IP>:<mapped-port>` and log in with the default credentials
3. Configure DNS providers, deployment targets, and notification channels, then build certificate workflows
4. Track issued certificates in the repository view, export them when needed, or automate delivery to targets
