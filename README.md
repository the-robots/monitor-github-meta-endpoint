# monitor‑github‑meta‑endpoint

A GitHub Actions–based monitor for the [`https://api.github.com/meta`](https://api.github.com/meta) endpoint, tracking subnet changes across GitHub services (e.g. Actions, API, webhooks, Pages).

---

## 🔧 What It Does

- **Fetches** GitHub’s current IP metadata (`hooks`, `actions`, `web`, `api`, `git`, `pages`, `packages`)
- **Compares** against a stored baseline to detect added or removed IPs
- **Creates a GitHub Issue** if changes are detected
- **Persists state** using `.meta.last.json` and `.meta.last.hash`
- **Skips issue creation** on initial run (baseline only)

---

## 🛠️ Use Cases

This project is ideal for teams that:

- Maintain **firewall allowlists** for GitHub-hosted services  
- Manage **webhook endpoints** that only accept GitHub IPs  
- Restrict **GitHub Actions runner traffic** behind IP rules  
- Need audit trails of **GitHub IP range changes over time**

---

## 🚀 Features

- Scheduled to run hourly via cron (`0 * * * *`)
- Generates human-readable diffs with separate **Added** and **Removed** IP sections
- Summarizes changes by pool (e.g. `actions`, `hooks`, etc.)
- Pushes updated baseline only when changes are detected
- Lightweight — no external dependencies or scripts beyond `jq`

---

## 📄 Issue Format Example

```markdown
:rotating_light: GitHub Meta IP Change Detected — 5 added, 2 removed

## :heavy_plus_sign: Added IPs

### actions
- 192.30.252.0/22
- 185.199.108.0/22

### hooks
- 140.82.112.0/20
- 143.55.64.0/20
- 20.201.28.0/24

## :heavy_minus_sign: Removed IPs

### hooks
- 192.30.252.0/23
- 140.82.114.0/24
```

---

## ⚙️ Setup Instructions

1. Fork or clone the repo
2. Enable GitHub Actions
3. Optional: Customize the schedule (`cron`) or IP categories
4. On first run, it will establish a baseline without opening an issue
5. On subsequent changes, a GitHub Issue is automatically opened

---

## 🔐 Permissions Required

Ensure the GitHub Actions workflow has permissions to:

- Read and write repo contents (`contents: write`)
- Create issues (`issues: write`)

---

## 🧩 Optional Enhancements

You can easily extend this setup to:

- Notify Slack or Discord via webhook
- Auto-close stale issues if changes revert
- Create a GitHub App for centralized policy monitoring

---

## 📚 Reference

- GitHub API Docs – [Meta Endpoint](https://docs.github.com/en/rest/meta/meta?apiVersion=2022-11-28#get-github-meta-information)

---

## 🤝 Contributions

Feedback, pull requests, and forks welcome!  
This project is maintained by the GitHub Reliability team for internal tooling and monitoring use cases.
