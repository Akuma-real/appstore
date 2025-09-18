# Hitokoto API

## Introduction
Hitokoto API delivers a plug-and-play quote service with built-in categories and lightweight caching. It powers websites, applications, or automation jobs with a simple random sentence endpoint.

## Default Credentials
- No account is required. The service exposes public HTTP APIs only.

## Key Features
- 12 built-in categories (animation, literature, internet, etc.) for random quotes
- Standard JSON response from `/api/hitokoto`, easy to consume in any language
- `/api/categories` lists all categories for UI filtering or client selection
- `/health` endpoint for monitoring or reverse proxy health checks
- Container logs stored under `/app/logs`, ready for troubleshooting

## Notes
- The container listens on port 3000; map it to an available host port in 1Panel (default 5203)
- Runs on Node.js and works well with 256 MB memory or above
- The API is open by default; secure it with firewall, reverse proxy, or authentication middleware if needed
- Logs are mounted to `./data/logs`; clean or rotate them regularly

## Setup Steps
1. Import or update the application in 1Panel and configure the exposed HTTP port (default 5203)
2. Start the stack and verify `http://<host-ip>:<port>/api/hitokoto` returns JSON data
3. Apply access control via reverse proxy or security policies if external exposure is required
4. Integrate the API with your frontend, backend, or scheduled jobs as needed