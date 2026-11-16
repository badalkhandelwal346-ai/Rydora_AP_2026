# Rydora Ride-Sharing & Carpooling Platform: Relational Database Normalization & Dependency Specification

**Document Version:** 1.0.0  
**Deliverable:** Database Normalization Specification (`normalization.md`)  
**Project:** Rydora — Community-First Ride-Sharing & Carpooling Ecosystem  
**Target Engine:** PostgreSQL 15+ / Prisma ORM v7  
**Schema Definition:** `backend/prisma/schema.prisma` (64 Relational Models, 49 Domain Enums)  
**Deliverable Context:** AP Project Checkpoint 2 (CP2)  
**Academic Cohort:** Team T0145 · Section A  
**Authors & Engineering Contributors:**
- Badal Khandelwal (Lead Backend & Database Architect)
- Anindya Sarkar (Database Administrator & Auth Specialist)
- Anshika Srivastava (Data Modeler & Systems Analyst)
- Atharv (Safety, Real-Time & Communications Engineer)

---

## Table of Contents

1. [Executive Summary & Normalization Objectives](#1-executive-summary--normalization-objectives)
2. [Theoretical Foundations of Relational Normalization](#2-theoretical-foundations-of-relational-normalization)
   - [2.1 Functional Dependencies (FDs)](#21-functional-dependencies-fds)
   - [2.2 Normal Forms Criteria (1NF through BCNF and 4NF)](#22-normal-forms-criteria-1nf-through-bcnf-and-4nf)
   - [2.3 Modification Anomalies Addressed](#23-modification-anomalies-addressed)
3. [Domain-by-Domain Normalization & Functional Dependency Analysis](#3-domain-by-domain-normalization--functional-dependency-analysis)
   - [Module 1: Identity, Authentication & Preferences](#module-1-identity-authentication--preferences-7-entities)
   - [Module 2: Communities, Domain Verification & Membership](#module-2-communities-domain-verification--membership-6-entities)
   - [Module 3: Fleet & Driver Management](#module-3-fleet--driver-management-3-entities)
   - [Module 4: Rides, Routes, Stops & Waypoints](#module-4-rides-routes-stops--waypoints-4-entities)
   - [Module 5: Bookings, Dynamic Quotes & Cancellations](#module-5-bookings-dynamic-quotes--cancellations-8-entities)
   - [Module 6: Financial Ledger, Escrow & Earnings](#module-6-financial-ledger-escrow--earnings-6-entities)
   - [Module 7: Safety, Real-Time Telemetry, SOS & Evidence](#module-7-safety-real-time-telemetry-sos--evidence-7-entities)
   - [Module 8: Ratings, Reviews, AI Feedback & Driver Tiers](#module-8-ratings-reviews-ai-feedback--driver-tiers-8-entities)
   - [Module 9: Multi-Channel Notifications & Preferences](#module-9-multi-channel-notifications--preferences-3-entities)
   - [Module 10: External Calendar Integrations](#module-10-external-calendar-integrations-2-entities)
   - [Module 11: Artificial Intelligence Services](#module-11-artificial-intelligence-services-3-entities)
   - [Module 12: Ecological Sustainability & Green Rewards](#module-12-ecological-sustainability--green-rewards-4-entities)
   - [Module 13: Platform Administration & Dynamic Policies](#module-13-platform-administration--dynamic-policies-3-entities)
4. [Lossless Join Decomposition & Dependency Preservation Proofs](#4-lossless-join-decomposition--dependency-preservation-proofs)
5. [Multi-Valued Dependency (4NF) & Join Dependency (5NF) Analysis](#5-multi-valued-dependency-4nf--join-dependency-5nf-analysis)
6. [Controlled Semi-Structured Attributes (JSON Justification)](#6-controlled-semi-structured-attributes-json-justification)
7. [Comprehensive Entity Relationship & Referential Cardinality Matrix](#7-comprehensive-entity-relationship--referential-cardinality-matrix)
8. [Conclusion & Production Readiness Verification](#8-conclusion--production-readiness-verification)

---

## 1. Executive Summary & Normalization Objectives

The **Rydora** database architecture is designed as an enterprise-grade, highly normalized relational foundation supporting a community-driven ride-sharing platform. Built upon PostgreSQL 15+ and managed via Prisma ORM, the physical schema consists of:
- **64 Distinct Relational Tables**
- **49 Strongly-Typed Domain Enums**
- **100% Cryptographic UUID Primary Keys** (`UUIDv4`) to eliminate enumeration attacks and partition bottlenecks.

The primary objective of this normalization specification is to formally demonstrate that the Rydora schema:
1. **Eliminates Redundancy:** Prevents duplicate facts from being stored across multiple rows or tables.
2. **Prevents Update, Insertion, and Deletion Anomalies:** Guarantees that updating a user attribute, deleting a vehicle, or cancelling a booking does not corrupt surrounding relational integrity or orphan historical audit logs.
3. **Achieves Boyce-Codd Normal Form (BCNF) / Third Normal Form (3NF):** Verifies that every determinant attribute is a candidate key for all functional dependencies.
4. **Guarantees Lossless Join Decomposition:** Proves that decomposes of complex domain entities (e.g., decomposing a raw Ride into `Ride`, `RideRoute`, `RideStop`, and `RideCommunity`) retain mathematical equivalence and can be reconstituted without spurious tuples.
5. **Maintains Financial & Safety Immutability:** Enforces strict separation between mutable operational states and immutable historical snapshots (e.g., `BookingCommunityEligibility`, `FareQuote`, `LedgerEntry`, and `SafetyEvidence`).

---

## 2. Theoretical Foundations of Relational Normalization

### 2.1 Functional Dependencies (FDs)
Given a relational schema $R$ and subsets of attributes $X, Y \subseteq R$, a **Functional Dependency** denoted as:
$$X \rightarrow Y$$
holds over $R$ if and only if for any two valid tuples $t_1, t_2 \in R$:
$$t_1[X] = t_2[X] \implies t_1[Y] = t_2[Y]$$
In Rydora, $X$ constitutes a **Superkey** if $X \rightarrow R$. If no proper subset $X' \subset X$ functionally determines $R$, then $X$ is a **Candidate Key** ($CK$). The primary key ($PK$) is chosen as a surrogate UUID from the set of candidate keys.

### 2.2 Normal Forms Criteria (1NF through BCNF and 4NF)

```mermaid
flowchart LR
    UNF["Unnormalized Form (UNF)"] -->|Atomic attributes, no repeating groups| 1NF["1NF"]
    1NF -->|No partial dependencies on composite keys| 2NF["2NF"]
    2NF -->|No transitive dependencies on non-prime attributes| 3NF["3NF"]
    3NF -->|Every determinant is a superkey| BCNF["BCNF"]
    BCNF -->|No non-trivial multi-valued dependencies| 4NF["4NF"]
```