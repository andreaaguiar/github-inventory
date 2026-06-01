# GitHub Org Inventory

A single-file browser tool for auditing members, outside collaborators, and teams across multiple GitHub organizations.

## Usage

Open `github_inventory.html` directly in any browser — no server or dependencies required.

1. Enter one or more GitHub org names (one per line or comma-separated)
2. Paste a [Personal Access Token](https://github.com/settings/tokens) with `read:org` scope (add `repo` scope to also see private repo activity)
3. Click **Fetch Data**

## Features

- **Org Dashboard** — overview card per org with plan, creation date, and counts
- **Members** — full member list with roles (admin/member), profile info, and last activity
- **Outside Collaborators** — external users with repo access
- **Teams** — team listing with privacy, permissions, and member roster
- **Multi-org deduplication** — "Unique only" view collapses the same user across orgs
- **Activity tracking** — last org event (from audit log) and last public GitHub event per user
- **Search** — filter by username, name, email, or org across any tab
- **CSV export** — download current view as a CSV file
- **Per-org filtering** — click any org pill to scope all tabs to that org

## Token Permissions

| Scope | Required for |
|---|---|
| `read:org` | Members, teams, outside collaborators |
| `repo` | Private repository activity timestamps |

Tokens are never sent anywhere except directly to the GitHub API from your browser.

## Limitations

- Activity data comes from the public events API (capped at the last 100 events per org/user)
- Fetching many orgs with large memberships can be slow due to per-member API calls
- Requires a token with org access; public orgs with private member lists will return fewer results
