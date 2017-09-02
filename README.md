# XML-Sound-Track-Data-into-Database
# Store Sound Track Data into a Database

#Retrieving Sound Track data from xml and store it in sqlite3 database


import xml.etree.ElementTree as ETimport sqlite3
conn = sqlite3.connect('Tracks.sqlite')
cur = conn.cursor()

cur.executescript('''   
  DROP TABLE IF EXISTS Track;   
  DROP TABLE IF EXISTS Album;    
  DROP TABLE IF EXISTS Artist;   
  DROP TABLE IF EXISTS Genre;
 
CREATE TABLE Artist(     
  id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,     
  name TEXT UNIQUE );

CREATE TABLE Album (    
  id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,    
  artist_id  INTEGER,    
  title  TEXT UNIQUE);

CREATE TABLE Genre(   
  id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,    
  name TEXT UNIQUE);

CREATE TABLE Track(   
  id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,  
  title TEXT UNIQUE,   
  album_id INTEGER, 
  genre_id integer,  
  len INTEGER, count INTEGER, rating INTEGER
  );
  ''')

def lookup(d,key):    
  found = False    
  for child in d:        
    if found: return child.text   
    if child.tag == 'key' and child.text == key:           
       found = True 
   return None

#fname = 'Library.xml'
fname = input ('Enter file name:')
if ( len(fname) < 1 ):fname = 'Library.xml'

stuff = ET.parse(fname)
all =  stuff.findall('dict/dict/dict')
print('Dict Count: ', len(all))

for entry in all:    
  if (lookup(entry,'Track ID') is None):continue
  
  name = lookup(entry,'Name')    
  album = lookup(entry,'Album')    
  artist = lookup(entry,'Artist')  
  genre = lookup(entry,'Genre') 
  length =  lookup(entry,'Total Time')  
  count = lookup(entry,'Play Count')  
  rating = lookup(entry,'Rating')
  
  if name is None or album is None or artist is None or genre is None:continue    

  #print(name,album,artist,genre,length,count,rating)
  cur.execute (''' INSERT OR IGNORE INTO Genre(name) VALUES(?) ''',(genre, ) )    
  cur.execute(''' SELECT id FROM Genre WHERE name = ?''',(genre, ) )   
  genre_id = cur.fetchone()[0]
  cur.execute ('''INSERT OR IGNORE INTO Artist(name) VALUES(?)''',(artist, ))  
  cur.execute(''' SELECT id FROM Artist WHERE name= ? ''',(artist, )) 
  artist_id = cur.fetchone()[0]
  cur.execute(''' INSERT OR IGNORE INTO Album(title,artist_id) VALUES( ?, ?) ''',(album,artist_id)) 
  cur.execute(''' SELECT id FROM Album WHERE title = ? ''',(album, ))  
  album_id = cur.fetchone()[0]
  cur.execute(''' INSERT OR REPLACE INTO Track(title, album_id, genre_id, len, count, rating) VALUES(?,?,?,?,?,?)''', (name, album_id,     genre_id, length, count, rating ))
  
  conn.commit()
