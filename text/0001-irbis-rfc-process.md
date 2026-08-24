---
rfc: 0001
title: The Irbis RFC process
type: process
authors:
  - anfragment
status: draft
discussion: https://github.com/irbis-sh/rfcs/pull/1
start-date: 2026-08-24
---

# The Irbis RFC process

## Summary

This RFC introduces the Request for Comments (RFC) process for Irbis - a consistent, public path for proposing, debating, and deciding on substantial changes to our projects and to the organisation itself.

## Motivation

We have so far made design decisions informally and mostly in private. That has worked, but it scales poorly in some ways:

1. Contributors lack a way to propose substantial changes. Without a design-first process, the only way to propose a significant change is to submit an informal discussion or issue, then build it and submit a PR. This risks weeks of work on something that could get declined for reasons that a design discussion would have surfaced. An RFC process lets ideas be evaluated before implementation.
2. Maintainers lack a way to discuss the design of substantial changes with the contributor community. Certain major or technically complex changes introduced by the maintainers would benefit from reaching a wider group of people, especially with certain specific expertise (e.g., systems programming, cybersecurity), depending on the change.
3. Decisions have no durable record. Why do projects work the way they do? The answer lives in commit messages, scattered issues, and the maintainers' heads. An RFC repository is a permanent and searchable record of what was decided and why.
4. Debates have no structured home. Some questions ("should we do this at all?") deserve serious discussion but are not yet proposals. Without a home for them, they either sit as half-committed roadmap items or spread across chat threads.
5. Design rigour benefits from collaboration. Writing a design down for public review and discussing it forces more precision than private notes.

In the spirit of the original IETF Request for Comments series, RFCs here are encouraged to be timely rather than polished. An RFC does not need to be authoritative to be worth writing - the point is to get ideas into a form where they can be openly discussed.

## Impact

This RFC affects contributors and maintainers. It changes nothing for users of Irbis projects.

**For contributors**: substantial changes now start with an RFC rather than an in-repo issue/discussion. This is additional up-front work in exchange for a guarantee: an accepted RFC means the design is agreed on and a well-executed implementation of it will not be rejected on design grounds. Reviewers engage with your design before you spend effort on an implementation.

**For maintainers**: the process adds review obligations (reading RFCs, driving final comment periods, writing decision rationales) in exchange for better-aligned designs.

## Design

### When an RFC is required

The vast majority of changes do not need an RFC and should go through the ordinary issue and PR workflow. An RFC is required for **substantial** changes:

- Adding, changing, or removing a significant user-facing capability of an Irbis project.
- Changing a project's security or privacy model.
- Introducing a new major subsystem or changing the architecture of an existing one.
- Adding constraints on future development: decisions that are expensive to revise once made, such as file formats, protocols, and public APIs.
- Changing organisation-wide policy or process, including this process itself.
- Anything else where public collaboration and scrutiny are beneficial to the final result.

An RFC is explicitly *not* required for: bug fixes, refactors, performance and quality improvements, documentation, dependency updates, and any other change where a wrong design is easy to revert. If unsure, open a ticket and ask.

RFCs are not feature requests - these belong in the project repository's Discussions.

### RFC types

Every RFC declares one of three types in its frontmatter:

- **Feature** proposes a concrete change to a product. Uses the full [template](../0000-template.md).
- **Process** proposes a change to how the organisation works: this process, contribution policy, release practices. Uses the full [template](../0000-template.md).
- **Discussion** poses a question: *should we do this at all?* A Discussion RFC might never produce code, and that is fine - a well-argued decision *not* to do something is just as valuable, and gives something to link to the next time the question comes up. Discussion RFCs may omit the [Impact](../0000-template.md#impact), [Design](../0000-template.md#design), and [Future possibilities](../0000-template.md#future-possibilities) sections of the [template](../0000-template.md). The remaining sections are required.

  When a Discussion RFC gets resolved towards action, the follow-up design is written as a fresh Feature RFC that supersedes it.

### Scope and the `project` field

This repository serves the whole Irbis organisation. Each RFC declares which project it applies to via the `project:` frontmatter field. RFCs that apply organisation-wide omit the field. RFC numbers form a single series across all projects.

### Roles

- **Authors** write the RFC and drive its discussion. An RFC may have any number of authors. The first listed is the **champion**, responsible for responding to review, revising the document, and moving the RFC forward. Co-authors may take over champion duties by reordering the author list, disputes are resolved by the maintainers. Substantial contributors from the discussion may be added as co-authors by the champion, or credited in an acknowledgements line.
- **Reviewers** are anyone who comments on the RFC. Feedback must be constructive: engage with the design, and write the kind of comment you would want to receive on your own work.
- **Maintainers** (currently [@anfragment](https://github.com/anfragment)) hold final decision authority over all RFCs. The process exists to make design open, to respect contributors' effort, and to record why decisions are made, but not to put decisions to a vote. The maintainers may co-author RFCs and retain decision authority over them given every decision comes with a written rationale in public.

### The RFC lifecycle

An RFC is in one of six states, recorded in its `status:` frontmatter field. Only the maintainers decide an RFC's status.

- `draft` - the RFC exists as an open pull request and is under discussion. Every RFC starts here.
- `active` - the RFC has been accepted and merged. A tracking issue is opened in the relevant project repository and linked in the RFC's `tracking:` frontmatter field.
- `shipped` - the design has shipped in a release.
- `closed` - evaluated and declined.
- `postponed` - a reasonable idea whose time has not yet come. May be picked up later when circumstances change.
- `superseded` - replaced by a later RFC, with a pointer to it in the frontmatter. This is the normal fate of a Discussion RFC that graduates into a Feature RFC, or of a design later revised by another proposal.

### How the process works

1. **Socialise (optional but highly encouraged)**. Before writing, raise the idea informally - in a GitHub Discussion in the project repository or in this one, or in a community forum. Five minutes of "has this been considered?" can save a week of writing. May be skipped by experienced contributors.
2. **Draft**. Fork this repository, copy `0000-template.md` to `text/0000-my-proposal.md`, and fill it in. Keep `0000` as the number placeholder. The template will guide you through the writing process.
3. **Submit**. Open a pull request. The PR link goes in the RFC's `discussion:` frontmatter field.
4. **Discuss and iterate**. The champion drives the discussion, iterating on the RFC in response to feedback. Substantive discussion that happens elsewhere must be summarised back into the PR. Authors may withdraw an RFC at any time by closing the PR. Withdrawn RFCs may be resurrected later by anyone, if possible in consultation with the original authors.
5. **Final comment period**. When discussion has converged - or stalled without prospect of new arguments - the maintainers announce a final comment period (FCP) in the PR, together with the intended decision: **accept, close, or postpone**. The FCP lasts **ten calendar days** and is a last call for objections. No rewrites of the RFC should happen during this period. If a maintainer is the RFC's champion, they may propose the FCP themselves. New arguments raised during FCP can cancel it and return the RFC to discussion, otherwise day ten ends with the decision.
6. **Decision**. Whatever the decision, the maintainers assign the next RFC number and the champion renames the file, sets the frontmatter `rfc:` field to that number, and updates `status:` to the decision. For closed and postponed RFCs a short Decision note at the top of the document summarising the rationale should be added. The maintainers then merge the PR. For accepted RFCs, the maintainers open a tracking issue in the project repository and link it from the `tracking:` field. If the champion is unresponsive at decision time, the maintainers make these edits themselves.
7. **After acceptance**. Implementation proceeds under the ordinary PR workflow in the project repository. Minor corrections and clarifications to a merged RFC go through ordinary PRs against this repository. Design changes require a new RFC that supersedes the old one. When the design ships in a release, the status moves to `shipped`.

### Participation

Anyone is welcome to propose an RFC and discuss and review ideas, with a few notes:

- Any contribution is expected to follow the org-wide [Code of Conduct](https://docs.irbis.sh/code-of-conduct), including the AI Policy.
- Reviewers are expected to be constructive and empathetic in the way they engage in the discussion.

### Changing this process

This process is recursively governed by itself, so any changes to the RFC process should be proposed as Process RFCs.

## Drawbacks

- **Process overhead**. Writing a good RFC takes effort, and for a project of our size this could exceed the benefit. The carve-outs in [When an RFC is required](#when-an-rfc-is-required) and the relatively lightweight lifecycle are designed to keep the process proportionate, but the risk is real.
- **A single point of failure**. The process depends entirely on the maintainers' bandwidth for review and decisions.
- **A public record is a double-edged sword**. Recording decisions makes them more visible but also more expensive to revisit since superseding an RFC is more work than rewriting the code.

## Rationale and alternatives

### Why a dedicated repository, rather than GitHub Discussions or in-repo docs?

A separate repository:
- Gives RFCs their own index,
- Keeps design debates out of the product issue trackers,
- Sets up proper unified infrastructure: template file, index, etc.,
- Allows for Feature designs that cut across projects, and Process designs that affect the entire org.

### Why a fixed ten-day FCP, rather than open-ended review or a full review schedule?

Open-ended review is how RFCs stay in draft for years. A full review schedule, on the other hand, requires an active roster of reviewers we do not currently have. A short, maintainer-triggered FCP is the minimum machinery that prevents stalls: it gives stakeholders a clear last call and gives the process a definitive end. Ten days is a good enough balance between more latency than a project this size needs, and too little time for contributors who check in only occasionally.

### The impact of not doing this

The status quo: substantial changes land without recorded design, and maintainers have no structured way of engaging with the community on design questions.

## Prior art

This process is inspired by and partly assembled from these examples:

- Rust's RFC process ([`rust-lang/rfcs`](https://github.com/rust-lang/rfcs/blob/master/text/0002-rfc-process.md))
- Python's [PEP 1](https://peps.python.org/pep-0001/)
- Fuchsia's [RFC-0001](https://fuchsia.dev/fuchsia-src/contribute/governance/rfcs/0001_rfc_process)
- Oxide's [RFD 1](https://rfd.shared.oxide.computer/rfd/0001#_rfd_life_cycle)

## Security & privacy considerations

The process itself introduces no attack surface and handles no user data. Two considerations apply:

- Public design discussion of security features is deliberate. Publishing threat models and designs before implementation invites scrutiny early on. The one exception is suspected vulnerabilities in shipped code - these are not RFC material and must follow the responsible disclosure process in the affected repository's security policy, not a public PR.
- The template makes security analysis mandatory. Irbis projects sit in a sensitive position on users' machines and in their network traffic, so every RFC, of any type (deliberately including Process), must contain a Security & privacy considerations section even when its content is an explicit "no implications".

## Unresolved questions

- Whether the ten-day FCP and the socialisation step are calibrated correctly for the current community's shape. Should be resolved by this RFC's own discussion and some follow-up ones.
- Whether additional process machinery earns its place. Each would arrive by Process RFC when a concrete need appears. Will be resolved through continued operation of the process.

## Future possibilities

- A rendered RFC index on docs.irbis.sh, generated from the frontmatter this process standardises.
- Programme metrics to monitor as the process goes on: RFCs sitting more than thirty days without maintainer response, zero community-authored RFCs after six months, or substantial PRs routinely arriving without RFCs would each indicate a calibration failure worth a Process RFC of its own.
