# RISC-V H Extension for Humans

A human-friendly, engineer-oriented guide to the RISC-V Hypervisor (H) extension.

Goal: explain the H extension in plain language, with enough system context to help engineers understand what it changes for virtualization, trap handling, interrupts, memory translation, and guest execution.

## Why this exists

The H extension is powerful, but it can feel dense when read directly from the privileged architecture specification.
This repository aims to turn the spec into an engineer-readable guide focused on mental models, execution flow, and real implementation concerns.

## Who this is for

- people learning RISC-V virtualization
- engineers reading KVM-on-RISC-V code paths
- hypervisor / kernel / firmware developers
- anyone who wants a practical explanation instead of only raw spec language

## Planned structure

- 00. What the H extension is and what problem it solves
- 01. Privilege model recap: M, HS, VS, VU
- 02. Two-stage translation in plain language
- 03. Trap and exception flow with virtualization enabled
- 04. Virtual interrupts and delegation mental model
- 05. Key CSRs introduced by the H extension
- 06. How this maps onto KVM on RISC-V
- 07. Common pitfalls and debugging checklist

## Style

Each chapter should try to cover:

- what the spec says
- what it means in plain language
- why the mechanism exists
- who uses it
- how it shows up in real systems
- what commonly goes wrong

## Status

Work in progress.
