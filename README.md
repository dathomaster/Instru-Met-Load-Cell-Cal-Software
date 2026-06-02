# HADI + OCR Capture App

## Install
```bash
pip install pyserial
```

## Run
```bash
python hadi_ocr_app.py
```

## Capture page

The Capture page is simplified for normal use:
- Select load cell
- Select Compression/Tension
- Connect HADI
- Start/Stop OCR
- Big ZERO HADI button
- Live HADI
- Live OCR
- Live % Error
- Capture table

Detailed COM/baud/OCR port/update-rate settings are on the **Connections** tab.

## Sync

Auto Sync now runs under the hood. The app estimates and saves OCR timing lag automatically in `sync_state.json`.

The capture table no longer shows sync details, but the CSV still saves sync lag/confidence for traceability.

## Load Cells

Open the **Load Cells** page to save calibration coefficients for any load cell.

Equation:
```text
Force (lbf) = B0 + B1*R + B2*R^2 + B3*R^3
```

Saved load cells are stored in `load_cells.json`.


## Visual cleanup

The HADI zero button is now a small **ZERO** button next to the HADI live display. The live percent error display changes color:
- green within ±0.5%
- amber within ±1.0%
- red above ±1.0%


## Table colors

The app uses the original/default theme again, but keeps live percent error coloring. Capture table rows are highlighted:
- amber when absolute error is over 0.5%
- red when absolute error is over 1.0%


## Edit OCR value

Double-click any OCR cell in the capture table to edit it. Press Enter or click away to save. The app recalculates percent error and updates the row color. Edited rows are marked in the CSV with `ocr_edited=True`.


## OCR edit override fix

After editing an OCR cell, the selected row is not armed for override. Override mode only starts from a normal single-click row selection, not from finishing a cell edit.


## Override button visibility

The Cancel Override button is hidden by default. It appears only when a table row override is armed, with clearer warning text.


## Override layout lock

The override controls now reserve their space even when override is off. The Cancel Override button is disabled instead of removed, so the capture table does not shift size.


## Point calibration table

The capture table is now a fixed point calibration table. Choose 10, 20, or Custom points.

- The highlighted row is where the next capture will go.
- Capturing writes into that row, then automatically moves down to the next empty point.
- Single-click any row to make the next capture go there.
- Double-click an OCR cell to edit it. After editing, the target goes back to the next empty point instead of staying on that edited row.
- The CSV saves only filled points.


## Point count resizing

Switching from 10 to 20 points, or increasing to a larger custom count, now keeps existing captured values and adds blank rows. Switching downward warns only if saved values would be removed.


## Capture button layout

The main capture button is now a large button under the HADI raw display. OCR age and last HADI text were removed from the Capture page for a cleaner layout.


## Clear warning and save button

The Clear button now asks for confirmation before clearing captured points. The CSV save button is larger and labeled **SAVE CAPTURE DATA TO CSV**.


## Save button label

The CSV button now says **SAVE TO CSV** while keeping the same wide button spacing.


## Coefficient notation

Load-cell coefficient fields accept scientific notation such as `-2.610015E+04`, `-2.610015e+04`, or `-2.610015e04`. When displayed in the Load Cells page, coefficients are normalized to uppercase scientific notation with two exponent digits, for example `-2.610015E+04`.


## Two-run point table

The capture table now has Run 1 and Run 2 side by side for each point. The point number column is slim, and percent error columns are slim.

Capture flow:
- The highlighted row/run is the next target.
- Capturing fills Run 1 first, then Run 2, then moves to the next point.
- Single-click a row to target the first empty run on that row.
- Double-click either OCR column to edit that run's OCR value.
- CSV saves one row per point with Run 1 and Run 2 columns.


## Run selection behavior

Run 1 and Run 2 are now separate passes. Capturing in Run 1 fills down Run 1 only. Capturing in Run 2 fills down Run 2 only.

- Click a Run 1 column to target Run 1 on that point.
- Click a Run 2 column to target Run 2 on that point.
- After capture, the target moves to the next point in the same run.
- Editing an OCR value does not switch runs.


## Run separation display

Run 1 and Run 2 now have a clear gap between them. The active target is shown with a ▶ marker in the R1 or R2 marker column instead of highlighting the whole row, so it is clear which run will receive the next capture.


## OCR edit fix

OCR cell editing in the separated Run 1 / Run 2 table is now stabilized. Double-clicking an OCR cell no longer lets the click event retarget the active run, and the table does not refresh cell values while the inline editor is open.


## Run table spacing

The empty separator column between Run 1 and Run 2 was removed to give the table more usable width.


## Split Run 1 / Run 2 tables

Run 1 and Run 2 are now two side-by-side tables. This allows the blue selection highlight to appear only in the active run table, instead of across both runs. The arrow/marker columns were removed.


## GPS / gravity weight reference

The app reads `GPS	<timestamp>	<latitude>	<longitude>` packets on the same UDP port as OCR. The iPhone app sends this GPS packet when streaming starts.

Use **SET SELECTED ROW AS WEIGHT** to fill the selected Run 1 or Run 2 target with a reference weight instead of the raw HADI force:
1. The app reads the current synced HADI value.
2. It applies the GPS-derived gravity factor.
3. It rounds to the nearest standard 1-2-5 / whole-pound weight.
4. It converts that nominal weight back to expected local lbf and compares it to OCR.

The CSV saves the nominal weight, gravity factor, GPS coordinates, and method for traceability.


## Manual weight entry

You can now populate a point without HADI connected:
- Type a weight in the **Weight lb** box and click **SET WEIGHT**.
- Or double-click any **HADI** cell and type the weight directly.

Typed weights are treated as nominal lb weights. The app applies the GPS gravity factor when available and stores the local lbf value in the HADI column. If OCR is already present for that run, percent error recalculates automatically.


## Batch manual weights and W marker

Each run table now has a small **W** column. A `W` appears when that row/run is populated from a manual or inferred weight reference.

You can apply weights to multiple rows:
- Select multiple rows in Run 1 or Run 2 with Shift/Ctrl.
- Type one weight, like `10`, to apply the same weight to every selected row.
- Or type a list, like `5 10 20`, to apply one weight per selected row.
- Click **SET WEIGHT**.

Double-clicking a HADI cell still lets you type one weight directly for that row.


## Weight marker cleanup

The separate W column was removed. Rows populated from a typed/manual weight now show the marker next to the point number, for example `1w`, `2w`, `3w`.

The old HADI-inferred weight button was removed from the main workflow. To populate weights:
- Select one or more rows in Run 1 or Run 2.
- Type one weight like `10` to apply it to all selected rows.
- Or type a list like `5 10 20` to apply one weight per selected row.
- Click **SET W**.

Double-click a HADI cell to type one weight directly for that row.


## SET W only

The weight entry prompt was removed. The workflow is now:
1. Enter or capture values in the HADI column.
2. Select one or more rows with Shift/Ctrl.
3. Click **SET SELECTED ROWS AS W**.

The selected rows are marked with `w` next to the point number, such as `1w` or `2w`. If a selected row has no HADI/weight value yet, the app warns you to double-click the HADI cell and enter it first.


## W toggle behavior

Editing a HADI cell no longer automatically marks the row as a weight. The workflow is:
1. Double-click the HADI cell and type the value.
2. Select one or more rows with Shift/Ctrl.
3. Click **SELECT / UNSELECT W**.

The button toggles W on selected rows. If a selected row is already marked `w`, clicking the button removes the `w` marker.


## Excel-style HADI entry

When editing a HADI/weight cell, pressing Enter now commits the value and immediately opens the next HADI cell down in the same run. This lets you type weight values like a spreadsheet: type, Enter, type, Enter.

This only happens for HADI cells. OCR cell editing still commits normally.


## HADI units, decimals, and mode badge

Added on the Capture page:
- **HADI Units** selector: `LBF`, `KGF`, `N`, `kN`, `gF`, `t`, and `mV/V`
- **Decimals** selector including `0.00001`
- A clearer **COMPRESSION / TENSION** badge directly under the HADI reading

The main live HADI display now follows the selected units and decimal resolution. The small raw display still shows the raw HADI response in `mV/V`.


## Theme + table stability update

- Compression badge moved to a shared top position and uses a teal light highlight with darker teal text.
- Tension badge moved to the same shared top position and uses a purple light highlight with darker purple text.
- The old badge under the HADI reading was removed.
- Run-table column widths are now locked so the tables stay visually stable when values are entered.
- Weight/status helper text was shortened and wrapped to avoid pushing the layout around.


## Waiting / stale-reading behavior

The live display no longer leaves stale numbers on screen:
- If HADI disconnects or no fresh HADI readings arrive for about 1.5 seconds, the HADI display shows **WAITING FOR HADI**.
- If OCR stops or no fresh OCR packets arrive for about 1.5 seconds, the OCR display shows **WAITING FOR OCR**.
- Live percent error clears while either source is stale.
- Capture is blocked until fresh HADI and OCR readings are available.


## Autosave and exit protection

The app now protects captured data:
- Every capture/edit/W-toggle automatically writes a backup CSV into an `autosaves` folder beside the app.
- If you try to close the app with unsaved capture data, it warns you and asks whether to save to CSV first.
- If you choose not to save manually, the autosave backup is still kept.
- Clearing the table also creates an autosave backup before removing the visible points.


## Smarter CSV names

The Save to CSV dialog now auto-populates a more useful filename:

`compression_run1_20260523_142530_SN.csv`

or, when both runs have data:

`compression_run1_run2_20260523_142530_SN.csv`

If only Run 2 has data, it uses `run2`. The final `SN` is intentionally left at the end so you can quickly replace it with the serial number before saving. Autosave files use the same pattern with an `autosave_` prefix.


## CSV filename time format

CSV filenames now use minute-level 24-hour time instead of seconds:

`compression_run1_20260523_1425_SN.csv`

Format:

`mode_runs_YYYYMMDD_HHMM_SN.csv`


## Simple CSV export format

Manual saves and autosaves now use a simple report-style CSV layout.

Run 1 is written first:

`Run 1`

`Point, OCR, HADI, % Error`

Then Run 2 is written underneath with a blank line break:

`Run 2`

`Point, OCR, HADI, % Error`

Only runs with data are included. Extra metadata columns were removed from the CSV to make the file easier to read and print.


## CSV point column removed

The simplified CSV export now omits the point column.

Current layout:

`Run 1`

`OCR,HADI,% Error`

Then Run 2 appears underneath, when it has data:

`Run 2`

`OCR,HADI,% Error`


## W toggle and auto-connect update

- Editing a HADI cell still enters the normal value and does not mark W automatically.
- Clicking **SELECT / UNSELECT W** now visibly converts the HADI value to the GPS-corrected local lbf value.
- Clicking **SELECT / UNSELECT W** again restores the original value you typed.
- On launch, the app automatically starts the OCR UDP listener and tries to connect to the first available HADI COM port. If HADI is not available, it waits without blocking the app.


## B4/B5 + exact OCR capture fix

- HADI force calculation now uses the full polynomial:
  `B0 + B1*R + B2*R^2 + B3*R^3 + B4*R^4 + B5*R^5`
- B4 and B5 are optional in the Load Cells page and act as zero when blank.
- Existing saved load cells without B4/B5 still work.
- OCR capture no longer interpolates between OCR packets. It uses the nearest actual OCR packet at the corrected sync time, so the saved OCR value stays exactly what the phone reported instead of creating extra decimals.
- HADI is still interpolated for timing alignment.


## W-row OCR-only capture

W rows can now be filled without HADI connected:
- If the selected target row is marked `w`, Capture only grabs the latest fresh OCR value.
- It preserves the existing HADI/weight value in that row.
- It recalculates percent error from the preserved HADI/weight value and the new OCR value.
- After capturing a W row, the target moves to the next W row that still needs OCR instead of jumping to the bottom/first empty row.
- Normal non-W rows still require fresh HADI and fresh OCR.


## W-row HADI edit behavior

Editing the HADI cell now works both ways:
- If the row is not marked W, editing HADI enters a normal manual HADI value.
- If the row is already marked W, editing HADI updates the nominal weight, keeps the W marker on, reapplies the GPS gravity factor, preserves any OCR value already in that row, and recalculates percent error.


## GPS altitude and MF

GPS packets may now include altitude in meters:

`GPS	<timestamp>	<latitude>	<longitude>	<altitude_m>`

The app calculates the multiplying factor:

`MF = local gravity / 9.80665`

Latitude uses the normal gravity formula. If altitude is supplied, the app applies a free-air elevation correction. If altitude is missing, the app assumes 0 m elevation, which matches the sea-level column of the ASTM-style MF table.


## ASTM Table 1 MF calculation

The W-row weight correction now uses the ASTM Table 1 "Multiplying Factor, MF, In Air at Various Latitudes" values directly.

Implementation:
- Table latitude rows: 0, 5, 10, ... 70 degrees
- Table elevation columns: 0, 500, 1000, 1500, 2000, 2500 m
- The app bilinearly interpolates between the nearest latitude/elevation entries.
- Latitude uses absolute latitude.
- If altitude is missing, elevation is assumed to be 0 m.
- If latitude/elevation is outside the displayed table range, the nearest table edge is used.

W row correction:

`local/reference lbf = typed nominal weight * ASTM MF`


## Optional B coefficients

All load-cell coefficients B0 through B5 are now optional. Blank coefficient boxes are saved as `0.0`.

The force equation still uses all six terms:

`Force = B0 + B1*R + B2*R^2 + B3*R^3 + B4*R^4 + B5*R^5`

So leaving a coefficient blank simply removes that term from the equation.
