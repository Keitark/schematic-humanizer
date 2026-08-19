# Repository validation checks

[日本語](validation-checks.ja.md) · [Back to README](../README.md)

`scripts/validate_repo.py` is the single gate for skill packaging, paired
documentation, link integrity, text hygiene, privacy, and asset licensing. The
[validate workflow](../.github/workflows/validate.yml) runs it on Linux and
Windows, and [CONTRIBUTING](../CONTRIBUTING.md) requires it before every pull
request. This page lists what it actually checks.

## Running it

```
python -m pip install "PyYAML==6.0.2"
PYTHONUTF8=1 python scripts/validate_repo.py
```

Every check appends to one shared error list, so a run reports all findings
instead of stopping at the first one. Exit `0` prints a one-line summary; exit
`1` prints `Repository validation failed:` and one `- message` line per finding
to stderr.

## Check groups

| Order | Function | Scope |
|---|---|---|
| 1 | `validate_required` | 23 required repository files |
| 2 | `validate_skills` | 8 skill packages under `.agents/skills/` |
| 3 | `validate_links` | Relative Markdown links in every tracked-tree `*.md` |
| 4 | `validate_text_and_privacy` | Encoding, whitespace, forbidden strings, privacy |
| 5 | `validate_assets` | Case-study and banner asset licensing |

## 1. Required files

Each path must exist as a file; a missing one is reported as
`missing required file: <path>`.

| Area | Files |
|---|---|
| Root | `README.md`, `README.ja.md`, `SUPPORT.md`, `SUPPORT.ja.md`, `CONTRIBUTING.md`, `CONTRIBUTING.ja.md`, `ASSET-LICENSES.md` |
| Licensing | `LICENSE`, `LICENSES/CC-BY-SA-4.0.txt` |
| Guides | `docs/installation`, `docs/choose-a-skill`, `docs/prompts`, `docs/artifact-contracts`, `docs/case-study-nescart`, each as a paired `.md` and `.ja.md` |
| Records | `docs/privacy-review.md` (English only) |
| Assets | `assets/banner.png` |
| Installers | `scripts/install-skills.ps1`, `scripts/install-skills.sh` |

## 2. Skill packages

The directory set under `.agents/skills/` must equal exactly
`manage-pcba-program`, `plan-electronic-product`, `qualify-pcba-sourcing`,
`design-and-review-circuit`, `schematic-humanizer`, `pcb-layout-review`,
`release-pcba-fabrication`, and `operate-jlcpcb-order`. Any extra or missing
directory is a set mismatch.

| Check | Rule |
|---|---|
| Package files | `SKILL.md`, `agents/openai.yaml`, and a standalone `LICENSE` all exist; otherwise the remaining checks for that skill are skipped |
| Standalone license | The skill `LICENSE` is byte-identical to the repository MIT `LICENSE` after stripping |
| Size | `SKILL.md` is at most 500 lines |
| Completeness | `SKILL.md` contains no `TODO` marker, in any case |
| Frontmatter | `SKILL.md` starts with YAML frontmatter that parses and contains only `name` and `description` |
| Body | Content after the frontmatter is not empty |
| Name | `name` equals the directory name and is lowercase kebab-case |
| Description | `description` is at least 40 characters after stripping |
| Interface fields | `openai.yaml` has an `interface` map with non-empty string `display_name`, `short_description`, and `default_prompt` |
| Short description | `short_description` is 25 to 64 characters inclusive |
| Default prompt | `default_prompt` mentions the skill as `$<skill-name>` |

## 3. Markdown links

Every `*.md` outside `.git` is scanned for inline links and images. Targets
starting with `http://`, `https://`, `mailto:`, or `#` are skipped. The rest are
percent-decoded, stripped of their fragment, resolved against the containing
file's directory, and must exist; otherwise the run reports
`broken link in <file>: <target>`.

## 4. Text hygiene and privacy

Files with a `.md`, `.yml`, `.yaml`, `.py`, `.ps1`, `.sh`, `.json`, `.csv`,
`.svg`, or `.txt` suffix are read as UTF-8 and scanned line by line.

| Check | Rule |
|---|---|
| Cache files | No git-tracked `.pyc` or `.pyo` file and no `__pycache__` member |
| Encoding | Every checked file decodes as UTF-8 |
| Whitespace | No line ends with a space or tab; the report includes `<file>:<line>` |
| Forbidden strings | The retired repository URL is absent, including its backslash-normalized form |
| Email address | No address other than the `@example.com` domain |
| Access token | No `gh[pousr]_` or `sk-` style secret of 20 or more characters |
| Credential value | No `api_key`, `access_token`, `secret`, `password`, `cookie`, or `authorization` assignment carrying a value of 12 or more characters |
| Private order identifier | No `pcbfile`, `quote`, `order`, or `project` identifier assignment of 8 or more characters |
| Private order URL identifier | No `pcbFileNo`, `quoteId`, or `orderId` query value of 8 or more characters |
| Absolute Windows path | Drive-letter paths are allowed only under the documentation placeholder `c:\path\to\project` |
| Private POSIX home path | Home paths are allowed only under `/home/user/` or `/Users/user/` |

The literal values `example-token`, `redacted`, `placeholder`, and `fixture-id`
are treated as safe placeholders, and each privacy pattern reports at most one
finding per file. `scripts/validate_repo.py` is exempt from the forbidden-string,
privacy, and absolute-path scans because it contains the patterns themselves; it
is still checked for encoding and trailing whitespace.

## 5. Assets and licensing

`assets/case-studies/nescart/` must exist, every file inside it must appear in
`ASSET-LICENSES.md` by its repository-relative path, and `assets/banner.png`
must be listed as well. A missing case-study directory ends this group early.

## Keeping the list current

Add both files of a new paired guide to the `REQUIRED` tuple in the same change,
update this page and its Japanese pair, and confirm that
[the privacy review](privacy-review.md) still covers every published asset. Text
scanning cannot prove raster privacy, so a new or replaced image needs a
full-resolution visual review instead.
