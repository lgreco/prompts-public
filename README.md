# Leo's prompts 

This a repo where I publish some of my operational mode prompts. The repository is meant mostly for friends and colleagues. This public repository contains content from my private prompts repository. Any prompt with the suffic `_public` in the **private** repo, will be published here through GitHub Actions. The suffix `_public` is dropped in this repository.

---

Table of contents (alphabetically)

* [aircraft_logbook_review.md](./prompts/aircraft_logbook_review.md): an agent to review PDF scans of airplane logs and prepare a consolidated summary of maintainance, highlighting issues of concern, for the purpose of a flying club purchasing the airplane.

**Note to self:** when Personal Access Token expires, go to github.com/settings/tokens, create a new one, copy it, execute

```bash
gh secret set PUBLIC_REPO_PAT --repo lgreco/prompts --body "<your-token>"
```

and commit/push