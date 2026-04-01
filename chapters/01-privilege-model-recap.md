# 01. Privilege Model Recap: M, HS, VS, VU

To understand the H extension, you need a clean mental model of the privilege levels involved.

## Before virtualization

In a simpler RISC-V system, you often think in terms of:

- **M-mode**: machine firmware / highest privilege
- **S-mode**: supervisor kernel
- **U-mode**: user applications

That model is enough when there is only one OS instance directly owning the machine.

## With the H extension

Virtualization splits the supervisor/user worlds into host-side and guest-side contexts.
A practical way to think about it is:

- **M-mode**: machine-level control and lowest-level platform authority
- **HS-mode**: host supervisor mode, where the hypervisor or host kernel virtualization code runs
- **VS-mode**: virtual supervisor mode, where a guest kernel believes it is running as a supervisor
- **VU-mode**: virtual user mode, where guest user programs run

## Why HS and VS both exist

A guest kernel wants to behave like a real kernel.
It wants privileged control over its guest OS world.
But the real machine still belongs to the host and platform.

So the architecture needs to distinguish:

- the **real supervisor context** managing virtualization
- the **guest-visible supervisor context** that should feel privileged from the guest’s point of view

That is the role of HS-mode versus VS-mode.

## Mental model

Think of **HS-mode** as the real controller of virtualization policy.
Think of **VS-mode** as the environment presented to the guest kernel.

The guest kernel can act like a supervisor inside its VM,
but some operations are redirected, virtualized, trapped, or represented differently so the host keeps actual control.

## Why this matters in debugging

When something goes wrong in a virtualized system, one of the first questions is:

- did this event happen in the host context or in the guest context?

That distinction affects:

- which CSRs are relevant
- where traps are delivered
- who is responsible for handling the fault
- whether a register value is host-visible or guest-visible

## Common pitfall

Do not read VS-mode as “just S-mode with a different name.”
It is a guest-facing privilege context embedded inside a larger host-controlled virtualization model.

That difference is exactly why the extra rules and CSRs exist.
