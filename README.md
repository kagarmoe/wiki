# gt-wiki

An unofficial code-grounded wiki for [gastownhall/gastown](https://github.com/gastownhall/gastown).

This wiki maps the gastown codebase to produce an authoritative rewrite of gastown's documentation. Code is the source of truth; docs are downstream and not authoritative. The upstream project is owned by Steve Yegge / the gastownhall organization, and this wiki exists as a reader's / contributor's map, not an authoritative product doc.

## Current Status

- **Phase 2 (mapping) complete**: All 213 pages of the gastown codebase have been mapped with code-grounded descriptions.
- **Phase 3 (drift analysis)**: Ongoing — comparing docs/ claims to code-grounded pages to identify discrepancies.

## How to Read This Wiki

Start at [index.md](index.md) for a catalog of all pages, organized by topic (currently gastown only).

Pages follow a consistent structure:
- **What it actually does**: Code-grounded description with absolute-path file:line references.
- **Docs claim**: What README, docs/, help text say — with source references.
- **Drift**: Contradictions between "actually does" and "docs claim".
- **Notes / open questions**: Freeform synthesis.

All cross-links are relative markdown links (no wikilinks) for portability.

## Contributing

This wiki welcomes contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

If gastownhall would like to adopt this wiki, I'd happily transfer it.

## License

This work is licensed under [Creative Commons Attribution 4.0 International](LICENSE).