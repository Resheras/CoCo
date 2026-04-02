---
name: agent-best-practices
description: Audit and proofread a Snowflake Cortex Agent's configuration (instructions, orchestration, tool descriptions, tool resources, profile) against a comprehensive best-practices checklist. Use when the user asks to review, audit, proofread, or improve an agent.
---

# Cortex Agent Best Practices Audit

## How to Use
1. Run `DESCRIBE AGENT <fully_qualified_agent_name>` to get the full agent_spec
2. If the spec is truncated, extract it in chunks using `SUBSTR($7, <offset>, 1000)` from `TABLE(RESULT_SCAN(LAST_QUERY_ID()))`
3. Evaluate every section of the agent_spec against ALL 15 checks below
4. Report findings in a table grouped by severity: Critical / High / Nice-to-have

## The 15-Point Checklist

### Correctness & Hygiene
1. **Typos & Spelling** — Scan all instruction text, profile display_name, tool names, and tool_resources for misspellings (e.g., "seach" → "search", "downtream" → "downstream", "VIES" → "VIEW")
2. **Orphan/Placeholder Text** — Look for dangling strings like a bare `url` on its own line, leftover TODO markers, or incomplete sentences ending with "..."
3. **Valid Tool Resource References** — Verify that semantic_view names, search_service names, and warehouse fields are non-empty and plausibly correct (no obvious typos in object names)
4. **Formatting Consistency** — Check for double spaces, inconsistent newlines, mixed bullet styles, or broken markdown

### Instruction Architecture
5. **No Cross-Layer Duplication** — Response instructions should own FORMATTING. Orchestration instructions should own ROUTING/LOGIC. Tool descriptions should own CAPABILITY SCOPE. Flag any rule that appears in more than one layer.
6. **Guardrails Positioned First** — Off-topic refusal, scope boundaries, and safety rules should appear at the TOP of orchestration instructions, not buried in the middle or end
7. **Consolidated "Do Not" Rules** — All negative constraints ("Do not speculate", "Do not suggest", etc.) should live in ONE authoritative list, not scattered across response, orchestration, and tool descriptions

### Domain & Context
8. **Domain Context Present** — Instructions should name the specific business domain, data domains, or organization so the agent understands its world (e.g., "This catalog contains assets spanning bookings, sales, finance...")
9. **Few-Shot Output Examples** — Response instructions should include 1-2 concrete examples of an ideal formatted response, showing the exact layout with links, emojis, metadata fields
10. **Few-Shot Routing Examples** — Orchestration instructions should include 2-3 examples mapping user intent → tool selection → search query construction

### Behavioral Controls
11. **Result Count Cap** — Instructions should explicitly limit how many results are displayed (e.g., "Return at most 5 results unless the user requests more")
12. **Graceful Edge Cases** — Check for explicit handling of: no results found, ambiguous queries, multiple possible interpretations. Each should have template phrasing the agent can use.
13. **Explicit Tool Priority** — Orchestration should state the tool selection order clearly (e.g., "Always use Searcher first; only use Analyst tool after obtaining an asset name from Searcher results")

### Infrastructure & Robustness
14. **Pinned Orchestration Model** — `models.orchestration` should be a specific model name, not `"auto"`, for deterministic behavior. Flag `"auto"` as a risk.
15. **Tuned max_results** — Check the `max_results` value on search tools. Values below 5 may be too restrictive; values above 10 may flood context. Recommend 5-6 for most catalog use cases.

## Output Format

Present findings as critical, high, nice-to-have.


If all 15 checks pass, confirm: "All 15 best-practice checks passed."

## Additional Notes
- Always extract the FULL agent_spec before auditing — truncated specs will miss issues
- Pay special attention to tool_resources section as typos there cause silent failures
- Check the profile section too (display_name, comment)
- If the agent has a Cortex Analyst tool, verify the semantic_view reference exists