# GitHub Meta Endpoint Monitor

A GitHub Actions workflow that monitors GitHub's [`/meta` endpoint](https://docs.github.com/en/rest/meta/meta?apiVersion=2022-11-28#get-github-meta-information) for changes to service IP ranges and reports additions and removals.

## What it monitors

The workflow tracks the `hooks`, `actions`, `web`, `api`, `git`, `pages`, and `packages` sections of `https://api.github.com/meta`. It sorts each section before comparison, so changes in response ordering alone do not trigger an alert.

Runs are scheduled hourly and can also be started manually with **Actions → monitor-meta → Run workflow**. The first run saves a baseline without opening an issue. Later runs compare against that baseline and, when ranges change:

- Create or update an open **GitHub Meta IP Change Detected** issue with separate added and removed ranges.
- Attach the complete diff as the `github-meta-ip-diff` workflow artifact. If the issue body is too large, it is truncated with a link to the artifact.
- Send an optional webhook notification.
- Save the new baseline for the next comparison.

The workflow creates the `ip-change` and `github-meta` labels if they do not already exist. Concurrent runs are queued rather than canceling an active run, and invalid or inconsistent baseline data causes the workflow to fail instead of silently reporting misleading changes.

## Setup

1. Fork or clone the repository and enable GitHub Actions.
2. Ensure the workflow runs on the repository's default branch.
3. To receive webhook alerts, add a repository Actions secret named `WEBHOOK_URL` (see [Webhook notifications](#webhook-notifications)).
4. Run `monitor-meta` manually once to initialize the baseline. Subsequent runs will report changes.

The workflow needs these repository permissions:

| Permission | Purpose |
| --- | --- |
| `contents: write` | Commit the initial and updated baseline |
| `issues: write` | Create labels and create or update the change issue |

These permissions are declared in `.github/workflows/monitor-meta.yml`. Organization or repository policies may restrict them; if so, allow the workflow the permissions listed above.

## Webhook notifications

Set the `WEBHOOK_URL` repository secret to an endpoint that accepts a JSON `POST`. The workflow sends a Slack-compatible payload containing the number of added and removed ranges, changed sections, and a link to the workflow run. If the secret is unset, the notification step is skipped.

Example payload:

```json
{
  "text": ":rotating_light: GitHub Meta IP Change Detected — 3 added, 1 removed\nChanged sections: actions,hooks\nDetails: https://github.com/org/repo/actions/runs/123"
}
```

Slack incoming webhooks are supported. Discord can use its [Slack-compatible webhook endpoint](https://discord.com/developers/docs/resources/webhook#execute-slackcompatible-webhook). Webhook requests have connection and overall timeouts; a failed request emits a workflow warning but does not prevent the new baseline from being saved.

## Baseline files

- `.meta.last.json` stores the last successfully processed IP ranges.
- `.meta.last.hash` stores the SHA-256 hash of that exact JSON file.

Keep both files committed and in sync. If either file is missing, malformed, or does not match the other, the workflow fails and reports the problem rather than updating the baseline from untrusted state.

## Contributing

Pull requests and feedback are welcome. Changes to tracked metadata sections should be reflected in the workflow's fetch, validation, and comparison steps.
