# Solution Design & Standard Operating Procedure (SOP)
## Client: EliMax Group Solar Companies
**Version:** 1.1  
**Date:** 2026-01-12  

---

## 1. Executive Summary
This document outlines the Salesforce Solution Design for EliMax Group, addressing challenges in sales lead quality, process visibility, regional pricing, and customer support scalability. The solution utilizes Salesforce Sales Cloud, Service Cloud, and Experience Cloud.

**Key Problems Addressed:**
- Incomplete/inaccurate lead data causing wasted sales effort
- Lack of visual sales process guidance for new agents
- No standardized lead scoring for prioritization
- Multi-currency regional pricing complexity
- Inconsistent support levels across customer tiers
- SLA violations for Premium customers
- High volume of support calls that could be deflected

---

## 2. System Configuration
**Environment:** Salesforce Developer Edition  
**Core Features Enabled:**
*   Multi-Currency (USD, EUR)
*   Lightning Knowledge
*   Entitlement Management
*   Digital Experiences (Customer Community)
*   Web-to-Case

> **Note:** This implementation is designed for a Developer Edition org. Some features like Email-to-Case and custom Quote templates have limited functionality in Dev Edition.

---

## 3. Data Model & Key Components

### 3.1 Leads (Sales Cloud)
| Component | Type | Purpose |
|-----------|------|---------|
| `Region__c` | Picklist | Territory assignment (North America, Europe, Asia) |
| `Lead_Score__c` | Formula (Number) | Prioritize high-value prospects (0-40 scale) |
| `Require_Phone_or_Email` | Validation Rule | Data quality enforcement |

**Lead Score Calculation:**
- +10 points: Email provided
- +10 points: Phone provided
- +20 points: Rating = Hot
- +10 points: Rating = Warm

### 3.2 Accounts
| Component | Type | Purpose |
|-----------|------|---------|
| `Region__c` | Picklist | Determines Price Book for Opportunities |
| `Support_Tier__c` | Picklist | Drives SLA and case routing (Basic/Standard/Premium) |

### 3.3 Opportunities & Products
*   **Regional Price Books**: Separate Price Books for North America (USD), Europe (EUR), and Asia (USD).
*   **Products**: EliMax Solar Panel 300W, EliMax Inverter 5kW.
*   **Quotes**: Standard Salesforce Quoting enabled for PDF generation.

### 3.4 Cases (Support)
| Component | Type | Purpose |
|-----------|------|---------|
| Assignment Rules | `EliMax Routing` | Routes cases to queues by Support Tier |
| Queues | Premium/Standard/Basic | Dedicated teams per tier |
| Escalation Rules | `EliMax Escalation` | Escalates cases breaching SLA |
| Web-to-Case | Enabled | Automated case creation from Help Center |
| **Case Deflection Flow** | Screen Flow | Guides customers through Knowledge before case creation |

### 3.5 Entitlements & Milestones
| Milestone | Premium SLA | Standard SLA |
|-----------|-------------|--------------|
| First Response | 1 Hour | 4 Hours |
| Case Resolution | 8 Hours | 24 Hours |

---

## 4. Standard Operating Procedures (SOP)

### 4.1 Sales Agent Workflow
**Objective**: Convert qualified leads into closed sales with accurate regional pricing.

#### Step 1: Lead Intake
1.  Agent creates a new Lead in Salesforce.
2.  **System Requirement**: Must enter Phone OR Email (validation rule enforces this).
3.  Select the correct **Region** (North America, Europe, or Asia).
4.  Save the Lead.

#### Step 2: Lead Qualification (Using Path)
1.  View the **Lead Score** on the record.
    *   Score > 30: Hot prospect. Call immediately.
    *   Score 20-30: Warm prospect. Follow up within 24 hours.
    *   Score < 20: Nurture via email campaigns.
2.  Follow the **Path** at the top of the page.
3.  Read the "Guidance for Success" tips at each stage.
4.  Update Status as you progress: `Open - Not Contacted` → `Working - Contacted` → `Closed - Converted`.

#### Step 3: Lead Conversion
1.  When the prospect is qualified, click **Convert**.
2.  Create: Account + Contact + Opportunity.
3.  Verify the **Region** field copied to the Account (required for correct pricing).

#### Step 4: Quoting
1.  Navigate to the Opportunity.
2.  Select the correct **Price Book** based on Account Region:
    *   North America → `North America Price Book`
    *   Europe → `Europe Price Book`
    *   Asia → `Asia Price Book`
3.  Add **Products** from the Products related list.
4.  Click **Quotes** related list > **New Quote**.
5.  Fill in Quote details (Discount, Expiration Date).
6.  Click **Save**.
7.  Click **Create PDF** > Select template > **Save to Quote**.
8.  Click **Email Quote** to send to the customer.

---

### 4.2 Support Agent Workflow
**Objective**: Resolve customer inquiries within SLA times.

#### Step 1: Case Triage
1.  Cases arrive via:
    *   **Web**: Customer submits via Help Center (Web-to-Case).
    *   **Email**: Email-to-Case (if configured).
    *   **Phone**: Agent creates manually.
2.  System **automatically assigns** the Case to the correct Queue:
    *   Premium Account → Premium Support Queue
    *   Standard Account → Standard Support Queue
    *   Basic Account → Basic Support Queue
3.  Agent views their Queue and accepts a Case.

#### Step 2: SLA Management
1.  Check the **Case Record** for the Milestone Tracker.
2.  Note the countdown timers:
    *   **First Response**: Time remaining to acknowledge the case.
    *   **Case Resolution**: Time remaining to fully resolve.
3.  Premium Customers: First response required within 1 hour.
4.  If approaching deadline, escalate immediately to Support Manager.

#### Step 3: Resolution Using Knowledge
1.  Open the **Knowledge** sidebar on the Case page.
2.  Search for keywords related to the customer's issue.
3.  If an article solves the problem:
    *   Click **Attach to Case**.
    *   Send the article link to the customer.
4.  If no article exists, resolve manually and consider creating a new Knowledge article.

#### Step 4: Case Closure
1.  Update the **Status** to `Closed`.
2.  Add a **Resolution Summary** describing how the issue was solved.
3.  Verify the Milestone shows as "Completed" (not "Violated").

---

### 4.3 Customer Self-Service Workflow
**Objective**: Deflect simple support tickets through self-service.

**Option A: Help Center Search**
1.  Customer navigates to the **EliMax Help Center** (Experience Cloud site).
2.  Customer enters keywords in the **Knowledge Search** (e.g., "inverter error", "billing").
3.  Customer views relevant Knowledge Articles.
4.  If resolved, customer exits (no Case created = deflection success).
5.  If unresolved, customer clicks **Contact Support** to submit a Case.

**Option B: Case Deflection Flow (Recommended)**
1.  Customer launches the **Case Deflection Flow**.
2.  Customer selects their issue category (Billing, Technical, Warranty, etc.).
3.  Flow displays relevant Knowledge Articles automatically.
4.  Customer indicates whether the articles solved their problem.
5.  **If Yes**: Flow ends with thank you message. *Case deflected.*
6.  **If No**: Flow collects case details and creates Case automatically.
7.  Customer receives confirmation with Case Number.

*The Flow approach ensures customers always see relevant articles before case creation, increasing deflection rates.*

---

## 5. Security & Access Control

### 5.1 Role Hierarchy
```
CEO
├── Sales Manager
│   └── Sales Representative
└── Support Manager
    └── Support Agent
```
*Managers can view and report on all records owned by their subordinates.*

### 5.2 Profiles
| Profile | Object Access |
|---------|---------------|
| EliMax Sales User | Leads (CRE), Opportunities (CRE), Quotes (CRE), Cases (R) |
| EliMax Support User | Cases (CRE), Knowledge (R), Entitlements (R), Accounts (R) |

*CRE = Create, Read, Edit. R = Read Only.*

---

## 6. Reporting & Dashboards

**EliMax Executive Overview Dashboard:**

| Component | Metric | Purpose |
|-----------|--------|---------|
| Donut Chart | Leads by Region | Territory distribution visibility |
| Bar Chart | Cases by Support Tier | Workload per tier |
| Gauge | SLA Compliance % | Monitor service quality |
| Table | Open Escalations | Identify at-risk cases |

---

## 7. Future Enhancements (Phase 2 Recommendations)
| Enhancement | Benefit | Estimated Effort |
|-------------|---------|------------------|
| Einstein Lead Scoring | Replace formula with AI-driven scoring | 2 weeks |
| Einstein Bots | Instant chat support before case creation | 3 weeks |
| ERP Integration | Live inventory for product availability | 4 weeks |
| CTI Integration | Screen pop for inbound calls | 2 weeks |
| SMS Notifications | SLA breach alerts via text | 1 week |

---

## 8. Appendix: Quick Reference

### Lead Score Thresholds
| Score | Action |
|-------|--------|
| 30-40 | Call immediately |
| 20-29 | Follow up within 24 hours |
| 10-19 | Add to nurture campaign |
| 0-9 | Low priority, review weekly |

### SLA Summary
| Tier | First Response | Resolution |
|------|----------------|------------|
| Premium | 1 hour | 8 hours |
| Standard | 4 hours | 24 hours |
| Basic | 8 hours | 48 hours |

### Price Book Selection
| Account Region | Price Book |
|----------------|------------|
| North America | North America Price Book (USD) |
| Europe | Europe Price Book (EUR) |
| Asia | Asia Price Book (USD) |
