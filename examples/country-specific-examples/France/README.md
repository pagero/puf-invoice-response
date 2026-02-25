# France Invoice Response and Document Status Examples

This directory contains example PUF Invoice Response and Document Status files for France's
e-invoicing lifecycle messaging, aligned with the French CDAR (Cross Domain Acknowledgement and Response) status system and the requirements of the French Tax Authority (DGFiP).

## Overview

French e-invoicing requires structured lifecycle messages beyond the basic invoice exchange.
Two types of messages are used:

| Type | Direction | CustomizationID |
|------|-----------|-----------------|
| **Invoice Response** | Buyer → Seller | `urn:pagero.com:puf:invoice_response:1.0` |
| **Document Status** | Seller → Buyer | `urn:pagero.com:puf:document_status:1.0` |

### French CDAR Status Codes

| PUF ResponseCode | CDAR Code | French Name | Who Sends |
|------------------|-----------|-------------|-----------|
| `SUBMITTED` | 200 | Déposée | Platform |
| `ISSUED_BY_PLATFORM` | 201 | Émise | Platform |
| `RECEIVED_BY_PLATFORM` | 202 | Reçue | Platform |
| `MADE_AVAILABLE` | 203 | Mise à disposition | Platform |
| `IP` | 204 | Prise en charge | Buyer |
| `AP` | 205 | Approuvée | Buyer |
| `PARTIALLY_ACCEPTED` | 206 | Approuvée partiellement | Buyer |
| `UQ` | 207 | Suspendue | Buyer |
| `ON_HOLD` | 208 | Mise en attente | Buyer |
| `INFORMATION_PROVIDED` | 209 | Complément d'informations | Buyer |
| `RE` | 210 | Refusée | Buyer |
| `PAYMENT_INITIATED` | 211 | Paiement Transmis | Buyer |
| `PAYMENT_RECEIVED` | 212 | Encaissée | Seller |
| `VF` | 213 | Rejetée | Platform |
| `225` | 225 | Affacturée | Seller |
| `226` | 226 | Affacturée Confidentiel | Seller/Factor |
| `227` | 227 | Changement de Compte à payer | Seller/Factor |
| `228` | 228 | Non Affacturée | Seller |

> **Note on codes 225–228:** These French-specific factoring codes are currently outside the
> standard PUF-016 code list and are used as raw numeric strings. Dedicated PUF response codes
> for these statuses are **under consideration/development** and may replace the numeric values
> in a future release of the PUF Invoice Response specification.

> **Mandatory Tax Authority reporting:** The statuses Déposée (200), Rejetée (213),
> Refusée (210) and **Encaissée (212)** must be reported to PPF. Encaissée is mandatory
> whenever VAT is due at collection (all service invoices S1/S2/S4/S5/S6/S7 and certain
> goods invoices where payment was already received).

---

## Example Files

### Generic Examples

These examples use the parties and invoice from
[PUF_France_Generic_Invoice.xml](../../puf-billing/examples/country-specific-examples/france/PUF_France_Generic_Invoice.xml)
(invoice `FR-INV-2026-001`, TechDistrib France SARL → Entreprise Client France SA, 1,200.00 EUR TTC @ 20%).

#### 1. `PUF_France_Generic_PaymentReceived.xml`

**Encaissée — Document Status (Seller → Buyer)**

Demonstrates the standard `PAYMENT_RECEIVED` (212) lifecycle message with:
- `puf:StatusExtension` containing a `MEN` amount (mandatory for French Tax Authority)
- Amount breakdown by VAT rate: 1,200.00 EUR TTC @ 20%
- `puf:ResponseExtension` correctly wrapping `puf:DocumentMatchingID`

---

#### 2. `PUF_France_Generic_Refused.xml`

**Refusée — Invoice Response (Buyer → Seller)**

Demonstrates the `RE` (210) refusal with:
- French reason code `CALCUL_ERR` (calculation error) — one of the 22 French-specific reason codes
- Action code `NIN` (OPStatusAction — issue a corrected invoice)
- Two `cac:Status` blocks: one for reason (`listID=OPStatusReason`), one for action (`listID=OPStatusAction`)

---

#### 3. `PUF_France_Generic_PaymentInitiated.xml`

**Paiement Transmis — Invoice Response (Buyer → Seller)**

Demonstrates `PAYMENT_INITIATED` (211) with:
- `puf:StatusExtension` containing a `MPA` amount (amount paid by buyer)
- Amount: 1,200.00 EUR TTC @ 20%

---

### Use Case Examples

#### UC2: Already Paid Invoice

##### 4. `PUF_France_UC2_PaymentReceived.xml`

**Encaissée — Document Status (Seller → Buyer)**
Reference: `PUF_France_UC2_AlreadyPaid_Invoice.xml` — invoice `INV-FR-2026-0235`

Demonstrates concurrent Encaissée (sent on the **same date as the invoice**) for an invoice
that was already paid before emission. MEN = full invoice TTC (2,400.00 EUR).

---

#### UC4: Partial Third-Party Payment (Insurance)

##### 5. `PUF_France_UC4_PaymentReceived_Buyer.xml`

**Encaissée (buyer portion) — Document Status (Seller → Buyer)**
Reference: `PUF_France_UC4_PartialThirdPartyPayment_Invoice.xml` — invoice `REP-2026-0845`

First of two Encaissée messages for a split-payment invoice (total 3,600.00 EUR):
- MEN = **1,100.00 EUR TTC @ 20%** — buyer's deductible portion

##### 6. `PUF_France_UC4_PaymentReceived_Insurer.xml`

**Encaissée (insurer portion) — Document Status (Seller → Buyer)**
Reference: `PUF_France_UC4_PartialThirdPartyPayment_Invoice.xml` — invoice `REP-2026-0845`

Second Encaissée for the same invoice:
- MEN = **2,500.00 EUR TTC @ 0%** — insurer's payment (insurance indemnity, not subject to VAT)

> Two separate Encaissée messages are required when payment is split between multiple payers
> with different VAT treatments.

---

#### UC10: Post-Factoring

##### 7. `PUF_France_UC10_Affacturee.xml`

**Affacturée — Document Status (Seller → Buyer)**
Reference: `PUF_France_UC10_PostFactoring_Invoice.xml` — invoice `UC10-IND-2026-0345`

Demonstrates the French-specific factoring notification code `225` (Affacturée):
- Seller notifies buyer that the invoice has been assigned (cédée) to a factor
- Factor's bank details communicated via `cbc:Note`
- Buyer must redirect payment to the factor

> Response code `225` is outside the standard PUF-016 code list and is specific to French CDAR.
> Related codes: 226 (Affacturée Confidentiel), 227 (Changement de Compte), 228 (Non Affacturée).
> Dedicated PUF response codes for these statuses are **under consideration/development** and may
> replace the numeric values in a future release of the PUF Invoice Response specification.

---

#### UC19b: Self-Billing

##### 8. `PUF_France_UC19b_PaymentReceived.xml`

**Encaissée — Document Status (Seller → Buyer)**
Reference: `PUF_France_UC19b_SelfBilling_Invoice.xml` — invoice `SMBF-AF-2026-01234`

Demonstrates Encaissée for a self-billing scenario:
- The invoice was **created by the buyer** on behalf of the seller (auto-facturation)
- Despite the reversed emission roles, the **seller (Ferme Biologique Rousseau) still sends
  the Encaissée** — payment direction is NOT reversed in self-billing
- MEN = 1,641.60 EUR TTC @ 20%

---

#### UC40: Netting / Compensation

##### 9. `PUF_France_UC40_PaymentReceived.xml`

**Encaissée — Document Status (Seller → Buyer)**
Reference: `PUF_France_UC40_Netting_Invoice2_FarmerToCoop.xml` — invoice `EARL-2026-0089`

Demonstrates Encaissée when payment was settled via netting/compensation:
- Farmer's invoice: 5,280.00 EUR TTC (grapes sold to cooperative)
- Cooperative's invoice: 2,400.00 EUR TTC (services provided to farmer)
- Net cash settlement: 2,880.00 EUR (cooperative pays farmer on 2026-02-15)
- **MEN = full invoice TTC (5,280.00 EUR), NOT the net settlement (2,880.00 EUR)**

> Per French Tax Authority requirements, MEN must report the **full amount of the taxable
> transaction** — the invoice was collected in full via legal set-off (compensation).
> The net payment amount must not be used for MEN.

---

## Key Structural Requirements

### DocumentMatchingID

Always wrap `puf:DocumentMatchingID` inside `puf:ResponseExtension`:

```xml
<puf:PageroExtension>
    <puf:ResponseExtension>
        <puf:DocumentMatchingID>
            <cbc:ID>...</cbc:ID>
        </puf:DocumentMatchingID>
    </puf:ResponseExtension>
</puf:PageroExtension>
```

### Encaissée (PAYMENT_RECEIVED) Amount Requirements

The `puf:StatusExtension` with `MEN` amounts is **mandatory** for Encaissée. Amounts must be
broken down by VAT rate:

```xml
<cac:Status>
    <ext:UBLExtensions>
        <ext:UBLExtension>
            <ext:ExtensionURI>urn:pagero:ExtensionComponent:1.0:PageroExtension:StatusExtension</ext:ExtensionURI>
            <ext:ExtensionContent>
                <puf:PageroExtension>
                    <puf:StatusExtension>
                        <puf:Amounts>
                            <puf:Amount>
                                <puf:AmountType>MEN</puf:AmountType>
                                <cbc:TaxExclusiveAmount currencyID="EUR">1000.00</cbc:TaxExclusiveAmount>
                                <cbc:TaxInclusiveAmount currencyID="EUR">1200.00</cbc:TaxInclusiveAmount>
                                <cac:ClassifiedTaxCategory>
                                    <cbc:Percent>20</cbc:Percent>
                                </cac:ClassifiedTaxCategory>
                            </puf:Amount>
                        </puf:Amounts>
                    </puf:StatusExtension>
                </puf:PageroExtension>
            </ext:ExtensionContent>
        </ext:UBLExtension>
    </ext:UBLExtensions>
    <cbc:StatusReasonCode>NON</cbc:StatusReasonCode>
</cac:Status>
```

### French Reason Codes (Refusée / RE)

Reason codes must use one of the 22 French-specific codes defined in the specification.
Two `cac:Status` blocks are expected for a refusal: one for the reason, one for the action:

```xml
<cac:Status>
    <cbc:StatusReasonCode listID="OPStatusReason">CALCUL_ERR</cbc:StatusReasonCode>
    <cbc:StatusReason>Human-readable description</cbc:StatusReason>
</cac:Status>
<cac:Status>
    <cbc:StatusReasonCode listID="OPStatusAction">NIN</cbc:StatusReasonCode>
    <cbc:StatusReason>Action description</cbc:StatusReason>
</cac:Status>
```

See [Status reason section](https://pagero.github.io/puf-invoice-response/index.html#_status_reason) for the full list of French reason codes
and their applicability per response code.

---

## Related Files

- [PUF Invoice Response - France](https://pagero.github.io/puf-invoice-response/#_france) — France country specifics (reason codes, amount types)
- [PUF Billing France examples](https://github.com/pagero/puf-billing/tree/master/examples/country-specific-examples/france) — reference invoices
- Best Practice Content France Lifecycle - Contact Pagero for more information on best practices for lifecycle messaging in France.
