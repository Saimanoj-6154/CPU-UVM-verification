# CPU UVM Verification

> A production-grade Universal Verification Methodology (UVM) environment built to exhaustively verify a RISC-V RV32I processor core — targeting corner-case bugs through constrained-random stimulus, functional coverage, and assertion-based checking across the full instruction space.

---

## Table of Contents

- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [UVM Environment Architecture](#uvm-environment-architecture)
  - [Testbench Hierarchy](#testbench-hierarchy)
  - [Component Breakdown](#component-breakdown)
- [Verification Plan (VPlan)](#verification-plan-vplan)
  - [Feature Coverage Goals](#feature-coverage-goals)
  - [Corner Cases Targeted](#corner-cases-targeted)
- [Test Suite](#test-suite)
- [Functional Coverage](#functional-coverage)
- [Assertion-Based Verification (SVA)](#assertion-based-verification-sva)
- [Directory Structure](#directory-structure)
- [Tools & Technologies](#tools--technologies)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Running Simulations](#running-simulations)
  - [Coverage Collection & Merge](#coverage-collection--merge)
- [Results & Metrics](#results--metrics)
- [Regression Flow](#regression-flow)
- [Known Bugs Found](#known-bugs-found)
- [Future Work](#future-work)
- [References](#references)
- [Author](#author)

---

## Overview

This project implements a complete **UVM-based functional verification environment** for the 5-stage pipelined RISC-V RV32I CPU (see companion project: *RISC-V CPU + ASIC Flow*). Functional verification of a pipelined processor is non-trivial — bugs hide in rare temporal sequences involving hazards, back-to-back instruction combinations, edge-case immediates, and pipeline flush/stall interactions that directed testing alone cannot expose.

This environment solves that problem through:
- A layered **UVM testbench** with reusable agents, monitors, scoreboards, and coverage collectors
- **Constrained-random test generation** that steers stimulus toward microarchitecturally interesting regions
- **Functional coverage** groups tied directly to the verification plan to measure completeness
- **SystemVerilog Assertions (SVA)** embedded at pipeline boundaries to catch bugs at their source
- A **self-checking scoreboard** using a golden reference model to automatically flag mismatches

**Outcome: Verification mastery** — demonstrating professional-grade verification closure through coverage-driven verification (CDV) methodology.

---


## Problem Statement

A pipelined CPU exposes a class of bugs that only appear under specific sequences of instructions — sequences unlikely to arise in simple directed tests:

| Bug Class | Example Corner Case |
|---|---|
| **Forwarding failure** | Back-to-back RAW with ALU→Load→ALU chain |
| **Load-use stall escape** | LW followed immediately by dependent branch |
| **Branch flush corruption** | Taken branch during a stall cycle |
| **Write-back collision** | Two instructions writing the same register in adjacent cycles |
| **Immediate sign-extension** | Negative immediates at 12-bit boundary (`0x800`) |
| **ALU overflow/underflow** | SLTU with `rs1=0, rs2=0xFFFFFFFF` |
| **PC misalignment** | JAL/JALR with non-word-aligned targets |
| **x0 write immunity** | Any instruction targeting register `x0` must not alter it |
| **NOP bubble propagation** | Bubbles (NOPs) flowing through all 5 stages cleanly |
| **Reset state correctness** | All pipeline registers and PC cleared on assertion of reset |

Directed tests cannot feasibly cover the combinatorial explosion of these interactions. A constrained-random UVM environment with functional coverage closure is the industry-standard solution.

---

