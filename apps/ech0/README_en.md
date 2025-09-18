# Ech0

## Product Introduction
Ech0 is a lightweight self-hosted publishing platform for individuals and small teams. It focuses on fast deployment, distraction-free writing, and seamless multi-device access for sharing ideas, links, and media.

## Default Account Information
- The first registered user is promoted to administrator automatically
- No default credentials are provided in the official image; create your own account after deployment

## Key Features
- Markdown editor with live preview and rich media cards
- Built-in CLI/TUI tools for one-click backup, restore, and export
- Multi-user permission management, Twikoo comments, and RSS subscription
- Ech0 Connect for cross-instance content sharing plus PWA support
- Native support for amd64/arm64 with memory usage below 15 MB

## Notes
- Make sure ports 6277 and 6278 are available before deployment
- Change the `JWT_SECRET` to a strong random string to enhance security
- Application data is stored under the mounted `./data` directory; plan backups accordingly
- Expose port 6278 only in trusted networks when enabling SSH/TUI management

## Configuration Steps
1. Import or install Ech0 in 1Panel and set HTTP/SSH ports and the JWT secret via the wizard
2. After the service starts, visit `http://<host-ip>:6277` and register the first admin account
3. Sign in to the console and complete site title, service URL, and other metadata
4. Configure Twikoo comments, Connect subscriptions, and backup strategy as needed
