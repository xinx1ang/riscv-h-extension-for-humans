# 04. Virtual Interrupts and Delegation Mental Model

Interrupt handling gets more complicated once guests enter the picture.

Without virtualization, the question is often just:

- which privilege level should receive this interrupt?

With the H extension, the question becomes:

- is this interrupt for the host or for a guest?
- should the host handle it directly, or present a virtual interrupt to the guest?
- which events are delegated, intercepted, or injected?

## Why this exists

A guest OS expects interrupts to behave like they would on a real machine.
It expects timer interrupts, software interrupts, and external interrupts to arrive in ways that make sense to the guest kernel.

But the host still owns the actual hardware interrupt sources and real platform state.
So the architecture needs a clean way to separate:

- **real interrupt ownership**
- **guest-visible interrupt delivery**

That is why virtualization needs both delegation and virtual interrupt machinery.

## The plain-language mental model

Think of the host hypervisor as the building manager and the guest as a tenant.
The building manager owns the real wiring and alarm system.
But the tenant still expects a working doorbell and smoke alarm inside the apartment.

Likewise:

- the host owns real interrupt routing policy
- the guest sees a virtualized interrupt world that should feel consistent and usable

The guest should not need direct ownership of the whole machine just to receive "its" timer or software interrupt.

## Three useful buckets

A simple engineering mental model is to put interrupt-related events into three buckets:

### 1. Host-owned events

These are interrupts or conditions the host must deal with itself.
They may concern real hardware management, host scheduling, or platform control.

The guest should not directly own these.

### 2. Guest-visible events

These are events that the guest should experience as interrupts within its own virtual machine context.
From the guest kernel's point of view, these should look like normal kernel-facing interrupts.

### 3. Mediated events

These are cases where the host observes a real event first,
then decides whether and how to inject or reflect a corresponding virtual interrupt into the guest.

This mediation is where a lot of hypervisor logic lives.

## Why delegation matters

Delegation is about deciding which layer should handle which class of events.

Without delegation rules, every interesting event would bounce to the highest control layer,
which would make virtualization expensive and messy.

With a clean delegation model, the architecture can support a more direct and structured execution path.
That reduces needless exits while preserving host control.

## What engineers should ask while debugging

When an interrupt path looks wrong, ask:

- was there a real interrupt but no corresponding virtual interrupt injection?
- was the event handled by the host when the guest expected to see it?
- was the event injected into the guest at the wrong time or in the wrong state?
- did delegation settings route the event differently than expected?

Those questions are usually more useful than asking only, "did an interrupt happen?"

## Common pitfall

Do not think "virtual interrupt" means fake and therefore unimportant.
A virtual interrupt is how the guest experiences real system progress.
If virtual interrupt delivery is wrong, the guest can lose scheduling ticks, miss wakeups, or behave as if devices are broken.

## Connection to KVM on RISC-V

When you read KVM code, a lot of the interrupt-related complexity is really about this boundary:

- receiving or tracking host-side state
- deciding guest-visible consequences
- injecting the right virtual interrupt at the right time

That is exactly the kind of machinery the H extension is meant to support more cleanly.
