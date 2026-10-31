# digital-zhupu | Digital 族譜

A long-term project to reconstruct and preserve Hakka family genealogy (族譜, zúpǔ) as a **private, portable and eventually distributed family archive**.

It is intended to become more than a family tree: a relationship diagram, a family contacts directory, and an evidence-based collection of stories, photographs, documents and oral histories.

**Status: early concept / architecture exploration.** No family data or application has been implemented.

## Read this first

**[Founding discussion and architecture considerations — 1 November 2026](docs/founding-discussion-2026-11-01.md)**

This document captures why the project was proposed, the requirements discussed, suggested technical architecture, multimedia storage choices, branching and reconciliation, privacy, AI assistance, and questions to revisit. It is a **record of thinking**, not a final technical specification.

## Working principles

- **Preserve evidence and uncertainty.** Keep original documents, remembered accounts and competing interpretations distinguishable.
- **Own the data.** Use readable, documented formats; make interfaces, indexes, storage providers and AI tools replaceable.
- **Start with one custodian; support many later.** Family branches may eventually maintain independent copies and reconcile selectively.
- **Keep historical records separate from current contacts.** Living relatives should be able to update private contact information without using Git.
- **Store media as an archive, not scattered attachments.** Link people, events and claims to source records and original digital files.
- **Use AI as an assistant, not an authority.** Require human review for inferred identities, relationships, transcriptions and conflict resolution.
- **No blockchain required.** Git already offers versioning and distributed collaboration; binary storage remains an open design choice.

## Privacy warning

**This GitHub repository was public when these notes were first recorded (2026-11-01).** Do not commit real family contacts, private family records, identifiable media, access tokens or sensitive annotations here. Before importing family data, choose whether this repo should be private or remain a public code/design repository with a **separate private archive**.

Even a private Git repository is not automatically end-to-end encrypted, and deleting a file does not remove it from prior Git history and copies.

## Current next step

Agree on a minimal record schema and privacy boundary, then test with **synthetic people, relationships and media** before importing any real family material.
