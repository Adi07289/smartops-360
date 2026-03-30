# SmartOps 360 — Data Model & Entity Relationship Diagram

> **Project:** SmartOps 360 — Intelligent Sales Operations Platform  
> **Client:** Vantage Solutions Pvt. Ltd.  
> **Prepared by:** Adi Sharma | PwC Consulting  
> **Platform:** Microsoft Dynamics 365 Sales + Power Platform  
> **Last Updated:** March 2026

---

## 1. Data Architecture Overview

SmartOps 360 leverages Microsoft Dataverse as the unified data layer, extending standard Dynamics 365 Sales entities with custom columns to support AI-driven lead scoring, intelligent pipeline management, and predictive analytics.

### Design Principles
- **Extend, Don't Replace** — Custom columns are added to standard D365 entities to preserve out-of-the-box functionality
- **Referential Integrity** — All relationships use Dataverse-managed lookups with cascading rules
- **Analytics-Ready** — Schema designed for seamless Power BI integration via Dataverse connector

---

## 2. Entity Relationship Diagram

```mermaid
erDiagram
    ACCOUNT ||--o{ LEAD : "generates"
    ACCOUNT ||--o{ OPPORTUNITY : "potential customer"
    ACCOUNT ||--o{ CONTACT : "employs"
    LEAD ||--o| OPPORTUNITY : "qualifies into"
    CONTACT ||--o{ OPPORTUNITY : "linked to"
    OPPORTUNITY ||--o{ ACTIVITY : "tracks"
    LEAD ||--o{ ACTIVITY : "tracks"

    ACCOUNT {
        guid AccountId PK
        string AccountName "Company Name"
        optionset Industry "Manufacturing, Retail, etc."
        currency AnnualRevenue "Yearly Revenue"
        int NumberOfEmployees "Employee Count"
        string MainPhone "Primary Contact Number"
        string Address1_City "City"
        string Address1_State "State/Province"
        string Address1_Country "Country/Region"
        datetime CreatedOn "Record Created Date"
        lookup OwnerId FK "Assigned Sales Rep"
    }

    LEAD {
        guid LeadId PK
        string FirstName "First Name"
        string LastName "Last Name"
        string CompanyName "Company Name"
        string Email "Email Address"
        string BusinessPhone "Phone Number"
        string Topic "Lead Topic/Interest"
        optionset LeadSourceSystem "Website, Referral, Partner, etc. (Custom)"
        int LeadScore "AI-Predicted Score 0-100 (Custom)"
        optionset LeadPriority "Hot, Warm, Cold (Custom)"
        datetime ExpectedCloseDate "Projected Close Date (Custom)"
        optionset Rating "Hot, Warm, Cold (Standard)"
        optionset Status "Open, Qualified, Disqualified"
        datetime CreatedOn "Record Created Date"
        lookup OwnerId FK "Assigned Sales Rep"
    }

    OPPORTUNITY {
        guid OpportunityId PK
        string Topic "Deal Name/Description"
        lookup PotentialCustomer FK "Links to Account"
        currency EstRevenue "Estimated Deal Value"
        datetime EstCloseDate "Target Close Date"
        optionset SalesStage "Qualify, Develop, Propose, Close"
        optionset RiskLevel "Low, Medium, High (Custom)"
        optionset DealCategory "New Business, Upsell, Renewal, Cross-sell (Custom)"
        string Competitor "Competitor Name (Custom)"
        percent Probability "Win Probability"
        optionset Status "Open, Won, Lost"
        datetime CreatedOn "Record Created Date"
        lookup OwnerId FK "Assigned Sales Rep"
    }

    CONTACT {
        guid ContactId PK
        string FirstName "First Name"
        string LastName "Last Name"
        string Email "Email Address"
        string BusinessPhone "Phone Number"
        string JobTitle "Designation"
        lookup AccountId FK "Parent Company"
        datetime CreatedOn "Record Created Date"
    }

    ACTIVITY {
        guid ActivityId PK
        string Subject "Activity Subject"
        optionset ActivityType "Email, Call, Task, Meeting"
        datetime ScheduledStart "Start Date/Time"
        datetime ScheduledEnd "End Date/Time"
        optionset Status "Open, Completed, Cancelled"
        lookup RegardingObjectId FK "Related Lead/Opportunity"
    }
```

---

## 3. Custom Columns Summary

### 3.1 Lead Table — Custom Extensions

| Column Name | Display Name | Data Type | Description | Business Purpose |
|-------------|-------------|-----------|-------------|-----------------|
| `cr_leadscore` | Lead Score | Whole Number (0–100) | AI-predicted conversion probability | Prioritize sales outreach based on ML scoring |
| `cr_leadpriority` | Lead Priority | Choice (Hot/Warm/Cold) | Priority classification based on score thresholds | Quick visual categorization for SDRs |
| `cr_leadsourcesystem` | Lead Source System | Choice (6 options) | Channel that generated the lead | Track marketing ROI by acquisition channel |
| `cr_expectedclosedate` | Expected Close Date | Date Only | Projected deal closure timeline | Revenue forecasting and pipeline planning |

### 3.2 Opportunity Table — Custom Extensions

| Column Name | Display Name | Data Type | Description | Business Purpose |
|-------------|-------------|-----------|-------------|-----------------|
| `cr_risklevel` | Risk Level | Choice (Low/Medium/High) | Deal risk assessment indicator | Proactive risk mitigation in pipeline |
| `cr_dealcategory` | Deal Category | Choice (4 options) | Revenue classification type | Distinguish new vs. expansion revenue |
| `cr_competitor` | Competitor | Text (Single Line) | Primary competitor in the deal | Competitive intelligence tracking |

---

## 4. Relationships & Cardinality

| Parent Entity | Relationship | Child Entity | Type | Cascade |
|--------------|-------------|-------------|------|---------|
| Account | 1 : N | Lead | Lookup | Referential |
| Account | 1 : N | Opportunity | Lookup (Potential Customer) | Referential |
| Account | 1 : N | Contact | Lookup (Parent Account) | Parental |
| Lead | 1 : 0..1 | Opportunity | System-managed (Qualification) | Referential |
| Contact | 1 : N | Opportunity | Lookup | Referential |
| Lead / Opportunity | 1 : N | Activity | Polymorphic (Regarding) | Referential |

---

## 5. Business Process Flow — Stage Mapping

```mermaid
graph LR
    A["🔵 Lead: Qualify<br/>(11 Data Steps)"] --> B["🔵 Lead: Score & Prioritize<br/>(2 Data Steps)"]
    B --> C["🟢 Opportunity: Develop<br/>(4 Data Steps)"]
    C --> D["🟢 Opportunity: Propose<br/>(4 Data Steps)"]
    D --> E["✅ Close"]

    style A fill:#1a73e8,stroke:#1557b0,color:#fff
    style B fill:#1a73e8,stroke:#1557b0,color:#fff
    style C fill:#34a853,stroke:#2d8f47,color:#fff
    style D fill:#34a853,stroke:#2d8f47,color:#fff
    style E fill:#0d652d,stroke:#094d22,color:#fff
```

### Stage Details

| Stage | Entity | Key Data Steps | Purpose |
|-------|--------|---------------|---------|
| **Qualify** | Lead | Contact info, Company, Budget, Timeline, Rating | Initial lead qualification |
| **Score & Prioritize** | Lead | Lead Score, Lead Priority | AI-driven scoring and prioritization |
| **Develop** | Opportunity | Customer Need, Proposed Solution, Stakeholders | Solution development and alignment |
| **Propose** | Opportunity | Proposal, Revenue, Close Date, Risk Assessment | Commercial proposal and negotiation |
| **Close** | Opportunity | Final Decision, Win/Loss Reason | Deal closure and outcome tracking |

---

## 6. Data Volume & Sample Records

| Entity | Sample Records | Production Estimate (Year 1) |
|--------|---------------|------------------------------|
| Account | 25 | 200–500 |
| Lead | 60 | 2,000–5,000 |
| Opportunity | 35 | 500–1,500 |
| Contact | 16 (pre-loaded) | 1,000–3,000 |
| Activity | — (auto-generated) | 10,000–25,000 |

---

## 7. Option Set Definitions

### Lead Source System
| Value | Label | Description |
|-------|-------|-------------|
| 1 | Website | Inbound web form or chat |
| 2 | Referral | Existing customer or partner referral |
| 3 | Cold Call | Outbound sales development |
| 4 | Partner | Channel partner sourced |
| 5 | Trade Show | Event or conference lead |
| 6 | Social Media | LinkedIn, Twitter, or other social |

### Lead Priority
| Value | Label | Score Range |
|-------|-------|-------------|
| 1 | Hot | 80–100 |
| 2 | Warm | 50–79 |
| 3 | Cold | 0–49 |

### Risk Level (Opportunity)
| Value | Label | Criteria |
|-------|-------|----------|
| 1 | Low | Strong champion, clear budget, short cycle |
| 2 | Medium | Some uncertainty in timeline or budget |
| 3 | High | No champion, budget unconfirmed, competitive |

### Deal Category (Opportunity)
| Value | Label | Description |
|-------|-------|-------------|
| 1 | New Business | First-time customer acquisition |
| 2 | Upsell | Expanding existing contract value |
| 3 | Renewal | Contract renewal with existing customer |
| 4 | Cross-sell | Selling additional product/service lines |

---

## 8. Integration Touchpoints

```mermaid
graph TD
    A["Microsoft Dataverse"] --> B["Power BI<br/>Analytics Dashboard"]
    A --> C["Power Automate<br/>Workflow Automation"]
    A --> D["Power Apps<br/>Sales Hub UI"]
    A --> E["Azure Logic Apps<br/>External Integration"]
    C --> F["Outlook/Teams<br/>Notifications"]
    E --> G["External CRM/ERP<br/>Data Sync"]

    style A fill:#742774,stroke:#5a1d5a,color:#fff
    style B fill:#f2c811,stroke:#d4a600,color:#000
    style C fill:#0066ff,stroke:#0052cc,color:#fff
    style D fill:#742774,stroke:#5a1d5a,color:#fff
    style E fill:#0078d4,stroke:#005a9e,color:#fff
    style F fill:#7b83eb,stroke:#5b6abf,color:#fff
    style G fill:#00a4ef,stroke:#0082c8,color:#fff
```

---

> **Note:** This data model document is a living artifact. Column schema prefixes (e.g., `cr_`) will vary based on the solution publisher prefix configured in the SmartOps360 environment.
