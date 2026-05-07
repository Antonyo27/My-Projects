# Local Computer Retail Business

> **POS, Inventory, and Service-Log System for a Computer Retail & Repair Shop.**
> Confidential client project · Live in production.

[![Status](https://img.shields.io/badge/Status-Confidential_Client-blue?style=flat-square)](#)
[![Stack](https://img.shields.io/badge/Stack-React_%2B_Express-61DAFB?style=flat-square&logo=react&logoColor=black)](#)
[![License](https://img.shields.io/badge/License-Proprietary-red?style=flat-square)](../../LICENSE)

---

> **Note on Confidentiality.** This project was built under a confidentiality agreement with a private client. The client name, branding, internal business logic, deployment specifics, and source code are **intentionally omitted**. What follows is a **generalized system overview** approved for portfolio demonstration only.

<!--
  Replace the placeholder below with a sanitized hero screenshot once available.
  Make sure to redact client name, logos, real customer data, and any branding before publishing.
-->

![System hero](./screenshots/home-page.png)

## System Overview

A comprehensive **web-based POS (Point of Sale), Inventory Management, and Service Logging platform** built for a local computer retail and repair business. The system replaced a paper-based workflow with a typed, audited digital platform — providing real-time data accuracy, role-based security, and an intuitive user experience.

It serves the dual nature of a computer business:

- **Retail** — selling hardware, peripherals, and accessories
- **Service** — logging repairs, technical consultations, troubleshooting sessions, and customer service tickets

## Goal

The primary goal was to **digitize and automate manual business processes** to:

- Reduce human error in stock counts and pricing
- Optimize stock levels and prevent stockouts
- Improve transaction efficiency at the counter
- Provide the owner with **actionable insights** through dashboards and exportable reports
- Establish accountability via audit logs and role-based access

## Key Features

- **Point of Sale (POS)** — Fast, secure transaction processing for daily retail sales with receipt generation.
- **Inventory Management** — Real-time stock levels, product variations, and **low-inventory alerts** that prevent stockouts.
- **Service Logging** — Records and tracks services rendered (repairs, diagnostics, consultations), maintaining a full history of each customer interaction and service detail.
- **Comprehensive Reporting** — Detailed sales, inventory, and audit reports exportable to Excel for accounting and strategic planning.
- **Role-Based Access Control (RBAC)** — Restricts access by role (Admin, Cashier, Staff), preventing unauthorized actions and protecting sensitive financial data.
- **Audit Logging** — Detailed, searchable log of user activities for accountability and troubleshooting.

## How It Helps

- **Operational Efficiency** — Consolidates POS, inventory, service tickets, and reporting into a single interface, eliminating administrative duplication.
- **Data-Driven Decisions** — Real-time dashboards enable informed decisions on purchasing, staffing, and sales strategy.
- **Improved Accuracy** — Automated calculations and inventory deductions eliminate the discrepancies common in manual bookkeeping.
- **Enhanced Security** — Secure authentication, rate limiting, and structured access controls protect sensitive business data.

## System Architecture Highlights

The system uses a **modern decoupled architecture**, separating the React frontend from the Express/Node.js backend API. The backend is **strictly typed** via TypeScript and Zod schemas, ensuring data integrity before persisting through Prisma to PostgreSQL.

The infrastructure supports **automated deployment** and is configured for high availability using **PM2** in production environments.

## Documentation

| Document | Description |
|----------|-------------|
| [Case Study](./docs/case-study.md) | Narrative-style write-up: client need → solution → outcome |
| [Engineering Challenges](./docs/challenges.md) | Selected hard problems and how they were solved |
| [Deployment](./docs/deployment.md) | Infrastructure and operational notes (sanitized) |
| [Scalability](./docs/scalability.md) | Scaling and performance considerations |
| [Tech Stack](./tech-stack.md) | Detailed stack breakdown with rationale |

## Visual Artifacts

- [`screenshots/`](./screenshots/) — UI captures (sanitized: client branding and real customer data redacted)
- [`architecture/`](./architecture/) — System design and data flow diagrams
- [`demo/`](./demo/) — Walkthrough video and animated previews

---

<sub>© 2026 Mark Anthony Resoso · See [LICENSE](../../LICENSE) · Provided for portfolio demonstration only. No confidential client information is disclosed.</sub>
