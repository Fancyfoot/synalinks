# Wiki Changelog

Append-only chronological log of all substantive edits to the wiki. See git history for full details of each change.

## 2025-09-26

- **2025-09-26 | Initial wiki creation** | Created schema.md, README.md, index.md, log.md (meta files) and all 24 content pages covering modules, training system, core framework, infrastructure, and patterns. Comprehensive coverage of 100+ framework classes synthesized from codebase survey.

---

## Future entries

When you edit the wiki:
1. Add one line to this file: `YYYY-MM-DD | path/to/page.md | One-sentence summary of change`
2. Commit with the change: `git add wiki/log.md` and `git commit -m "docs(wiki): <change>"`
3. Keep entries brief and factual (one sentence per edit; group multiple edits in one commit if they're cohesive)

Example entries:
- `2025-09-27 | modules/core.md | Added new Decision parameter min_confidence`
- `2025-09-27 | training/optimizers.md | Fixed OMEGA signature; added novelty_search_metric param`
- `2025-09-28 | patterns/design-decisions.md | Expanded rationale for composition-over-inheritance`
