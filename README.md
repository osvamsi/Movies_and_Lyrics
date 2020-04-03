# Movies and Lyrics Project
- Introduction:
    - This  project scrapes information about movies and lyrics from the web and store them into your local database(SQLite)
    - Several features includes: 
      1. search for the movie.
      2. search for the lyrics.
      1. saving the lyrics to the text file
      2. Saving the database into excel sheet.
      3. saving the poster of your movie.
      4. print the contents of your database.
    - The skills that are required for this project are:
        1. python
        2. communicating with the web services using API calls
        3. creating and maintaining your database(SQLite)
        4. Handling JSON objects
        5. SQL
        6. Motivation :)
        
# The Flow Diagram of the system
![](/images/mini_project.PNG "Architecture")



## The Required modules
- urllib is the gateway from Python to web is done through urllib module. It is a Python module for fetching URLs (Uniform Resource Locators).
- json module allows us to handle JSON Objects.
-  re module is used for extracting data of certain pattern

```python
import urllib.request, urllib.parse, urllib.error
import json
import re
```


```python
with open('./lyrics_project/apikeys.json') as f:
    keys = json.load(f)
    omdbapi = keys['OMDBapi']
```


```python
serviceurl = 'http://www.omdbapi.com/?'
apikey = '&apikey='+omdbapi
```


```python
def print_json(json_data):
    list_keys=['Title', 'Year', 'Rated', 'Released', 'Runtime', 'Genre', 'Director', 'Writer', 
               'Actors', 'Plot', 'Language', 'Country', 'Awards', 'Ratings', 
               'Metascore', 'imdbRating', 'imdbVotes', 'imdbID']
    print("-"*50)
    for k in list_keys:
        if k in list(json_data.keys()):
            print(f"{k}: {json_data[k]}")
    print("-"*50)
```


```python
def save_poster(json_data):
    import os
    title = json_data['Title']
    poster_url = json_data['Poster']
    # Splits the poster url by '.' and picks up the last string as file extension
    poster_file_extension=poster_url.split('.')[-1]
    # Reads the image file from web
    poster_data = urllib.request.urlopen(poster_url).read()
        
    savelocation=os.getcwd()+'\\'+'Posters'+'\\'
    # Creates new directory if the directory does not exist. Otherwise, just use the existing path.
    if not os.path.isdir(savelocation):
        os.mkdir(savelocation)
    
    filename=savelocation+str(title)+'.'+poster_file_extension
    f=open(filename,'wb')
    f.write(poster_data)
    f.close()

```


```python
def save_in_database(json_data):
    print("\n")
    db_name = input("Please enter a name for the database(extension not needed, it will be added automatically):")
    conn_path = 'lyrics_project/'+db_name+'.db'
    
    import sqlite3
    conn = sqlite3.connect(str(conn_path))
    cur=conn.cursor()
    if db_name == 'MovieInfo':
        title = json_data['Title']
        # Goes through the json dataset and extracts information if it is available
        if json_data['Year']!='N/A':
            year = int(json_data['Year'])
        if json_data['Runtime']!='N/A':
            runtime = int(json_data['Runtime'].split()[0])
        if json_data['Country']!='N/A':
            country = json_data['Country']
        if json_data['Metascore']!='N/A':
            metascore = float(json_data['Metascore'])
        else:
            metascore=-1
        if json_data['imdbRating']!='N/A':
            imdb_rating = float(json_data['imdbRating'])
        else:
            imdb_rating=-1

        # SQL commands
        cur.execute('''CREATE TABLE IF NOT EXISTS MovieInfo 
        (Title TEXT, Year INTEGER, Runtime INTEGER, Country TEXT, Metascore REAL, IMDBRating REAL)''')

        cur.execute('SELECT Title FROM MovieInfo WHERE Title = ? ', (title,))
        row = cur.fetchone()

        if row is None:
            cur.execute('''INSERT INTO MovieInfo (Title, Year, Runtime, Country, Metascore, IMDBRating)
                    VALUES (?,?,?,?,?,?)''', (title,year,runtime,country,metascore,imdb_rating))
        else:
            print("Record already found. No update made.")
            
    elif db_name == 'LyricsInfo':
        
        artist = json_data['artist']
        song = json_data['song']
        lyrics = json_data['lyrics']
        
        # SQL commands
        cur.execute('''CREATE TABLE IF NOT EXISTS LyricsInfo 
        (Artist TEXT,Song TEXT, Lyrics TEXT)''')
        
        cur.execute('SELECT Song FROM LyricsInfo WHERE Song = ? ', (song,))
        row = cur.fetchone()
        
        if row is None:
            cur.execute('''INSERT INTO LyricsInfo (Artist,Song,Lyrics)
                    VALUES (?,?,?)''', (artist,song,lyrics))
            print("Song added to the LyricsInfo database!")
        else:
            print("Record already found. No update made.")
    else:
        print("check the name of the database! There are only two 1.MovieInfo and 2.LyricsInfo")
        

    # Commits the change and close the connection to the database
    conn.commit()
    conn.close()
```


```python
def print_database(database):
    conn_path = 'lyrics_project/'+database+'.db'
    import sqlite3
    conn = sqlite3.connect(str(conn_path))
    cur=conn.cursor()
    if database == "MovieInfo":
        for row in cur.execute('SELECT * FROM MovieInfo'):
            print(row)
    elif database == "LyricsInfo":
        for row in cur.execute('SELECT * FROM LyricsInfo'):
            print(row)
    conn.close()
```


```python
def save_in_excel(filename, database):
    
    if filename.split('.')[-1]!='xls' and filename.split('.')[-1]!='xlsx':
        print ("Filename does not have correct extension. Please try again")
        return None
    
    import pandas as pd
    import sqlite3
    
    #df=pd.DataFrame(columns=['Title','Year', 'Runtime', 'Country', 'Metascore', 'IMDB_Rating'])
    
    conn = sqlite3.connect(str(database))
    #cur=conn.cursor()
    
    df=pd.read_sql_query("SELECT * FROM MovieInfo", conn)
    conn.close()
    
    df.to_excel(filename,sheet_name='Movie Info')
```


```python
def save_in_text(song_name,database):
    import pandas as pd
    dbname = 'lyrics_project/'+database+'.db'
    import sqlite3
    conn = sqlite3.connect(str(dbname))
    cur=conn.cursor()
    cur.execute('SELECT Lyrics FROM LyricsInfo WHERE Song = ? ', (song_name,))
    row = cur.fetchone()
    if row is None:
        print("Song not found in database please save the song to database and try again!")
    else:
        text_file = open("{}.txt".format(song_name), "w")
        text_file.writelines(row) 
        text_file.close()
        print("Song: {} lyrics saved to a text file".format(song_name))
    
   
    
    
```


```python
def search_movie(title):
    if len(title) < 1 or title=='quit': 
        print("Goodbye now...")
        return None

    try:
        url = serviceurl + urllib.parse.urlencode({'t': title})+apikey
        print(f'Retrieving the data of "{title}" now... ')
        uh = urllib.request.urlopen(url)
        data = uh.read()
        json_data=json.loads(data)
        
        if json_data['Response']=='True':
            print_json(json_data)
            
            # Asks user whether to download the poster of the movie
            if json_data['Poster']!='N/A':
                poster_yes_no=input ('Poster of this movie can be downloaded. Enter "yes" or "no": ').lower()
                if poster_yes_no=='yes':
                    save_poster(json_data)
            # Asks user whether to save the movie information in a local database
            save_database_yes_no=input ('Save the movie info in a local database? Enter "yes" or "no": ').lower()
            if save_database_yes_no=='yes':
                save_in_database(json_data)
        else:
            print("Error encountered: ",json_data['Error'])
    
    except urllib.error.URLError as e:
        print(f"ERROR: {e.reason}")
```

### Let's Search for a movie called The Prestige
 - pass the movie name to the search_movie function.
 - It searches for the information about movie and extracts it from the web.
 - It gives you an option whether to save the movie poster or not 
 - It also asks you whether to save this movie info into the local database. If you asked it to save, the save_database() function is called and it check whether the database already has information on this movie or not. If it has, then it will not write into database else it adds a row to the database.


```python
search_movie("The Prestige")
```

    Retrieving the data of "The Prestige" now... 
    --------------------------------------------------
    Title: The Prestige
    Year: 2006
    Rated: PG-13
    Released: 20 Oct 2006
    Runtime: 130 min
    Genre: Drama, Mystery, Sci-Fi, Thriller
    Director: Christopher Nolan
    Writer: Jonathan Nolan (screenplay), Christopher Nolan (screenplay), Christopher Priest (novel)
    Actors: Hugh Jackman, Christian Bale, Michael Caine, Piper Perabo
    Plot: After a tragic accident, two stage magicians engage in a battle to create the ultimate illusion while sacrificing everything they have to outwit each other.
    Language: English
    Country: USA, UK
    Awards: Nominated for 2 Oscars. Another 6 wins & 38 nominations.
    Ratings: [{'Source': 'Internet Movie Database', 'Value': '8.5/10'}, {'Source': 'Rotten Tomatoes', 'Value': '76%'}, {'Source': 'Metacritic', 'Value': '66/100'}]
    Metascore: 66
    imdbRating: 8.5
    imdbVotes: 1,104,197
    imdbID: tt0482571
    --------------------------------------------------
    Poster of this movie can be downloaded. Enter "yes" or "no": yes
    Save the movie info in a local database? Enter "yes" or "no": yes
    
    
    Please enter a name for the database(extension not needed, it will be added automatically):MovieInfo


### Let's see the downloaded poster of The prestige movie


```python
from IPython.display import Image
Image("Posters/The_Prestige.jpg")
```




![](/images/The_Prestige.jpg "The Prestige movie poster")




```python
def clean_lyrics(bad_lyrics):
#     pattern = r'[\n]+'
#     better_lyrics = re.sub(pattern, "/n", bad_lyrics)
    good_lyrics = bad_lyrics.split("/n")
    for line in good_lyrics:
        print(line)
    
```


```python

def search_lyrics(artist,song_name):
    
    service_url_lyrics = "https://api.lyrics.ovh/v1"
    
    try:
        url = service_url_lyrics + '/'+artist+'/'+song_name
        print('Retrieving the lyrics of song : {} from the artist :{} now... '.format(song_name,artist))
        print('~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~')
        uh = urllib.request.urlopen(url)
        data = uh.read()
        json_data=json.loads(data)
        if json_data["lyrics"]:
            json_data['artist'] = artist
            json_data['song'] = song_name
            clean_lyrics(json_data['lyrics'])

            # Asks user whether to save the movie information in a local database
            print("\n")
            save_database_yes_no=input ('Save the lyrics info in a local database? Enter "yes" or "no": ').lower()
            if save_database_yes_no=='yes':
                save_in_database(json_data)
        else:
            print("No lyrics available for this song")
    
    except urllib.error.URLError as e:
        print(f"ERROR: {e.reason}")
    
    
```

### Let's search for some lyrics
- pass the name of the artist and name of the song to search.
- search_lyrics() function finds the lyrics online and retrieves it.
- It gives you an option whther to save these lyrics into your local database or not and If you said yes it adds a row into your database if it doesn't have that information.

```python
search_lyrics("coldplay","Clocks")
```

    Retrieving the lyrics of song : Clocks from the artist :coldplay now... 
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    The lights go out and I can't be saved
    Tides that I tried to swim against
    Have brought me down upon my knees
    
    Oh I beg, I beg and plead, singing
    Come out of things unsaid
    
    Shoot an apple off my head and a
    
    Trouble that can't be named
    
    A tiger's waiting to be tamed, singing
    
    
    
    You are
    
    You are
    
    
    
    Confusion never stops
    
    Closing walls and ticking clocks
    
    Gonna come back and take you home
    
    I could not stop that you now know, singing
    
    
    
    Come out upon my seas
    
    Cursed missed opportunities
    
    Am I a part of the cure?
    
    Or am I part of the disease? Singing
    
    
    
    You are, you are, you are
    
    You are, you are, you are
    
    
    
    And nothing else compares
    
    Oh nothing else compares
    
    And nothing else compares
    
    
    
    Home, home where I wanted to go
    
    Home, home where I wanted to go
    
    
    
    Home, (you) home where I wanted to (are) go
    
    Home, (you) home where I wanted to (are) go
    
    
    
    
    
    (Thanks to MusicLoars and iheartpenguins113 for correcting these lyrics)
    
    
    Save the lyrics info in a local database? Enter "yes" or "no": yes
    
    
    Please enter a name for the database(extension not needed, it will be added automatically):LyricsInfo
    Record already found. No update made.



```python
print_database("LyricsInfo")
```

    ('coldplay', 'Clocks', "The lights go out and I can't be saved\nTides that I tried to swim against\nHave brought me down upon my knees\nOh, I beg, I beg and plead, singing\n\nCome out of things unsaid\nShoot an apple off my head, and a\nTrouble that can't be named\nA tiger's waiting to be tamed, singing\n\nYou are\nYou are\n\nConfusion never stops\nClosing walls and ticking clocks, gonna\nCome back and take you home\nI could not stop that you now know, singing\n\nCome out upon my seas\nCurse missed opportunities, am I\nA part of the cure\nOr am I part of the disease? Singing\n\nYou are\nYou are\nYou are\nYou are\n\nYou are\nYou are\n\nAnd nothing else compares\nOh, nothing else compares\nAnd nothing else compares\n\nYou are\nYou are\n\nHome, home, where I wanted to go\nHome, home, where I wanted to go\nHome, home, where I wanted to go (You are)\nHome, home, where I wanted to go (You are)")



```python
save_in_text("Clocks","LyricsInfo")
```

    Song: Clocks lyrics saved to a text file


### Summary
- My weekend was fun and productive because of this project(Movies and Lyrics).
- Some of the challenges even though they are trivial which annoyed me much are environment issues. I was using mini conda for developing the code and even though I have installed required modules in my local environment I was getting no Module Found Error. This issue was resolved by creating a separate enviroment for this project (credits to stackoverflow).
- I learned how to play with JSON objects.
- My SQL query writing also got refreshed. 
