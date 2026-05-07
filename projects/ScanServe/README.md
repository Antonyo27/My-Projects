# ScanServe

> **Queue smarter. Serve better.**
> Real-Time Multi-Tenant Queue Management & Feedback System

[![Status](https://img.shields.io/badge/Status-Independent_Project-success?style=flat-square)](#)
[![Stack](https://img.shields.io/badge/Stack-TALL-FF2D20?style=flat-square&logo=laravel&logoColor=white)](#)
[![License](https://img.shields.io/badge/License-Proprietary-red?style=flat-square)](../../LICENSE)

---

<!--
  Replace the placeholder below with a hero screenshot once available.
  Recommended: a clean composite of (1) the live TV board, (2) the customer
  ticket on a phone, and (3) the staff admin dashboard.
-->

![ScanServe hero](./screenshots/home-page.png)

## System Overview

**ScanServe** is a modern, real-time **Queue Management and Feedback System** engineered for multi-tenant organizations. Designed with a digital-first approach, it streamlines customer flow and gathers actionable insights across organizational branches and service points through a robust, QR-based workflow.

This repository is a **high-level overview and architectural summary**. The proprietary source code, deployment configuration, and internal business logic remain private.

## System Goals

ScanServe aims to **eliminate physical queues and reduce customer wait times** by fully digitizing the queueing process. It provides organizations with a structured methodology to handle visitor transactions, monitor staff performance in real time, and seamlessly collect customer feedback immediately upon service completion.

Whether deployed in a **university registrar**, a **government office**, or a **corporate service center**, ScanServe acts as a comprehensive "digital lobby" coordinating the end-to-end customer journey from arrival to final resolution.

## Key Features

- **Multi-Tenant Architecture** — Each onboarded organization receives a dedicated subdomain, ensuring strict data isolation and customizable environments per client.
- **Dynamic and Static QR Routing**
  - **Dynamic QRs** broadcast on TV displays using short-lived, signed URLs to guarantee secure queue access.
  - **Static QRs** anchored at physical service points for designated selection and remote feedback submission.
- **Real-Time TV Queue Boards** — WebSocket-powered display boards instantly update queue status the moment a ticket is called.
- **Comprehensive Feedback System** — Integrated surveys directly tied to individual transactions, processing windows, or broader service points.
- **Priority Return Pass System** — Secure tokens issued to customers who must leave the premises and return later (e.g., subsequent document pickup).
- **Multi-Tier RBAC** — Granular permissions covering Super Admin, Organization Admin, Branch Admin, Unit Head, and Staff roles.
- **Advanced Analytics & Reporting** — Automated generation of impact logs, compliance summaries, and audit trails exportable as Excel and PDF.

## Impact & Results

| Metric | Outcome |
|--------|---------|
| **60%** | Average reduction in perceived customer wait times through real-time visibility and digital ticketing. |
| **2×** | Increase in customers served per hour per window through streamlined call-and-serve workflows. |
| **Integrated** | Post-service feedback collection built into the flow, giving management actionable service quality data. |

## Operational Flow

1. **For Customers** — Visitors scan a QR code upon entry, receive a secure digital ticket on their mobile device, and can wait anywhere without losing their queue position. They monitor status in real time and provide instant, context-aware feedback after their transaction.
2. **For Staff and Unit Heads** — Staff manage the queue via a dedicated console (call, serve, redirect). Unit Heads use a real-time rescue dashboard to monitor missed slots, idle periods, and abandoned "zombie" transactions.
3. **For Administrators** — Executive admins gain data-driven insights into queue efficiency, average wait times, staff performance metrics, and overall customer satisfaction across multiple branches.

## Documentation

| Document | Description |
|----------|-------------|
| [Case Study](./docs/case-study.md) | Narrative-style write-up: challenge → solution → impact |
| [Engineering Challenges](./docs/challenges.md) | Hard problems encountered and how they were solved |
| [Deployment](./docs/deployment.md) | Infrastructure, hosting, and CI/CD overview |
| [Scalability](./docs/scalability.md) | Scaling strategy and architectural trade-offs |
| [Tech Stack](./tech-stack.md) | Detailed stack breakdown with rationale |

## Visual Artifacts

- [`screenshots/`](./screenshots/) — UI captures: home page, admin dashboard, mobile views
- [`architecture/`](./architecture/) — System design, database schema, and API flow diagrams
- [`demo/`](./demo/) — Walkthrough video and animated previews

---

<sub>© 2026 Mark Anthony Resoso · See [LICENSE](../../LICENSE) · This documentation is provided for portfolio demonstration only.</sub>
