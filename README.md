# Leo's prompts 


This repository is where I publish selected operational-mode prompts, primarily for friends and colleagues. It is populated from my private prompts repository: any prompt in the private repo with the suffix `_public` is published here automatically through GitHub Actions. When published to this repository, the `_public` suffix is removed.

The prompts are typed up as markdown (`.md`) files. To use them, it's best to download them or switch to raw view, instead of copying the static HTML rendering.

---

## Table of contents (alphabetically)

* [aircraft_logbook_review.md](./prompts/aircraft_logbook_review.md): an agent to review PDF scans of airplane logs and prepare a consolidated summary of maintainance, highlighting issues of concern, for the purpose of a flying club purchasing the airplane.

---

**Note to self:** when Personal Access Token expires, go to [github.com/settings/tokens](github.com/settings/tokens), create a new one, copy it, execute
```bash
gh secret set PUBLIC_REPO_PAT --repo lgreco/prompts --body "<new-token>"
```
and commit/push.