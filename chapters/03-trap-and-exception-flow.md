# 03. Trap and Exception Flow with Virtualization Enabled

Once virtualization is enabled, trap handling becomes more subtle.

In a non-virtualized mental model, you often ask:

- did execution fault in user mode or supervisor mode?
- which privileged handler should receive the trap?

With the H extension, you also need to ask:

- did the event come from the host or from a guest?
- should this trap be reflected to the guest-visible context or handled by the host hypervisor?

## Why this matters

A guest OS should feel like it is running on a real machine.
That means many events need to look normal from the guest kernel's point of view.

But the host still owns the real machine and must preserve isolation and control.
So trap routing is no longer just about privilege level.
It is also about virtualization context.

## Mental model

A useful way to think about traps under virtualization is:

- some events are genuinely for the **host** to handle
- some events should be presented to the **guest** as if they happened on its own machine
- some events require the host to inspect, translate, or emulate behavior before guest execution can continue

So trap handling becomes a policy boundary as much as an exception boundary.

## Typical questions during debugging

When a virtualized workload breaks, the first debugging questions are often:

- what instruction was the guest executing?
- was the failure during normal execution, page translation, or interrupt delivery?
- did the event trap to the host because the guest truly did something wrong,
  or because the hypervisor must virtualize the operation?

Those are different situations, even if they all show up as "a trap happened."

## Why this feels hard at first

Virtualization introduces more than one privileged world.
So the old intuition of "trap goes to the kernel" is no longer enough.
You need to know:

- which kernel-like context is active
- whether the current state is guest-visible or host-visible
- whether the architecture expects direct handling, reflection, or emulation

## Common pitfall

Do not collapse all virtualization-related traps into "guest bugs."
Often the guest did something completely expected,
and the trap exists because the hypervisor needs to mediate or virtualize the behavior.

That distinction is essential when reading hypervisor code paths such as KVM on RISC-V.
