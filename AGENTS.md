# AGENTS.md

## What this repo is

A [Copier](https://copier.readthedocs.io) template, not an application. `copier.yml` sets
`_subdirectory: template`, so everything under `template/` is the payload copied into target repos.
The top-level files are this repo's own tooling and are not copied. There is no test suite.

## Main gotcha: duplicated files

These files exist twice, byte-for-byte identical, and must stay that way:

- `.editorconfig`, `.yamllint.yaml`, `.pre-commit-config.yaml`, `.gitignore`
- `.vscode/extensions.json`, `.github/dependabot.yml`, `.github/workflows/lint.yml`

Change the root copy so this repo lints, and the `template/` copy so generated repos get the change.
`mise.toml`, `.bumpversion.toml` and `CHANGELOG.md` differ on purpose: the template versions are the
minimal/generic ones shipped to users (no `copier`/`git-cliff` tools, no git-cliff bump hook).

## Commands

- `mise run lint` — `prek run --all-files` (yamllint `--strict`, shellcheck, pre-commit hooks).
  This is the only CI check, run on PRs and pushes to `main`.
- `mise run generate [dir]` — end-to-end check of the template. Cleans `dir` (default
  `output/example-repo`) then runs `copier copy --trust --vcs-ref HEAD . <dir>`. Inspect the
  rendered files. `output/` is **not** gitignored; never commit it.
- `mise run update-changelog -- --commit` — regenerate `CHANGELOG.md` with git-cliff and commit
  it.
- `mise run bump-version` — takes the next version from `git-cliff --bump`, then bump-my-version
  commits `chore: bump version to X.Y.Z`, updates `CHANGELOG.md`, and creates the `vX.Y.Z` tag.
- `mise run git-push` — pushes HEAD and tags. Do not run unless explicitly asked.
- Single hook/file: `prek run <hook-id> --files <paths>` (e.g.
  `prek run yamllint --files copier.yml`).

## Template mechanics

- `.jinja` files are rendered and the suffix stripped (`LICENSE.jinja` → `LICENSE`,
  `README.md.jinja` → `README.md`).
- `{{ _copier_conf.answers_file }}.jinja` writes `.copier-answers.common.yml` into the target.
  Copier owns that file; never edit it by hand. Users update with
  `copier update -a .copier-answers.common.yml`.
- Variables in `copier.yml`: `repo_name`, `repo_description`, `repo_author_name`, used only by
  `template/README.md.jinja` and `template/LICENSE.jinja`.
- `_min_copier_version` is `9`; keep template syntax compatible with it.

## Conventions

- Conventional commits (git-cliff maps types to changelog groups). Work on a branch; `main` only
  receives PR merges.
- YAML: 2-space indent, 120-column limit (`yamllint --strict`). `.copier-answers*.yml` is exempt
  from document-start and `.github/workflows/` from the truthy rule.
- Markdown: 100-column limit (`.editorconfig`).
- Pre-commit refs are pinned commit hashes annotated `# frozen: vX.Y.Z`; bump both copies together.
