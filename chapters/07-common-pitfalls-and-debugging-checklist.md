# 07. Common Pitfalls and Debugging Checklist

By this point, the H extension should feel less like a random CSR explosion
and more like a structured virtualization model.

This final chapter is about practical failure modes and a debugging posture that helps in real systems.

## The biggest recurring pitfall

The most common mistake is to mix up **host state** and **guest state**.

When something breaks, engineers often ask only:

- what did the guest do?

But under the H extension, that is not enough.
You also need to ask:

- what virtualization context was active?
- what did the host think the guest state was?
- what trap, interrupt, or translation path crossed the host/guest boundary?

A lot of confusion comes from collapsing two worlds into one mental model.

## Pitfall 1: treating guest physical as truly physical

Guest physical addresses are not final machine addresses.
They are only "physical" from the guest's point of view.

Debugging consequence:

- if memory access fails, do not stop after checking the guest page tables
- also check the guest-physical to host-physical stage

## Pitfall 2: assuming every trap means guest misbehavior

Many traps under virtualization are not bugs in guest logic.
They are expected mediation points.

Examples include:

- operations the hypervisor must virtualize
- faults the host must decode and reflect
- interrupt or timer events that require host-side handling before guest injection

Debugging consequence:

- first classify the trap
- only then decide whether it represents guest error, host policy, or normal virtualization flow

## Pitfall 3: looking only at classic supervisor CSRs

If you debug a virtualized system using only the non-virtualized S-mode mental model,
you can miss half the story.

Debugging consequence:

- inspect virtualization-specific CSRs too
- check both guest-visible and host-visible state
- verify the currently active translation and interrupt context

## Pitfall 4: losing track of interrupt ownership

Interrupts are easy to misunderstand in a guest system.
There is a difference between:

- a real interrupt source existing on the host
- the host deciding the guest should observe a virtual interrupt
- the guest actually receiving it in the expected state

Debugging consequence:

- ask where delivery failed: source, mediation, injection, or guest handling

## Pitfall 5: debugging translation failures too late in the flow

By the time a guest crashes, the original memory mistake may be far upstream.
A wrong mapping, stale state, or bad invalidation can surface later in confusing ways.

Debugging consequence:

- validate translation assumptions early
- treat two-stage translation and remote synchronization as first-class suspects

## A practical debugging checklist

When a guest or hypervisor path looks wrong, walk through this checklist:

### Execution context

- were we in host context or guest context?
- was the active world HS, VS, or VU related?
- what entry/exit path just ran?

### Trap / exception path

- what was the reported trap or exception?
- did the host handle it directly, emulate it, or reflect it?
- was the trap expected as part of virtualization?

### Memory translation

- what guest virtual address was involved?
- what guest physical address did it map to?
- what host physical mapping backed it?
- did stage-1, stage-2, or permission checks fail?

### Interrupt path

- was there a real interrupt source?
- should the guest have seen a virtual interrupt?
- was the relevant interrupt pending / enabled / injected?
- did the guest receive it in the right state?

### State ownership

- is the relevant register or state host-owned, guest-visible, or both?
- are you reading the host's view, the guest's view, or a mediated representation?

## Code-reading checklist for KVM-style implementations

When reading KVM-on-RISC-V or similar hypervisor code, label each path as one of:

- guest entry
- guest exit
- trap decode / exit reason handling
- vCPU state save/restore
- interrupt injection
- guest memory management
- remote synchronization / invalidation

This keeps the architecture concepts grounded in implementation work.

## Final takeaway

The H extension is hard mainly because it joins multiple moving parts:

- privilege context
- trap routing
- translation layering
- interrupt virtualization
- duplicated or mediated control state

If you debug it one layer at a time,
using a strict host-versus-guest mental model,
it becomes much easier to reason about.
