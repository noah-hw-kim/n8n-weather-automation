# Daily Weather Automation (n8n + Supabase + OpenWeather)

This workflow runs once per day to automate weather updates. It fetches data from **OpenWeather**, generates a formatted summary with custom alerts, logs the execution to **Supabase**, and sends an update via **email**.

---

## 1. Setup

### OpenWeather API
1. Create an account at [OpenWeatherMap](https://openweathermap.org).
2. Generate an **API key**.
3. The workflow targets the following endpoint:  
   `https://api.openweathermap.org/data/2.5/weather?q=<CITY>&appid=<KEY>&units=metric`

### Supabase
1. Create a new Supabase project.
2. Create a table named `weather_logs` with the following schema:

| Column | Type |
| :--- | :--- |
| `run_at` | `timestamptz` |
| `city` | `text` |
| `temperature` | `float4` |
| `temperature_unit` | `text` |
| `condition` | `text` |
| `humidity` | `int` |
| `wind_speed` | `float4` |
| `alert_type` | `text` |
| `alert_message` | `text` |
| `summary` | `text` |
| `raw_response` | `jsonb` |

3. **Security:** Enable Row Level Security (RLS). Use the `service_role` key in n8n or apply an insert policy:
```sql
create policy "Allow all inserts"
on public.weather_logs
for insert
to public
with check (true);
```

### Email (SMTP) Setup
You can use Gmail with an App Password or any SMTP provider. For Gmail, use the following configuration:

* **Host:** `smtp.gmail.com`
* **Port:** `587` (TLS off)
* **User:** Your Gmail address
* **Password:** Your App Password (generated in Google Account settings)

---

## 2. How the Workflow Works

* **Schedule Trigger:** Runs once per day (7 am).
* **Set Cities & Units:** Defines the list of cities and temperature units.
* **Split Out:** Turns the city list into separate workflow items.
* **Normalize City Field:** Ensures city names are formatted correctly before calling the API.
* **Weather API Call:** Fetches current weather for each city.
* **Code (JavaScript):** Processes raw data into a structured format:
    * **Precipitation Alerts:** Specifically identifies **Rain, Snow, Drizzle, or Storms**.
    * **Temperature Alerts:** Flags **Heat** (Above 32°C) or **Frost** (Below 0°C).
    * **Metrics:** Calculates "Feels Like" temperature alongside standard readings.
    * **Summary Generation:** Joins all metrics into a clean, multi-line string for the email body.
* **Insert to Supabase:** Saves a timestamped log including alerts and the raw JSON response.
* **Send Weather Email:** Dispatches a daily update with the city-specific summary and alerts.

---

## 3. Importing the Workflow

1. In **n8n**, create a new workflow.
2. Click **Import** and select `weather_workflow.json`.
3. Add or update **Credentials**:
    * **OpenWeather API:** Your API key.
    * **Supabase API:** Your Service Role key and Project URL.
    * **SMTP:** Your email provider details.
4. **Activate** the workflow to start the daily schedule.

---

## 4. Included Files

* `weather_workflow.json` – Exported n8n workflow.
* `Screenshots/` – Visuals of the workflow nodes and sample email outputs.
