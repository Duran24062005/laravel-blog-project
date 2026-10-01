# Vercel deployment plan

## Problem and goal

Make this Laravel application deployable as a serverless PHP application on Vercel while keeping local development unchanged.

## Scope

- Add a Vercel PHP entrypoint that forwards requests to Laravel's existing front controller.
- Build Vite assets during deployment and expose them from `public/build`.
- Route application requests to the Laravel entrypoint.
- Document the environment and storage requirements that Vercel cannot provide by itself.

## Implementation approach

Vercel uses the community `vercel-php` runtime for `api/index.php`. The project keeps Laravel's normal `public/index.php` and adds only a thin adapter under `api/`. `vercel.json` installs Composer and pnpm dependencies, builds frontend assets, serves known static files, and forwards the remaining requests to Laravel.

## Required production configuration

The Vercel project must define at least `APP_KEY`, `APP_ENV=production`, `APP_DEBUG=false`, `APP_URL`, and the external database variables required by `config/database.php`. The default SQLite configuration in `.env.example` is suitable for local development, but should not be used as persistent production storage in a serverless deployment.

Sessions, cache, queues, and uploaded files must use durable external services or database-backed drivers. The local filesystem and `/tmp` are ephemeral between function invocations.

## Risks and limitations

- `vercel-php` is a community runtime, not an official Vercel PHP runtime.
- Vercel Functions are stateless; local writes and long-running queue workers are not deployment primitives.
- A database must be provisioned separately and migrations must be run as an explicit release step.
- This change prepares the repository for deployment but does not create a Vercel project, configure secrets, or run a production deployment.
