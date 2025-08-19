

# Stacks-Credenza - Skill Reputation & Assessment Smart Contract

## Overview

This Clarity smart contract provides a **decentralized skill reputation and assessment system**.
It enables:

* User registration and skill association.
* Decentralized peer assessments of skills.
* Calculation of **mean scores** and **standard deviation** for fair validation.
* A **reputation system** that rewards valid assessments and penalizes invalid ones.

The system ensures assessments are objective by requiring multiple assessors, calculating variance, and enforcing thresholds for approval.

---

## ✨ Features

* **User Management**

  * Register new users.
  * Track skills, reputation, and assessment history.

* **Skill Management**

  * Create new skills with names, descriptions, and categories.
  * Define required number of assessments per skill.

* **Assessment System**

  * Peer-to-peer assessments with scoring.
  * Enforces a maximum number of assessors per skill.
  * Calculates **mean** and **standard deviation** for validation.
  * Ensures statistical integrity by rejecting highly deviant assessments.

* **Reputation System**

  * Rewards valid assessments.
  * Penalizes invalid assessments.
  * Tracks per-skill reputation and assessment quality.

---

## ⚙️ Constants

| Constant                       | Description                                 | Value       |
| ------------------------------ | ------------------------------------------- | ----------- |
| `contract-owner`               | Contract deployer                           | `tx-sender` |
| `min-assessors`                | Minimum number of assessors required        | `u3`        |
| `assessment-threshold`         | % of approval needed for skill verification | `u70`       |
| `max-assessors`                | Maximum number of assessors per skill       | `u20`       |
| `standard-deviation-threshold` | Allowed deviation from mean                 | `u15`       |
| `reputation-penalty`           | Penalty for invalid assessments             | `u5`        |
| `reputation-reward`            | Reward for valid assessments                | `u2`        |

---

## ❌ Error Codes

| Code                                    | Meaning                          |
| --------------------------------------- | -------------------------------- |
| `err-not-authorized (err u100)`         | Caller is not authorized         |
| `err-already-registered (err u101)`     | User already registered          |
| `err-not-registered (err u102)`         | User not registered              |
| `err-insufficient-assessors (err u103)` | Not enough assessors to verify   |
| `err-already-assessed (err u104)`       | User already assessed this skill |
| `err-max-assessors-reached (err u105)`  | Too many assessors assigned      |
| `err-invalid-score (err u106)`          | Score is out of valid range      |
| `err-invalid-skill-id (err u107)`       | Skill ID does not exist          |
| `err-invalid-input (err u108)`          | Invalid input data               |

---

## 🗂 Data Structures

### `users` (map)

Tracks registered users and their reputation.

```clarity
principal => {
    registered: bool,
    skills: (list 20 uint),
    reputation: uint,
    total-assessments: uint,
    invalid-assessments: uint
}
```

### `skill-reputation` (map)

Tracks reputation and assessment history **per skill per user**.

```clarity
{user: principal, skill-id: uint} => {
    reputation: uint,
    assessments-given: uint,
    valid-assessments: uint
}
```

### `skills` (map)

Holds skill metadata.

```clarity
uint => {
    name: (string-ascii 50),
    description: (string-ascii 200),
    required-assessments: uint,
    category: (string-ascii 50)
}
```

### `skill-assessments` (map)

Tracks all assessments for a user’s skill.

```clarity
{skill-id: uint, user: principal} => {
    assessors: (list 20 principal),
    scores: (list 20 uint),
    verified: bool,
    timestamp: uint,
    mean-score: uint,
    standard-deviation: uint
}
```

### `skill-id-counter` (data-var)

Keeps track of skill IDs.

---

## 📐 Helper Functions

### Validation Functions

* `is-valid-skill-id` → Checks if skill exists.
* `is-valid-string-50` / `is-valid-string-200` → Validates input strings.
* `is-valid-name` → Additional rules for skill names.
* `is-valid-category` → Rules for categories.
* `is-valid-description` → Rules for descriptions.

### Statistical Functions

* `calculate-mean` → Computes mean score from list of scores.
* `calculate-standard-deviation` → Computes SD to check score consistency.
* `square` / `square-diff-from-mean` → Helpers for variance calculation.
* `sqrt` → Approximate integer square root.

---

## 🔄 Workflows

### 1. User Registration

1. User calls `register-user`.
2. Added to `users` map with default reputation and no skills.

### 2. Skill Creation

1. Contract owner (or authorized account) adds a new skill.
2. Skill assigned an auto-incremented `skill-id`.
3. Stored in `skills` map.

### 3. Skill Assessment

1. Assessors submit scores for another user’s skill.
2. Score validated (within allowed range).
3. If deviation is within `standard-deviation-threshold`, assessment is considered valid.
4. Assessment stored in `skill-assessments`.

### 4. Skill Verification

1. When enough assessments are collected (`>= required-assessments`).
2. Calculate **mean** and **standard deviation**.
3. If approval ≥ `assessment-threshold`, mark skill as `verified`.

### 5. Reputation Adjustment

* Valid assessment → assessor gains `+2 reputation`.
* Invalid assessment → assessor loses `-5 reputation`.

---

## 📊 Example Usage

### Register User

```clarity
(contract-call? .skill-contract register-user)
```

### Create New Skill

```clarity
(contract-call? .skill-contract create-skill "Solidity" "Smart contract development" u5 "Blockchain")
```

### Submit Assessment

```clarity
(contract-call? .skill-contract assess-skill u1 'ST123... 85)
```

### Verify Skill (after enough assessments)

```clarity
(contract-call? .skill-contract verify-skill u1 'ST123...)
```

### Query User

```clarity
(contract-call? .skill-contract get-user 'ST123...)
```

---

## 🔐 Security Considerations

* Prevents self-assessment.
* Caps assessors per skill (`max-assessors`).
* Uses statistical thresholds to prevent manipulation.
* Reputation system discourages spam/inaccurate scoring.

---
