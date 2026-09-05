# agent-skills — operating manual

Instructions for any coding agent working in this repository. Read this before changing
anything here.

---

## 1. This repository is public. Treat every change as a publication.

This is a personal repository, which is exactly why it is dangerous. The author's private
notes, work environment, employer's systems, and personal data all sit close at hand while
editing it, and any of them can arrive in a commit by accident.

**Assume nothing is ever deleted.** Git history is public, permanent, and searchable.
Removing a secret in a later commit does not remove it — it stays in the history, in forks,
in clones, and in anything that scraped the repo in between. Repository content, commit
messages, author metadata, branch names, and pull request text are all published.

### Never commit

- **Absolute paths containing a real username.** Write `~/.config/<tool>/` or
  `$HOME/...`, never `/Users/<someone>/...`.
- **Email addresses, phone numbers, physical addresses, account names, or IDs.**
- **Hostnames, IP addresses, internal URLs, VPN or cluster endpoints.**
- **Credentials of any kind** — tokens, keys, passwords, session cookies, `.env` contents.
  This includes expired ones and ones that "aren't real".
- **Names of private repositories, internal tools, plugins, or employers**, including in
  code comments, commit messages, and CI files. A comment naming a private system tells a
  reader it exists and where to look.
- **Real machine output** — directory listings, process lists, shell history, screenshots,
  or config files copied verbatim from a working machine without review.
- **Anyone else's information.** Colleagues, clients, and family members did not consent to
  appear here.

### The rule that decides the ambiguous cases

**A skill documents *method*, not the author's *environment*.**

If a line describes how a tool behaves, it belongs here. If it describes how *this
particular machine* is set up, it does not. A config example is fine when its values are
placeholders or defaults; it is not fine when it is a copy of a real file. When a real
value is genuinely needed to make a point, replace the identifying part with a placeholder
and say so.

If you are unsure whether something is safe to publish, **leave it out and say why** in the
pull request. Omitting a useful detail costs the reader a little. Publishing a private one
cannot be undone.

### Before every commit

Run this and read the output. Empty is the only passing result other than deliberate,
reviewed exceptions.

```sh
git diff --cached -U0 | grep -nE '/(Users|home)/[a-zA-Z0-9._-]+|[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-z]{2,}|\b([0-9]{1,3}\.){3}[0-9]{1,3}\b'
```

Then reread the diff for the categories above that no pattern can catch — private project
names, employer references, and anything that only a human recognises as identifying.

---

## 2. What this repository is

Personal [Agent Skills](https://agentskills.io) for the developer environment, published as
a Claude Code plugin and marketplace. Skills here are vendor-neutral and MIT licensed, so
they are safe to install anywhere, including a machine the author does not own.

```
.claude-plugin/
  marketplace.json          # marketplace "maemresen"
  plugin.json               # plugin "personal-ai" — version gates every PR
skills/<skill-name>/
  SKILL.md                  # frontmatter + mental model + cheap-to-state traps
  references/*.md           # depth, loaded on demand via relative links
```

`skills/` is the plugin's skills directory. A new skill is a new directory under it; no
manifest change is needed to register one.

## 3. Writing a skill

The bar is **non-obvious, and not already in the tool's manual**. A skill earns its place by
covering what costs an hour to discover, not by restating documentation. If a reader could
find it in the first page of the official docs, leave it out and link the docs instead.

Frontmatter requirements from the spec, enforced in CI:

| Field | Required | Constraint |
|---|---|---|
| `name` | yes | Max 64 chars, lowercase letters, digits and hyphens, must match the directory name |
| `description` | yes | Max 1024 chars, says what the skill does *and when to use it* |

The `description` is the routing surface — an agent reads only that to decide whether to
load the skill at all. Write it for matching, and include the phrases a user would actually
type.

Keep `SKILL.md` small. The spec recommends under 500 lines. Depth belongs in `references/`,
linked by relative path, so it costs nothing until it is needed.

Mark version-specific claims with the version they were checked against, and say how they
were checked. A claim that cannot be reproduced should be labelled as unverified rather
than stated as fact.

## 4. Gates on every change

`main` is protected. Direct pushes are rejected for everyone including the owner, so all
work goes through a pull request.

Three checks must pass:

| Check | What it runs |
|---|---|
| Agent Skills spec | `skills-ref validate` over every directory in `skills/` |
| Plugin and marketplace manifests | `claude plugin validate . --strict` |
| Plugin version bumped | `.claude-plugin/plugin.json` version must increase |

**Every pull request must raise the plugin version** in `.claude-plugin/plugin.json`
following semver. A change that forgets it cannot merge.

Validate locally before pushing:

```sh
claude plugin validate . --strict
claude plugin validate ./skills --strict
```
