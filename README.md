# weblingappointments

This repository contains a small utility script to create and post calendar events to a WeBling-based API using data from an Excel (`appointment.xlsx`) file and a JSON template (`appointment.json`).

This README explains how to set up a local environment, prepare your data, and run `main.py` safely.

Requirements
------------

- Python 3.8+ (3.10 recommended)
- pip
- Optional: `virtualenv` or `venv` to isolate the Python environment

Files in this repository
-------------------------

- `main.py` - The main script that reads events from `appointment.xlsx`, updates `appointment.json` and sends it to the target API.
- `appointment.json` - JSON template used when creating events.
- `appointment.xlsx` - The Excel file (not included) that contains event rows. The script expects a sheet named `Termine` with the columns described below.
- `sample_appointments.csv` - A small CSV example demonstrating the expected columns and types for the Excel file.
- `requirements.txt` - Python dependencies to install.

Quick setup
-----------

1. Create a Python virtual environment and activate it (macOS / zsh):

```bash
python3 -m venv .venv
source .venv/bin/activate
```

2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Prepare event data.

The script expects an Excel file named `appointment.xlsx` with a sheet named `Termine` and the following columns:

- `title` (string) — title of the event
- `begin` (datetime) — start of event in a format pandas recognizes, e.g. `2025-11-13 18:30:00`
- `duration` (integer — minutes) — length of event in minutes
- `place` (string) — location
- `description` (string) — event description

You can convert the included `sample_appointments.csv` into `appointment.xlsx` with pandas if you like:

```bash
python - <<'PY'
import pandas as pd
df = pd.read_csv('sample_appointments.csv')
df.to_excel('appointment.xlsx', sheet_name='Termine', index=False)
print('Created appointment.xlsx with sheet `Termine`')
PY
```

Alternatively, create an Excel file manually with the `Termine` worksheet and matching column names.

Configure the API key
---------------------

The script reads the API key from the environment variable `API_KEY`.

Set it in your shell before running (zsh example): or add it to a `.env` file if using `python-dotenv`.

```bash
export API_KEY="your_webling_api_key_here"
export BASE_LINK="your_webling_base_link_here"
```

Add `appointment.json`
----------------------
Create an `appointment.json` file in the repository root. You can use the following minimal example as a starting point (similar to sample_appointment.json):

```json
{
  "properties": {
	"title": "",
	"begin": "",
	"end": "",
	"duration": 0,
	"place": "",
	"description": ""
  }
}
```

Run the script
--------------

To run the script and post the events to the API:

```bash
python main.py
```

What the script does
---------------------

- Loads `appointment.json` as a template
- Loads rows from `appointment.xlsx` (`Termine` sheet)
- For each row, fills the `appointment.json` template with the row values and posts the created JSON to the API endpoint in `main.py`
- The script prints the generated JSON and prints a response from the server (or a failed status code)

Dry-run mode (no external network requests)
------------------------------------------

If you'd like to generate the JSON for all rows and see the payload without sending any HTTP requests, use this small snippet. It loads `appointment.json` and `appointment.xlsx` and prints the JSON payload for each row so you can inspect it locally.

```bash
python - <<'PY'
import json
import pandas as pd
from datetime import timedelta

with open('appointment.json','r') as f:
	json_data = json.load(f)

df = pd.read_excel('appointment.xlsx', sheet_name='Termine')

def set_params(row, json_template):
	j = json.loads(json.dumps(json_template))
	begin_time = pd.to_datetime(row['begin'])
	end_time = begin_time + pd.Timedelta(minutes=int(row['duration']))
	j['properties']['title'] = row['title']
	j['properties']['begin'] = begin_time.strftime('%Y-%m-%d %H:%M:%S')
	j['properties']['end'] = end_time.strftime('%Y-%m-%d %H:%M:%S')
	j['properties']['duration'] = int(row['duration'])
	j['properties']['place'] = row['place']
	j['properties']['description'] = row['description']
	return j

for _, row in df.iterrows():
	payload = set_params(row, json_data)
	print(json.dumps(payload, indent=2, ensure_ascii=False))
PY
```

Notes and troubleshooting
-------------------------

- FileNotFoundError: `appointment.xlsx` — verify the file exists and has the `Termine` sheet.
- `pandas.errors.EmptyDataError` or parse errors — ensure `begin` column is in a recognizable datetime format.
- HTTP errors — ensure `API_KEY` is valid and network access is enabled. If you want to test without posting to production, use the dry-run steps above.

Security
--------

Don’t commit your `API_KEY` to version control. Use environment variables or a secure credentials manager.

Contributions
-------------

If you'd like to add features (e.g. a true `--dry-run` flag, logging, parallel requests, or error handling) please open a PR.

License
-------

See the `LICENSE` file in this repository.
