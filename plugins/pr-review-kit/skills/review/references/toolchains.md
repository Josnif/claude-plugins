# Toolchain detection

Mirror the project's CI (`.github/workflows/*`, `.gitlab-ci.yml`, `bitbucket-pipelines.yml`) when it exists. Otherwise, detect the stack from the files below. Prefer frozen or locked installs.

| Marker | Install | Checks |
|---|---|---|
| `composer.json` (+ `artisan` = Laravel) | `composer install --no-interaction --prefer-dist` | `vendor/bin/pint --test` or `vendor/bin/php-cs-fixer fix --dry-run` · `vendor/bin/phpstan analyse` / larastan if configured · `php artisan test` (or `vendor/bin/pest` / `vendor/bin/phpunit`) |
| `package.json` + lockfile | `pnpm i --frozen-lockfile` / `npm ci` / `yarn --immutable` / `bun i --frozen-lockfile` | the `build`, `lint`, `typecheck` and `test` scripts that exist (in monorepos, build before test) |
| `pyproject.toml` / `requirements*.txt` | `uv sync` / `poetry install` / `pip install -r` | `ruff`, `mypy`, `pytest` if configured |
| `go.mod` | — | `go build ./... && go vet ./... && go test ./...` |
| `Cargo.toml` | — | `cargo build && cargo clippy && cargo test` |
| `pom.xml` / `build.gradle*` | — | `mvn -q verify` / `./gradlew check` |
| `Gemfile` | `bundle install` | `rubocop`, `rspec` / `rails test` |
| `Makefile` | — | the `lint` and `test` targets, if present |

## Laravel specifics

- **Env:** copy `.env.example` to `.env` in the worktree, then run `php artisan key:generate`. Never copy the user's real `.env`.
- **Database:** if `phpunit.xml` uses sqlite `:memory:`, the tests need no container. Otherwise start a throwaway MySQL or Postgres container and point `DB_*` at it.
- **Migrations:** `php artisan migrate --force` on an empty DB, then `php artisan migrate:rollback --force` and migrate again. This checks that `down()` works and that migrations are re-runnable. For data-dependent migrations, seed realistic rows before migrating.
- **Sail:** if the project uses Laravel Sail (`docker-compose.yml` with `laravel.test`), you may use `./vendor/bin/sail` instead. Don't start the user's own Sail stack, because it shares their volumes; use separate containers.
- **Frontend assets:** if `vite.config.*` exists and the PR touches `resources/js` or `resources/css`, also run `npm ci && npm run build`.
- **Laravel review hot spots:** mass assignment (`$fillable` / `$guarded`), missing `authorize()` or policies on new routes and form requests, N+1 queries (eager loading), unvalidated `$request->all()`, queued jobs not being idempotent, `config()` called outside config files, missing `->index()` on foreign keys, and `env()` used outside `config/` (it breaks under `config:cache`).
