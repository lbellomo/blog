# Scraping spotify artists

Spotify does not offer an api to get all the artists, only to get the artist info once we have their id. But we can abuse the search api to catch as many artists as we can.

We can use the [search api](https://developer.spotify.com/documentation/web-api/reference/search) to search for artists (`type=artsits`) by genre and year (`q=genre:rock year:1999-2002`). The first is to find the artists and the second is to limit the number of results (the api returns at most 1000 items).

First we need to find some genres to start looking for artists. The api [/recommendations/available-genre-seeds](https://developer.spotify.com/documentation/web-api/reference/get-recommendation-genres) is perfect for us, with this we will be able to search for the first artists.

```
$ http GET https://api.spotify.com/v1/recommendations/available-genre-seeds \
  Authorization:'Bearer BQ..G0' | jq ".genres"
[
"acoustic",
"afrobeat",
"alt-rock",
"alternative",
[...]
"turkish",
"work-out",
"world-music"
]
```

Then, we will iterate through all these genres to search for artists. As the api returned us a maximum of 1000 items we will cut the search in years to have more items.

```
# search for genre 'world-music'
# less that 1000
$ http GET 'https://api.spotify.com/v1/search?q=genre: world-music&type=artist' \
  Authorization:'Bearer BQ..G0' | jq .artists.total
831

# search for genre 'alternative'
# 1000 results, surely there are many more
$ http GET 'https://api.spotify.com/v1/search?q=genre: alternative&type=artist' \
  Authorization:'Bearer BQ..G0' | jq .artists.total
1000

# serach for genre 'alternative' but filter by year '1980-1995'
# less that 1000
$ http GET 'https://api.spotify.com/v1/search?q=genre: alternative year: 1980-1995&type=artist' \
  Authorization:'Bearer BQ..G0' | jq .artists.total
753
```

For the most popular genres we have no choice but to iterate from one year at a time to get as many results as we can. We will end up with many duplicate artists so we have to remove duplicates from the results. Also for these genres we are going to be missing some results that are going to be hidden behind the 1000 item limit.

```
$ http GET 'https://api.spotify.com/v1/search?q=genre: rock year: 1999&type=artist' \       
  Authorization:'Bearer BQ..G0' | jq .artists.total
1000
$ http GET 'https://api.spotify.com/v1/search?q=genre: rock year: 2000&type=artist' \
  Authorization:'Bearer BQ..G0' | jq .artists.total
1000

[...]

$ http GET 'https://api.spotify.com/v1/search?q=genre: rock year: 2022&type=artist' \
  Authorization:'Bearer BQ..G0' | jq .artists.total
1000
$ http GET 'https://api.spotify.com/v1/search?q=genre: rock year: 2023&type=artist' \
  Authorization:'Bearer BQ..G0' | jq .artists.total
1000
```

Finally, once we have downloaded all of these genres for each of the years and removed the duplicates we can get new genres from the artists and repeat the process again until we find nothing new (3 or 4 times) and get many more artists.Approximately half a million artists can be obtained in this way.

Finally, [this](https://gist.github.com/lbellomo/b3c7f4f38f51d9ac1ce20dc5c0696ab9#file-scraping_spotify_artists-py) is the script to download all the genres and [here](https://gist.github.com/lbellomo/b3c7f4f38f51d9ac1ce20dc5c0696ab9#file-spotify_artists_ids_2023-08-csv) I leave the ids of the artists sorted by popularity to be able to hydrate them with the [api /artists](https://developer.spotify.com/documentation/web-api/reference/get-multiple-artists) (much faster than doing all this process).
