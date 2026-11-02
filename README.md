# This is just a readme to see a sample of the project.  The main is private

# Bus Interface

Local browser interface for a bus climate, safety, water, propane, and power system. It talks to an AutomationDirect CLICK PLC over Modbus TCP using clickplc and pymodbus.

Double-click **Start Bus Interface.cmd** on Windows. Keep the server window open. The dashboard opens in your browser. Closing the browser tab leaves the server running; close the server window to stop it.

## Screens

These are the current pages. More captures, including earlier builds, are in the sections below. The image files are in `Screenshots/`.

![Overview with temperatures, Auto mode, tank levels, and safety and propane ready](Screenshots/overview-tanks.png)

![Climate page in Auto with writes enabled](Screenshots/climate-controls-restored.png)

![Propane page with the temperature-adjusted percentage and raw PSI](Screenshots/propane-percentage.png)

![Water tanks with fresh and black faults active](Screenshots/water-tanks.png)

![Safety page with permissives OK and monitored faults clear](Screenshots/clean-operating-labels.png)

![Faults page with nothing active](Screenshots/faults-page.png)

![I/O checkout filtered to CPU outputs](Screenshots/io-checkout.png)

![Settings register mapping](Screenshots/clickplc-setup.png)

## Requirements

- Python 3.13 (`py -3.13` on Windows)
- A CLICK PLC reachable on the network, or demo mode
- clickplc 0.14.2 and pymodbus 3.13.1

Install the libraries with:

```bat
py -3.13 -m pip install -r "Configuration/requirements.txt"
```

On the original computer, those packages may already be present under `PLC Communications/vendor/`. The launcher uses them when they are there.

## Start

```bat
py -3.13 "API Server/bus_interface.py"
```

| Flag | Effect |
| --- | --- |
| `--demo` | Start without a PLC connection. Sample values only. |
| `--no-browser` | Do not open a browser window. |
| `--port 58616` | Listen on a fixed port. The default `0` picks a free local port. |
| `--share-address` | Opt-in IPv4 address for trusted Wi-Fi sharing. |
| `--share-network` | Allowed subnet, for example `10.0.0.0/24`. |

The server listens on loopback unless sharing is turned on. It connects to the PLC at startup and retries a lost connection.

Default PLC settings: **10.0.0.240**, port **502**, unit ID **1**. Change them under **Settings**, save, then select **Connect to PLC**. Compare readings with CLICK Data View before enabling writes. Addresses in Settings are native CLICK names such as `DS1` and `DF110`.

## Overview

Overview is the home screen: cabin, tank-bay, and outside temperatures, climate mode, safety and propane status, and tank levels. **Safety Drop** stays in the header on every page.

![Overview with temperatures, Auto mode, tank levels, and safety and propane ready](Screenshots/overview-tanks.png)

Outside temperature also has its own card on the current Overview.

![Overview with the outside temperature card at 100 °F](Screenshots/outside-temperature-card.png)

Safety and propane show as ready indicators once those readings are in. In this capture, outside temperature is still listed under On board.

![Overview with safety and propane ready indicators](Screenshots/status-indicators.png)

An early live connection, before outside had a card and before safety monitoring was filled in. Temperatures are 0.0 °F and writes are locked.

![Early overview with zero temperatures and writes locked](Screenshots/interface-preview.png)

Tank bay on `DF17` with the PLC connected and writes still locked.

![Overview showing a live tank-bay temperature with writes locked](Screenshots/df17-preview.png)

## Climate

Climate sets HVAC mode and the cabin target. Only those two values are written, and only after the mapping is verified and **Enable PLC writes** is selected.

- `DS1` — Off `0`, Heat `1`, Cool `2`, Auto `3`
- `DF110` — cabin target, 35–100 °F. A Celsius entry is converted before the write.

Each write is read back. Controls drop out on disconnect and do not replay old commands when the link returns. PLC temperatures stay in Fahrenheit. Display can be Fahrenheit or Celsius. Calibration is `decoded °F × gain + offset`.

Default temperature reads: front `DF113`, rear `DF114`, outside `DF115`, tank bay `DF17`.

![Climate page in Auto with writes enabled](Screenshots/climate-controls-restored.png)

![Climate page in Off with writes enabled and the target still at 0.0 °F](Screenshots/live-controls.png)

## Propane

The Propane page shows the PLC’s temperature-adjusted pressure percentage from `DF40` (0–100). That is pressure percentage, not tank fullness. The displayed low reference is 20%. Raw pressure stays on `DF10` in PSI. The app does not write `DF40` or change PLC thresholds. Pressure permissive is `C3`. Low-propane fault is `C102`.

![Propane page with the temperature-adjusted percentage, raw PSI, and PLC status](Screenshots/propane-percentage.png)

This earlier capture showed raw `DF10` pressure before the percentage display was added.

![Earlier propane page showing supply pressure in PLC units](Screenshots/propane-page.png)

## Water tanks

Water levels come from the PLC. Fresh and gray show percent and gallons. Black level comes from the tank switches. Pump, fill, and the pressure switch stay under PLC control. The pressure switch is `X309`.

Fresh-water low is `C105`, gray full is `C106`, and black full is `C107`. Those three are recorded in fault history.

![Water tanks with fresh and black faults active and the fill valve on](Screenshots/water-tanks.png)

## Power

Power shows battery voltage `DF200`, battery current `DF201`, and AC leg amps `DF203` and `DF204`. Shore power is on when shore power good `X301`, shore override `X302`, or the HMI override `C15` is on. Generator is selected on `C14` and flashes while generator run `C28` is on. Quiet time latches `C24`. Battery low `C112` and generator fault `C113` are shown here and recorded with the other equipment faults. There is no Power-page capture in `Screenshots/` yet.

## Safety

The Safety page reads E-stop permissive `C1`, safety latch `C2`, pressure permissive `C3`, E-stop fault `C101`, low-propane fault `C102`, tank-bay sensor fault `C52`, reset request `C103`, and HMI safety drop `C110`. Pressure is `DF10` in PSI. The low threshold on this page is `DF101`.

![Safety page with permissives OK and monitored faults clear](Screenshots/clean-operating-labels.png)

This earlier capture is from before Safety Drop was on the page. E-stop and low-propane faults are active, and the reset control is still waiting on setup.

![Earlier safety page with active faults and reset not yet available](Screenshots/safety-page.png)

Safety Drop pulses `C110` for 0.3 seconds, then the server turns it off before the request returns. Release drop and reset requires a live connection and verified mapping. It will not pulse `C103` unless E-stop permissive `C1` is ready. A failed or uncertain reset locks writes. There is no automatic reset at startup.

The next three captures are from the build that described Safety Drop as a latch. The running program pulses `C110` and then turns it off.

![Safety page while a safety drop is requested](Screenshots/safety-controls.png)

![Safety page after the drop is released, with reset still unavailable](Screenshots/safety-buttons-steady.png)

![Safety page with the latch OK and the drop released](Screenshots/reset-ready.png)

This screen stop uses an ordinary PLC and network. It is not a safety-rated emergency stop. Keep the physical E-stop circuit effective on its own. Before using reset with equipment, make the PLC reset request self-limiting with a one-shot or maximum-on timer. AutomationDirect’s CLICK Ethernet analog PLC safety insert is at https://cdn.automationdirect.com/static/manuals/c0userm/CLICK_Ethernet_Analog_PLC_Insert.pdf

## Faults

The banner on every page lists active E-stop, low-propane, tank-bay sensor, fresh-water low, gray full, black full, battery low, generator fault, HMI safety-drop, and communication faults. Open **Faults** for the active list and history.

`Data and History/fault-history.sqlite3` records Active and Cleared transitions for each PLC endpoint. The file is created on the computer running the server. The Faults page shows the newest 200 events. Times are America/New_York. Demo mode does not record history. A lost connection is not logged as an equipment fault clearing.

![Faults page with nothing active and an empty history](Screenshots/faults-page.png)

## I/O checkout

I/O Checkout is read-only. CPU inputs are `X001` through `X004`. `X005` through `X011` are not physical inputs and are not listed. Module 3 carries shore, generator-leg, black-tank, and fresh-water pressure points (`X301`–`X309`). Output bits show the PLC command, not proof that a device is energized. Names can be edited and saved in `Configuration/io-labels.local.json`.

This capture is filtered to outputs. It was taken before `X005`–`X011` were removed, so the signal count on screen is higher than the current list.

![I/O checkout filtered to CPU outputs](Screenshots/io-checkout.png)

## Settings

Open **Settings** to set the PLC address, display unit, register mapping, and sample values. Mark the verification checkbox, save, connect, then select **Enable PLC writes**. Climate writes stay enabled until you turn them off. Saving setup or losing the connection locks them.

This capture still has tank bay entered as `DF116`. The current default is `DF17`.

![Settings register mapping and the address-verification checkbox](Screenshots/clickplc-setup.png)

## Wi-Fi sharing

Sharing is off unless you start it. Close any running server, then use **Startup Scripts/Start Wi-Fi Sharing.cmd**, or pass `--share-address` and `--share-network`. The server binds to that IPv4 address and loopback, and rejects clients outside the subnet. The launcher in this repo is set for host `10.0.0.80` and subnet `10.0.0.0/24`. Change those if this PC’s address is different.

Settings on the host shows a private invitation link. The key is in the URL fragment and is removed from the address bar after sign-in. Everyone who accepts the link has the same controls. This is plain HTTP for a trusted Wi-Fi network. Do not forward the port to the internet.

`Configuration/sharing-code.private.txt` and `Configuration/sharing-link.private.txt` stay on the host. Do not commit them.

## Layout

- `Dashboard/index.html` — browser interface
- `API Server/bus_interface.py` — local HTTP API
- `PLC Communications/` — CLICK driver
- `System Controls/` — climate, safety, water, power, and Wi-Fi session logic
- `Configuration/` — I/O catalog, requirements, and local settings
- `Data and History/` — fault recorder
- `Screenshots/` — the captures linked above
- `Startup Scripts/` — opt-in Wi-Fi launcher
- `Documentation/README.md` — longer operator notes

## Not in git

Local files matched by `.gitignore` stay on the computer that created them:

- `Configuration/settings.json`
- `Configuration/io-labels.local.json`
- `Configuration/*.private.txt`
- `Data and History/fault-history.sqlite3`
- `PLC Communications/vendor/`
- Python cache, and PNG files outside `Screenshots/`
