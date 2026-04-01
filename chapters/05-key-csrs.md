# 05. Key CSRs Introduced by the H Extension

The H extension introduces or redefines several control and status registers that matter for virtualization.

You do not need to memorize all of them at once.
What matters first is understanding what *kind* of problem each CSR helps solve.

## The right way to learn these CSRs

Do not learn them as an isolated register list.
Instead, group them by purpose:

- **state tracking**: what virtualization context are we in?
- **trap reporting**: what fault happened, and for whom?
- **address translation**: how do guest addresses get translated?
- **interrupt virtualization**: what guest-visible interrupt state exists?

That mental grouping makes the register set much easier to digest.

## Important categories

### Virtual supervisor state

Some CSRs exist so the architecture can represent a guest-visible supervisor world.
That includes guest-facing versions of state that a guest kernel expects to control or observe.

The point is not merely duplication.
The point is to preserve the illusion that the guest has a coherent supervisor environment,
while the host still retains actual authority.

### Hypervisor control state

Some CSRs are primarily for the host hypervisor.
These control virtualization behavior, trap interpretation, interrupt state, and translation policy.

These are the registers you often end up inspecting when the host and guest disagree about what should have happened.

### Guest address translation state

Two-stage translation needs control state for guest-related page-table interpretation and translation behavior.
If you are debugging guest memory faults,
this is one of the first CSR categories to keep in mind.

### Trap and fault context

Virtualization introduces more nuanced fault reporting.
The host often needs more context than a non-virtualized supervisor would need,
especially when faults relate to guest instruction fetch, guest memory access, or virtualized execution state.

## A practical short list

Engineers reading H-extension code paths often end up repeatedly seeing registers such as:

- **hstatus**: major hypervisor status/control context
- **hedeleg / hideleg**: exception/interrupt delegation choices relevant to virtualization flow
- **hvip / hip / hie**: hypervisor-visible interrupt-pending / enable style state
- **vsstatus / vsie / vstvec / vsscratch / vsepc / vscause / vstval / vsip / vsatp**: guest-visible supervisor state
- **hgatp**: guest-physical to host-physical translation control
- **htval / htinst**: extra trap-reporting context useful for virtualization-related faults

You do not need the exact bitfields memorized on day one.
But you should know the job each register family is doing.

## Why these matter in practice

When virtualization breaks, you often need to answer questions like:

- what mode/context was active?
- what fault was reported?
- did the host or guest own the relevant state?
- what translation structure was active?
- what interrupt looked pending or enabled from each side?

Those answers usually live in CSR state.

## Common pitfall

A frequent beginner mistake is to look only at traditional supervisor CSRs and ignore the virtualization-specific ones.
That often leads to an incomplete picture.
Under the H extension, the guest-visible state and host-visible virtualization state are both part of the story.

## Suggested reading posture

When you see a new H-extension CSR, ask four questions:

1. is this host-visible, guest-visible, or both?
2. is it about traps, interrupts, or translation?
3. does it describe real machine state, virtual state, or mediation between them?
4. when would a hypervisor engineer inspect or write it?

That approach is much more useful than trying to memorize names in alphabetical order.
