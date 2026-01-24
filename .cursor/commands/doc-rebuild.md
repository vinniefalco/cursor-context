---
description: Refresh documentation after API changes
---

# Doc Rebuild

## 1. Load Context
- [ ] Find agent guide: `doc/agent-guide.md` > `agent-guide.md` > search repo
- [ ] Read current `doc/` structure
- [ ] Diff public API (`include/`) against existing docs
- [ ] Exclude `detail/`, `impl/` dirs and `detail` namespaces (not public)

## 2. Analyze API & Structure
- [ ] List new/removed/changed public symbols
- [ ] Apply agent-guide section directives first (explicit sections, ordering)
- [ ] Group remaining API into topical sections
- [ ] Order non-specified sections: dependencies first, simple to complex

## 3. Build Introduction Page
- [ ] Context is now rich—write intro page while API understanding is fresh
- [ ] Follow Intro Page Structure (see below) exactly
- [ ] This page sets expectations for entire doc set

## 4. Light Pass (Structure)
- [ ] Create/reorganize section outline
- [ ] One file per section, placeholder content only

## 5. Medium Pass (Skeleton)
- [ ] Add key concepts to each section
- [ ] Insert code example stubs
- [ ] Preserve relevant existing prose

## 6. Final Pass (Complete)
- [ ] Web search each section's domain for context (pain points, common examples)—don't write about findings
- [ ] Fill all content, examples, edge cases
- [ ] Update examples for changed signatures
- [ ] Ensure simple → complex progression

## 7. Verify & Polish
- [ ] For each section: web search domain, confirm coverage, apply touchups
- [ ] All public symbols documented
- [ ] No references to removed API
- [ ] Cross-references valid
- [ ] Delete stale pages, update index files (nav.adoc, etc.)

---

## Key Behaviors

- **Agent guide is authoritative**: If it specifies sections or ordering, follow it exactly
- **Agent guide lookup**: `doc/agent-guide.md` → root `agent-guide.md` → glob search
- **Fill the gaps**: After applying agent-guide directives, structure remaining API smartly
- **Incremental, not scorched earth**: Preserve good existing prose, reorganize as needed
- **Handle guide changes**: If `agent-guide.md` changed, re-read it and adjust approach accordingly
- **API-driven sections**: For non-specified API, identify natural groupings
- **Dependency ordering**: If section B requires section A, A comes first
- **Simple to complex**: Always progress from basic concepts to advanced usage

## Section Arc

Opening (what reader will accomplish) → Simple case with code → Build complexity → Bridge to next

**Exposition/Code Balance:**

- Early: more prose, full paragraphs, short code snippets (API flavor, not full programs)
- Middle: longer examples as concepts build
- End: full program if appropriate—skip boilerplate (long include lists, etc.)

## Non-Obvious Rules

- Document gotchas over happy-path (thread safety, lifetimes, platform quirks)
- "That" (essential, no comma) vs "which" (nonessential, comma)
- Introduce before referencing—no unexplained forward refs
- What? Why? How?—in that order
- Don't list all class members (reference exists); write prose about important ones with example code

## Intro Page Structure (Required)

1. Title and two-ish sentences what problem it solves, with flair
2. What This Library Does (bullet points)
3. What This Library Does Not Do
4. Target Audience (e.g., "experts in networking")
5. Design Philosophy
6. Requirements (assumed knowledge, C++ standard, compilers, dependencies, linking notes)
7. Code Convention Note - callout: "Code examples assume these declarations are in effect:" with `#include` and `using namespace` (agent-guide.md may override)
8. Quick Example (minimal working code)
9. Next Steps (links)
