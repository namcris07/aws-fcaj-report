---
title : "Stage 5 – SAST Scan (SonarQube)"
date : 2026-07-06 
weight : 4
chapter : false
pre : " <b> 5.5.4. </b> "
---

# 5.5.4 Stage 5 – Static Code Analysis (SAST Scan - SonarQube)

---

SonarQube detects 4 intentional code issues in `app/src/component/Game/index.js`:
- **S1763:** Unreachable code after `return`.
- **S2871:** `sort()` without compare function leading to incorrect numeric sorting.
- **S3776:** Cognitive Complexity = 18 > threshold 15.
- **S3358:** Nested ternary operators.

---

### Remediation

- **S2871:** Pass explicit numeric comparison `numScores.sort((a, b) => a - b)`.
- **S1763:** Remove dead code.
- **S3776:** Refactor complex functions into smaller modular helpers.
