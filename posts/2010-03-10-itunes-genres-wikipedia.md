
A long time ago I altered my iTunes library to include only artist name and song title. That's it. I removed any
additional information including album name, genre, all of that. 

Today I decided to enable iTunes sharing and it occurred to me that some people who choose to take a gander at my
library might wonder what genre my songs fall into. Yes, labels are for soup cans, but when someone tells me to go
listen to R&amp;B I can tell them I'm not interested. See?

<!--more-->

So where do we get the genres from? Wikipedia is one resource. Most well-known (and some unknown) bands have
entries in Wikipedia listing the types of genres they fall into. Searching through Wikipedia for each artist can
get boring very quickly. So I've put together a python script that will do it for us. 

Here's how it works. The script reads the user's library (exported as an XML file), grabs each artist, looks it up
on Wikipedia, grabs the genres the artist is listed as (if it is listed), and creates a new XML file with the genre
information. Now in some cases, it won't find a Wikipedia entry for an artist, or the genres on the page won't be
listed. In that case, it won't change the genres entry for that artist in your music library. Artists whose genres
can't be found are printed out at the end of the script so you can look it up yourself. 

Here's the script:

{% gist 10927867 %}

Save the script as getgenre.

Launch iTunes and xport any custom playlists that you want to keep! You will need to restore these at the end of
this excercise.  To export a playlist, select it then click on File > Library > Export Playlist. Next,  export your library: File > Library > Export Library and save it as Library.xml in the same folder as getgenre.  

Run getgenre passing it the Library.xml as the parameter.  The script will begin reading the Library.xml and connecting to Wikipedia. 

```
$ python getgenre Library.xml
```

The script will begin reading the Library.xml and connecting to Wikipedia. Here's a sample of 
what you'll see: 


```
Searching: http://en.wikipedia.org/wiki/Special:Search/The_Sisters_Of_Mercy
The Sisters Of Mercy is labeled as post-punk, gothic rock
Searching: http://en.wikipedia.org/wiki/Special:Search/45_Grave
45 Grave is labeled as horror punk, deathrock, gothic rock
Searching: http://en.wikipedia.org/wiki/Special:Search/Siouxsie_And_The_Banshees
Siouxsie And The Banshees is labeled as punk rock, post-punk, gothic rock, alternative rock
Searching: http://en.wikipedia.org/wiki/Special:Search/Xmal_Deutschland
Xmal Deutschland is labeled as gothic rock, new wave, neue deutsche wellerock, dark cabaret
Searching: http://en.wikipedia.org/wiki/Special:Search/Skeletal_Family
Searching: http://en.wikipedia.org/wiki/Special:Search/Skeletal_Family_(band)
Couldn't find genre for Skeletal Family
Searching: http://en.wikipedia.org/wiki/Special:Search/Penis_Flytrap
Searching: http://en.wikipedia.org/wiki/Special:Search/Penis_Flytrap_(band)
Couldn't find genre for Penis Flytrap
.
.
.
####################################################
# Could not find genres for the following artists: #
####################################################
Skeletal Family
Penis Flytrap
Novocaine Mausoleum
Antiworld
Las Gorgonas
La Unidad Del Dolor
.
.
.
```

At the end of all this, you'll have a new XML file called NEW.Library.xml which contains the updated genre
information that was pulled from Wikipedia.

Now Shut down iTunes and go to your iTunes folder. Copy or backup the following files to another location: iTunes
Library, iTunes Library Extras.itdb, iTunes Library Genius.itdb, and iTunes Music Library.xml.  

Delete iTunes Library, iTunes Library Extras.itdb, iTunes Library Genius.itdb, and iTunes
Music Library.xml and startup iTunes. You'll notice that your music library and playlists are now replaced with
the default iTunes playlists. Delete them. 

Click on File > Library > Import Playlist. Select NEW.Library.xml and in a few minutes you should have your old
library with artist genres listed. You'll see your playlists, but if you click on any of them you'll notice they're
empty so go ahead and delete them. Now import the playlist backups you took earlier and everything should be back
to normal.

I've only tested this script with my iTunes library on iTunes 9.0.3(15) on OS X 10.6.2, so I'm not sure how it will
behave with other libraries and other operating systems. If you're feeling adventurous, give it a go. 
