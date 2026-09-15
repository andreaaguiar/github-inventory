# GitHub Org Inventory

A single-file browser tool for auditing members, outside collaborators, and teams across multiple GitHub organizations.

## Usage

Open `github_inventory.html` directly in any browser.

1. Enter one or more GitHub org names (one per line or comma-separated)
2. Paste a [Personal Access Token](https://github.com/settings/tokens) with `read:org` scope (add `repo` scope to also see private repo activity)
3. Optionally check **Skip activity timestamps** first if you're fetching a large org — it skips the audit-log call and the per-member/collaborator "last public event" call (the most expensive part of a big fetch, one extra API call per person)
4. Click **Fetch Data**

## Features

- **Search** — the default landing tab. Type a login, name, email, team, or repo name to find it across every org and jump straight to it, correctly scoped and filtered. Press `/` or `Cmd/Ctrl+K` from anywhere to jump into it
- **Remembered org list** — the org names you enter are saved locally and pre-filled next time (the token never is)
- **Per-org refresh** — a ↻ button on each org's dashboard card refetches just that org instead of everything
- **Org Dashboard** — overview card per org with plan, creation date, and counts
- **Members** — full member list with roles (admin/member), profile info, last activity, which teams they're on, and repo access reachable via team membership
- **Outside Collaborators** — external users with repo access
- **Teams** — team listing with privacy, default permission, member roster, and the repos each team grants access to (with permission level)
- **Repos** — every org repo with visibility, archived status, which teams grant access at what permission level, and an estimated unique-user count derived from those teams
- **Members w/o Team stat** — one click to find everyone not on any team
- **Multi-org deduplication** — "Unique only" view collapses the same user (or team) across orgs
- **Activity tracking** — last org event (from audit log) and last public GitHub event per user; optionally skippable (see below) to cut API usage
- **Per-column filtering** — click the funnel icon on any filterable column header for a text or dropdown filter, including an "Empty only" option where blank is a meaningful state (e.g. members with no team) — omitted where it isn't (e.g. a repo always has a name). Active filters show as removable chips above the table, with a "Clear all"
- **Column sorting** — click a sortable column's label to sort ascending/descending/off
- **Column visibility** — the Columns menu lets you show/hide any column per view, remembered across reloads
- **Clickable stat cards** — jump straight to the relevant tab/view/filter
- **Sticky table header and first column** — stay oriented while scrolling long or wide tables
- **Risk-colored badges** — repo Visibility (private=green, internal=amber, public=red) and permission levels (read-only=cool blue/cyan, write levels=warm amber→orange→red) are colored consistently regardless of which naming GitHub's API happens to use for that field (`pull`/`push` vs. `read`/`write` — displayed text always matches whichever GitHub actually returned, only the color is unified)
- **Dismissible info banners** — the data-completeness caveats on Outside Collaborators/Repos can be dismissed and restored
- **Rate-limit badge** — tracks `X-RateLimit-Remaining` from responses already being made, no extra calls
- **CSV export** — current tab/view as a CSV file, respecting the active sort (not the active filters — exports full data)
- **JSON export** — a full nested dump (members, collaborators, teams, repos, and the computed team→repo→user access relationships) for every org in the current org-filter scope, independent of whichever tab is active
- **Per-org filtering** — click any org pill to scope all tabs to that org
- **Token show/hide toggle** — verify what's actually in the token field before fetching

## Access-control coverage (read this before trusting the numbers)

Team → repo → user access is fully computed from `GET /orgs/{org}/teams/{slug}/repos` and `GET /orgs/{org}/teams/{slug}/members` — no extra API calls per repo. That means:

- **Repo "Est. Users" and "Team Access" only count access granted through a team.** A user added as a *direct collaborator* on a repo (bypassing teams entirely) is invisible here — listing direct collaborators requires push/admin rights on that specific repo (`GET /repos/{owner}/{repo}/collaborators`), which a read-only org token typically doesn't have. If your org grants access outside of teams, treat these counts as a lower bound.
- **Org owners have implicit admin on every repo**, regardless of team grants. The Repos tab shows the owner count per org as a note but doesn't fold it into "Est. Users," since it applies uniformly rather than per-repo.

## Token Permissions

| Scope | Required for |
|---|---|
| `read:org` | Members, teams, outside collaborators, public repos |
| `repo` | Private repos, team-repo access on private repos, activity timestamps |

Tokens are never sent anywhere except directly to the GitHub API from your browser.

## Limitations

- Activity data comes from the public events API (capped at the last 100 events per org/user)
- Fetching many orgs with large memberships/teams can be slow — each team costs 2 extra calls (members + repos)
- Requires a token with org access; public orgs with private member lists will return fewer results
- Direct (non-team) repo collaborators are not enumerated — see "Access-control coverage" above
