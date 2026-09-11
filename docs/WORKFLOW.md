# How it works — end-to-end flow

One picture of how the three pages, their tabs and forms drive the whole
Proactive Outbound Calling loop. `client: rizz-calling` calls hit our service;
everything on `/tickets`, `/routing/shift` and the `provider:` data is Pulse-native.

```mermaid
flowchart TD
    subgraph MGR["Campaign Manager · campaign-manager/page.json"]
        CFG["Configure form<br/>update-campaign-config"]
        PQ["Pool Query form<br/>update-combined-query"]
        DBX["Run SQL in Databricks<br/>→ combined CSV"]
        UP["Upload Leads form<br/>ingest-leads (pool only)"]
        DISP["Dispatch Leads<br/>dispatch-leads?lob=REMITTANCE"]
        CFG --> PQ --> DBX --> UP --> DISP
    end

    DISP -->|"rizz.lead_dispatch (Kafka)"| PULSE["Pulse creates tickets<br/>in category"]

    PULSE --> QUEUE["Campaign Calling Queue<br/>calling-queue/page.json"]
    QUEUE -->|"Assign to Me → /claim"| DETAIL["Ticket Detail<br/>ticket-details/page.json"]
    DETAIL -->|"Start Call (action START_CALL)"| ONCALL{"On call —<br/>outcome?"}

    ONCALL -->|Answered| ANS["Answered form<br/>RESOLVE"]
    ONCALL -->|No pickup| NA["Not Answered form<br/>FAIL"]
    ONCALL -->|Callback later today| CBT["Callback Today form<br/>HOLD_SAME_DAY_CALLBACK"]
    ONCALL -->|Callback later date| ANSCB["Answered form<br/>+ callback slot"]

    ANS -->|"submit-feedback"| RIZZ["rizz records disposition"]
    NA -->|"submit-feedback"| RIZZ
    CBT -->|"held with agent"| DETAIL
    ANSCB -->|"submit-feedback"| RIZZ

    RIZZ -->|"pulse.ticket_closed (Kafka)"| CLOSE["Ticket closed"]
    RIZZ -->|"retry / later-date callback due"| REDISP["rizz re-dispatches<br/>fresh ticket"]
    REDISP -->|"rizz.lead_dispatch"| PULSE

    DISP -.->|"Cancel remaining (cancel-remaining)"| RELEASE["Open tickets cancelled,<br/>leads released to rizz"]
    QUEUE -.-> RELEASE
```

**In words:** the Campaign Manager sets up the campaign and pool query → runs the
SQL in Databricks → uploads one CSV → **Dispatch** hands leads to Pulse over Kafka,
which materialises tickets. Agents work the **Queue**, open a **Ticket Detail**,
call, and file a disposition — which flows back to rizz and either closes the
ticket or schedules a retry / callback that re-dispatches. **Cancel remaining**
clears the open pool at any time.
