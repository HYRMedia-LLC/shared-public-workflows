# Shared public workflows

This repository contains reusable GitHub Actions workflows for HYRMedia-LLC repositories.

## New issue automation

The `issue-opened.yml` workflow does two tasks when a repository calls it from an `issues: opened` event:

1. It assigns the configured GitHub users to the new issue.
2. It sends the issue title, description, author, repository, and link to Discord.

The workflow uses GitHub CLI, `jq`, and `curl` from the standard GitHub-hosted runner. It does not use third-party actions.

### Organization setup

Create these Actions settings in the GitHub organization or in each issue-tracking repository:

| Type | Name | Required | Value |
| --- | --- | --- | --- |
| Variable | `ISSUE_ASSIGNEES` | For assignment | Comma-separated GitHub usernames, for example `octocat,hubot` |
| Secret | `DISCORD_ISSUES_WEBHOOK` | For notifications | Discord channel webhook URL |
| Variable | `DISCORD_ISSUE_MENTION` | No | Discord mention, for example `<@&123456789012345678>` |

If you use organization settings, grant the issue-tracking repositories access to each variable and secret. The configured GitHub assignees must have access to the repository.

This repository is public so that public issue-tracking repositories can call its reusable workflows.

### Repository setup

Copy [`examples/issue-opened.yml`](examples/issue-opened.yml) to `.github/workflows/issue-opened.yml` in each issue-tracking repository. The caller explicitly grants `issues: write` so that the reusable workflow can add assignees. A reusable workflow cannot increase the caller's token permissions.

The example uses the `main` branch and receives updates immediately. For stricter change control, create a release tag such as `v1` and replace `@main` with `@v1` in each caller.

### Options

The reusable workflow accepts these inputs:

| Input | Default | Purpose |
| --- | --- | --- |
| `assignees` | Empty | Comma-separated GitHub usernames |
| `discord_username` | `GitHub Issues` | Webhook display name |
| `discord_avatar_url` | Empty | Webhook avatar URL |
| `discord_mention` | Empty | Optional Discord user or role mention |

The `discord_webhook_url` secret is optional. If it is empty, the workflow skips the Discord notification. If `assignees` is empty, the workflow skips assignment.

### Permissions and security

The workflow requests only `contents: read` and `issues: write`. Discord receives issue content, so do not enable this notification in repositories where issue text can contain confidential data. Store the webhook only as an Actions secret; do not store it in a variable or in a workflow file.

## Public and private issue synchronization

The issue synchronization workflows keep public reports linked to private development repositories without publishing internal work:

1. `sync-public-issue-to-private.yml` creates a private mirror and syncs the public title, body, edits, close, and reopen state.
2. `sync-private-issue-state-to-public.yml` sends only close and reopen state back to the public issue.

Private comments, internal notes, labels, assignees, and title or body edits do not go to the public repository. Developers can add information below the `Internal notes` heading in a mirrored issue. A later public edit preserves that information.

### Repository map

| Public issue tracker | Private destination | Routing label |
| --- | --- | --- |
| `team-tracker-issues` | `team-tracker` | None |
| `arspogo-web-issues` | `arspogo-web` | None |
| `whats-the-hundo-issues` | `whats-the-hundo` | `iOS` |
| `whats-the-hundo-issues` | `whats-the-hundo-android` | `Android` |

For What's The Hundo, the workflow waits for a routing label before it creates a private mirror. If both routing labels are present, it creates and maintains both mirrors.

### Authentication

Create an organization Actions secret named `ISSUE_SYNC_TOKEN`. Use a fine-grained personal access token from a service account. Give the token Issues read and write permission and Metadata read permission for these repositories:

- `team-tracker-issues`
- `team-tracker`
- `arspogo-web-issues`
- `arspogo-web`
- `whats-the-hundo-issues`
- `whats-the-hundo`
- `whats-the-hundo-android`

Grant all seven repositories access to the organization secret. The normal `GITHUB_TOKEN` cannot write to a different repository.

### Callers

Use these examples to add the synchronization events to each repository:

- [`sync-public-issue-fixed.yml`](examples/sync-public-issue-fixed.yml) for Team Tracker and ARS Pogo Web.
- [`sync-public-issue-routed.yml`](examples/sync-public-issue-routed.yml) for What's The Hundo.
- [`sync-private-issue-state.yml`](examples/sync-private-issue-state.yml) in each private destination.

The public callers handle `opened`, `edited`, `closed`, `reopened`, `labeled`, and `unlabeled` events. The private callers handle only `closed` and `reopened` events.
