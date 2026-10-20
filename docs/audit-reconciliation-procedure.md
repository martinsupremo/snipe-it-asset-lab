# Audit & Reconciliation Procedure (SOP)

Purpose: prove that physical stock and Snipe-IT records agree — and fix
them, on the record, when they don't. Mirrors warehouse-style inventory
audits: scan everything, explain every difference, sign off.

## 1. Choose the audit mode

| Mode | Scope | When | Disrupts operations? |
|---|---|---|---|
| Cycle count | One location *or* one category | Weekly/biweekly rotation — every location covered each quarter | No |
| Full audit | Every asset, every location | Yearly, after a major move/refresh, or when cycle counts show a pattern of errors | Plan a low-activity window |

Rule of thumb: cycle counts keep you honest; a full audit resets the
baseline. If cycle counts keep finding the same kind of error, stop and
fix the process — don't just audit harder.

## 2. Prepare

- [ ] Freeze non-essential moves for the audit window: no checkouts or
      location changes for in-scope assets (emergency swaps are logged
      and excluded, noted on the report).
- [ ] Print/prepare the expected-assets list for the scope (from
      Snipe-IT: assets by location/category) — this is the "system says"
      side of the comparison.
- [ ] Charge the handheld scanners; verify each scanner is paired and
      acts as a keyboard wedge (scan a test tag into a text field).
- [ ] Label printer available with spare labels — you'll find unlabeled
      or mislabeled gear; that's expected.

## 3. Scan-based verification

1. Walk the location systematically (left to right, aisle by aisle —
   same route every time so nothing is skipped or double-scanned).
2. Scan **every** asset tag you physically touch into Snipe-IT's audit
   screen (or a scan list you import the same day). No tag? Write the
   serial and model on the found-items list.
3. Checked-out assets: verify with the holder (scan at their desk /
   confirm by serial + tag photo for remote holders).
4. Mark each verified asset's `Last Audit Date` in Snipe-IT as you go.

## 4. Discrepancies — the four types and their resolutions

### Type 1: Missing (system says present, not found)
1. Re-check the location once (behind/below/inside adjacent cabinets —
   most "missing" gear is mis-shelved, not gone).
2. Check recent history: last checkout, last audit, last location change.
3. Contact the last holder; give a short, stated deadline (e.g. 3
   business days) to produce or account for it.
4. Unresolved → status `Lost/Stolen`, note with dates and who was
   contacted. Escalate per site policy. **Never delete the record.**

### Type 2: Found but unrecorded (physical device, no system record)
1. Record serial + model + physical condition.
2. Search Snipe-IT by serial (it may exist under a wrong tag/location).
3. Truly new → full intake: create the asset, print a tag, set location.
   Note "found in audit <date>" in the record.
4. A pattern of these = an intake leak (gear deployed without being
   logged). Flag it in the report; fix the intake step, not just the data.

### Type 3: Wrong location (found, but record says elsewhere)
1. Correct the location to where it physically is.
2. Check the history: was it moved without a checkout/location update?
3. Remind the mover of the rule — assets move **with** a record, or they
   don't move. Repeated offenders/locations get named in the report
   (factually, not punitively) so the pattern is visible.

### Type 4: Duplicate tag (two records — or two labels — same tag)
1. Physically identify both devices by serial number; the serial is the
   tie-breaker, the tag is not.
2. Keep the record whose history matches the physical device; correct or
   retire the other record (merge notes into the survivor — never lose
   history silently).
3. Reprint/relabel as needed so label = record, and re-scan to verify.

## 5. Reconciliation report checklist

The audit isn't done until this exists and is filed:

- [ ] Scope audited (locations/categories) and audit mode (cycle/full)
- [ ] Date(s) and who performed the count
- [ ] Total expected vs total scanned
- [ ] Discrepancy list by type (missing / found / wrong location /
      duplicate), each with its resolution or its open status + owner
- [ ] All record corrections made in Snipe-IT (or ticketed, with IDs)
- [ ] Known losses (`Lost/Stolen`) confirmed separately from new misses
- [ ] Process findings (intake leaks, move-without-record patterns)
- [ ] Next cycle count scheduled

## 6. Physical-to-system sign-off

Sign-off means a named person attests the count was actually walked, not
just that the spreadsheet balances:

```
Location/scope: ______________________   Audit date: ____________
Counted by (name/signature): ______________________
Reconciled by (name/signature): ______________________
Discrepancies open at sign-off: ____ (list attached)
Next scheduled count: ____________
```

File the signed report with the audit records. An audit without a
sign-off is a rumor; an audit with one is evidence the next audit can
build on.
