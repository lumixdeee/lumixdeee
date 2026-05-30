# Repo Sources and Update Check

Generated: 2026-05-30
Owner: lumixdeee

Purpose: record source repo URLs for regenerated docs, and provide a repeatable way to check for new or updated repos.

## Source discovery

Primary index:

```text
https://github.com/lumixdeee?tab=repositories
```

This page is enough for public repo discovery. Add extra links only for private repos, non-default branches, renamed repos, mirrors, forks that should be included, or external sources outside this GitHub account.

## Current public repo list

| Repo | URL |
|---|---|
| lmxdi | `https://github.com/lumixdeee/lmxdi` |
| lumixdeee | `https://github.com/lumixdeee/lumixdeee` |
| amphi | `https://github.com/lumixdeee/amphi` |
| robot_bugs_and_frogs | `https://github.com/lumixdeee/robot_bugs_and_frogs` |
| CSP-106 | `https://github.com/lumixdeee/CSP-106` |
| mogri | `https://github.com/lumixdeee/mogri` |
| staff | `https://github.com/lumixdeee/staff` |
| transwhatification | `https://github.com/lumixdeee/transwhatification` |
| system_witch | `https://github.com/lumixdeee/system_witch` |
| lexii | `https://github.com/lumixdeee/lexii` |
| StoryForge101 | `https://github.com/lumixdeee/StoryForge101` |
| dragi | `https://github.com/lumixdeee/dragi` |

## Check for new public repos

### Browser method

1. Open the primary index.
2. Sort by "Last updated".
3. Compare the repo names against the table above.
4. Add any new repo to this file before regenerating docs.

### API method

```bash
curl -s 'https://api.github.com/users/lumixdeee/repos?per_page=100&sort=updated' \
  | jq -r '.[] | [.name, .html_url, .updated_at] | @tsv'
```

Expected check:

```bash
curl -s 'https://api.github.com/users/lumixdeee/repos?per_page=100&sort=updated' \
  | jq -r '.[].name' \
  | sort
```

Compare with:

```text
CSP-106
StoryForge101
amphi
dragi
lexii
lmxdi
lumixdeee
mogri
robot_bugs_and_frogs
staff
system_witch
transwhatification
```

## Update existing local clones

From the parent folder that contains the cloned repos:

```bash
for repo in */.git; do
  dir="${repo%/.git}"
  echo
  echo "== $dir =="
  (
    cd "$dir" || exit 1
    git remote -v
    git remote update --prune
    git status --short --branch
  )
done
```

This fetches remote refs and prunes removed remote branches. It does not merge changes into local branches.

## Pull default branches before doc regen

Use this only when local work is committed or intentionally absent.

```bash
for repo in */.git; do
  dir="${repo%/.git}"
  echo
  echo "== $dir =="
  (
    cd "$dir" || exit 1
    branch="$(git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null | sed 's|^origin/||')"
    if [ -z "$branch" ]; then
      branch="$(git branch --show-current)"
    fi
    git checkout "$branch"
    git pull --ff-only origin "$branch"
  )
done
```

## Clone missing repos

```bash
for url in \
  https://github.com/lumixdeee/lmxdi \
  https://github.com/lumixdeee/lumixdeee \
  https://github.com/lumixdeee/amphi \
  https://github.com/lumixdeee/robot_bugs_and_frogs \
  https://github.com/lumixdeee/CSP-106 \
  https://github.com/lumixdeee/mogri \
  https://github.com/lumixdeee/staff \
  https://github.com/lumixdeee/transwhatification \
  https://github.com/lumixdeee/system_witch \
  https://github.com/lumixdeee/lexii \
  https://github.com/lumixdeee/StoryForge101 \
  https://github.com/lumixdeee/dragi
do
  name="$(basename "$url")"
  if [ ! -d "$name/.git" ]; then
    git clone "$url"
  fi
done
```

## Include extra links when any of these apply

- repo is private
- source is not under `lumixdeee`
- repo has been renamed but old name still appears in generated docs
- docs must use a branch other than the default branch
- docs must include a submodule or external archive
- generated docs depend on releases, issues, wiki pages, or GitHub Pages output
- local folder name differs from repo name, for example `storyforge101` vs `StoryForge101`

## Regeneration rule

Before regenerating docs:

1. Check the source index or API output.
2. Update or clone repos.
3. Record the date.
4. Record any excluded folders.
5. Regenerate docs from the current local checkout.
