<img width="2186" height="1520" alt="image" src="https://github.com/user-attachments/assets/cf79980b-c220-478d-a18b-bc7318e53171" />
# HADI + OCR Capture Software

A Windows desktop application for comparing force readings from a **Morehouse HADI digital indicator** against an **OCR stream from an iPhone camera app**. The software is designed for calibration/check workflows where a HADI/load cell reading and a displayed OCR reading need to be captured together, compared, and saved to CSV.

## What It Does

The app reads live force data from a HADI indicator over a serial COM port and receives OCR values from an iPhone over UDP. It displays both readings live, calculates percent error, and lets the user capture points into Run 1 and Run 2 tables.

It also supports manual weight-reference rows, GPS-based ASTM multiplying factor correction, automatic autosaves, and simple CSV exports.

## Main Features

### Live HADI Reading

* Connects to a HADI indicator over serial COM.
* Default communication settings:

  * Baud rate: `19200`
  * Read command: `GN`
  * Tare command: `SZ`
  * Line ending: `\r`
* Displays live HADI force.
* Supports Compression and Tension modes.
* Supports selectable HADI display units:

  * `LBF`
  * `KGF`
  * `N`
  * `kN`
  * `gF`
  * `t`
  * `mV/V`
* Supports selectable decimal display resolution, including `0.00001`.

### Load Cell Calibration

Load cells are stored locally in `load_cells.json`.

Each load cell can have separate Compression and Tension coefficients:

```text
Force = B0 + B1*R + B2*R^2 + B3*R^3 + B4*R^4 + B5*R^5
```

Where:

```text
R = raw HADI response in mV/V
```

All B coefficients are optional. Blank coefficients are treated as `0.0`.

The app supports scientific notation such as:

```text
-2.610015E+04
```

### OCR Stream

The iPhone OCR app sends OCR values to the laptop over UDP.

Default port:

```text
9999
```

The app listens for OCR packets and displays the latest value live.

OCR capture uses the nearest real OCR packet at the corrected capture time. It does not interpolate OCR values, so it does not invent extra decimal values that were not actually sent by the phone.

### Timing Sync

The software keeps a timing correction between HADI readings and OCR packets.

* HADI is interpolated for timing alignment.
* OCR uses the nearest actual OCR packet.
* The app saves timing sync state in:

```text
sync_state.json
```

This file is generated automatically and is not shipped with the app.

### Capture Table

The Capture page supports:

* Run 1
* Run 2
* HADI value
* OCR value
* Percent error
* Compression/Tension visual mode indicator
* Editable HADI and OCR cells
* Excel-style HADI entry:

  * type value
  * press Enter
  * moves to the next HADI cell

Percent error is calculated as:

```text
Percent Error = (HADI - OCR) / OCR * 100
```

### Weight Reference Rows

Rows can be marked as weight-reference rows using **SELECT / UNSELECT W**.

When a row is marked `W`:

* The typed HADI value is treated as the nominal weight.
* The app applies the ASTM multiplying factor.
* The displayed HADI value becomes the local/reference lbf value.
* Capturing a W row only fills the OCR value.
* HADI is not required for W-row capture.
* Existing HADI/weight values are preserved.
* Percent error is recalculated after OCR capture.

Clicking **SELECT / UNSELECT W** again removes the W marker and restores the original typed value.

If a W row already exists and the HADI cell is edited, the row stays marked W and the new typed value is treated as the updated nominal weight.

### ASTM Multiplying Factor

The app supports GPS-based ASTM Table 1 multiplying factor correction for weights.

Supported GPS packet formats:

```text
GPS	<timestamp>	<latitude>	<longitude>
```

or:

```text
GPS	<timestamp>	<latitude>	<longitude>	<altitude_m>
```

The app uses the ASTM Table 1 multiplying factor values for latitude and elevation.

Behavior:

* Uses absolute latitude.
* Uses altitude in meters when supplied.
* If altitude is missing, assumes `0 m`.
* Interpolates between table latitude/elevation values.
* Applies the factor to W rows:

```text
local/reference lbf = nominal weight * ASTM MF
```

### Autosave

The app automatically saves backup CSV files whenever data changes.

Autosaves are stored in:

```text
autosaves/
```

Autosave happens when:

* a capture is made
* a HADI or OCR cell is edited
* a W row is toggled
* the table is cleared

If the app is closed with unsaved data, it warns the user and offers to save manually first.

### CSV Export

Manual CSV export creates a simple report format.

Example:

```csv
Run 1
OCR,HADI,% Error
10.0000,9.9908 LBF,0.09%
20.0000,19.9506 LBF,0.25%

Run 2
OCR,HADI,% Error
10.0000,9.9980 LBF,0.02%
20.0000,19.9910 LBF,0.05%
```

CSV filenames are auto-populated in this format:

```text
compression_run1_YYYYMMDD_HHMM_SN.csv
```

or:

```text
compression_run1_run2_YYYYMMDD_HHMM_SN.csv
```

The `SN` at the end is intended to be replaced manually with the serial number.

## Local Network Setup

The iPhone and laptop must be on the same local network.

A small travel router works well for field use. Internet is not required.

Example:

```text
iPhone: 192.168.8.201
Laptop: 192.168.8.131
Port:   9999
```

The iPhone OCR app should send to the laptop IP address.

If packets are not received, allow inbound UDP port `9999` through Windows Firewall.

## Installation

Use the provided V1 installer ZIP.

Recommended install steps:

1. Right-click the installer ZIP.
2. Choose **Extract All**.
3. Open the extracted folder.
4. Run:

```text
START_HERE_INSTALL.cmd
```

5. Launch from the desktop shortcut:

```text
HADI OCR Software V1
```

The installer creates a no-console launcher and installs the app to:

```text
%LOCALAPPDATA%\HADI OCR Software
```

## Important Files

```text
hadi_ocr_app.py       Main desktop app
ocr_stream.py         UDP OCR/GPS receiver
gps_receiver.py       Standalone GPS receiver test utility
load_cells.json       Saved load-cell calibrations
sync_state.json       Auto-generated timing sync state
autosaves/            Auto-generated backup CSV files
```

## Notes

* `sync_state.json` is generated automatically.
* `autosaves/` is generated automatically.
* `load_cells.json` stores saved load cell calibration data.
* HADI normal capture requires fresh HADI and OCR readings.
* W-row capture only requires fresh OCR because the HADI/weight value is already entered.
* The app is intended for offline/local-network use.
