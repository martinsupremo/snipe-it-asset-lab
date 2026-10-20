# Architecture & Design Decisions

## Topology

- **App:** Snipe-IT (PHP/Laravel) in Docker, published on host port 8080 → container 80.
- **Database:** MariaDB 11 in a second container, reachable only from the app
  container (no host port published). Named volume `db_data` persists it.
- **Clients:** IT admin via browser; warehouse staff via handheld scanners that
  act as keyboard wedges (see barcode workflow below); a label printer for
  asset tags.
- **State that matters:** the MariaDB volume plus `storage/app` (uploaded files,
  label templates, backups). Back both up together.

## Why Snipe-IT

- Purpose-built for IT assets: check-in/check-out, assignment history, license
  seats, and audit logging are native — not spreadsheet workarounds.
- Free and open source, self-hostable in Docker, with a CSV importer that matches
  how inventory data actually arrives (vendor packing lists, old spreadsheets).
- Barcode/QR support and an audit trail answer the two questions inventory work
  always asks: "where is it?" and "who had it last?"

## Asset tag scheme

Format: `<PREFIX>-<NNNN>`, sequential within a prefix, printed on the physical label.

| Prefix | Meaning | Example |
|---|---|---|
| `IT-1xxx` | End-user computers (laptops/desktops) | `IT-1001` |
| `IT-2xxx` | Peripherals (docks, monitors) | `IT-2001` |
| `WH-3xxx` | Warehouse devices (handheld scanners) | `WH-3001` |
| `WH-4xxx` | Warehouse infrastructure (label printers) | `WH-4001` |

Rules: a tag is issued once and never reused, even after retirement — history
stays unambiguous. The tag is the barcode content; the serial number is a
separate field (vendor serials collide and change, tags don't).

## Categories, models, status labels

- **Categories** group assets by kind: Laptops, Desktops, Peripherals,
  Barcode Scanners, Label Printers, Networking, Mobile Devices.
  Category drives which custom fields and checkout rules apply (e.g. laptops
  require an assignee acceptance; scanners check out to a location/shift).
- **Models** live under categories (e.g. category *Laptops* → model
  "Business Laptop 14in"). Purchase cost/date defaults hang off the model;
  per-asset values override.
- **Status labels** are the lifecycle states, kept deliberately small:
  - `Ready to Deploy` (deployable) — in stock, imaged, available
  - `Deployed` (deployable, in use) — checked out to a user or location
  - `Pending` (non-deployable) — awaiting imaging, repair, or data wipe
  - `Out of Repair` (non-deployable) — with vendor / on the bench
  - `Broken - Not Fixable` (archived)
  - `Lost/Stolen` (archived)
  - `Retired` (archived) — end of life, disposed or recycled

  Anything not deployable must not show up as available stock — that's how
  "ghost inventory" starts.

## Custom fields

Added on top of Snipe-IT's built-ins (serial, purchase date/cost, supplier, warranty):

| Field | Applies to | Why |
|---|---|---|
| IMEI | Mobile devices, cellular scanners | Carrier/activation and loss reporting |
| MAC address (Wi-Fi) | Laptops, scanners, printers | Network troubleshooting, DHCP reservations, NAC |
| MAC address (Ethernet) | Docks, desktops, printers | Same, wired side |
| Purchase order # | All | Ties asset back to procurement paperwork |
| Warranty expiration | All | Drives repair-vs-replace decisions |
| Data wipe verified (Y/N + date) | Returned storage devices | Gate before redeploy or disposal |

## Locations: office vs warehouse

Two top-level locations with sub-locations, because "which building" and
"which room/aisle" are different questions:

```
Office - HQ
├── Office - IT Closet        (spares, imaging bench)
└── Office - Dock Office      (warehouse lead's desk)
Warehouse
├── Warehouse - Receiving
├── Warehouse - Aisle 1
├── Warehouse - Shipping
└── Warehouse - Repair Bench
```

Every asset has exactly one location at all times. An asset checked out to a
person still carries the location it physically sits in — user assignment and
location are separate axes in Snipe-IT, and confusing them is the #1 cause of
"the system says it's here but it isn't."

## Barcode label workflow (Zebra-style printers + scanners)

1. **Print at intake.** When an asset is created, print its tag label from
   Snipe-IT (Settings → Labels) to the label printer: asset tag as barcode,
   plus human-readable tag, model, and serial. Affix before the device leaves
   the bench — an unlabeled device is an untracked device.
2. **Label stock:** direct-thermal or thermal-transfer labels sized to the
   device; use durable (synthetic) stock for warehouse gear that gets handled
   and bumped all shift.
3. **Scanner as keyboard wedge.** Handheld scanners (USB or Bluetooth) present
   as HID keyboards: a scan types the tag + Enter into whatever field has
   focus. No special integration needed — in Snipe-IT you click into the asset
   lookup/audit field and scan. This is also the failure mode to know: if a
   scanner "stops working," check that it's paired/charged and that focus is in
   the right field before suspecting the app.
4. **Verify after printing.** Scan every new label once against its record
   before deployment; a misprinted or duplicate tag caught at the bench is
   free, caught in the field it costs an audit discrepancy.

## Hardening notes

- `APP_DEBUG=false` in production; debug pages leak stack traces and config.
- App key and DB credentials live only in `.env` on the host (gitignored here).
- MariaDB publishes no host port; only the app container can reach it.
- Least privilege inside Snipe-IT: day-to-day inventory users get
  create/edit/checkout rights; delete and admin settings stay with IT admins.
- Backups: scheduled DB dump + `storage/app` snapshot, stored off the host;
  a restore you've never tested is a hope, not a backup.
- Put the app behind HTTPS (reverse proxy) before any real data goes in —
  session cookies and credentials cross the network on every login.
