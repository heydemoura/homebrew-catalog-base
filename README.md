# Homebrew catalog base

A template for your own PS5 homebrew catalog: a GitHub repository that publishes the
[store API](docs/api.md) ProsperoStore reads, plus a website, from one JSON file per app. It
is the tooling of [darkxex/ps5-homebrew-catalog](https://github.com/darkxex/ps5-homebrew-catalog),
the catalog behind [prosperostore.net](https://prosperostore.net), without that catalog's apps,
keys and bots, so a new catalog can start in minutes.

## Launch your catalog

1. **Use this template** (the green button above) to make a repository of your own. Any name
   works; the examples below call it `my-catalog`.
2. In the new repository's **Settings → Pages**, set **Source** to **GitHub Actions**.
3. Push to `main` (or run **Actions → CI → Run workflow**). The CI tests the records, builds
   the site and publishes it to the repository's GitHub Pages site. After the first run:

   ```
   https://<you>.github.io/my-catalog/            the website
   https://<you>.github.io/my-catalog/api/v1/     the store API
   ```

4. In ProsperoStore, press **Options**, go to the **Catalogs** room (R1) and:
   - switch **Verify custom catalog signatures** off and confirm the warning, since a new
     catalog is published unsigned (see [Signing](#signing));
   - press **Add a GitHub catalog** and type `<you>/my-catalog`.

   Close and reopen the store. Your catalog's apps are added to prosperostore.net's, and each
   one's page names your catalog. Switch **Official catalog** off to see yours alone.

The template lists one app, ProsperoStore itself (`apps/PPSA99000.json`), so the first build has
something to show and a store that reads only your catalog can still update itself. Replace or
remove it as you like.

## Name your catalog

`catalog.json` at the root of the repository names the catalog:

```json
{
  "name": "Homebrew catalog base",
  "maintainer": "",
  "repository": ""
}
```

- `name` is what the website calls itself: the brand in its header, the site name in link
  previews and the `name` of the older `catalog/v1.json` feed. Empty means the repository's name.
- `maintainer` is the GitHub account named in the footer ("Maintained by"). Empty means the
  repository's owner, which is right for most catalogs.
- `repository` is the catalog's GitHub address, only needed for a build outside GitHub Actions
  from a checkout without a GitHub `origin`. The CI always knows its own repository.

The footer's "Built from" link and commit are filled in at build time from the repository and
the commit the CI builds, so neither needs editing.

## Add an app

One file per app in `apps/`, named after its title ID: `apps/PPSA01234.json`.

```json
{
  "titleid": "PPSA01234",
  "name": "Example App",
  "kind": "app",
  "category": "utilities",
  "description": "What the app does, in one or two sentences.",
  "license": "MIT",
  "author": "janedoe",
  "version": "1.0.0",
  "source_repo": "https://github.com/janedoe/example-app",
  "artifact_url": "https://github.com/janedoe/example-app/releases/download/v1.0.0/PPSA01234.zip",
  "sha256": "<sha256 of that exact asset>",
  "icon_url": "https://raw.githubusercontent.com/janedoe/example-app/v1.0.0/sce_sys/icon0.png"
}
```

- `artifact_url` must be a file attached to a published GitHub release: ProsperoStore downloads
  release files from github.com only. `python3 -m catalog digest <artifact_url>` prints the
  `sha256` GitHub reports for it.
- `kind` is `app`, `game` or `tool`; `category` is one of the [categories](docs/metadata.md#categories).
- An app that shares its title ID with a listed one goes in `apps/PPSA01234-<owner>.json`.
- `python3 -m catalog check` validates every record; the CI runs it on each pull request too.

The full record format, release notes, reservations and the review rules are in
[docs/submitting.md](docs/submitting.md), [docs/metadata.md](docs/metadata.md) and
[docs/review-policy.md](docs/review-policy.md). They describe the official catalog's process;
yours can be as strict or as relaxed as you want.

## What the repository does for you

| Workflow | What it does |
| --- | --- |
| **CI** (`ci.yml`) | On every push to `main`: tests, checks the records, builds the website and the API for `https://<you>.github.io/<repository>/`, signs the API when a key is set, and publishes it to GitHub Pages. Pull requests only run the tests. |
| **Submission check** (`pull-request.yml`) | Checks a pull request's records with the base branch's code, reading the PR's files as data only. |
| **Release updates** (`updates.yml`) | Three times a day, looks for newer releases of every listed app and opens one pull request with the bumps. |
| **Catalog health** (`health.yml`) | Re-verifies that listed releases are still published and unchanged. |
| **Scan the catalog** (`scan-catalog.yml`) | Manual: downloads and statically scans the listed releases and writes a report. |

The official catalog's discovery bot and comments sync are not included: one searches GitHub for
apps to list, the other needs a Discussions category; ProsperoStore reads likes and comments from
the official catalog only.

## Signing

A catalog made from this template publishes its API unsigned, and ProsperoStore reads it with
**Verify custom catalog signatures** off. Everything else is still checked: the SHA-256 of each
API file is in `manifest.json`, each download is checked against the record's `sha256`, and
downloads come from GitHub over HTTPS only. [keys/README.md](keys/README.md) says how to add a
signing key when you want one.

## Run the checks locally

```sh
python3 -m pip install -r requirements.txt
python3 -m unittest discover -s tests
python3 -m catalog check
python3 -m catalog build --no-icons --out /tmp/site --site-url https://<you>.github.io --base /my-catalog/
```

More about the build in [docs/website.md](docs/website.md) and about the API in
[docs/api.md](docs/api.md).

## Website branding

The website's words live in `site/shared/i18n.js` and its templates in `site/themes/`; they still
name ProsperoStore and prosperostore.net in places. Change them as you like; the store API does not
depend on them.

## Credits and license

The tooling, documentation and website come from
[darkxex/ps5-homebrew-catalog](https://github.com/darkxex/ps5-homebrew-catalog) by BlackBearReloaded
and its contributors, under the [GNU GPL v3](LICENSE). Third-party notices are in
[THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md). Homebrew is used at your own risk; a catalog
lists apps, it doesn't build or support them.
