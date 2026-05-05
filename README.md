# Lunch Calculator (Android & Desktop)

A cross-platform application for splitting lunch bills among colleagues, now optimized for mobile devices. It handles Malaysian SST and service charge, tracks per-person totals, and compares against the actual receipt amount.

This branch focus on the **Android version**, featuring a completely redesigned mobile-first UI while maintaining compatibility with the original desktop version.

---

## Mobile Features (New!)

- **Mobile-First UI:** Responsive portrait layout with a modern dark theme.
- **Intuitive Navigation:** Stack-based navigation for managing people and individual orders.
- **Touch-Optimized Controls:** Uses bottom sheets and popups for editing items, picking names, and selecting currencies.
- **Currency Search:** Built-in searchable list of 150+ global currencies.
- **Discounts Support:** Enter negative prices to handle vouchers or line-item discounts.
- **Quick Sharing:** Buttons to copy a bill summary to the clipboard or save it as an image for sharing via WhatsApp/Telegram.
- **Batch Actions:** Toggle SC? or SST? for all items with a single switch.

---

## Core Features (Shared)

- Add unlimited bill items with quantity and unit price.
- Independent **SC?** and **SST?** flags per item.
- Automatic split calculation among selected persons.
- Per-person breakdown: food cost, SC+SST share, and total.
- Receipt amount validation with a difference indicator.
- Persistent names list managed in-app.

---

## Requirements (Development)

| Tool | Version | Purpose |
|---|---|---|
| **Qt** | 6.10.2 | Framework |
| **Android SDK/NDK** | Latest | Required for Android builds |
| **OpenJDK** | 17 or later | Required for Android builds |
| **MinGW 64-bit** | Bundled with Qt | Required for Desktop builds |
| **CMake** | 3.21.1 or later | Build system |

---

## Building

### For Android
1. Open **Qt Design Studio** (or Qt Creator).
2. Open `CMakeLists.txt` at the project root.
3. Select an **Android** kit (e.g., `Android Qt 6.10.2 Clang x86_64` for emulator or `arm64-v8a` for physical device).
4. Click **Build → Build All**.
5. Connect your device and press **Ctrl+R** to deploy.

### For Desktop (Windows)
1. Open `CMakeLists.txt` at the project root.
2. Select the **Qt 6.10.2 MinGW 64-bit** kit.
3. Click **Build → Build All**.
4. Press **Ctrl+R** to run.

---

## Project Structure

```
LunchCalculator/
├── App/
│   ├── main.cpp                # App entry point (switches UI based on OS)
│   └── CMakeLists.txt
├── LunchCalculatorContent/
│   ├── App.qml                 # Desktop UI (Light theme)
│   ├── AppAndroid.qml          # Mobile UI (Dark theme)
│   ├── PersonNames.qml         # Default names list
│   └── CMakeLists.txt
├── LunchCalculator.h/cpp       # Main controller — calculations, totals, reset
├── LunchModel.h/cpp            # Table model logic
├── LunchItem.h                 # Data struct for bill items
└── ImageExporter.h/cpp         # Screenshot and clipboard functionality
```

---

## How to Use (Mobile)

1. **Setup:** Enter the Place, Date, and Tax percentages at the top.
2. **Add People:** Tap **Manage** to add colleagues. Pick from the defaults or type new names.
3. **Add Items:** Tap the **+** FAB (Floating Action Button) to add items. Set Qty, Price, and Tax flags.
4. **Assign Orders:** Tap a person's avatar to open their checklist and tick the items they ordered.
5. **Review:** Check the summary card at the bottom. Enter the **Receipt Amount** to see the difference.
6. **Share:** Use the **Copy** or **Save** buttons in the bottom bar to share the result.

---

## How to Use (Desktop)

### Filling a Bill

1. Enter the **Place** and **Date** at the top.
2. Set **Service Charge %** and **SST %** (yellow fields).
3. Add persons using **+ Add Person** — pick from the list or type a custom name.
4. Click **+ Add Item** for each menu item:
   - Enter the item name.
   - Set **Qty** (defaults to 1) and **Unit Price**.
   - Tick **SC?** if the item is subject to service charge.
   - Tick **SST?** if the item is subject to SST.
   - Tick the checkboxes under each person's name who ordered that item.
5. Enter the **Receipt Amt** from the actual bill — the **Difference** row shows any discrepancy.

### Quantity

When multiple people ordered the same item, enter the count in **Qty** instead of adding separate rows. For example, 3 people each ordering Kopi Ice at MYR 6.20 → Qty=3, Unit Price=6.20, tick all three persons. Each person's share works out to MYR 6.20.

### SC? and SST? Flags

Each item independently controls whether service charge and SST apply:

| SC? | SST? | Effect |
|-----|------|--------|
| ✓ | ✓ | Both SC and SST applied (most common) |
| ✗ | ✓ | SST only — e.g. some beverages |
| ✓ | ✗ | SC only |
| ✗ | ✗ | Neither — e.g. vouchers, non-taxable items |

### Per-Person Totals

The blue bar at the bottom shows each person's food cost, combined SC+SST, and total amount to pay. It scrolls horizontally if there are more than a few persons.

---

## Calculation Rules

| Field | Formula |
|---|---|
| Subtotal | Sum of (Qty × Unit Price) for all items |
| Service Charge | SC-flagged items total × SC% |
| SST | SST-flagged items total × SST% |
| Grand Total | Subtotal + Service Charge + SST |
| Per-person SC+SST | Proportional to each person's flagged food share |

SC and SST are calculated independently and do not compound.

---

## Deploying to Android

To generate a signed APK or AAB for distribution:
1. In Qt Creator, go to **Projects → Build Settings → Build Steps**.
2. Under **Build Android APK**, ensure the correct keystore is selected.
3. Build the project in **Release** mode.
4. The `.apk` file will be generated in your build directory under `android-build/build/outputs/apk`.

---

## Deploying to Windows

Follow these steps to produce a standalone folder that colleagues can run without installing Qt.

### 1. Build in Release mode

In Qt Creator, switch the kit selector from **Debug** to **Release**, then run **Build → Rebuild All**.

### 2. Create a staging folder and copy the executable

```powershell
mkdir "$env:USERPROFILE\Desktop\LunchCalculator-dist"
copy "C:\Users\na87995\Documents\QtDesignStudio\LunchCalculatorApp-Release\LunchCalculatorApp.exe" "$env:USERPROFILE\Desktop\LunchCalculator-dist\"
```

### 3. Run `windeployqt`

```powershell
$exe = "$env:USERPROFILE\Desktop\LunchCalculator-dist\LunchCalculatorApp.exe"
$qml = "C:\Users\na87995\Documents\QtDesignStudio\LunchCalculator\LunchCalculatorContent"
& "C:\Qt\6.10.2\mingw_64\bin\windeployqt.exe" --qmldir $qml $exe
```

### 4. Smoke test

Open `LunchCalculator-dist\` and double-click `LunchCalculatorApp.exe` to verify it launches correctly on a machine without Qt installed.

### 5. Zip and distribute

Right-click the `LunchCalculator-dist` folder → **Send to → Compressed (zipped) folder**.
