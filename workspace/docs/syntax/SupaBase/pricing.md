# Pricing

> Project contract: Browser -> Next.js -> Express on Node.js -> Supabase.
> SDKs, credentials, authorization and provider integration live in Express only.

> Source review: 2026-09-27. This is a template reference, not a running project.
> Historical execution/measurement statements below are author reports without a supplied
> reproduction harness; they are not acceptance evidence. This review ran no Supabase
> CLI, SQL, deployed function, or real provider integration.

Billing is **organization-based**: plan covers platform features and quotas; compute is billed separately per project (hourly). **Pro and Team include $10/month compute credits** (covers one Micro instance). Free includes no credits.

Source: https://supabase.com/pricing (source-checked 2026-09-27; recheck at implementation)

## Plans

| | Free | Pro | Team | Enterprise |
|---|------|-----|------|------------|
| **Price** | $0 (2 projects max) | $25/mo | $599/mo | Custom |
| **Database** | 500 MB | 8 GB, then $0.125/GB | 8 GB+ | Custom |
| **File Storage** | 1 GB | 100 GB | 100 GB | Custom |
| **Egress** | 5 GB + separate 5 GB cached | 250 GB + separate 250 GB cached | Same as Pro | Custom |
| **MAU (Auth)** | 50,000 | 100,000, then $0.00325/MAU | Same as Pro | Custom |
| **Edge Functions** | 500K invocations | 2M, then $2/M | Same as Pro | Custom |
| **Realtime connections** | 200 concurrent | 500, then $10/1000 | Same as Pro | Custom |
| **Realtime messages** | 2M/mo (256 KB max msg) | 5M/mo (3 MB max msg) | Same as Pro | Custom |
| **Backups** | None | Daily, 7-day retention | Daily, 14-day retention | Custom |
| **PITR** | No | **$100/mo add-on** per 7 days | Same add-on | Same add-on |
| **Branching** | No | **Yes** ($0.01344/branch/hr) | Yes | Custom |
| **Image Transforms** | No | 100 origin images incl., then $5/1000 | Same as Pro | Custom |
| **Global file-size configuration ceiling** | 50 MB | 500 GB | 500 GB | Custom |

**Pro spend caps cover selected usage categories; verify the organization's setting.** They are not
an invoice ceiling: compute, additional projects and add-ons can still increase charges.
Source: https://supabase.com/docs/guides/platform/cost-control

## Compute Add-Ons (per project)

| Size | $/month | RAM | Notes |
|------|---------|-----|-------|
| Micro | $10 | 1 GB | Included via credits on Pro/Team |
| Small | $15 | 2 GB | |
| Medium | $60 | 4 GB | Common production choice |
| Large | $110 | 8 GB | Dedicated CPU starts here |

## Real-World Cost Expectations

- **Pro + one project**: $25 base + Micro compute − $10 credit = **$25/month baseline**, excluding add-ons, uncovered usage and taxes.
- **Two projects on Pro**: $25 + $10 + $10 − $10 = **$35/month**.
- **Overage**: distinguish capped usage categories from compute and other uncapped charges.

## Where Costs Creep Up

1. **Extra projects**: each adds full compute cost; credits cover only one.
2. **Storage bandwidth**: public buckets + no CDN caching = repeated egress.
3. **Edge Function invocations**: 2M included, then $2 per million.
4. **Database size**: text-heavy tables grow fast; disk overage is $0.125/GB.
5. **Add-ons stack**: PITR $100/mo, custom domain $10/mo per project, phone MFA and other add-ons: verify current feature-specific charges.

## Free Tier Limitations

- Pauses after 1 week of inactivity (unpause manually).
- Max 2 active projects.
- No backups/PITR, no image transformations, no branching.
- Basic CDN only (paid tiers get Smart CDN).
- 200 realtime concurrent connections, 256 KB max message size.
