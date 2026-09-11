# rizz-sdui-configs

Source-of-truth backup of the **Horus SDUI configs** behind Proactive Outbound
Calling (a.k.a. Campaign Calling). Each file is the JSON exactly as it lives in
the Pulse config store — pages, their tab-content configs, and their form configs.

Use it as the **porting checklist** when standing the product up on a new
environment, and as a version-controlled record of the live config.

> **Naming:** on **stage** everything is prefixed `rizz-*` (pageIds, form ids,
> permissions). On **prod** the product is **Campaign Calling** and the roles are
> **Campaign Manager** / **Campaign Agent**. The `rizz-calling:*` permission
> strings below are the *stage* strings.
>
> **Two backends:** `client: rizz-calling` → our service (`/pulse-service/api/v1/data/rizz-calling/*`,
> `/forms/*/submit`, `/page/*`). Everything on `/tickets`, `/routing/shift` and
> the `provider:` data (ticket, alphaDeskUser, userVaultDevice, rewards-v2,
> app-server) is **Pulse-native**.

## Layout

```
.
├── README.md
├── docs/
│   └── WORKFLOW.md                     # the end-to-end flow diagram
├── campaign-manager/                   # pageId: rizz-campaign-manager
│   ├── page.json
│   ├── tabs/
│   │   ├── campaigns.json               # rizz-campaigns-tab
│   │   ├── dispatch-history.json        # rizz-dispatch-tab
│   │   ├── audit-log.json               # rizz-audit-tab
│   │   └── pool-queries.json            # rizz-queries-tab
│   └── forms/
│       ├── upload-leads.json            # rizz-upload-leads-form
│       ├── campaign-config.json         # rizz-campaign-config-form
│       └── pool-queries.json            # rizz-pool-queries-form
├── calling-queue/                      # pageId: rizz-calling-queue
│   ├── page.json
│   └── forms/
│       └── cancel-remaining.json        # rizz-cancel-remaining-form
└── ticket-details/                     # pageId: rizz-calling-ticket-details
    ├── page.json
    ├── tabs/
    │   ├── lead-context.json            # tabId: lead_context
    │   ├── customer-profile.json        # tabId: customer_profile
    │   └── ticket-details.json          # tabId: ticket_details
    └── forms/
        ├── call-answered.json           # rizz-call-answered-form
        ├── call-notanswered.json        # rizz-call-notanswered-form
        └── call-callback.json           # rizz-call-callback-form
```

## The three pages

| Page | pageId | Who | Tabs | Forms |
|------|--------|-----|------|-------|
| **Campaign Manager** | `rizz-campaign-manager` | Campaign Manager | Campaigns · Dispatch History · Audit Log · Pool Queries | Upload Leads · Configure · Pool Queries |
| **Campaign Calling Queue** | `rizz-calling-queue` | Manager + Agents | *(none — it's a table)* | Cancel Remaining |
| **Ticket Detail** | `rizz-calling-ticket-details` | Campaign Agent | Lead Context · Customer · Ticket Details | Answered · Not Answered · Callback Today |

See [`docs/WORKFLOW.md`](docs/WORKFLOW.md) for how they connect end-to-end.
