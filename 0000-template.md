---
rfc: 0000
title: (a short, descriptive title)
project: (project this applies to; omit for org-wide RFCs) # zen
type: feature # feature | process | discussion
authors:
  - your
  - GitHub
  - username(s)
status: draft # draft | active | shipped | closed | postponed | superseded (see RFC 0001)
discussion: https://github.com/irbis-sh/rfcs/pull/0000
tracking: # link to the implementation tracking issue, added on acceptance
start-date: YYYY-MM-DD
---

# (title)

## Summary

One paragraph explanation of the proposal.

## Motivation

Any change should solve a problem that users, organisation members, or contributors are actually having. Explain that problem in detail, including any background needed to understand it, and give specific scenarios where the proposal helps. Those scenarios should then guide the design.

This is one of the most important sections of any RFC, and can be lengthy. If the motivation is weak, no amount of design quality will justify the proposal.

## Impact

Explain what changes from the addressed group of people's - maintainers, contributors, or users - point of view, as concretely as possible. For example, for user impact:

- New features, screens, prompts, or settings - include the actual proposed wording where you can.
- Behaviour on upgrade: what happens to existing installations, and whether migration is automatic, prompted, or manual.
- Platform differences (Windows, macOS, Linux - and desktop environment caveats where relevant).
- Changes to performance and resource usage, if any.

## Design

The technical portion of the RFC. Explain the design in enough detail for somebody familiar with the project to understand, and for somebody familiar with the implementation to implement. This should get into specifics, terminology, and edge cases.

Return to the scenarios given in earlier sections, and explain how the design makes them work. Diagrams, data formats, and API sketches are welcome; pseudocode is fine where real code would be premature.

## Drawbacks

Why should we *not* do this?

## Rationale and alternatives

- Why is this design the best in the space of possible designs?
- What other designs were considered, and why were they not chosen?
- What is the impact of not doing this at all?

## Prior art

Discuss prior art, both good and bad, in relation to this proposal:

- Do other ad blockers, privacy tools, proxies, or VPNs do this? What has their experience been?
- Are there relevant standards, papers, or write-ups? Post-mortems of others' failed attempts are especially valuable.
- For process proposals: how do other projects or communities handle this, and what lessons can we take?

If there is no prior art, that is fine - say so. Ideas are welcome whether they are brand new or adapted from elsewhere. Note that precedent alone does not justify an RFC: other projects' choices are evidence, not authority.

## Security & privacy considerations

This section is required for every RFC, including Process and Discussion RFCs.

Address, as applicable:

- What new attack surface does this introduce? What could a malicious actor do with it?
- Does this change how keys, certificates, or intercepted traffic are stored, transmitted, or exposed?
- Does this change what data the project collects, processes, or could be compelled to reveal?
- What is the blast radius if this component is compromised?
- Are there failure modes that silently degrade the user's protection rather than failing loudly?

If the honest answer is "no security or privacy implications", write that.

## Unresolved questions

- What parts of the design do you expect to resolve through the RFC discussion before this is accepted?
- What parts do you expect to resolve during implementation?
- What related questions are out of scope for this RFC but could be addressed independently in the future?

## Future possibilities

Think about the natural extensions and evolution of your proposal, and how it fits the direction of the project as a whole. This is also a good place to park ideas that came up during design but are out of scope.

If you cannot think of any future possibilities, you may simply say so.

Note that content in this section is not a reason to accept the current or a future RFC; arguments for acceptance belong in Motivation and Rationale. This section merely provides additional context.
