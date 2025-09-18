# Vaultwarden MySQL

## Product Overview
Vaultwarden is a lightweight, self-hosted implementation of the Bitwarden server. This MySQL edition relies on an external relational database to deliver stable reads and writes while remaining compatible with web, desktop, mobile, and browser extensions.

## Default Accounts
- No default account is provided; the first registered user becomes a regular user
- Configure `ADMIN_TOKEN` during installation to unlock the `/admin` console after deployment

## Key Features
- Full compatibility with Bitwarden clients to sync passwords, secure notes, cards, and identities
- Supports TOTP MFA, password generator, organizations, collections, and event logs
- Handles file attachments, temporary share links, and granular user management
- Ships with a responsive web UI and API endpoints for browser and mobile integrations
- Uses MySQL/MariaDB as the primary store, ideal for multi-user or high-concurrency scenarios

## Notes
- Prepare a reachable MySQL or MariaDB service and verify connectivity before installation
- Set or rotate the `ADMIN_TOKEN` immediately after startup and secure access via HTTPS or a reverse proxy
- Data is persisted in the host-mounted `./data` directory; schedule regular backups
- When exposing the service publicly, consider disabling self-signup or enforcing invitations to prevent abuse

## Setup Steps
1. Import the Vaultwarden MySQL app in 1Panel, select the database service, and confirm host/port parameters
2. Keep or customize the generated database name, user, and password, then configure the HTTP port and optional domain
3. Optionally supply an admin token, deploy the stack, and visit `http://<host>:<mapped-port>` to create your account
4. Access `/admin` to tune mail delivery, organization policies, WebAuthn, and other security settings
