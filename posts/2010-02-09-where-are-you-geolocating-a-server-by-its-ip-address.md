
I often see IP addresses in log files on my servers and wonder where they might be coming from. Tools such as
traceroute and whois are great when you need to dig further, But if you just want a quick answer to the question
"where are you?", here's a possible solution. 

<!--more-->

Several websites already offer services that let you enter an IP address in a form and it returns a nice Google Map
along with additional information about that IP address. That's handy most of the time, but not when you're SSH'd
into the server and have no access to a pretty GUI.

So here's a python script that I put together that accepts both IP addresses and hostnames, and will allow you to
get the information you need via command line:

{% gist 10912004 %}

The script uses the API provided by [http://www.hostip.info](http://www.hostip.info) to retrieve country, city, latitude, and longitude of
the server. Save it as whereru. You can pass it an IP address using the -i flag, or a hostname using -h. Here's an
example searching by hostname: 

```
$ whereru -h facebook.com
Hostname: facebook.com
IP address: 69.63.181.12
Country: UNITED STATES (US)
City: San Francisco, CA
Latitude: 37.8133
Longitude: -122.505
Google map: http://maps.google.com/maps?q=37.8133,+-122.505
```

And an example searching by IP address:

```
$ whereru -i 209.191.93.53
Hostname: b1.www.vip.mud.yahoo.com
IP address: 209.191.93.53
Country: PHILIPPINES (PH)
City: Lucena City
Latitude: 13.9333
Longitude: 121.617
Google map: http://maps.google.com/maps?q=13.9333,+121.617
```

The Google map link is provided, although the coordinates I receive from hostip.info don't seem all that accurate
all the time. Oh well. 

**Update 17/02/2010**: It looks like the API changed a little. They're now adding a newline in the results, which gets
added to the result list. This causes the script to break because it expects the latitude to be in results[2] but
finds a newline there instead. So the script's been changed to ignore newlines and to scan the lines for 'latitude'
and 'longitude' and use that instead of relying on indexes. 
