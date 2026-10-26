# NextDNS Denylist Auto-Updater

Automatically sync your NextDNS denylists from one or more upstream blocklists using GitHub Actions. Runs daily, removes stale domains (those no longer in the source), and imports new ones — keeping your denylist aligned with the lists you trust.

---

## Why This Exists

NextDNS offers curated blocklists under their Privacy tab, but doesn't support adding custom list URLs directly; you're limited to whatever they've vetted and chosen to include. Community contributors often publish broader or more targeted lists, but those sources are not natively supported by NextDNS.

So instead of waiting for NextDNS to acknowledge community lists, this workflow takes matters into its own hands: download, normalize, merge, and prune denylists automatically via the API. If they won't host your preferred source, this project will.

---

## Features

- **Multi-blocklist support** — combine any number of blocklist URLs into one unified denylist
- **Stale domain cleanup** — automatically remove domains deleted from upstream sources
- **Delta-aware operations** — only imports additions and removals that matter, reducing needless API churn
- **Zero local state** — all state lives in NextDNS, nothing to commit beyond your config
- **Graceful upstream noise handling** — comments, blank lines, and malformed entries are ignored during processing

---

## How the workflow works

The workflow is intentionally simple and resilient:

1. Reads `blocklists.txt` line by line
2. Ignores blanks and comment lines starting with `#`
3. Downloads each source list and concatenates the results into a temporary combined file
4. Normalizes the data by stripping comments and whitespace
5. Deduplicates entries with `sort -u`
6. Exports the current denylist from NextDNS
7. Imports the cleaned list using `nextdnsctl denylist import --skip-invalid`

This means upstream sources with a few malformed entries do not break the run: the import step skips invalid values instead of failing the entire sync.

---

## Setup

### 1. Fork the Repository

1. Click **Fork** at the top right of this repository
2. Click **Create fork**

You now have your own copy at `github.com/<your-username>/custom-blocklists-nextdns-updater`.

### 2. Configure the required secrets

The current workflow is built around the `nextdnsctl` CLI and uses one configured profile per run.

Add these repository secrets in GitHub:

- `NEXTDNS_ACCOUNT_1_API_KEY` — your NextDNS API key
- `NEXTDNS_PROFILE_ID` — the profile ID for the denylist to update

These are the values the workflow exports into the environment before running `nextdnsctl`.

The included `blocklists.txt` ships with the [ph00lt0 blocklist](https://github.com/ph00lt0/blocklist) by default. You can add more later.

First, get your API key:
1. Log in to [my.nextdns.io/account](https://my.nextdns.io/account)
2. Scroll to the **API** section
3. Copy your API key

Then add it to GitHub:
1. Go to your forked repo on GitHub
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**

### 3. Run the Workflow

The workflow runs automatically every day at 03:00 UTC. To trigger it manually right away:

1. Go to the **Actions** tab in your fork
2. Select **Update NextDNS Denylist** from the left sidebar
3. Click **Run workflow** → select your branch → **Run workflow**
4. Click the run to view live logs

---

## Adding More Blocklists (Optional)

The included `blocklists.txt` contains the ph00lt0 blocklist by default. Add additional sources with one URL per line, keeping comments and blank lines allowed.

Example:

```text
# One blocklist URL per line
# Lines starting with # are ignored
https://raw.githubusercontent.com/ph00lt0/blocklist/master/domains.txt
https://example.com/another-blocklist.txt
```

All sources are merged and deduplicated automatically. If you remove a URL from this file, all domains unique to that source will be cleaned up on the next run.

---

## Recent behavior and caveats

- The workflow uses the `nextdnsctl` package rather than a custom API wrapper.
- Invalid or malformed entries in upstream sources are skipped by `nextdnsctl denylist import --skip-invalid`; this is intentional and prevents a single bad line from breaking the import.
- The current implementation is profile-based and expects a single active profile per workflow run (`NEXTDNS_PROFILE_ID`).
- If you have multiple NextDNS accounts or profiles, you will need to either duplicate the workflow for each target or adapt the workflow to iterate over them.
- The workflow does not keep local state; it only compares the downloaded source data against the live NextDNS denylist.

---

## First Run Warning

> [!WARNING]
> On the first run after enabling stale cleanup, any domains already in your NextDNS denylist that are not in the blocklists will be flagged as stale and removed. If you have manually added domains that you want to keep, make sure they are present in your source lists before the first sync.

---

## Limitations

- Extremely large blocklists may hit NextDNS API rate limits. `nextdnsctl` handles retries, but monitor logs for warnings.
- The script expects a single configured profile per run, identified by `NEXTDNS_PROFILE_ID`.
- GitHub Actions has a maximum runtime of 6 hours per job. Very large setups should monitor execution time.

---

## Attribution

- nextdnsctl — Community CLI for NextDNS management (MIT licensed)
- ph00lt0/blocklist — Default included blocklist source

---

## License

MIT License — see LICENSE file for details.
