# PythonAPI-Challenge

#### Dependencies and Setup ####

# Initial Import
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
import requests

# Import API key
from api_keys import api_key

# Incorporated citipy to determine city based on latitude and longitude
from citipy import citipy


#### Generate Cities List ####

# Generate 600 cities with randomly chosen latitude and longitudes
n = 600
cities = []


for x in range(n):
    lat = np.random.uniform(-90, 90) # Range of latitudes and longitudes
    lng = np.random.uniform(-180, 180) 
    city = citipy.nearest_city(lat, lng) # Identify nearest_city in the corresponding latitudes and longitudes
    
    # If the city is unique, then add it to a our cities list
    while city.city_name in cities:
        lat = np.random.uniform(-90, 90)
        lng = np.random.uniform(-180, 180)
        city = citipy.nearest_city(lat, lng)
    
    # Add the data to our lists
    cities.append(city.city_name)

#### Perform API Calls ####

# Create base URL and indicate imperial units
url = "http://api.openweathermap.org/data/2.5/weather?"
units = "imperial"

# Build partial query URL
query = f"{url}appid={api_key}&units={units}&q="

# Placeholder for API Calls
latitude = []
longitude = []
temperature = []
humidity = []
cloudiness = []
wind_speed = []

# Extract temperature, humidity, cloudiness and wind speed
for x in range(len(cities)):
    response = requests.get(f"{query}{cities[x]}").json()
    
    # Some of the cities we generate don't have data in openweathermap, so set their values to numpy's NaN
    try:
        temperature.append(response['main']['temp_max'])
        latitude.append(response['coord']['lat'])
        longitude.append(response['coord']['lon'])
        humidity.append(response['main']['humidity'])
        wind_speed.append(response['wind']['speed'])
    except KeyError:
        temperature.append(np.nan)
        latitude.append(np.nan)
        longitude.append(np.nan)
        humidity.append(np.nan)
        wind_speed.append(np.nan)
    
    # if "clouds" do not exist, then append the value to "0"
    try:
        cloudiness.append(response['clouds']['all'])
    except KeyError:
        cloudiness.append(0)
        
    # Print each city name and query string    
    print(f"Processing record {x + 1} | {cities[x]}")
    print(f"{query}{cities[x]}")

print("--------------------------------------------")
print("End of API Call")
print("--------------------------------------------")

                         

#### Clean-Up the Data Frame ####

# Assemble everything into a data frame
cities_df = pd.DataFrame({"City": cities,
                           "Latitude": latitude,
                           "Longitude": longitude,
                           "Humidity": humidity,
                           "Max Temp": temperature,
                           "Cloudiness": cloudiness,
                           "Wind Speed": wind_speed, })

cities_df=cities_df.rename(columns={0:"City"})
cities_df["City"]=cities_df["City"].str.capitalize()
cities_df=cities_df.dropna(how='any') #Drop any NA Value

ncities=len(cities_df)

print(f"There are",ncities,"cities, which is above the required number of 500 cities")

# Safe as CSV file
cities_df.to_csv("cities_df.csv", index=False, header=True)


#### Graphics ####

# 01: Temperature (F) vs. Latitude

plt.scatter(x = cities_df['Latitude'], y = cities_df['Max Temp'], marker="x",
             alpha=0.75, edgecolors = 'black', facecolor = 'maroon', s=30)
plt.grid()

plt.title("Temperature (F) vs. Latitude")
plt.xlabel("Latitude")
plt.xlim(-80,100)
plt.ylabel("Temperature (F)")
plt.savefig("tempvslat.png")
plt.show()

# 02: Humidity (%) vs. Latitude

plt.scatter(x = cities_df['Latitude'], y = cities_df['Humidity'], marker="x",
             alpha=0.75, edgecolors = 'black', facecolor = 'saddlebrown', s=30)

plt.grid()
plt.title("Humidity (%) vs. Latitude")
plt.xlabel("Latitude")
plt.xlim(-80,100)
plt.ylabel("Humidity (%)")
plt.savefig("humidvslat.png")
plt.show()


# 03: Cloudiness (%) vs. Latitude

plt.scatter(x = cities_df['Latitude'], y = cities_df['Cloudiness'], marker="x",
             alpha=0.75, edgecolors = 'black', facecolor = 'darkblue', s=30)
plt.grid()
plt.title("Cloudiness (%) vs. Latitude")
plt.xlabel("Latitude")
plt.xlim(-80,100)
plt.ylabel("Cloudiness (%)")
plt.savefig("cloudvslat.png")
plt.show()

# 04: Wind Speed (mph) vs. Latitude

plt.scatter(x = cities_df['Latitude'], y = cities_df['Wind Speed'], marker="x",
             alpha=0.75, edgecolors = 'black', facecolor = 'green', s=30)
plt.grid()
plt.title("Wind Speed (mph) vs. Latitude")
plt.xlabel("Latitude")
plt.xlim(-80,100)
plt.ylabel("Wind Speed (mph)")
plt.savefig("windvslat.png")
plt.show()
