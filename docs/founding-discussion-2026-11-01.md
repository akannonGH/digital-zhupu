# Digital 族譜 (Zúpǔ) — Founding discussion and architecture notes

**Recorded:** 2026-11-01  
**Status:** Exploration / design notes, not an implemented specification  
**Repository:** https://github.com/akannonGH/digital-zhupu

> This is a dated record of the ideas discussed when the project began. It distinguishes requirements expressed by the founder from possible implementation choices. No real genealogical records have been added yet.

## 1. Why this project exists

The starting question was how to reconstruct and preserve a lost Hakka 族譜 (family genealogy), using AI where helpful. The desired result is **more than a conventional family tree**:

- A genealogy of ancestors, descendants, marriages and branches.
- A family contacts directory for living relatives.
- An interactive diagram of relationships.
- A historical archive of photographs, documents, letters, audio/video and remembered accounts.
- A knowledge system where historical evidence and uncertainty can be traced.
- A long-lived, portable archive that could eventually be maintained by different family branches.

The original analogy was blockchain-like: multiple copies, independent contributors and eventual reconciliation. The preference was **a Git repository, not an actual blockchain or a public-facing genealogy service**.

### Founder requirements stated in the conversation

1. Start with one maintainer.
2. Let family members update their own contact details somehow, without expecting everyone to use Git.
3. Allow independently maintained copies as family branches diverge.
4. Be able to reconcile those copies later.
5. Support non-text source material, not just structured records.
6. Keep family information private; use Git as infrastructure, not as a public directory.

A useful additional design idea discussed: support the traditional Hakka/patrilineal 族譜 view, but do **not** limit the underlying family graph to paternal ancestry. Maternal and married-in families should also be representable.

## 2. The main architectural idea

**Separate the durable records, the original media and the interpretations of evidence.** Interfaces, indexes and AI models sit on top and can be replaced.

| Layer | Purpose | Proposed implementation |
| --- | --- | --- |
| Genealogical records | People, kinship links, events, branches, places | Versioned open text files (YAML or JSON) in Git |
| Evidence catalogue | Sources, claims, citations, provenance, uncertainty | Structured text records in Git |
| Media archive | Scans, photos, recordings, documents, video | Original files managed outside normal Git blobs; candidate: git-annex |
| Private contacts | Current email, phone, address, contact preferences | Separate, restricted and encrypted data store; NOT in the shared history repo |
| Search and graph | Browse relationships and full-text media derivatives | Rebuildable index (e.g. SQLite); optional graph/search UI |
| AI workflows | OCR, transcription, translation, extraction, matching | Proposals and derived annotations, with human review |
| Family interface | Tree, profiles, archive browsing, contact self-service | Future website/app; people need not know Git |
| Interchange | Exchange with other tools and relatives | GEDCOM 7 for genealogy, carefully scoped vCard for contacts, open media formats |

Git records changes and supports branch reconciliation, but **Git alone does not resolve conflicting genealogical claims**. The application needs domain-level merging.

### Why not blockchain?

A blockchain is not needed for the initial goals. Git already offers distributed copies, branches, revisions and change history. Blockchain would add governance and privacy complexity without an identified benefit. Cryptographic hashes are useful for preserving file integrity **without** a blockchain.

## 3. Suggested data model (not final)

Primary record types:

- **Person:** stable opaque ID, names and aliases, relevant dates, narrative notes.
- **Relationship:** explicit typed connections (parent/child, spouse, adoptive, etc.), with validity and evidence where needed.
- **Family branch:** a lineage or custodian group; can overlap or be represented differently by different families.
- **Event:** birth, marriage, migration, death, reunions and other historical events.
- **Place:** villages, ancestral origins, historical and present-day names, countries and coordinates where supportable.
- **Source:** a physical or conceptual original (photograph, family register, oral interview, letter, document).
- **Digital asset:** a particular digital file or scan of a source; original and derivative files are distinct.
- **Claim/assertion:** a statement (e.g. a date, identity or relationship) attributed to one or more sources.
- **Annotation:** a transcription, translation, identification, correction or contextual note.
- **Contribution/review:** who submitted or reviewed information, when, and on what basis.

**IDs must not be derived from names or family position.** For truly independent branches, collision-resistant IDs such as UUIDs or ULIDs are preferable to local sequential numbers alone. When two branches create separate IDs for the same historical person, preserve both until identity reconciliation is reviewed.

### Hakka-specific information worth accommodating

Optional fields may include Chinese-character names, Hakka romanisation/pronunciation, Cantonese/Mandarin/Burmese/English names and aliases, ancestral village (祖籍), generation-character naming conventions (字輩), ancestral-hall identity (堂號), branch (房支), and migration history. These are **research possibilities, not assumed facts about this family**. Missing values must remain missing.

### Evidence is not certainty

Preserve distinctions among:

- A document's content.
- Someone's recollection of an event.
- A proposed interpretation of either.
- A fact independently corroborated by other evidence.
- An unknown or disputed detail.

Do not silently turn approximate dates into exact dates, resolve contradictions by replacing one claim, or infer people/relationships without a documented basis.

**Example:** One branch recalls an ancestor born in 1895; another has a document suggesting 1897. Store both claims with citations and review status. A visual view may show "birth year disputed" rather than selecting a winner.

## 4. Non-text records: archival media and evidence

One original photograph might contain many people, show a wedding, support a date and depict a place. It should exist **once** as an archival source, with links from multiple people/events/claims.

Distinguish:

1. **Source:** e.g. one physical wedding photograph with a handwritten reverse.
2. **Assets:** original front scan, original reverse scan, possibly a later enhanced copy.
3. **Derivatives:** thumbnail, OCR/transcription, colour restoration, translated caption.
4. **Assertions:** which people are depicted, whether the event is a wedding, the likely date, etc.
5. **Provenance:** who supplied or digitised it, when, where the physical original resides, restrictions, and confidence.

Each file should have integrity checks (e.g. SHA-256), appropriate metadata and durable references. Content hashes identify **byte-identical files**, not different scans of the same physical photograph. Maintain a source-level relationship across distinct files.

Store originals unchanged. Corrections, enhanced versions, translations and AI-generated text should be new, linked derivatives. Include transcription language, model/tool information when relevant, review status and attribution.

Media could include TIFF/JPEG/PNG photographs, scans/PDF, audio and video. Preserve the uploaded original format; produce accessible/searchable derivatives without replacing it.

### Binary storage candidates

| Option | Strength | Trade-off |
| --- | --- | --- |
| Normal Git | Simplest for a tiny pilot | Repository grows unwieldy with large/changing binary files |
| Git LFS | Familiar Git workflow, pointers to separately stored binaries | Usually relies on a shared LFS service/remote; storage and bandwidth limits matter |
| **git-annex** | Distributed storage, selective retrieval, tracking which copies hold content | More operational complexity and a learning curve |

**Preliminary preference:** explore git-annex for the long-term federated archive, because branches may wish to keep different subsets of media on NAS devices, local drives or independent servers. **Not yet selected.** A small pilot might use ordinary files or Git LFS instead.

Critical limitation: synchronising metadata does **not** guarantee a branch has downloaded the corresponding media. Availability and backup status need to be managed and made visible.

## 5. Federation, independent family branches and reconciliation

The likely evolution:

1. One original repository is stewarded by the founder.
2. Other family members contribute data through a simple interface or a reviewed proposal.
3. Branches may keep their own full copies or independent repositories.
4. They exchange selected genealogical records and media metadata.
5. Human reviewers reconcile duplicate people, contradictory claims and changing relationships.

Git can handle routine text changes, but a domain-aware merge process needs to address:

- Two records referring to the same person under different IDs or names.
- Two source-backed claims that disagree.
- Changes to a relationship that cannot be resolved by line-by-line merge.
- Media present on some remote stores but absent on others.
- Permission boundaries: a relative may share lineage details but not current contacts or private photos.
- Schema version differences as independently maintained archives evolve.

Do not let the first repository become the mandatory central authority. Branch custodians should be able to preserve their own accounts and to decline a proposed reconciliation.

**Selection of a master/federation protocol is an open question.** Independent Git remotes plus explicit import/export may be sufficient at first.

## 6. Living contacts and privacy

The directory of living relatives is useful but should be architecturally separate from historical genealogy.

A future user flow: a relative opens an authenticated private link, verifies their identity, edits their own contact details and selects who may see them. The application updates the restricted contact store. Changes to historical genealogical claims can enter a review queue.

Avoid committing phone numbers, home addresses, private emails, minors' details, authentication secrets or sensitive media references to a repository that others may clone. **Removing a file later does not erase it from existing Git history or copies.**

GitHub repository visibility is **public as of this note's creation (2026-11-01)**. This repository must contain **conceptual design documentation and non-sensitive examples only**, unless visibility and an explicit privacy model are addressed. Even a GitHub *private* repository is access-controlled, not automatically end-to-end encrypted.

Privacy boundaries also apply to:

- Images/videos with identifiable living people.
- EXIF/location data embedded in photos.
- Restricted family stories and annotations.
- Search indexes, logs, thumbnails, exported genealogies and AI processing.
- Selective sharing with branches that have different permissions.

Prefer explicit consent, least-privilege access and separate encrypted storage for restricted materials. Reconsider whether third-party AI/cloud services should receive family documents; local processing may be preferable for sensitive content.

## 7. AI: a research assistant, not a historian of record

Potential workflows:

- Transcribe Hakka or other language oral histories (with a reviewable transcript).
- OCR handwritten/printed documents when feasible.
- Translate names and texts while preserving the original.
- Suggest structured people, events, places and relationships from interviews or photos.
- Link source material to existing records.
- Flag possible duplicate people across repositories.
- Compare inconsistent accounts and identify unresolved questions.
- Search genealogical records conversationally.
- Generate a traditional-style printable 族譜 or a modern interactive family graph.

AI output should be saved as an **attributed proposal or derivative**, not accepted automatically. Record evidence, uncertainty and human approval. Do not rely on automatic face matching or AI guesses as proof of identity.

## 8. Reference implementation sketch

Illustrative **future** structure; folders are not created yet:

~~~text
digital-zhupu/
  README.md
  docs/
    founding-discussion-2026-11-01.md
    ...
  schema/
  records/
    people/
    relationships/
    branches/
    events/
    places/
    sources/
    claims/
    assets/
    annotations/
  media/                   # storage approach to be chosen
    originals/
    derivatives/
  tools/                   # import, validation, export, indexes
~~~

**Private contact details should not be added inside this shared tree**; give them a separately administered store with its own access policy.

A generated SQLite index or graph projection can power search and visualisation. The index is **not the source of truth** and should be reproducible from records and permitted media-derived text. Graphviz or Cytoscape.js are possible viewers. Gramps offers a useful genealogy data-model reference. GEDCOM 7 export helps with interoperability, though a custom evidence model may contain information that GEDCOM cannot fully express; exports should be treated as projections.

## 9. Longevity and preservation

The archive should survive the disappearance of its original website, cloud vendor, AI model or custodian.

Priorities:

- Open, documented schema and stable identifiers.
- Human-readable structured records and ordinary media files.
- Preserved originals, with derivatives clearly labelled.
- Checksums and periodic integrity verification.
- Multiple independently controlled backups, including offline/offsite copies where practical.
- Documented import/export and migration procedures.
- Avoid reliance on GitHub as the only copy or on any one AI provider.
- Plan explicitly for continuity when custodians change or a branch stops maintaining its storage.

Replication/synchronisation is **not** a substitute for backup.

## 10. What is decided versus only suggested

### Requirements expressed by the founder

- Start small with the founder as initial custodian.
- Genealogy + contacts + graph + multimedia.
- Private information, not a public family directory.
- Distributed copies with future reconciliation.
- Git as the initial collaboration/versioning model.
- Ability for relatives to update contacts without dealing with Git directly.

### Working architectural recommendations (not final decisions)

- Version open text records in Git; avoid blockchain.
- Use an evidence/claim model with sources and uncertainty.
- Separate original media from the human-readable catalogue.
- Consider git-annex for distributed binary storage.
- Keep contact data in a separate secured system.
- Use a rebuildable local index rather than making a graph database canonical.
- Use AI to suggest changes, with human review.
- Support GEDCOM and portable exports.

### Unanswered questions

1. Should this GitHub repository be private, or be a public code/design repository with all real records in private repositories?
2. YAML vs JSON as the canonical format; what schema and validation rules?
3. Git-annex vs Git LFS vs simple media storage for the first real archive?
4. Which information may be shared between branches, and who decides?
5. How will living relatives authenticate and control their contact privacy?
6. How should contributors establish identity and cite source material?
7. What is the initial scope of the genealogical graph (one Hakka lineage, maternal lines, married-in families, other branches)?
8. What is the minimum viable tree viewer and record editor?
9. How should remote branches discover each other and reconcile independently created identities?
10. Who holds physical originals, digital backups and long-term custody responsibilities?

## 11. A sensible staged approach

**Stage 0 — Design (current):** document goals, architecture, privacy risks and unresolved choices. No real family data.

**Stage 1 — Text-only pilot:** design a small versioned schema, validation and a handful of **synthetic** records; demonstrate relationships and contradictory claims.

**Stage 2 — Media pilot:** attach sample scans/audio with source-asset-claim references; test integrity, derivatives and backup/retrieval.

**Stage 3 — Browser and AI assistance:** build an interactive tree, search, evidence views and reviewable AI-assisted extraction.

**Stage 4 — Family contribution:** authenticated updates, contact controls, review queues and audit trail.

**Stage 5 — Federation:** independent branch replicas, selective media exchange, identity matching and semantic reconciliation.

These are sequencing ideas, not deadlines or a commitment to a software stack.

## 12. Discussion trajectory (summary, not a verbatim transcript)

1. **Lost Hakka 族譜:** Asked how AI could reconstruct a genealogy and store it digitally as both contacts and a relationship diagram; proposed a blockchain-like, private Git-repository concept.
2. **Initial proposal:** A private family knowledge graph with people/relationships/history/media records, human-readable files, provenance, AI lookup and GEDCOM compatibility.
3. **Clarification about shared ownership:** Founder would begin alone; relatives might update their own contacts; descendants and branches would eventually need separate copies and reconciliation.
4. **Federated direction:** Independent family repositories, branch stewardship, stable IDs, selective sharing, conflict-preserving claims and domain-aware merges rather than blind overwrite.
5. **The multimedia question:** Asked how to store documents, photos and accounts when the project extends beyond text.
6. **Archive direction:** Distinguish original sources from digital assets, derivatives, annotations and evidence claims; use Git for records, consider git-annex for large distributed media, and rebuildable search/graph indexes.
7. **Repository created:** The founder created this repository so these ideas can be preserved and revisited.

**Central guiding principle:** preserve the family history and its evidence in a portable, private, distributed form that future generations can understand and maintain—without depending on a specific application, host or AI.
