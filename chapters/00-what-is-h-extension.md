# 00. What the H Extension Is

The RISC-V H extension adds architectural support for virtualization.

In plain language, it helps a hypervisor run guest operating systems more cleanly and efficiently, without having to emulate every privileged behavior in an awkward or overly expensive way.

## The problem it solves

Without dedicated virtualization support, a hypervisor has to intercept and emulate too many operations manually.
That increases complexity, slows things down, and makes correctness harder.

The H extension gives the architecture a vocabulary for virtualization:

- guest privilege levels
- virtualized traps and interrupts
- guest-visible control state
- support for two-stage address translation

## Mental model

A useful first approximation is:

- **M-mode** is still the highest machine privilege
- **HS-mode** is where the host hypervisor-oriented supervisor runs
- **VS-mode** is where a guest supervisor kernel runs
- **VU-mode** is where guest user programs run

So the H extension does not just add “one more bit.”
It creates a structured way to distinguish the host supervisor world from the guest supervisor world.

## Why engineers care

If you work on KVM, a hypervisor, low-level kernel code, trap handling, or MMU behavior, the H extension changes how you think about execution flow.

You are no longer asking only:

- did this trap come from S-mode or U-mode?

You also need to ask:

- did it come from the host or a guest?
- is this interrupt virtualized?
- are we translating guest virtual addresses, guest physical addresses, or host physical addresses?

## What makes it feel hard at first

The H extension brings several concepts together at once:

- extra privilege contexts
- extra CSRs
- extra trap-routing rules
- two-stage translation
- virtualization-aware interrupt behavior

Each individual piece is understandable.
The difficulty comes from how they interact.

## Guiding idea for this guide

Do not try to memorize every CSR first.
Start with the system model:

1. host vs guest
2. where traps go
3. how memory translation changes
4. how interrupts are virtualized

Once those are clear, the individual register details make much more sense.
