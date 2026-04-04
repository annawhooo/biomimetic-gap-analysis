# Biomimetic Gap Analysis: Immune System Structural Patterns Applied to Agentic AI Security

**Authors:** Anna Hix, Shaun Milligan

**DOI:** [10.5281/zenodo.19411502](https://doi.org/10.5281/zenodo.19411502) (v2) | [10.5281/zenodo.19393455](https://doi.org/10.5281/zenodo.19393455) (v1)

**Status:** Working draft — independent research (April 2026)

## Abstract

As AI agents increasingly operate with autonomous access to enterprise identity infrastructure — holding credentials, calling APIs, making tool-use decisions — the security community lacks a coherent framework for detecting when these agents are compromised or misbehaving. This paper introduces *biomimetic gap analysis*: a methodology that decomposes biological immune mechanisms across multiple kingdoms of life (mammals, plants, bacteria, insects, parasites, viruses) into abstract structural patterns and maps them against current agent security architecture. Feasibility assessment against current agent frameworks (MCP, LangChain) finds that 14 of 19 attack scenarios (74%) are trivially or feasibly exploitable using documented framework behaviors.

## Contents

- **36** cross-domain mappings from immune mechanisms to agent security equivalents
- **35** risk-prioritized design principles for agent behavioral monitoring
- **19** biologically-derived attack scenarios with paired defensive mitigations
- **8** immune failure modes with safeguards identifying which design principles are at risk
- **3** framework compliance mappings (NIST CSF 2.0, OWASP LLM Top 10 2025, MITRE ATT&CK)
- **1** feasibility assessment grading all 19 scenarios against MCP and LangChain
- **170** references spanning immunology, computer science, and security architecture

## v2 Changelog (April 2026)

- Added mappings #35 (autotomy / sacrificial decoy) and #36 (decoy antigen shedding)
- Added 7 immune failure modes with safeguards (Section 13.1)
- Added framework alignment mappings: NIST CSF 2.0, OWASP LLM Top 10, MITRE ATT&CK (Section 12.5)
- Added feasibility assessment of all 18 attack scenarios against MCP/LangChain (Section 12.6)
- Expanded Related Work with 8 additional references from Google Scholar sweep
- Updated abstract with 72% feasibility finding

## v2.1 Changelog (April 2026)

- Added Attack Scenario #19: Motivation-Aligned Fabricated Authorization (TRIVIAL feasibility)
- Added immune failure mode #8: Motivation-Aligned Tolerance (human-granted immune privilege)
- Added cross-reference block connecting 5 mappings (#3, #7, #8e, #14, #30) via fabricated authorization chain
- Updated feasibility summary: 6 TRIVIAL, 8 FEASIBLE, 4 ADVANCED — 14 of 19 (74%)
- Added MITRE ATT&CK and OWASP LLM06 mappings for Scenario #19

## Files

- `immune-security-mappings.pdf` — formatted paper (cite via the Zenodo DOI above)
- `immune-security-mappings.md` — markdown source

## Related

- [biomimetic-defense-catalog](https://github.com/annawhooo/biomimetic-defense-catalog) — the open catalog project growing from this paper (MITRE ATT&CK for biological defense mechanisms)
- [coffer-mcp](https://github.com/annawhooo/coffer-mcp) — the credential management tool for MCP servers whose development exposed the gap this paper addresses

## License

CC-BY-4.0
