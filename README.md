# ♻ Smart Bin Fleet Optimizer

**HTH-IT-04 · Fleet-Wide Waste Collection Route Optimizer Under Budget**

An IoT + optimization project that decides **which bins a limited fleet of garbage trucks should collect first**, using live fill-level and smell data, and proves the improvement against a naive fixed-schedule baseline. One bin is real (ESP32 + ultrasonic sensor); the other 40 are simulated around New Delhi.

---

## The problem

A city has a fixed number of trucks and truck-hours per day, which is not enough to empty every bin. Collecting in a fixed order wastes time on half-empty bins while overflowing ones wait. This project prioritises bins by **overflow risk**, plans routes inside the truck-hour budget, and quantifies the gain over the naive method.

## Features

- **Real bin** – ESP32 + HC-SR04 ultrasonic sensor, 3 LEDs, buzzer, and a built-in web page / JSON API.
- **40 simulated bins** – fill level and smell change logically over simulated time (rush hours, bin type, time since last collection, random smell spikes).
- **Two Critical triggers** – fill level ≥ 80 % **or** air quality (smell index) > 70.
- **Reason tracking** – every Critical bin shows *why* it is Critical.
- **Budget-constrained optimizer** – truck-hour limit (including return to depot) and truck load limit.
- **Naive vs Optimized comparison** – critical bins missed, coverage, distance, wasted visits, load and more.
- **Real-time re-routing (bonus)** – a mid-demo fill-rate spike on one bin changes the plan on the next tick.
- **Live dashboard** – Delhi map with per-truck routes, KPI cards, real-bin panel, truck plans, history chart and a searchable bin table.

## How it works

### Risk model (`risk_model.py`)

| Tier | Rule (current values, same as the ESP32) |
|---|---|
| **Critical** | fill ≥ 80 % **or** air quality > 70 |
| **Medium** | fill ≥ 50 % |
| **Low** | otherwise |

The optimizer uses a *priority* score: `risk = max(predicted fill ÷ 100, air quality ÷ 100)`, plus 1.0 for Critical bins so they always come first. Predicted fill looks a few hours ahead using each bin's fill rate.

### Optimized method (`route_optimizer.py`)
1. Skip low-risk bins (below `MIN_RISK_TO_VISIT`) unless Critical.
2. Take bins from highest priority down.
3. Insert each bin at the **cheapest position** across all trucks, if it still fits the truck-hour budget (including the trip back to the depot) and the truck load limit.
4. Improve each route with **2-opt**.

### Naive baseline (`baseline_schedule.py`)
Bins are handed out in fixed `bin_id` order, round-robin between trucks, ignoring fill level and smell, until a truck runs out of hours or capacity.

### Real bin rules (ESP32)
| Condition | LED | Buzzer | Status shown |
|---|---|---|---|
| fill ≥ 80 % | Red | **ON** | Critical (reason: fill) |
| air quality > 70 | not affected | **off** | Critical (reason: bad smell) |
| fill ≥ 50 % | Yellow | off | Medium |
| otherwise | Green | off | Low |

The LEDs and buzzer follow the fill level only. Smell has no gas sensor in this prototype, so the ESP32 simulates it.

## Project structure

```
smart-bin-optimizer/
├── main.py                 # live loop: simulate → risk → optimize → compare → save
├── utils.py                # ALL settings (depot, thresholds, trucks, hours, capacity)
├── generate_bins.py        # creates the 40 simulated bins (simulated_bins.csv)
├── simulator.py            # makes bins fill up / smell over time
├── risk_model.py           # tiers, reasons, priority score
├── route_optimizer.py      # optimized routing
├── baseline_schedule.py    # naive fixed-schedule routing
├── evaluate.py             # naive vs optimized metrics → latest_results.json
├── bin_reader.py           # reads the real bin from the ESP32
├── visualize_map.py        # optional Folium map (route_map.html)
├── dashboard.html          # web dashboard (reads latest_results.json)
├── firmware/
│   └── sketch_sep24a/
│       └── sketch_sep24a.ino   # ESP32 code
├── requirements.txt
└── README.md
```

## Hardware

| Part | ESP32 pin |
|---|---|
| HC-SR04 TRIG | GPIO 5 |
| HC-SR04 ECHO | GPIO 18 |
| Green LED | GPIO 26 |
| Yellow LED | GPIO 27 |
| Red LED | GPIO 14 |
| Buzzer | GPIO 12 |

The HC-SR04 sits at the top of the bin pointing down. `BIN_HEIGHT` in the sketch is the empty-bin distance in cm.

## Setup

**Requirements:** Python 3.9+, Arduino IDE (with ESP32 board support), a browser with internet access (map tiles).

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
pip install -r requirements.txt
python generate_bins.py
```

### ESP32
1. Open `firmware/sketch_sep24a/sketch_sep24a.ino` in the Arduino IDE.
2. Set `WIFI_SSID` and `WIFI_PASSWORD` to your Wi-Fi (or phone hotspot).
3. Upload, then open the Serial Monitor at **115200 baud** and note the IP address.
4. In `bin_reader.py`, set `ESP32_IP` to that address.
5. Check `http://<esp32-ip>/data` in a browser – you should see JSON.

*No hardware?* Set `MOCK_REAL_BIN = True` in `bin_reader.py` for a fake real bin.

## Run

Open two terminals in the project folder.

```bash
# Terminal 1 – simulation + optimization (refreshes every 10 s)
python main.py

# Terminal 2 – local web server for the dashboard
python -m http.server 8000
```

Open **http://localhost:8000/dashboard.html** (do not double-click the HTML file, browsers block reading the JSON that way).

`python main.py --once` runs a single cycle and exits.

## Dashboard

- KPI cards: critical bins missed, coverage, distance, wasted visits, risk covered (naive vs optimized)
- Delhi map: switch between Optimized and Naive routes, missed critical bins ringed in red, optional road-following routes
- Real-bin panel: fill, smell, tier, reason, buzzer state, route assignment
- Truck plans: time and load against budget, ordered stops
- History chart of critical bins missed per tick
- Searchable, filterable table of all bins

## ESP32 API

`GET http://<esp32-ip>/data`

```json
{
  "fill_percent": 43.8,
  "distance_cm": 7.6,
  "air_quality": 41.1,
  "status": "Low",
  "reason": "none",
  "reason_text": "Normal",
  "buzzer": false
}
```

## Configuration

Everything is in `utils.py`:

| Setting | Meaning |
|---|---|
| `DEPOT_LAT`, `DEPOT_LON` | Depot location (default: New Delhi) |
| `NUM_TRUCKS`, `HOURS_PER_TRUCK` | The fleet budget |
| `TRUCK_CAPACITY_L` | Truck load limit (litres) |
| `TRUCK_SPEED_KMPH`, `SERVICE_TIME_MIN` | Driving speed, time to empty a bin |
| `FILL_CRITICAL`, `AQ_CRITICAL` | Critical thresholds (80 % / 70) |
| `MIN_RISK_TO_VISIT` | Optimizer ignores bins below this risk |

At the top of `main.py`: `TICK_SECONDS`, `SPIKE_AT_TICK`, `SPIKE_BIN`, `EXECUTE_EVERY_N_TICKS`.

A tighter budget (fewer trucks or hours) widens the gap between the two methods.

## Sample result

In a sample run at 06:00, the naive plan missed **5 of 7 critical bins**; the optimized plan missed **0**, while also skipping low-risk bins. Numbers change every tick as the simulation runs.

## Limitations and future work

- Distances and times use straight-line (haversine) distance at a fixed speed, not real road distance.
- Air quality is simulated; a real MQ-series gas sensor could replace it.
- Only one real bin; its fill rate is a placeholder value.
- Greedy insertion + 2-opt is a heuristic, not a guaranteed optimum. OR-Tools could be added for comparison.
- Possible extras: multi-depot, time windows, bin-history drill-down, alerts by SMS.

## Tech stack

Python (pandas, numpy, requests, folium) · ESP32 / Arduino C++ · HTML, CSS, vanilla JavaScript · Leaflet + OpenStreetMap

## Team

- Catalyst
