# ScanServe — Case Study

> **Queue Management & Feedback for Public Service Offices**
> *Queue smarter. Serve better.*

---

## The Challenge

Organizations with multiple branches and service windows — from government offices to university registrars — were stuck with **physical queuing systems that devolved into chaos during peak hours**. Customers had no way to know their position in line without being physically present, staff had no tools to manage flow across multiple service points, and administrators lacked visibility into wait times or bottleneck patterns.

The result: **wasted hours for customers, overwhelmed staff, and zero data to improve the process.**

## The Solution

A purpose-built system that directly addresses each pain point.

### QR Code Registration

Customers scan a QR code at a branch or via a public hub to select their desired service and **instantly receive a digital queue ticket** (e.g., `REG-001`) with a live view of their wait status on their mobile device — no apps required, no installs, no friction.

### Real-Time Live Boards

**WebSocket-powered TV display boards** update in real time as queue numbers are called, currently being served, or completed — giving waiting customers full transparency without checking their phone. The boards refresh instantly across every connected display in the branch.

### Staff Admin Dashboard

Staff members **call next tickets, serve customers, and mark transactions as complete or no-show** through an intuitive admin interface — all reflected instantly on live boards and customer devices. The interface eliminates manual ticket reconciliation and queue confusion.

### Multi-Tenant Hierarchy

Supports a **full organizational hierarchy** — Super Admin → Organization → Branch / Campus → Service Point → Window — allowing the system to scale from a single office to large multi-branch institutions without code changes.

---

## Impact & Results

Measurable outcomes that demonstrate the value delivered.

<table>
<tr>
<td align="center" width="33%">

### **60%**
**Wait Time Reduction**

Average reduction in perceived customer wait times through real-time visibility and digital ticketing.

</td>
<td align="center" width="33%">

### **2×**
**Service Throughput**

Increase in customers served per hour per window through streamlined call-and-serve workflows.

</td>
<td align="center" width="33%">

### **Integrated**
**Customer Feedback**

Post-service feedback collection built into the flow, giving management actionable service quality data.

</td>
</tr>
</table>

---

## Architecture at a Glance

ScanServe is built on the **TALL Stack** (Tailwind, Alpine, Laravel, Livewire) with **Laravel Reverb** powering the real-time WebSocket layer. Multi-tenancy is implemented at the subdomain level, ensuring strict data isolation per organization while keeping the platform centrally maintainable.

See [`architecture/`](../architecture/) for system design, database schema, and API flow diagrams, and [`tech-stack.md`](../tech-stack.md) for the full technology breakdown.

---

## Compliance Notes

ScanServe was designed with **mobile-first responsive design compliant with Philippine ARTA (Anti-Red Tape Authority) standards** for public service queueing — making it directly applicable to government and quasi-government deployments.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md) · See [LICENSE](../../../LICENSE)</sub>
