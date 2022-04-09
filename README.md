# spotify_playlist_randomizer

Ruby scripts to take an existing Spotify playlist and create a copy with contents of playlist in randomized order.

# How to use scripts

You will need to log into https://developer.spotify.com/ and register an app to get a client id and client secret to use with these scripts.

To get an access token, first you will need to run a curl command similar to

```bash
client_id=<your client id>
domain=<your domain>
curl -v "https://accounts.spotify.com/authorize?response_type=code&client_id=$client_id&redirect_uri=$domain&scope=playlist-modify-private"
```

Then take the location response header and paste that into a browser and login to your account. The redirect to your domain will include a code=... parameter. Copy and paste that into your terminal window.

You need to combine your client id and secrety with a colon and base64 encode them to call the curl commands to get an access token to feed to the scripts.

```bash
base64_encoded=`echo -n "$client_id:$client_secret" | base64`
```

To get an access token run curl command:

```bash
curl -v -X POST -H "Authorization: Basic $base64_encoded" -H Content-Type:application/x-www-form-urlencoded "https://accounts.spotify.com/api/token" -d "grant_type=authorization_code&code=$code&redirect_uri=$domain"
```

# Running playlist_downloader

You need to have the playlist id for the playlist you want to download. Assuming it is one of your own playlists, you can get it with this curl command:

```bash
curl -v -H "Authorization: Bearer $access_token" -H Content-Type:application/json https://api.spotify.com/v1/me/playlists
```

Usage of the `playlist_downloader` script is then 

```bash
./playlist_downloader playlist_id output_file access_token
```

The output is a CSV with headers uri, song, album and artists.

# Running playlist_randomizer

The file output by `playlist_downloader` is one of the inputs into `playlist_randomizer`.

```bash
./playlist_randomizer new_playlist_name input_filename user_id access_token [input_json_songs_to_match]
```

## Keeping songs together

The optional parameter is for the filename of a JSON formatted file with lists of files to keep together in a specific order. (It doesn't matter if these songs are together in this order in the original play list or not for the randomizer script to find the files and keep them in the desired order.) The script can keep two or more songs together. The matching is done with simple case insensitive regular expressions that do not require whole string, so that for example you don't have to worry that the album name on Spotify might actually be "Sgt. Pepper's Lonely Hearts Club Band (Deluxe Edition)".

Example JSON:

```javascript
[
  {
    "artist": "Beatles",
    "album": "Sgt. Pepper's Lonely Hearts Club Band",
    "songs": [ "Sgt. Pepper's Lonely Hearts Club Band", "With A Little Help From My Friends" ]
  },
  {
    "artist": "INXS",
    "album": "Kick",
    "songs": [ "Need You Tonight", "Mediate"]
  },
  {
    "artist": "Queen",
    "album": "News Of The World",
    "songs": [ "We Will Rock You", "We Are the Champions"]
  }
]
```
