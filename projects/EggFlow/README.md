# EggFlow

> **Farm-to-shelf egg production management.**
> End-to-end production, inventory, and distribution platform for agribusiness.

[![Status](https://img.shields.io/badge/Status-Independent_Project-success?style=flat-square)](#)
[![Stack](https://img.shields.io/badge/Stack-TALL-FF2D20?style=flat-square&logo=laravel&logoColor=white)](#)
[![License](https://img.shields.io/badge/License-Proprietary-red?style=flat-square)](../../LICENSE)

---

<!--
  Replace the placeholder below with a hero screenshot once available.
  Recommended: a composite of (1) the executive dashboard, (2) the POS sale flow,
  and (3) the multi-location inventory view.
-->

![EggFlow hero](./screenshots/home-page.png)

## System Overview

**EggFlow** is a modern, multi-tenant **production, inventory, and POS platform** purpose-built for agribusinesses managing the full lifecycle of egg production — from daily farm collection through warehouse processing to retail sale across multiple locations.

This repository serves as a **high-level system overview**. The proprietary source code, deployment configuration, and internal business logic remain private.

## System Goals

EggFlow's primary objective is to **replace manual logbooks and disconnected spreadsheets** with a centralized digital hub that gives owners, managers, and staff real-time, actionable insight into every stage of operations.

By digitizing daily yield tracking, inventory movements, sales, expenses, and shift accountability, the platform eliminates invisible wastage, automates discrepancy detection, and enables truly data-driven decisions across single-farm or multi-location agribusinesses.

## Core Capabilities

- **Production & Yield Tracking** — Digitally logs daily egg gatherings, categorizes yields by size (Jumbo, Large, Medium, etc.), and computes per-farm production analytics including source damage rates.
- **Multi-Location Inventory** — Real-time stock across simultaneous warehouse and retail locations, delivery logistics between sites, and supply-chain wastage logged at every transfer point.
- **POS & Sales Hub** — Retail and wholesale transactions with dynamic pricing, including a **VIP reservation system** that syncs with live inventory to prevent double-booking of reserved stock.
- **Shift & Staff Management** — Tracks staff clock-ins and **automatically calculates shift-end discrepancies** to enforce accountability for cash and stock during each shift.
- **Financials & Expense Approval** — Logs operational costs (feed, logistics, utilities) and **routes high-value expenses through management approval workflows** before finalization.
- **Executive Dashboards** — Transforms raw operational data into business intelligence via interactive charts and stat cards — production comparisons, sales trends, inventory valuations.
- **Multi-Tenant Architecture** — Strict per-shop data isolation with hierarchical aggregation for upper management.
- **Real-Time Customer Engagement** — Floating live chat widget connecting customers directly to shop staff via secure WebSockets.
- **Audit & RBAC** — Every critical action is tracked via automated audit logs; access is gated by hierarchical role permissions (Admin, Manager, Staff, Customer).

## Impact & Results

| Metric | Outcome |
|--------|---------|
| **100%** | **Wastage visibility** — full supply-chain wastage tracking from farm to shelf, eliminating previously invisible losses. |
| **85%** | **Reduction in shift discrepancies** — unresolved cash and stock variances dropped via automated shift-end reconciliation. |
| **3×** | **Faster executive decisions** — real-time dashboards replace weekly manual report compilation. |

## Operational Flow

1. **At the farm** — Staff log daily gatherings and categorized yields directly from the field. Source damage rates and mortality are tracked per farm.
2. **In the warehouse** — Inventory transfers, breakage, and category re-grading are logged at every step. Multi-location stock stays synchronized in real time.
3. **At the POS** — Cashiers process retail and wholesale transactions with dynamic pricing. VIP customers' reservations sync automatically with inventory to prevent double-booking.
4. **For management** — High-value expenses route through approval workflows. Executive dashboards compare production across farms, surface sales trends, and visualize true inventory valuation across all locations.

## Documentation

| Document | Description |
|----------|-------------|
| [Case Study](./docs/case-study.md) | Narrative-style write-up: challenge → solution → impact |
| [Engineering Challenges](./docs/challenges.md) | Hard problems encountered and how they were solved |
| [Deployment](./docs/deployment.md) | Infrastructure, hosting, and CI/CD overview |
| [Scalability](./docs/scalability.md) | Scaling strategy and architectural trade-offs |
| [Tech Stack](./tech-stack.md) | Detailed stack breakdown with rationale |

## Visual Artifacts

- [`screenshots/`](./screenshots/) — UI captures: home, executive dashboard, POS, mobile views
- [`architecture/`](./architecture/) — System design, database schema, and module flow diagrams
- [`demo/`](./demo/) — Walkthrough video and animated previews

---

<sub>© 2026 Mark Anthony Resoso · See [LICENSE](../../LICENSE) · This documentation is provided for portfolio demonstration only.</sub>
