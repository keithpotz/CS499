# CS 499 – Computer Science Capstone ePortfolio

**Author:** Keith Pottratz
**Institution:** Southern New Hampshire University
**Course:** CS 499 – Computer Science Capstone
**Live Site:** [keithpotz.github.io/CS499](https://keithpotz.github.io/CS499)

---

## Overview

This repository hosts my CS 499 capstone ePortfolio, published as a GitHub Pages site. The portfolio demonstrates professional growth in software engineering through three enhanced artifacts drawn from prior coursework. Each artifact was analyzed, critiqued, and improved to reflect industry-standard practices in software design, algorithms, and database management.

---

## Portfolio Sections

| Page | Description |
|------|-------------|
| [Home / Self-Assessment](index.md) | Professional self-assessment and program outcomes overview |
| [Code Review](code-review.md) | Formal code review of all three original artifacts |
| [Software Design & Engineering](software-design.md) | Enhancement of a Java Contact Management System |
| [Algorithms & Data Structures](algorithms.md) | Enhancement of an Android Inventory Management App (sorting engine) |
| [Databases](databases.md) | Enhancement of the same Android app (database normalization) |

---

## Artifacts Enhanced

### 1. Contact Management System (Java)  Software Design & Engineering
**Origin:** CS 320 (December 2024)
**Enhancement:** January 2026

The original system was a basic Java application with HashMap-backed contact storage and unit tests. The enhancement transformed it into a production-ready, secure, and professionally architected system by:

- Replacing `HashMap` with `ConcurrentHashMap` to resolve thread-safety issues
- Building a `ContactValidator` class to reject XSS patterns, SQL injection strings, and control characters
- Enforcing a `MAX_CONTACTS` resource limit to prevent memory exhaustion
- Adding a full audit logging trail via SLF4J
- Applying OWASP Top 10 principles throughout, validated by dedicated security tests

---

### 2. Inventory Management App (Android / Java)  Algorithms & Data Structures
**Origin:** CS 360
**Enhancement:** January 2026

The original app had zero sorting capability  all ordering was controlled by a single hardcoded SQL query. The enhancement introduced a complete in-memory sorting engine:

- **Four algorithms implemented from scratch:** QuickSort, MergeSort, Counting Sort, Insertion Sort
- **Nine sort criteria:** name, quantity, price, date added, low-stock priority (asc/desc)
- **Smart automatic algorithm selection** based on dataset size and criteria type
- **Real-time search filtering** that preserves the current sort order
- **Sort preference persistence** via SharedPreferences across app sessions
- Performance improvement: ~60ms (database round-trip) → ~6–14ms (in-memory)

---

### 3. Inventory Management App (Android / Java)  Databases
**Origin:** CS 360
**Enhancement:** January 2026

The original database had 2 tables with no relationships. The enhancement redesigned it to Third Normal Form (3NF):

- Expanded from **2 tables to 6 tables** with 7 foreign key relationships
- Added `categories`, `suppliers`, `locations`, and `inventory_history` tables
- Implemented a **safe Room migration** (v1 → v2) with zero data loss
- Added advanced DAO queries: JOINs, aggregation, low-stock filtering, full-text search
- Added **ACID transaction support** for multi-step operations
- Added unique constraint on usernames and a full audit trail for compliance

---

## Program Outcomes Demonstrated

1. **Collaborative Environments & Communication**  Structured code review and in-code documentation
2. **Professional-Quality Communication**  Technical writing and oral code review presentation
3. **Algorithms and Problem Solving**  Four sorting algorithms with complexity analysis and smart selection
4. **Computing Tools and Industry Practices**  Version control, modular design, organized project structure
5. **Security Mindset**  OWASP-aligned input validation, thread safety, audit logging, referential integrity

---

## Repository Structure

```
CS499/
├── index.md              # Home page / professional self-assessment
├── code-review.md        # Formal code review
├── software-design.md    # Software Design & Engineering artifact
├── algorithms.md         # Algorithms & Data Structures artifact
├── databases.md          # Databases artifact
├── _config.yml           # Jekyll / GitHub Pages configuration
├── _includes/            # Jekyll includes
├── _sass/                # Site stylesheets
└── assets/               # Images and static files
```

---

## Technology

- **Site generator:** Jekyll with the Hacker remote theme
- **Hosting:** GitHub Pages
- **Artifact languages:** Java (contact system), Java / Android (inventory app)
- **Database:** Android Room (SQLite)
