# Changelog

All notable changes to the **x-hots** skill are documented here.
This project follows [Semantic Versioning](https://semver.org/).

## [1.4.1] — 2026-07-20

### Added
- `CHANGELOG.md` tracking notable changes per release.
- Release notes attached to the ClawHub publication.

## [1.4.0] — 2026-07-20

### Added
- Bilingual project README: `README.md` (English) and `README.zh-CN.md` (Chinese), cross-linked.
- `CLAUDE.md` with architecture guidance for the OpenClaw skill / `scripts/x_hots.py` coupling.
- `.gitignore` to keep Python bytecode out of the repo.

### Changed
- Framed the project as an **OpenClaw** skill (driven by an OpenClaw agent) across all docs.
- Bumped skill version to `1.4.0`.

### Removed
- Accidentally committed `scripts/__pycache__/` from the tree.

## [1.3.1] — 2026-07-20

### Added
- Initial ClawHub publication of the X (Twitter) Hot Topics skill.
- Existing `SKILL.md` workflow spec (interest-domain analysis → keyword fetch → background/opinions/controversies) and the `scripts/x_hots.py` data-fetching implementation using Tweepy / X API v2.

## [1.1.1] — 2026-07-16

### Added
- Initial commit: `SKILL.md`, `scripts/x_hots.py`, and `LICENSE`.
