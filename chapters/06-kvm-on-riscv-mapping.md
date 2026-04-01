# 06. How This Maps onto KVM on RISC-V

The H extension becomes much easier to understand once you connect it to a real hypervisor design.
KVM on RISC-V is a good anchor for that.

This chapter is intentionally conceptual.
It is about mapping architecture ideas to implementation responsibilities.

## The high-level picture

At a very high level, KVM on RISC-V is trying to do four recurring jobs:

1. **enter guest execution**
2. **leave guest execution when something needs host attention**
3. **manage guest-visible CPU and memory state**
4. **inject or reflect events back into the guest**

The H extension exists to make these jobs more direct and architectural.

## Mapping the concepts

### 1. Privilege model -> host versus guest execution contexts

The architectural split between HS and VS maps naturally onto KVM's need to distinguish:

- host kernel/hypervisor control paths
- guest kernel-visible execution state

That means KVM is not inventing the idea of guest supervisor context out of thin air.
It is using the architecture's virtualization model.

### 2. Two-stage translation -> guest memory management

KVM needs to let the guest kernel believe it controls memory,
while still ensuring the host controls real machine mappings.

That maps directly onto the two-stage translation mental model:

- guest manages its own page-table world
- host controls the final machine backing

In implementation terms, memory virtualization is not just allocating RAM to a VM.
It is also controlling translation and fault behavior across the guest/host boundary.

### 3. Trap flow -> VM exits and host handling

When the guest does something that needs host mediation,
execution leaves the guest context and returns to host handling logic.

That may happen because of:

- privileged guest operations that must be virtualized
- faults during translation or access
- interrupt and timer related events
- emulation needs

So many KVM code paths are really concrete instances of the trap-flow model described earlier.

### 4. Virtual interrupts -> event injection logic

A hypervisor must decide when a guest should observe a timer, software interrupt, or external event.
That is the implementation face of virtual interrupt support.

In other words, guest interrupt injection logic in KVM is where the abstract architecture turns into scheduler and event-delivery code.

## A practical debugging lens for KVM

When reading or debugging KVM on RISC-V, it helps to classify every path into one of these questions:

- are we entering the guest or returning from it?
- are we saving/restoring guest-visible CPU state?
- are we handling a trap that belongs to the host?
- are we translating or validating guest memory access?
- are we injecting a guest-visible interrupt or exception?

If you keep that frame in mind, the code becomes much less mysterious.

## Why the H extension matters here

Without architectural virtualization support,
a lot of this logic would require heavier software emulation and more awkward trap handling.

With the H extension, KVM gets clearer hardware-level structure for:

- guest privilege contexts
- translation layering
- trap reporting
- virtual interrupt state
- guest-visible supervisor state

That does not remove implementation complexity,
but it makes the complexity more principled.

## Common pitfall

Do not assume the H extension "is KVM."
It is not.
KVM is still a software implementation with policy, lifecycle, scheduling, and memory-management responsibilities.

The H extension is the architectural support layer that makes those responsibilities feasible and cleaner.

## What to read next in real code

After building the mental model, useful next code-reading targets are typically:

- guest entry/exit paths
- vCPU state save/restore logic
- trap handling and exit-reason dispatch
- interrupt injection paths
- guest memory / page-fault handling paths

Those are the places where H-extension concepts become visible as real engineering machinery.
