import streamlit as st
import requests
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime
from matplotlib.patches import Patch

# 🔑 Your OpenWeatherMap API Key
API_KEY = "5c2f32ea7b78d92d88ec52982dd7fb75"

# 🌍 City dropdown with coordinates
city_coords = {
    "Chennai": (13.08, 80.27),
    "Delhi": (28.61, 77.21),
    "Mumbai": (19.07, 72.87),
    "Kolkata": (22.57, 88.36),
    "Bengaluru": (12.97, 77.59),
    "Hyderabad": (17.38, 78.48),
    "Ahmedabad": (23.02, 72.57),
    "Pune": (18.52, 73.85),
    "Jaipur": (26.91, 75.79),
    "Lucknow": (26.85, 80.95)
}

# 🧭 UI — Title and Dropdown
st.set_page_config(page_title="Air Quality Dashboard", layout="wide")
st.title("🌫️ Air Quality Dashboard (Current & Forecast)")
city = st.sidebar.selectbox("📍 Select a City", list(city_coords.keys()))
lat, lon = city_coords[city]

# 🌐 API URLs
current_url = f"https://api.openweathermap.org/data/2.5/air_pollution?lat={lat}&lon={lon}&appid={API_KEY}"
forecast_url = f"https://api.openweathermap.org/data/2.5/air_pollution/forecast?lat={lat}&lon={lon}&appid={API_KEY}"

# 🔴 Fetch current AQI
current_data = requests.get(current_url).json()

if "list" not in current_data:
    st.error("❌ Failed to fetch data. Check API key or city coordinates.")
    st.stop()

aqi = current_data["list"][0]["main"]["aqi"]
components = current_data["list"][0]["components"]

st.subheader(f"🟢 Current Air Quality in {city}")
st.write(f"**AQI Index (1 = Good, 5 = Very Poor):** {aqi}")
st.write("Pollutant Levels (µg/m³):")
st.json(components)

# 🔮 Fetch forecast data
forecast_data = requests.get(forecast_url).json()
forecast_list = forecast_data["list"]
data_rows = []

for entry in forecast_list:
    dt = datetime.fromtimestamp(entry["dt"])
    aqi = entry["main"]["aqi"]
    pm25 = entry["components"]["pm2_5"]
    pm10 = entry["components"]["pm10"]
    no2 = entry["components"]["no2"]
    o3 = entry["components"]["o3"]
    data_rows.append([dt, aqi, pm25, pm10, no2, o3])

df_forecast = pd.DataFrame(data_rows, columns=["DateTime", "AQI", "PM2_5", "PM10", "NO2", "O3"])

# 🟨 AQI Color Mapper
def get_color(aqi_value):
    return {
        1: "green",
        2: "yellow",
        3: "orange",
        4: "red",
        5: "purple"
    }.get(aqi_value, "gray")

colors = df_forecast["AQI"].apply(get_color)

# 📈 Plot AQI Forecast Chart
st.subheader(f"📈 Forecasted AQI for {city} (Color-Coded)")
fig, ax = plt.subplots(figsize=(10, 4))
ax.scatter(df_forecast["DateTime"], df_forecast["AQI"], c=colors, s=50)
ax.plot(df_forecast["DateTime"], df_forecast["AQI"], color="gray", alpha=0.3)
ax.set_title(f"AQI Forecast - {city}")
ax.set_xlabel("Date & Time")
ax.set_ylabel("AQI (1 = Good to 5 = Very Poor)")
ax.grid(True)
plt.xticks(rotation=45)

# 📊 Add AQI Legend
legend_elements = [
    Patch(facecolor='green', label='Good (1)'),
    Patch(facecolor='yellow', label='Fair (2)'),
    Patch(facecolor='orange', label='Moderate (3)'),
    Patch(facecolor='red', label='Poor (4)'),
    Patch(facecolor='purple', label='Very Poor (5)')
]
ax.legend(handles=legend_elements, title="AQI Levels")

st.pyplot(fig)

# 📄 Forecast Data Table
st.subheader("📄 Forecast Data Table")
st.dataframe(df_forecast)
