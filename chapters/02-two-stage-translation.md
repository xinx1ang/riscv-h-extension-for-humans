# 02. Two-Stage Translation in Plain Language

Two-stage translation is one of the core ideas behind the RISC-V H extension.

In plain language, guest memory addresses are not translated only once.
Instead, the system may translate them in two steps before reaching real machine memory.

## The short version

A guest OS thinks in terms of its own virtual memory system.
So from the guest's point of view, a guest virtual address should become a guest physical address.

But the guest does not actually own the real machine.
The hypervisor still controls where that guest physical memory lives in real system memory.

So the full path can look like this:

1. **guest virtual address (GVA)**
2. **guest physical address (GPA)**
3. **host physical address (HPA)**

That is what people mean by **two-stage translation**.

## Why one translation is not enough

Without virtualization, a normal OS maps:

- virtual address -> physical address

Under virtualization, the guest kernel still wants to manage memory for its own processes.
So it still needs its own page tables.

At the same time, the hypervisor must keep final authority over the real machine memory layout.
So it needs a second mapping layer.

That creates two different translation responsibilities:

- the **guest** manages GVA -> GPA
- the **hypervisor/platform** manages GPA -> HPA

## Mental model

Think of the guest as renting an apartment in a building.
Inside the apartment, the guest is free to organize furniture however it wants.
But the building owner still decides where that apartment exists in the building.

Likewise:

- the guest controls its own virtual memory view
- the hypervisor controls where the guest's memory lives on the real machine

## Why engineers care

This matters because virtualization bugs are often really address-translation bugs.
When a guest memory access fails, you need to ask:

- did the guest page-table walk fail?
- did the guest-physical to host-physical mapping fail?
- did permissions fail at stage 1, stage 2, or both?

If you do not keep the stages separate in your head, debugging becomes confusing fast.

## What changes in practice

With the H extension, the architecture gives the hypervisor cleaner support for this layered model.
That means guest execution can happen with virtualization-aware translation behavior rather than relying on ad hoc software emulation for everything.

## Common pitfall

Do not think of guest physical memory as truly physical.
It is only "physical" from the guest's perspective.
The host still decides what real machine memory backs it.

That naming is one of the biggest sources of confusion when first learning virtualization.
