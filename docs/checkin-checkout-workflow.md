# Check-in / Check-out Workflow (SOP)

Applies to: all tracked IT assets in Snipe-IT (laptops, peripherals,
barcode scanners, label printers).
Goal: every asset's location and holder is known, with a paper trail,
at all times.

## A. Checkout

Use for: new deployments, replacements, loaners, scanner/printer assignment
to a warehouse area.

1. **Verify ready state.** Asset status must be `Ready to Deploy`.
   Never check out an asset that is `Pending`, `Out of Repair`, or
   missing from the shelf — resolve the discrepancy first (see
   `audit-reconciliation-procedure.md`).
2. **Assign.** In Snipe-IT → asset → Check Out:
   - To a **user** (laptops, phones): the named individual, always.
     Never "the department" or a free-text name.
   - To a **location** (scanners, label printers): the sub-location
     (e.g. `Warehouse - Shipping`), and update the asset's location field
     to match the physical place it will live.
3. **Expected checkin date.** Required. Default: permanent assignment →
   the next scheduled audit date; loaner → actual return date (start
   with 14 days).
4. **Acceptance.** The receiver confirms receipt in one of two ways:
   - Email acceptance (Snipe-IT's "require acceptance" + EULA flow), or
   - For walk-ups: receiver signs the printed checkout receipt at handoff.
   No acceptance, no handoff. The acceptance is the proof of custody.
5. **Physical prep at handoff.** Device imaged/configured for the role,
   charger/accessories noted in the checkout note, asset tag label
   intact (relabel if damaged — see architecture notes on tagging).

## B. Check-in

Use for: returns (offboarding, refresh, repair intake, loaner return,
scanner/printer swaps).

1. **Receive & inspect.** Power on. Record condition: physical damage,
   missing parts (chargers, cables, printer heads), obvious tampering.
2. **Check in** in Snipe-IT with:
   - Actual checkin date (today).
   - Condition note written plainly ("hinge cracked, charger missing").
   - New location = where it now physically is (IT Closet, Repair Bench).
3. **Set status correctly (do not skip):**
   - Clean, reusable as-is → `Ready to Deploy`.
   - Needs data wipe, repair, or inspection → `Pending`.
   - Vendor repair / bench work → `Out of Repair`.
4. **Data-wipe gate (returned laptops and any storage-bearing device).**
   - A returned laptop is **never** reissued while status is `Pending`.
   - Run the documented wipe procedure, set custom field
     `Data wipe verified = Yes` + date, then reimage. Only then may
     status move to `Ready to Deploy`.
   - If wipe can't be verified → stays `Pending`/quarantined; it feeds
     the decommissioning flow, not the redeploy pool.
5. **Accessories.** Returning chargers/cables are scanned back with the
   device (as accessories/components in Snipe-IT). Missing accessories
   go in the checkin note and on the user's record.

## C. Overdue assets

1. Run the overdue-assets view/report weekly (daily in a busy warehouse).
2. First contact: friendly reminder to the holder (email or in person) —
   "system shows X due on <date>, confirm you still have it / need it?"
3. No response in 3 business days → notify the holder's supervisor and
   re-set the expected date only if there's a legitimate extension;
   don't silently extend forever.
4. 30+ days unresolved → escalate to IT management; treat as potential
   loss and start the lost/damaged path below while follow-up continues.

## D. Lost / damaged

**Lost or stolen**
1. Confirm via holder + last known location + last checkout record.
2. Change status to `Lost/Stolen` (archived) with a note: reported by,
   date, circumstances. Do not delete the asset — the record is the proof.
3. If it holds data: notify whoever owns incident handling per your
   site's policy (remote wipe/lock if the tooling exists).
4. Device is flagged in the next audit report as a known-loss, so it
   isn't double-counted as a fresh discrepancy.

**Damaged**
1. Check in (or log against current holder), attach photos if possible.
2. Status: repairable → `Out of Repair` + repair ticket/work order
   reference in the note; not repairable → `Broken - Not Fixable`.
3. Warranty expiration (custom field) drives the call: in warranty →
   vendor RMA; out of warranty → repair vs replace decision documented
   in the asset notes.
4. Broken/not fixable assets move to the decommissioning/disposal flow;
   storage media goes through the data-wipe gate first, always.
