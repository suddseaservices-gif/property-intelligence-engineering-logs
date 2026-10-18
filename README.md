# Property Intelligence — Engineering Logs

An evolving, locally operated property-intelligence research project: computer vision, camera systems, automation, robotics, environmental sensing, and practical machine learning.

This repository preserves **what was attempted, what worked, what did not work, and what to test next**. An unfinished experiment is not erased; it becomes a reproducible starting point for the next iteration.

## Engineering field reports

- [2026-10-18 — From a broken YOLOv5 environment to a working training pipeline](logs/2026-10-18-yolov5-chicken-detector.md)

## Current focus

Build a custom chicken detector for an existing Blue Iris camera system, then generalize the same data-collection and evaluation process to other animals, tools, keys, wallets, and property objects.

**Status (2026-10-18):** YOLOv5 inference restored; custom model trained and requested by Blue Iris; chicken detections **not yet achieved**. Next: representative, carefully labeled local images and held-out camera-footage evaluation.

## Principles

- Train and evaluate for the environment the system will actually see.
- Distinguish observed results from hypotheses.
- Prefer repeatable experiments, clear logs, and modular components.
- Preserve unsuccessful attempts and the lessons they reveal.
- Keep sensitive footage, identifying details, credentials, and private property data out of public commits.

This is an engineering journal, not a claim that every planned subsystem is complete.
