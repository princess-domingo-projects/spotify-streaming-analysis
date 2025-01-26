<center><h1>Spotify Streaming Analysis - Tableau & Python Project</h1></center>
<h2>Contents</h2>
<ul>
  <li><a href="#introduction">Introduction</a>
  <li><a href="#libraries">Import Required Libraries</a>
  <li><a href="#transform">Data Cleaning & Transforming</a>
  <li><a href="#spotifywebapi">Spotfiy Web API: Artist Discovery</a>
  <li><a href="#genreassociation">Genre Association 
</ul>

<br>
<h1><a name="introduction">Introduction</a></h1>
<p>
  This project is commemorate my first full year in Data. Spotify is known for its incredibly rich data and their Spotify API documentation is super indepth so I decided to create own (but not quite) Spotify Wrapped. I developed a custom API that can associate an artist to a string of genres and then further classify them into umbrellas. The purpose of this analysis is to identify trends in my streaming behaviour over the last two years (2023 and 2024). I have chosen <a href="https://public.tableau.com/app/profile/princess.domingo/viz/SpotifyProjectv1_17378454361870/Dashboard22#1">Tableau</a> as my BI tool due to its interactivity and flexible parameter building.
</ul>

Spotify offer a free download of historic streaming data via <a href="https://www.spotify.com/uk/account/privacy/"> Spotify | Account Privacy</a><p>

Please feel free to reach out with any questions on the <a href="https://public.tableau.com/app/profile/princess.domingo/viz/SpotifyProjectv1_17378454361870/Dashboard22#1">dashboard</a> or suggestions at <a href="https://www.linkedin.com/in/princess-d-491a3b182/"> LinkedIn | Princess Domingo
 </p>

 <ul><h3>Tools Used:</h3>
 <li>Programming Language: Python</li>
 <li>Libraries: pandas, numpy, requests, dotenv</li>
 <li>Modules: re, time, os, warnings, logging, datetime, calendar</li>
 <li>IDE: VS Code </li>
 <li>Visualisation tool: Tableau</li>
 </ul>

<br>
<h1><a name="libraries">Import Required Libaries</a></h1>

```python
# Required packages/libraries/modules
import pandas as pd 
import numpy as np 
import os
import re
import time 
import requests
import warnings
warnings.filterwarnings('ignore')
import logging
from datetime import datetime
import calendar

```
 <ul>
  <li><code>import pandas as pd</code>: This imports the library Pandas under the alias pd. Pandas can be used for data analysis and manipulation and provides data in a tabular structure like a DataFrame.</li>
  <li><code>import numpy as np</code>: This import the library NumPy under the alias np. NumPy is used for the numerical computation and supports arrays and matrices.</li>
  <li><code>import re</code>: The module 're' is used for regular expression matching operations similar to Perl.</li>
  <li><code>import requests</code>: The module 'requests' is used to send HTTP requests through Python. This will be the way we interact with the Spotify API.</li>
  <li><code>import time</code>: The module 'time' is used to manage time-related tasks. </li>
  <li><code>import os</code>: This is a Python module that provides function for interacting with the operation system. It enables us to interact with environment variables like the API client ID and secret key.</li>
  <li><code>from dotenv import load_dotenv</code>: This is a library used to read key-value pairs from .env files.</li>
  <li><code>warnings.filterwarnings('ignore')</code>: This module enables us to manage warnings in the code that aren't necessarily errors but highlights depreciated feaures or non-critical issues.</li>
  <li><code>import logging</code>: This module to records the execution of functions within the .py file.</li>
  <li><code>import calendar </code>: This module to provides calendar style formatting of datetime objects.</li>
  <li><code>import datetime </code>: This module enables the manipulation of datetime objects.</li>
</ul>

<br>
<h1><a name="transform"><h3>Data Cleaning & Transformation</a></h1>

```python
# Capture the current date
current_date = datetime.now().strftime('%Y-%m-%d')

# Set up a log file
log_file = f'spotify_api_log_notes_{current_date}.txt'

# Set up your logging configuration
logging.basicConfig(
    filename=log_file,
    filemode='w',
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s"
)

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
    }]

# Import the JSON files
df1 = pd.read_json('stream_2023.json')
df2 = pd.read_json('stream_2023_1.json')
df3 = pd.read_json('stream_2024.json')
df4 = pd.read_json('stream_2024_1.json')

# Create two new dataframes for 2023 and 2024 respectively
df5 = pd.concat([df1, df2], axis=0)
df6 = pd.concat([df3, df4], axis=0)

# Convert the columns from camelCase to snake_case 
df5.columns = [re.sub(r'(?<!^)(?=[A-Z])', '_', col).lower() for col in df5.columns]
df6.columns = [re.sub(r'(?<!^)(?=[A-Z])', '_', col).lower() for col in df6.columns]

def check_dates(df, year):
    df['date'] = pd.to_datetime(df['end_time'].str[:10])
    if year == 2023:
        year == 2023
    else: 
        year == 2024
    logging.info(f'The earliest recorded date in the {year} DataFrame is {df["date"].min().strftime("%Y-%m-%d")}.')
    logging.info(f'The latest recorded date in the {year} DataFrame is {df["date"].max().strftime("%Y-%m-%d")}.')

# Remove any overlapping data
df7 = df5[df5['date'] < '2024-01-01']
df8 = df6[df6['date'] >= '2024-01-01']

check_dates(df7, year=2023)
check_dates(df8, year=2024)

df10 = pd.concat([df8, df7], axis=0)
logging.info(f'Spotify data has been successfully concatenated.')

```
<h6>Explanation:</h6>
<p> I first verify that the data is not overlapping across both years. Once I'm confident it's been removed, per the log notes, I join the data together on the axis=0 or horizontally.</p>

<h6>Columns:</h6>
 <ul>
  <li><code>end_time</code>: The date and time the user stopped listening to the song</li>
  <li><code>artist_name</code>: The name of the artist</li>
  <li><code>track_name</code>: The name of the song </li>
  <li><code>ms_played</code>: The duration of the song being played in milliseconds </li>
</ul>
<br>

<h1><a name="spotifywebapi">Spotfiy Web API: Artist Discovery</a></h1>
<p> I would like more qualiative data for the artists and songs in my streaming history so I will be connecting to the Spotify Web API.
We will first get genres and the number of followers of each unique artist.</p>

```python
# The API needed to query unique artists. I recommend creating a DataFrame that captures all unique artists rather than running the entire DataFrame which will contain duplicates.

spotify_id = os.getenv("spotify_client_id") # Replace with own client ID
if spotify_id: 
    logging.info("Spotify ID found. Searching for secret key...")
else:
    logging.error("Spotify ID not found, required for API connection!")
spotify_secret = os.getenv("spotify_client_secret") # Replace with own secret key

if spotify_secret: 
    logging.info("Spotify secret key found.")
else:
    logging.error("Spotify ID not found, required for API connection!")

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

access_token = get_access_token(spotify_id, spotify_secret)

if access_token:
    logging.info("Credentials successfully verified.")
else:
    logging.error("Failed to retrieve access token. Check your credentials.")

# Set headers for API requests
headers = {"Authorization": f"Bearer {access_token}"}

# This is the function to find the relevant artist information that we want for the artist.
def get_artist_info(artist_name):
    # Set up the parameters; the query is the artist name.
    # The type is 'artist' to find artist information.
    # Look for 1 unique artist at a time.
    params = {"q": artist_name, "type": "artist", "limit": 1}
    try:
        response = requests.get(search_url, headers=headers, params=params)
        response.raise_for_status()
        artist_data = response.json()
        # If the JSON contains the key 'items', then return the relevant column values.
        if artist_data["artists"]["items"]:
            artists = artist_data["artists"]["items"][0]
            return {
                  "genres": artists.get("genres", []),
                  "followers": artists["followers"].get("total", ''),
                  "artist_popularity": artists.get("popularity", '')
            }
        return None
    except requests.exceptions.RequestException as e:
        # Where the artist isn't found, the log text will detail which artist failed.
        # Usually, it is unstable internet connection or special characters [%@^$] that can affect the success of the API.
        logging.warning(f"Error fetching artist info for '{artist_name}': {e}")
        return None
    

# The recommended batch is fifty (50). It takes an average of two (2) to three (3) minutes for one (1) batch.
# This can be in/decreased based on personal preferences.
def process_artist_dataset(df, batch_size=50):
    processed_file = "spotify_api_artist_info.csv"

   # Ensure the necessary columns exist
    required_columns = ["genres", "followers", "artist_popularity"]

    # If the value is found, add to the relevant column; else return None.
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

        # Save progress in every batch to add to the log file
        if i % batch_size == 0 or i == len(df) - 1:
            df.to_csv(processed_file, index=False)
            logging.info(f"Saved progress: {i + 1}/{len(df)} rows processed.")

        # Throttle requests to avoid hitting API limits
        time.sleep(1)

    return df

def add_spotify_info(df):

    if len(df) <= 10: 
        chunks = [df] 
    else:
        # Define the size of each split
        split_size = len(df) // 8

        # Split the DataFrame into 8 parts
        chunks = [
            df.iloc[i * split_size : (i + 1) * split_size] if i < 7 else df.iloc[i * split_size :]
            for i in range(8)
        ]

    # List to store processed chunks
    processed_chunks = []
    # Define the size of each split
    split_size = len(df) // 8
    
    # Split the DataFrame into 8 parts
    sf1 = df.iloc[:split_size]
    sf2 = df.iloc[split_size:2*split_size]
    sf3 = df.iloc[2*split_size:3*split_size]
    sf4 = df.iloc[3*split_size:4*split_size]
    sf5 = df.iloc[4*split_size:5*split_size]
    sf6 = df.iloc[5*split_size:6*split_size]
    sf7 = df.iloc[6*split_size:7*split_size]
    sf8 = df.iloc[7*split_size:]
    
    # List to store processed data
    processed_chunks = []
    
    # List of chunks
    chunks = [sf1, sf2, sf3, sf4, sf5, sf6, sf7, sf8]
    
    for i, chunk in enumerate(chunks):
        # Process the chunk
        processed_chunk = process_artist_dataset(chunk)
        processed_chunks.append(processed_chunk)
        
        # Wait for 2 minutes after every even-numbered chunk (sf2, sf4, etc.)
        if (i + 1) % 2 == 0 and i != len(chunks) - 1:
            logging.info(f"Processed chunk {i + 1}. Waiting for 2 minutes...")
            time.sleep(120)  # Wait for 2 minutes (120 seconds)
    
    # Combine all processed chunks into a single DataFrame
    df2 = pd.concat(processed_chunks, axis=0)
    
    return df2
```
<h6>Output:</h6>
<img width="500" alt="Coding" src="https://github.com/princess-domingo-projects/spotify-streaming-analysis/blob/main/Screenshot%202024-12-25%20110613.png">
 <ul>
  <li><code>genres</code>: This returns a string of genres associated to the artist.</li>
  <li><code>followers</code>: This returns the number of followers of the artist's page. NOTE: the followers count is a few days in arreas.</li>
  <li><code>artist_popularity</code>: This determines the popularity of the artist on a scale of 0-100 where 0 is least popular and 100 is most popular.</li>
</ul>
<br>

<p> More information on what data can be extracted from the Spotify Search API can be found <a href="https://developer.spotify.com/documentation/web-api/reference/search">here.</a></p>
<br>
<h1><a name="genreassociation">Genre Associations</a></h1>

```python
# Create a DataFrame for all unique artists
df12 = df10[['artist_name']].drop_duplicates()

df13 = add_spotify_info(df12)

umbrella_genre_map = {
    'indie' : 'Indie', 
    'pop' : 'Pop', 
    'drill' : 'Rap', 
    'rave' : 'EDM',
    'hip hop' : 'Hip Hop', 
    'r&b' : 'R&B', 
    'rap' : 'Rap', 
    'soul' : 'Soul', 
    'rock' : 'Rock', 
    'funk' : 'Funk', 
    'country' : 'Country',
    'punk' : 'Rock', 
    'grime' : 'Hip Hop', 
    'phonk' : 'Indie', 
    'techno': 'EDM', 
    'latin' : 'Latin', 
    'edm' : 'EDM', 
    'breakcore': 'EDM', 
    'dnb': 'EDM',
    'nigerian' : 'Afrobeats', 
    'afro': 'Afrobeats', 
    'jazz': 'Jazz', 
    'sierra leonean' : 'Afrobeats', 
    'plugg' : 'Rap', 
    'jungle' : 'EDM', 
    'psychedelic' : 'Psychedelic', 
    'metal' : 'Rock',
    'bass trip' : 'EDM', 
    'dancehall': 'Dancehall', 
    'house' : 'House', 
    'reggage' : 'Reggage', 
    'disco': 'Funk', 
    'native american' : 'World', 
    'amapiano' : 'Afrobeats', 
    'boom bap' : 'Rap', 
    'drum and bass' : 'EDM', 
    'electronic' : 'EDM', 
    'dubstep': 'EDM', 
    'garage': 'EDM', 
    'anime': 'World', 
    'japanese' : 'World', 
    'surf': 'Psychedelic', 
    'dance': 'EDM', 
    'bassline' : 'House', 
    'argentina' : 'Latin', 
    'jersey' : 'EDM', 
    'reggae' : 'Reggae', 
    'ghanaian': 'Afrobeats', 
    'azonto' : 'Afrobeats', 
    'gaze': 'Indie', 
    'kompa' : 'World', 
    'south african' : 'Afrobeats', 
    'corecore' : 'Indie', 
    'chillhop' : 'Hip Hop', 
    'zouk' : 'Afrobeats', 
    'ukg' : 'EDM', 
    'sertanejo' : 'Latin', 
    'clubbing' : 'EDM', 
    'sped up' : 'EDM', 
    'aussietronica' : 'EDM', 
    'chillwave': 'Indie', 
    'chinese' : 'World'

}

# Function to find umbrella genres
def find_umbrella_genres(genres, mapping):
    matched_genres = set()
    for genre in genres:
        for keyword, umbrella_genre in mapping.items():
            if keyword in genre.lower():  # Use lower() to make the search case-insensitive
                matched_genres.add(umbrella_genre)
    return list(matched_genres)

# Apply the function to create a new column
df13['umbrella_genres'] = df13['genres'].apply(
    lambda x: find_umbrella_genres(x, umbrella_genre_map) if x is not None else None
)

df13['primary_genre'] = df13['umbrella_genres'].apply(lambda x: x[0] if x else 'Other')
logging.info('Genres have been applied to the artist DataFrame. Calculating uncategorized artists....')

uncategorized_artists = df13[df13['primary_genre'].str.contains('Other')]
uncategorized_artists_list = uncategorized_artists['artist_name'].unique()
percent_total = round((len(uncategorized_artists) / len(df13))*100, 1)

if percent_total <= 30:
    logging.info(f'There are {len(uncategorized_artists)} artists with no associated genre, {percent_total}% of all artists.') 
    logging.info('A genre map is recommended for more accurate results.')
else:
    logging.info(f'There are {len(uncategorized_artists)} artists with no associated genre, {percent_total}% of all artists.') 

# 48% of my total artists were uncategorized so the genre association is a mix of ChatGPT results, memory and Google searching. This is not
# recommended if you have over 60% uncategorized artists.

genre_map = {
    'Chase Shakur': 'R&B', 'DC': 'Hip Hop', 'BreezyLYN': 'Hip Hop', 
    'Koto.': 'EDM', 'Mushkilla': 'Hip Hop', 'dexter in the newsagent': 'Indie', 
    'Honey Mooncie': 'Jazz', 'Mcbaise': 'Pop', 'Kamggarn': 'Indie', 'lucidbeatz': 'EDM', 
    'Ghoulavelii': 'Hip Hop', 'Domsta': 'Hip Hop', 'Ark Woods': 'Hip Hop',
    'Vector': 'Rock', 'daste.': 'Indie', 'Talktofranc': 'Hip Hop', 'niquo': 'EDM',
    'goddard.': 'EDM', 'DARGZ': 'Hip Hop', 'dedwrite': 'Hip Hop',
    '99GINGER': 'EDM', 'NOT THE TWOS': 'Indie', 'Ari Afsar': 'Pop',
    'WMG Lab Records': 'Pop', 'Mysie': 'Pop', 'The Drums': 'Indie',
    'Castro D’destroyer': 'Afro', 'Austin Millz': 'EDM',
    'Mathey': 'Afro', 'SEB': 'Pop', 'Michael Abraham': 'Hip Hop',
    'Pity Party (Girls Club)': 'Indie', 'Mind’s Eye': 'Rock', 'Cottonwood Firing Squad': 'Folk/Country',
    'Peyton': 'EDM', 'Alexa Demie': 'Pop', 'Michel Martelly': 'Afro', 'Sparky Deathcap': 'Indie',
    'Redbone': 'R&B', "Ni'jah": 'R&B', 'Fresh X Reckless': 'Hip Hop', 'CassKidd': 'Hip Hop',
    'Trevor Spitta': 'Hip Hop', 'Swsh': 'R&B', 'UG Vavy': 'Hip Hop',
    'T3': 'Hip Hop', 'Venna': 'Jazz', 'Billen Ted': 'EDM', 'Lil Vada': 'Hip Hop',
    'Sika Deva': 'Hip Hop', "Tre' Amani": 'Hip Hop', 'Saint Joshua': 'R&B',
    'Jontha Links': 'EDM', 'Jordan Hawkins': 'R&B', 'DIPS': 'R&B',
    'General Lee': 'Folk/Country', 'Mejiwahn': 'Jazz', 'Juvie': 'Hip Hop',
    'Ventry': 'Indie', 'GROOVY': 'Hip Hop', 'blondead': 'Hip Hop',
    'Grace Sorensen': 'R&B', 'ahmed Zou': 'Indie', 'Scribbles Who': 'R&B',
    'Eduardo Luzquiños': 'World', 'Crave Moore': 'Hip Hop', 'JJYuMusic': 'Experimental',
    'K Meta': 'Experimental', 'John Wells': 'Indie', 'Zombie Juice': 'Hip Hop',
    'Isaiah Falls': 'R&B', 'Dave $tokes': 'Hip Hop', 'Nudashe': 'Hip Hop', 
    'artistbasm': 'Hip Hop', 'EDMy coleman!': 'Hip Hop', 'Heno.': 'Jazz', 
    'Dinner Party': 'Indie', 'Quadroon': 'Indie', '3rd Wxrld': 'Hip Hop',
    'Take Van': 'Pop', 'Frank Ivy': 'Indie', 'Yellow Trash Can': 'Hip Hop', 
    'Arc De Soleil': 'Indie', 'Blu DeTiger': 'Indie', 'Gloria Tells': 'R&B',
    'Humble the Great': 'Hip Hop', 'Shenie Fogo': 'Indie', 'Marigold': 'Rock', 
    'Harvey Whyte': 'Hip Hop', 'Chillz': 'Afro', 'Hannymoon': 'Pop', 
    'Steph Ocho': 'Afro', 'Reapz': 'Hip Hop', 'BYELUH': 'Pop', 'WINTER ENDS': 'Experimental',
    'KOTU': 'EDM', 'King Jelz': 'Hip Hop', 'Nazio': 'Unknown',
    'Ty Savage': 'Afro', 'LONESTAR': 'Folk/Country', 'FERNE': 'Indie',
    'AYOCHILLMANNN': 'Hip Hop', 'Jazz Playaz': 'Jazz', 'YAZ': 'Indie',
    'ParanoidRapper': 'Hip Hop', 'Saiming': 'Hip Hop', 'Rizz Capolatti': 'Hip Hop',
    'Avelino': 'Hip Hop', 'ARTAN': 'Hip Hop', 'OTG': 'Hip Hop', 'BABYLON MAYNE': 'Hip Hop',
    'KiltKarter': 'Hip Hop', '8SZN': 'Hip Hop', 'RichGains': 'Hip Hop', 'JAYPROB': 'Hip Hop', '27Delly': 'Hip Hop',
    'Joony': 'Hip Hop', 'Paxslim': 'Hip Hop', 'Flower in Bloom': 'Indie', 
    'Kaelin Ellis': 'Hip Hop', 'Dejima': 'Rock', 'Lokowat': 'EDM', 'Miguel KR': 'Hip Hop', 
    'Mulatoh Prod': 'EDM', 'Pale Jay': 'R&B', 'Dare Devil': 'Rock', "MC's Zaac": 'Hip Hop',
    'Kelly Doyle': 'EDM', 'Ruben Moreno': 'EDM', 'strongboi': 'Hip Hop',
    'Blended Babies': 'Hip Hop', 'Chek': 'Indie', 'Brad stank': 'Indie', 'Langston Bristol': 'Hip Hop',
    'Oladipo': 'Hip Hop', 'CoryaYo': 'Hip Hop', 'LBL': 'Hip Hop', 'Wale the Sage': 'Hip Hop',
    'Tommy Guerrero': 'Indie', 'Andy Clockwise': 'Indie', 'Khi Infinite': 'Hip Hop', 
    'Derek Simpson': 'Indie', 'T. Evann': 'Indie', 'Honeywhip': 'Indie', 'Ambassadeurs': 'EDM', 
    'KINGDM': 'EDM', 'Jianbo': 'Jazz', 'SamRecks': 'Hip Hop', 'Mugzz': 'Hip Hop', 'Lil Seyi': 'Hip Hop', 
    'M-Beat': 'Hip Hop', 'No Guidnce': 'Hip Hop', 'Haraket': 'EDM', 'DJ Zone': 'EDM',
    'DOMi & JD BECK': 'Jazz', 'Martha Eve': 'Indie', 'FendiDa Rappa': 'Hip Hop', 'tomcbumpz': 'Hip Hop',
    '8Nights': 'Pop', 'KiLLOWEN': 'Hip Hop', 'Keys the Prince': 'Hip Hop', 'AntsLive': 'Hip Hop',
    'CLAVIS 7EVEN': 'Hip Hop', 'Jim Legxacy': 'Hip Hop', 'Sad Night Dynamite': 'Experimental',
    'Brandon Nembhard': 'Hip Hop', 'Banditt': 'Hip Hop', 'matt mcwaters': 'R&B',
    'Kindness': 'Hip Hop', 'Merges': 'Hip Hop', 'dialE': 'Hip Hop', 'COLDKiDCLUB': 'Hip Hop',
    'Blaize Jenkins': 'Hip Hop', 'Common Saints': 'R&B', 'Holy Hive': 'Folk/Country',
    'Kelz2busy': 'R&B', 'Fimiguerrero': 'Hip Hop', 'brazy': 'Hip Hop', 
    'Soft Powers': 'Indie', 'Joe Hisaishi': 'World', 'Lauren Faith': 'EDM', 
    'veggi': 'Hip Hop', 'In Deep We Trust': 'EDM', 'King Imprint': 'Hip Hop', 'Claud': 'Indie',
    'kostromin': 'Hip Hop', 'Vague Detail': 'R&B', 'Beach Party Music': 'Indie', 
    'How DBlack Do Dat': 'Hip Hop', 'R616U': 'Hip Hop', 'Mari.': 'Pop',
    'NBE': 'Hip Hop', 'OTF Zay': 'Hip Hop', 'DJ Yones': 'EDM', 
    'AFrmDaBank': 'Hip Hop', 'OGHollywood': 'Hip Hop', "Jumpin' Joe The Rapper": 'Hip Hop', 
    'NuNu': 'Hip Hop', 'OutTheWay_Jay': 'Hip Hop', 'Paperboy Fabe': 'Hip Hop',
    'arpsweatpants': 'Hip Hop', 'Ms. Ca$H': 'Hip Hop', 'Woo Da Savage': 'Hip Hop', 
    'Anycia': 'Hip Hop', 'DEELA': 'Hip Hop', 'Dj Akoza': 'EDM', 'DJ Acura': 'EDM',
    'Mula Axe': 'Hip Hop', 'Bbyafricka': 'Hip Hop', 'Zoeyy B.': 'Hip Hop', 'Vae Vanilla': 'Hip Hop', 
    'Alfonso Ramos': 'Hip Hop', 'Baby Monii': 'Hip Hop', 'Lanae': 'Hip Hop', 'FlowFly': 'Hip Hop',
    'Teya & Salena': 'Pop', 'Vincent Thiery': 'Jazz', 'Luckie_L': 'Indie', 'K.A.G': 'Hip Hop',
    'Minu': 'Indie', 'Makohna': 'Pop'
}

def clean_artist_data(df):
    df['new_genre'] = df.apply(
    lambda row: genre_map[row['artist_name']] if row['artist_name'] in genre_map else row['primary_genre'],
    axis=1)

    df['genre_found_by_api'] =  np.where(df['artist_name'].\
                            isin(uncategorized_artists_list), False, True)

    df2 = df.drop(['genres', 'umbrella_genres', 'primary_genre'], axis=1).rename(columns={'new_genre': 'genre'})

    df2.to_csv(f'spotify_artist_info_{current_date}.csv', index=False)

    return df2

df14 = clean_artist_data(df13)
```

