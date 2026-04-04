# AGENTS Guide

## Project shape (what this repo is)
- This is a JetBrains Writerside documentation project; content lives under `Writerside/topics`, not application source code.
- The published docs instance is `ddd` (`Writerside/writerside.cfg` -> `<instance src="ddd.tree" web-path="/dan-dev-diary" version="2.0"/>`).
- Treat `Writerside/ddd.tree` as the navigation contract; adding a topic file alone does not publish it.
- `Writerside/topics/Overview.md` is the start page and intent statement for the whole knowledge base.

## Core boundaries and data flow
- **Content layer**: Markdown + Writerside XML tags in `Writerside/topics/*.md`.
- **Structure layer**: TOC hierarchy in `Writerside/ddd.tree`, metadata in `Writerside/c.list` and `Writerside/v.list`.
- **Presentation assets**: images in `Writerside/images` referenced by filename (for example `loki-installed-pods.png`).
- **Delivery pipeline**: `.github/workflows/deploy.yml` builds `webHelpDDD2-all.zip`, validates with checker, deploys to GitHub Pages, then publishes Algolia indexes.

## Authoring conventions specific to this repo
- Use Writerside procedural markup already used across docs: `<procedure>`, `<step>`, `<note>`, `<warning>`, `<tip>`, `<code-block ...>`.
- Prefer task/runbook style pages with concrete commands and outcomes (see `Writerside/topics/Loki-Setup.md`).
- Keep operational postmortem sections when relevant (`Overview`, `Symptoms`, `Root Cause Analysis`, `Timeline` pattern in `Writerside/topics/Booting-Issues-with-Samsung-T7.md`).
- Image usage pattern is inline with local assets (example: `Writerside/topics/Configure-Nexus-As-a-Proxy-for-PyPi.md`).
- Preserve existing filename conventions and spelling quirks unless explicitly asked to refactor (for example `Common-Scipts.md`, `Find-Availabe-Diskspace.md`) because TOC references are exact.

## Safe edit workflow for agents
- When creating a new page: add `Writerside/topics/<Topic>.md` **and** register it in `Writerside/ddd.tree` at the correct location.
- When renaming/removing pages: update `Writerside/ddd.tree` and add/update redirect entries in `Writerside/redirection-rules.xml` to avoid broken published URLs.
- For links between pages, use existing relative-doc style (for example `[Secure Boot Configuration](Secureboot.md)`).
- Keep command examples in fenced blocks via Writerside code blocks (for example `<code-block lang="bash">...</code-block>`).

## CI/CD and external integrations
- Main branch deploy path is in `.github/workflows/deploy.yml` (build -> test -> pages deploy -> Algolia publish).
- Feature branches (`feature/**`) run documentation build/check in `.github/workflows/lint.yaml`.
- Algolia config is split between workflow env and Writerside variables (`Writerside/cfg/buildprofiles.xml` + GitHub secret `ALGOLIA_KEY`).
- Do not introduce or rotate keys/tokens in repo content; treat values in build profiles as configuration artifacts only.

## Practical "where to change what"
- Add/edit article content: `Writerside/topics/*.md`
- Change sidebar/order: `Writerside/ddd.tree`
- Adjust instance/site settings: `Writerside/writerside.cfg`, `Writerside/cfg/buildprofiles.xml`
- Maintain legacy URL compatibility: `Writerside/redirection-rules.xml`
- Update publish behavior: `.github/workflows/deploy.yml`, `.github/workflows/lint.yaml`

