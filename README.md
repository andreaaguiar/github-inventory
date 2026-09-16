# GitHub Org Inventory

A single-file browser tool for auditing members, outside collaborators, and teams across multiple GitHub organizations.

## Usage

Open `github_inventory.html` directly in any browser.

1. Enter one or more GitHub org names (one per line or comma-separated)
2. Paste a [Personal Access Token](https://github.com/settings/tokens) with `read:org` scope. Add `repo` scope too if you also want to see private repos and team-repo access on private repos
3. Optionally check **Skip activity timestamps** first if you're fetching a large org. It skips the org public-events call and the per-member/collaborator "last public event" call. That's the most expensive part of a big fetch: one extra API call per person
4. Click **Fetch Data**

## Features

### Setup & fetching
- **Remembered org list** — the org names you enter are saved locally and pre-filled next time (the token never is)
- **Token show/hide toggle** — verify what's actually in the token field before fetching
- **Rate-limit badge** — tracks `X-RateLimit-Remaining` from responses already being made, no extra calls

### Views
- **Search** — the default landing tab. Type a login, name, email, team, or repo name to find it across every org and jump straight to it, correctly scoped and filtered. Press `/` or `Cmd/Ctrl+K` from anywhere to jump into it
- **Org Dashboard** — overview card per org with plan, creation date, and counts
- **Per-org refresh** — a ↻ button on each org's dashboard card refetches just that org instead of everything
- **Members** — full member list with roles (admin/member), profile info, last activity, which teams they're on, and repo access reachable via team membership
- **Outside Collaborators** — roster of external users with access to the org's repos (which specific repos isn't exposed by GitHub's API here — see Access-control coverage)
- **Teams** — team listing with privacy, default permission, member roster, and the repos each team grants access to (with permission level)
- **Repos** — every org repo with visibility, archived status, which teams grant access at what permission level, and an estimated unique-user count derived from those teams
- **Activity tracking** — last public event in this org ("In org") and last public event anywhere on GitHub ("On GitHub") per user, pulled from GitHub's public Events API (see Limitations for what this can't see). Skipping this is optional (see below) and reduces API usage

### Stats & scoping
- **Clickable stat cards** — jump straight to the relevant tab/view/filter
- **Members w/o Team stat** — one click to find everyone not on any team
- **In Multiple Orgs stat** — one click to find members who belong to 2+ of the currently loaded orgs (the intersection, not just the union "Unique only" already gives you)
- **Multi-org deduplication** — "Unique only" view (Members, Collaborators) collapses the same user across orgs, matched by their GitHub login. Teams intentionally don't get this treatment — see Access-control coverage below
- **Per-org filtering** — click any org pill to scope all tabs to that org

### Table tools
- **Per-column filtering** — click the funnel icon on any filterable column header for a text or dropdown filter. An "Empty only" option appears where blank is a meaningful state (e.g. members with no team). It's omitted where blank isn't meaningful (e.g. a repo always has a name). Active filters show as removable chips above the table, with a "Clear all"
- **Column sorting** — click a sortable column's label to sort ascending/descending/off
- **Column visibility** — the Columns menu lets you show/hide any column per view, remembered across reloads. The "☰ Columns" button itself turns blue and shows a count (e.g. "2 hidden") whenever anything's hidden, so it's visible without opening the dropdown
- **Sticky table header and first column** — stay oriented while scrolling long or wide tables
- **Risk-colored badges** — repo Visibility (private=green, internal=amber, public=red) and permission levels (read-only=cool blue/cyan, write levels=warm amber→orange→red) are colored consistently. GitHub's API sometimes calls the same permission `pull`/`push` and sometimes `read`/`write`, depending on the field. The displayed text always matches whichever GitHub actually returned. Only the color is unified across both naming schemes
- **Dismissible info banners** — the data-completeness caveats on Outside Collaborators/Repos can be dismissed and restored
- **CSV export** — current tab/view as a CSV file, matching exactly what's on screen: active sort, active column filters, and hidden columns (a hidden column's field is dropped from the export too) all apply
- **JSON export** — a full nested dump (members, collaborators, teams, repos, and the computed team→repo→user access relationships) for every org in the current org-filter scope, independent of whichever tab is active

## Access-control coverage (read this before trusting the numbers)

Team → repo → user access is fully computed from `GET /orgs/{org}/teams/{slug}/repos` and `GET /orgs/{org}/teams/{slug}/members` — no extra API calls per repo. That means:

- **Repo "Est. Users" and "Team Access" only count access granted through a team.** A user added as a *direct collaborator* on a repo (bypassing teams entirely) is invisible here. Listing direct collaborators needs push/admin rights on that specific repo (`GET /repos/{owner}/{repo}/collaborators`), which a read-only org token typically doesn't have. If your org grants access outside of teams, treat these counts as a lower bound. The same blind spot applies to a Member's "Repo Access (via teams)" column, for the same reason (hence the "via teams" label).
- **Org owners have implicit admin on every repo**, regardless of team grants. The Repos tab shows the owner count per org as a note but doesn't include it in "Est. Users," since it applies uniformly rather than per-repo.
- **Outside Collaborators don't show which specific repos they have access to.** `GET /orgs/{org}/outside_collaborators` returns who the collaborators are, not what they can access. Getting that would mean calling `GET /repos/{owner}/{repo}/collaborators` once per repo in the org. That call needs push/admin on that specific repo, not just `read:org`. It also scales with repo count instead of team count, unlike everything else this tool computes. Both reasons are why it's deliberately not done. Treat the Collaborators tab as a roster, not an access map.
- **Teams are never deduped by name across orgs, unlike Members/Collaborators.** A GitHub login is globally unique, so collapsing "Unique only" by login is a correct identity match. A team name is only unique *within* one org. Two unrelated orgs that happen to both have a team called "Engineering" are not the same team. Grouping by name alone would silently merge their member lists and repo grants into one fabricated row. The Teams tab always shows one row per team per org, and the "Teams" stat is a plain total count, not a deduped one.

## Token Permissions

| Scope | Required for |
|---|---|
| `read:org` | Members, teams, outside collaborators, public repos |
| `repo` | Private repos, team-repo access on private repos |

Activity timestamps ("In org" / "On GitHub") don't need `repo` scope. They come from GitHub's public Events API, which serves public data regardless of token scope. See Limitations below for what that means in practice.

This table uses classic personal access token scope names. A fine-grained PAT uses different, more granular permission names. Grant it read access to organization members, teams, and repository contents/metadata to cover the same ground.

If your org enforces SAML SSO, a classic token must also be explicitly **authorized for that org** (Settings → Developer settings → your token → "Enable SSO"). Without that, its calls against that org will silently return partial or no data instead of an obvious error.

Tokens are never sent anywhere except directly to the GitHub API from your browser.

## Limitations

- **Activity data only reflects public-repo/public-event activity, and only within the last 100 events per org and the single most recent event per user.** It cannot see private or internal repo activity for anyone but yourself, no matter how much access your token has. That data simply isn't exposed by GitHub's Events API. Only the Enterprise Audit Log API can see it. A member showing "In org: —, On GitHub: —" is very often not inactive — their work may just live entirely in private/internal repos this tool can't see. Don't read blank activity as a signal to offboard someone without checking further.
- Fetching many orgs with large memberships/teams can be slow — each team costs 2 extra calls (members + repos)
- Requires a token with org access. Public orgs with private member lists will return fewer results
- Direct (non-team) repo collaborators are not enumerated — see "Access-control coverage" above

## License

[MIT](LICENSE)
