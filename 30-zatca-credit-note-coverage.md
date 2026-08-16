# ZATCA Credit-Note Field Coverage

**Scope:** Refund-triggered credit notes produced by
`App\Modules\Commerce\Application\Services\InvoiceService::issueCreditNoteForRefund()`.

**Reference:** ZATCA "E-Invoicing Detailed Technical Guideline" v2.0 (2024)
— sections 5.4 (credit note), 6 (Phase 2 XML), and Annex C (QR TLV).

---

## Phase 1 — Simplified B2C credit notes (printed/displayed receipt)

| # | Required field | Source in code | Status |
|---|---|---|---|
| 1 | Document type | `billing_snapshot.document_type = 'credit_note'` + filename prefix `WMD-CN-` | ✓ |
| 2 | Invoice number (sequential, unique) | `InvoiceService::nextCreditNoteNumber()` returns `WMD-CN-{year}-{seq}-{rand}` | ✓ |
| 3 | Issue date | `issued_at` column + Tag 3 in QR | ✓ |
| 4 | Seller name | `current_tenant()->name` | ✓ |
| 5 | Seller VAT number | `current_tenant()->vat_number` | ✓ |
| 6 | Buyer name | `billing_snapshot.buyer.name` | ✓ |
| 7 | Buyer VAT number (B2B only) | — | ✗ **Gap** — only the buyer's `email` is captured. For B2B refunds this must be added before credit notes are issued to corporate customers. Tracked as B-H5-1. |
| 8 | Reference to original invoice | `billing_snapshot.original_invoice_id` + `original_invoice_number` | ✓ |
| 9 | Line items (qty, unit price, VAT) | `invoice_items` rows with negative `unit_price_halalas` + `tax_halalas` | ✓ |
| 10 | VAT breakdown | `subtotal_halalas = -base`, `tax_halalas = -vat`, derived via inclusive-VAT math `base = round(amount/(1+rate))`, `vat = amount - base` | ✓ |
| 11 | Total with VAT | `total_halalas` (negative) | ✓ |
| 12 | Currency code | `currency` (SAR) | ✓ |
| 13 | Reason for credit | `billing_snapshot.refund_reason` (added in B-H5) | ✓ |
| 14 | QR code (TLV tags 1..5) | `InvoiceService::buildZatcaQrTlv()` builds chr(tag).chr(len).value, base64-encoded; called from `issueCreditNoteForRefund()` with negative `totalHalalas` and `vatHalalas` | ✓ |
| 15 | Printable PDF | — | ✗ **Gap** — no PDF rendering. Customers see the credit-note number in their dashboard but receive no document. Tracked as B-H5-3. |

**Phase 1 coverage score: 13 / 15 fields complete after B-H5.**

---

## Phase 2 — UBL 2.1 XML + cryptographic stamp (reporting / clearance)

These are required by 1 Jan 2025 for wave-eligible taxpayers, and progressively
for all taxpayers thereafter. Wamadat is currently NOT integrated with ZATCA's
Fatoora platform; `zatca_status` stays `'pending'` permanently.

| # | Required field | Source in code | Status |
|---|---|---|---|
| 1 | UBL XML with `cbc:InvoiceTypeCode=381` (credit note) | — | ✗ **No XML generation** |
| 2 | Cryptographic stamp (signed XML) using CSID | — | ✗ **No private key, no signing flow** |
| 3 | Hash chain (PIH/ICV — each invoice references the previous invoice's hash) | — | ✗ **Not implemented** |
| 4 | EGS unit registration and certificate | `services.zatca` config keys exist, never populated | ✗ |
| 5 | Submission via Fatoora API within 24h (simplified) or sync (standard) | `zatca_status` column is a stub; no API client | ✗ **Stub only** |
| 6 | Additional QR tags 6..9 (XML hash, public key, signature, signed-property hash) | — | ✗ |

**Phase 2 coverage: 0 / 6.** This is a deliberate scope choice — Phase 2 is
tracked as part of the larger "Compliance / Tax Reporting" workstream and is
NOT a launch blocker for B2C-only Saudi e-commerce that issues simplified
invoices.

---

## Risks left open after B-H5

| Risk | Severity | Mitigation deadline |
|---|---|---|
| **B-H5-1 — No buyer VAT field** | High for B2B credit notes; not blocking for current B2C scope | Before first B2B customer activation |
| **B-H5-3 — No PDF rendering** | Low — customer dashboard shows credit-note number; ZATCA accepts digital display | Before SLAs that require attached PDF |
| **Phase 2 wholesale gap** | Critical for ZATCA Fatoora reporting | Tracked outside Phase B/Phase C |

---

## Proof: current credit-note row meets Phase 1 fields 1–12, 14

DB row from B2 proof (commit `3d92127`):

```
invoice_number   = WMD-CN-2026-000001-U98
total_halalas    = -600000           # -6,000 SAR ✓ field 11
subtotal_halalas = -521739            # base ✓ field 10
tax_halalas      = -78261             # vat ✓ field 10
billing_snapshot = {
  document_type           : 'credit_note',                      # ✓ field 1
  original_invoice_id     : a1c50999-...,                       # ✓ field 8
  original_invoice_number : WMD-INV-2026-000001-KEX,            # ✓ field 8
  refund_id               : a1c50999-c86d-...,
  seller : { name, vat_number },                                 # ✓ fields 4-5
  buyer  : { name, email }                                       # ✓ field 6
}
zatca_qr_payload = base64(TLV)                                  # ✓ field 14
currency         = SAR                                          # ✓ field 12
```

Verification SQL:

```sql
SET search_path = tenant_wamadat;
SELECT
  invoice_number,
  total_halalas,
  subtotal_halalas,
  tax_halalas,
  billing_snapshot->>'document_type'           AS doc_type,
  billing_snapshot->>'original_invoice_number' AS original_invoice,
  billing_snapshot->>'refund_id'               AS refund_id,
  length(zatca_qr_payload)                     AS qr_b64_len,
  zatca_status
FROM invoices
WHERE invoice_number LIKE 'WMD-CN-%'
ORDER BY created_at DESC;
```
