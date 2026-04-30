---
name: Policy change proposal (H / P / C / MVVM / AT)
about: Propose adding, modifying, or reversing an opinionated rule
title: "[policy] <H/P/C/MVVM/AT>### — <rule summary>"
labels: policy, proposal
---

> Policies are the heart of this kit. Changes to them require ≥7 days of community feedback before merging (unless trivially typographic).

## Type of change

- [ ] Add a new rule (e.g., H10)
- [ ] Modify an existing rule (clarification, edge case)
- [ ] Reverse / remove an existing rule (this is a BREAKING CHANGE)

## Rule

<!-- State the proposed rule in one sentence, in the same voice as the existing rules.
     Example: "H10 — Avoid late initialization in constructors." -->

## Why (the incident, constraint, or trade-off)

<!-- This is the most important section. Be specific.
     - What bug or class of bugs does this prevent?
     - Did you encounter this in production? Cite the symptom.
     - What constraint (compiler, framework, mobile target) makes this necessary?
     - What's the trade-off? (every rule has one) -->

## How to apply

<!-- When does this rule kick in? When does it NOT apply?
     Give 1-2 concrete code examples (good vs. bad). -->

## Counter-arguments you considered

<!-- The strongest case AGAINST this rule. If you can't think of one, you haven't thought hard enough. -->

## Existing rules this interacts with

<!-- Does it conflict with H4? Does it strengthen P3? Does it make C2 redundant? -->

## Adoption evidence

- [ ] I have applied this in production for ≥3 months
- [ ] I have applied this in production for <3 months
- [ ] I have prototyped it but not used in production
- [ ] I have not used it; this is a theoretical proposal

## Proposed RATIONALE.md entry (draft)

<!-- Write the section as it should appear in RATIONALE.md, with **Why:** and **How to apply:** lines. -->
