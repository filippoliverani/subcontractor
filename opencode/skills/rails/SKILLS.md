---
name: Rails
description: >
Conventions and workflow for working on Ruby on Rails projects. Use whenever editing, adding, or refactoring Ruby/Rails code, running tests, or linting. Triggers on any change to .rb files, RSpec specs, models, controllers, services, jobs, migrations, or when the user asks to test, lint, or follow project standards.
---

# Rails

Role: you are a senior software engineer working on a Ruby on Rails codebase. Follow these conventions for every change.

## Mandatory workflow after editing code

After modifying ANY Ruby file, run BOTH of these before considering the task done:

1. **Lint and auto-correct** the changed ruby file(s):
  ```bash
   bundle exec rubocop -A {filename}
  ```
2. **Lint and auto-correct** the changed erb file(s):
  ```bash
   bundle exec erb_lint -a {filename}
  ```
3. **Lint and auto-correct** the changed js file(s):
  ```bash
   yarn run lint-javascript --fix {filename}
  ```
4. **Lint and auto-correct** the changed css file(s):
  ```bash
   bundle yarn run lint-scss --fix {filename}
  ```
5. **Run the tests** for the changed file(s):
  ```bash
   bundle exec rspec {spec_filename}
  ```

Rules:

- Always use `bundle exec` — never call `rspec` or `rubocop` directly.
- Replace `{filename}` with the actual file path(s) you changed. Run on the specific file(s), not the whole suite, unless explicitly asked.
- After `rubocop -A` auto-corrects, re-read the file to confirm the changes are correct, then re-run RSpec.
- Iterate until both rubocop reports no offenses AND all specs pass. Never leave failing tests or lint offenses behind.

## Testing conventions

- Write specs for every new method, class, model, controller, service, and job.
- Follow the existing spec structure and helpers (FactoryBot, shared examples, request specs) — read a neighboring spec first to match the style.
- In unit tests cover the happy path, edge cases, and failure cases, in system tests cover only the happy path
- Use factories, not fixtures, unless the project already uses fixtures.
- Keep unit tests fast and isolated — avoid hitting external services; stub/mock them (WebMock/VCR if present).
- Run the full suite (`bundle exec rspec`) only when explicitly asked or before declaring a large change complete.

## Linting &amp; style

- The project's `.rubocop.yml` is the source of truth. Do not override it inline unless necessary; if you must, use a scoped `# rubocop:disable`/`# rubocop:enable` with a reason.
- Run `bundle exec rubocop -A {filename}` to auto-correct safe and unsafe offenses, then review the diff.
- Match existing code style: 2-space indentation, frozen string literals if the project uses them, double vs. single quotes per config.

## Rails best practices

- **Fat models / skinny controllers**: keep business logic out of controllers. Prefer service objects, query objects, or model methods.
- **Service objects** for multi-step business operations
- **ActiveRecord**: avoid N+1 queries (use `includes`/`preload`/`eager_load`); add database indexes for foreign keys and frequently queried columns; use scopes for reusable queries.
- **Migrations**: reversible (`change` or paired `up`/`down`); never edit a migration that has run in production — add a new one. Keep `schema.rb`/`structure.sql` in sync.
- **Validations &amp; constraints**: enforce data integrity at both the model and database level (NOT NULL, unique indexes, foreign keys).
- **Callbacks**: use sparingly; prefer explicit service calls over hidden side effects.
- **Background jobs** (Sidekiq/ActiveJob) for slow or external work; keep jobs idempotent and pass IDs, not objects.
- **Security**: rely on strong params, avoid raw SQL interpolation (use parameterized queries), respect authorization (Pundit/CanCanCan if present), never log secrets.
- **I18n**: put user-facing strings in locale files when the project uses them.
- **Naming**: follow Rails conventions (singular models, plural tables, RESTful controller actions).

## Before finishing any task

Checklist:

- `bundle exec rubocop -A {filename}` → no offenses
- `bundle exec rspec {spec_filename}` → all green
- New/changed behavior is covered by specs
- No N+1 queries, no leftover debug output (`binding.pry`, `puts`, `byebug`)
- Migrations are reversible and schema is updated
