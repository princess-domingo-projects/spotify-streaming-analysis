<center><h1>Spotify Streaming Analysis Python Project</h1></center>
<h2>Contents</h2>
<ul>
  <li><a href="#introduction">Introduction</a>
  <li><a href="#libraries">Import Required Libraries</a>
  <li><a href="#explore">Explore the Dataset</a>
  <li><a href="#spotifywebapi">Spotfiy Web API</a>
  <li>Exploratory Analysis and Insights
  <li>Hypotheses for 2025
  <li>Diagnostic Analysis of Spotify API Integration
</ul>

<br>
<h1><a name="introduction">Introduction</a></h1>
<p>
  As an avid music lover, Spotify has become an integral part of my daily routine. 

  The purpose of my project is to identify trends in my streaming behaviour for the past 2 years (2023 and 2024).

  In this project, we will be measuring the following:
  <ul>
    <li>Total Days on Spotify
    <li>Average Hours on Spotify per Day
    <li>Streaming Behaviour throughout the Day
    <li>Change in Spotify Usage Year-on-Year
    <li>Top Genres, Artists & Songs
    <li>Total New Artists (2024)
    <li>The relationship between popularity and music preferences
    <li>Listening Streaks - Artists & Songs
    <li>The relationship between seasonality and genre preference evoluation
    <li>The success of genre attribution from Spotify API 
</ul>

  Spotify offer a free download of historic streaming data via <a href="https://www.spotify.com/uk/account/privacy/"> Spotify | Account Privacy</a><p>

Please feel free to reach out with any questions or suggestions at <a href="https://www.linkedin.com/in/princess-d-491a3b182/"> LinkedIn | Princess Domingo
 </p>

 <ul><h3>Tools Used:</h3>
 <li>Programming Language: Python</li>
 <li>Libraries: pandas, numpy, requests, seabo:rn, dotenv</li>
 <li>Modules: re, time, os, matplotlib.pyplot, warnings</li>
 <li>IDE: VS Code </li>
 </ul>

<br>
<h1><a name="libraries">Import Required Libaries</a></h1>

```python
import pandas as pd
import numpy as np
import re
import requests
import time
import os
import seaborn as sns
import matplotlib.pyplot as plt
from dotenv import load_dotenv
import warnings
warnings.filterwarnings('ignore')
```

 <ul>
  <li><code>import pandas as pd</code>: This imports the library Pandas under the alias pd. Pandas can be used for data analysis and manipulation and provides data in a tabular structure like a DataFrame.</li>
  <li><code>import numpy as np</code>: This import the library NumPy under the alias np. NumPy is used for the numerical computation and supports arrays and matrices.</li>
  <li><code>import re</code>: The module 're' is used for regular expression matching operations similar to Perl.</li>
  <li><code>import requests</code>: The module 'requests' is used to send HTTP requests through Python. This will be the way we interact with the Spotify API.</li>
  <li><code>import time</code>: The module 'time' is used to manage time-related tasks. </li>
  <li><code>import os</code>: This is a Python module that provides function for interacting with the operation system. It enables us to interact with environment variables like the API client ID and secret key.</li>
  <li><code>import seaborn as sns</code>: This is a data visualisation library built on top of matplotlib. It's useful for identifying relationships between variables.</li>
  <li><code>import matplotlib.pyplot as plt</code>: This is a data visualisation library that builds static, animated and interactive visualisations like line plots, bar charts and histograms.</li>
  <li><code>from dotenv import load_dotenv</code>: This is a library used to read key-value pairs from .env files.</li>
  <li><code>warnings.filterwarnings('ignore')</code>: This module enables us to manage warnings in the code that aren't necessarily errors but highlights depreciated feaures or non-critical issues.</li>
</ul>

<br>
<h1><a name="explore">Exploring the Dataset</a></h1>
<p>Spotify's streaming history data is delivered to users in JSON format</p>

```python
# Add a snippet of the Spotify streaming history JSON data
json_data = [
    {
        "endTime" : "2023-01-24 07:20",
        "artistName" : "Jay Electronica",
        "trackName" : "Fruits Of The Spirit",
        "msPlayed" : 24760
    },
    {
        "endTime" : "2023-01-24 07:24",
        "artistName" : "Kali Uchis",
        "trackName" : "I Wish you Roses",
        "msPlayed" : 244123
    },
    {
        "endTime" : "2023-01-24 07:28",
        "artistName" : "SZA",
        "trackName" : "Snooze",
        "msPlayed" : 215985
    }

# Import all JSON files
spot_1 = pd.read_json('spotify_streaming_history_2023.json')
spot_2 = pd.read_json('spotify_streaming_history_2023_1.json')
spot_3 = pd.read_json('spotify_streaming_history_2024.json')
spot_4 = pd.read_json('spotify_streaming_history_2024_1.json')

# Create a function to view each DataFrame's metadata and structure
def view_metadata(df):
    print('statistical summary:')
    print(df.describe().transpose())
    print('DataFrame size:', df.shape)
    print('metadata:')
    print(df.info())
    print('First five DataFrame rows:')
    print(df.head())

view_metadata(spot_1)
```
<h6>Output:</h6>
<img width="500" alt="Coding" src="https://github.com/princess-domingo-projects/spotify-streaming-analysis/blob/main/Screenshot%202024-12-25%20100817.png">
<img width="500" alt="Coding" src="https://github.com/princess-domingo-projects/spotify-streaming-analysis/blob/main/Screenshot%202024-12-25%20100850.png">

<h3>Data Cleaning & Transformation</h3>

```python
# Create 2 new DataFrames for the 2023 and 2024 data respectively.
hist_2023 = pd.concat([spot_1, spot_2], axis=0)
hist_2024 = pd.concat([spot_3, spot_4], axis=0)

# I want to change the columns from camelCase to snake_case but this step can be skipped
hist_2023.columns = [re.sub(r'(?<!^)(?=[A-Z])', '_', col).lower() for col in hist_2023.columns]
hist_2024.columns = [re.sub(r'(?<!^)(?=[A-Z])', '_', col).lower() for col in hist_2024.columns]
print('Spotify 2023 columns:', hist_2023.columns )
print('Spotify 2024 columns:', hist_2024.columns )
```
<h6>Output:</h6>
<img width="500" alt="Coding" src="https://github.com/princess-domingo-projects/spotify-streaming-analysis/blob/main/Screenshot%202024-12-25%20193526.png">

```python
# When reviewing the metadata for the streaming history, I can see there is some overlap in data.
# I want to know the earliest and latest dates recorded for each DataFrame
# so I can delete any duplicates, if neccessary.

def check_dates(df):
    # Create a new date column
    df['date'] = pd.to_datetime(df['end_time'].str[:10])
    print(f'The earliest recorded date is {df['date'].min().strftime('%Y-%m-%d')}.')
    print(f'The larest recorded date is {df['date'].max().strftime('%Y-%m-%d')}.')

check_dates(hist_2023)
check_dates(hist_2024)
```
<h6>Output:</h6>
<p>Dates for 2023 DataFrame</p>
<img width="500" alt="Coding" src="https://github.com/princess-domingo-projects/spotify-streaming-analysis/blob/main/Screenshot%202024-12-25%20194251.png">
<h6>Output:</h6>
<p>Dates for 2024 DataFrame</p>
<img width="500" alt="Coding" src="https://github.com/princess-domingo-projects/spotify-streaming-analysis/blob/main/Screenshot%202024-12-25%20193555.png">

```python
# Omit any data that overlaps 
clean_2023 = hist_2023[hist_2023['date'] < '2024-01-01']
clean_2024 = hist_2024[hist_2024['date'] > '2024-01-01']

check_dates(clean_2023)
check_dates(clean_2024)
```
<h6>Output:</h6>
<img width="500" alt="Coding" src="https://github.com/princess-domingo-projects/spotify-streaming-analysis/blob/main/Screenshot%202024-12-25%20193619.png">

```python
# Now that we are confident there is no overlapping data
# you can concat all spotify history into 1 DataFrame for easier handling 
full_hist = pd.concat([clean_2023, clean_2024], axis=0)
full_hist.head(10)
```
<h6>Output:</h6>
<img width="500" alt="Coding" src="">

```python
# For this analysis, I will be using various datetime aggregations so will need to create some extra columns 
def add_datetime_objects(df):
    # Month column 
    df['month'] = pd.to_datetime(df['date'].dt.to_period('M').dt.start_time.dt.strftime('%Y-%m-%d'))

    # Quarter column 
    df['quarter'] = pd.to_datetime(df['date'].dt.to_period('Q').dt.start_time.dt.strftime('%Y-%m-%d'))

    # Year column 
    df['year'] = df['date'].dt.year

    # Streaming hour
    df['end_time'] = pd.to_datetime(df['end_time'])
    df['stream_hour'] = df['end_time'].dt.hour

    return df
full_hist = add_datetime_objects(full_hist)
```
<h6>Output:</h6>
<img width="500" alt="Coding" src="https://github.com/princess-domingo-projects/spotify-streaming-analysis/blob/main/Screenshot%202024-12-25%20111754.png">
<br>

<h1><a name="spotifywebapi">Spotfiy Web API</a></h1>
<p> I would like more qualiative data for the artists and songs in my streaming history so I will be connecting to the Spotify Web API.
We will first get genres and the number of followers of each unique artist.</p>

```python
# Create a DataFrame for all unique artists
unique_artists = full_hist[['artist_name']].drop_duplicates()

# Create a smaller DataFrame that will be used to test the API functionality
unique_artists_small = unique_artists.head(10)

# Set up the API connectoon to generate the associated genre to each artist
spotify_id = os.getenv("spotify_client_id") # Replace with own client ID
spotify_secret = os.getenv("spotify_client_secret") # Replace with own secret key

# Spotify API Endpoints
auth_url = "https://accounts.spotify.com/api/token"
search_url = "https://api.spotify.com/v1/search"

# Authenticate and get access token 
def get_access_token(client_id, client_secret):
    response = requests.post(auth_url, {
        "grant_type": "client_credentials",
        "client_id": client_id,
        "client_secret": client_secret
    })
    response_data = response.json()
    return response_data.get("access_token")

# Fetch access token
access_token = get_access_token(spotify_id, spotify_secret)
if not access_token:
    raise Exception("Failed to retrieve access token. Check your credentials.")

# Set headers for API requests
headers = {"Authorization": f"Bearer {access_token}"}

# Get the associated genres for each artist
def get_artist_info(artist_name):
    params = {"q": artist_name, "type": "artist", "limit": 1}
    try:
        response = requests.get(search_url, headers=headers, params=params)
        response.raise_for_status()
        artist_data = response.json()
        if artist_data["artists"]["items"]:
            artists = artist_data["artists"]["items"][0]
            return {
                  "genres": artists.get("genres", []),
                  "followers": artists["followers"].get("total", ''),
                  "artist_popularity": artists.get("popularity", '')
            }
        return None
    except requests.exceptions.RequestException as e:
        print(f"Error fetching artist info for '{artist_name}': {e}")
        return None

# Add the function to the artist dataset
def process_artist_dataset(df, batch_size=150):
    processed_file = "spotify_api_artist_info.csv"

   # Ensure the necessary columns exist
    required_columns = ["genres", "followers", "artist_popularity"]

    for col in required_columns:
      if col not in df.columns:
          df[col] = None

    df.reset_index(drop=True, inplace=True)

    for i in range(len(df)):
        artist_name = df.at[i, "artist_name"]

        # Fetch artist information
        artist_info = get_artist_info(artist_name)
        if artist_info:
            df.at[i, "genres"] = artist_info.get("genres")
            df.at[i, "followers"] = artist_info.get("followers")
            df.at[i, "artist_popularity"] = artist_info.get("artist_popularity")

        # Save progress in every batch
        if i % batch_size == 0 or i == len(df) - 1:
            df.to_csv(processed_file, index=False)
            print(f"Saved progress: {i + 1}/{len(df)} rows processed.")

        # Throttle requests to avoid hitting API limits
        time.sleep(1)

    return df

# Test the data API functionality first
unique_artists_small = process_artist_dataset(unique_artists_small)
```

```python
# Run the API on the full dataset
unique_artists = process_artist_dataset(unique_artists)

# The average batch processing time is ~3 minutes per 150 rows for dataset with ~2000 rows.
```

<h6>Output:</h6>
<img width="500" alt="Coding" src="https://github.com/princess-domingo-projects/spotify-streaming-analysis/blob/main/Screenshot%202024-12-25%20110613.png">

 <ul>
  <li><code>genres</code>: This returns a string of genres associated to the artist.</li>
  <li><code>followers</code>: This returns the number of followers of the artist's page. NOTE: the followers count is a few days in arreas.</li>
  <li><code>artist_popularity</code>: This determines the popularity of the artist on a scale of 0-100 where 0 is least popular and 100 is most popular.</li>
</ul>

<p> More information on what data can be extracted from the Spotify Search API can be found <a href="https://developer.spotify.com/documentation/web-api/reference/search">here.</a></p>
<br>

<h1><a name="exploratoryanalysis">Exploratory Analysis and Insights</a></h1>

<h4> 1. How many days did I spend on Spotify in 2023 and 2024? </h4>

<h4> 1b. What was the year-on-year change in Spotify consumption? </h4>

<h4> 2. What was the highest recorded listening day in 2023 and 2024? </h4>

<h4> 3. What were the average hours spent on Spotify over a daily, weekly, monthly basis in 2023 and 2024? </h4>

<h4> 3b. How did the average hours spent using Spotify change YoY? </h4>

<h4> 4. Explore the top sub-genres over the last 2 years</h4>

```python
# Get all unique sub-genres 
all_genres = [genre for sublist in unique_artists['genres'] for genre in sublist]

# Get the count of each genre
genre_counts = Counter(all_genres)

# Create the word cloud
wordcloud = WordCloud(width=800, height=400, background_color='white').generate_from_frequencies(genre_counts)

# Display the word cloud using matplotlib
plt.figure(figsize=(10, 5))
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis('off')
plt.show()
```
<h6>Output</h6>
<img width="500" alt="Coding" src="https://github.com/princess-domingo-projects/spotify-streaming-analysis/blob/main/all-sub-genre-preferences.png">
<br>

<h4> 5. What were the top genres in 2023 and 2024?</h4>

<h4> 5b. How have my genre preferences changed quarter-on-quarter and year-on-year? </h4>

<h4> 5c. What were the top genres on the highest recorded listening day for 2023 and 2024? </h4>

<h4> 10. The relationship between an artist's popularity and discovery of their associated genres. </h4>

```python
sns.histplot(data=df2, x="artist_popularity", bins=10, hue='genre_found_via_api')
plt.show()
```
<img width="500" alt="Coding" src="https://github.com/princess-domingo-projects/spotify-streaming-analysis/blob/main/artist-popularity-genre-attribution.png">



