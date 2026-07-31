# Personal Fork Context

## Purpose

This repository is a personal visual fork of Ashell.

The goal is to create a custom Hyprland rice experience while preserving Ashell functionality.

This is NOT a rewrite of Ashell.
This is NOT a new shell implementation.

Primary focus:

- visual design;
- theme;
- layout;
- spacing;
- typography;
- icons;
- colors;
- transparency.

---

## Development Rules

Preserve upstream functionality.

Prefer:

1. Existing configuration options.
2. Existing theme/layout mechanisms.
3. Minimal UI code changes.

Before changing Rust logic:

- explain why configuration is insufficient;
- identify affected files;
- keep the change minimal.

Avoid modifying:

- services;
- backend integrations;
- IPC;
- compositor logic;
- architecture.

---

## Design Direction

Target:

- clean;
- dark;
- technical;
- functional;
- lightweight;
- polished Linux rice.

Inspired by Mutagen-like aesthetics.

Avoid:

- AI-generated looking interfaces;
- decorative slogans;
- unnecessary information;
- fake futuristic UI;
- excessive blur;
- excessive animations;
- visual clutter.

The interface should prioritize information density and usability.

---

## Hardware Information Policy

Do not display hardware identity in the UI.

Forbidden:

- laptop manufacturer;
- laptop model;
- CPU marketing names;
- GPU marketing names.

Examples:

"Hasee S7-DA5NP"
"Intel Core i5-12500H"
"RTX 3050 Laptop"

Allowed:

- CPU usage;
- RAM usage;
- GPU utilization;
- temperature;
- battery;
- network state.

---

## Performance Requirements

Visual improvements must not significantly increase:

- RAM usage;
- CPU usage;
- startup time.

Avoid unnecessary dependencies and background services.

---

## Workflow

Before implementing visual changes:

1. Analyze current implementation.
2. Explain the proposed changes.
3. Prefer small incremental commits.

Keep the fork close to upstream when possible.
