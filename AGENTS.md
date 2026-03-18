# Blogging Style Profile

- Tone: Technical but witty.
- Constraints: No corporate jargon.
- Evolutionary History:
  - [Rule 1]: Use Go-style code blocks for analogies.

# Tooling & Workflow

- Always use `mise` for managing dependencies and running tasks.
- For linting markdown, run `mise run lint`.
- For formatting markdown, run `mise run format`.
- For building the site, run `mise run build`.
- Avoid running `hugo`, `go`, or other tools directly if a `mise` task exists.
