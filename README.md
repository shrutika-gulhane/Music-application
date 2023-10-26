

## Overview

Music player and server for ownCloud and Nextcloud. Shows audio files stored in your cloud categorized by artists and albums. Supports mp3, and depending on the browser, many other audio formats too. Supports shuffle play and playlists. The Music app also allows serving audio files from your cloud to external applications which are compatible either with Ampache or Subsonic.


## Supported formats

* MP3 (`audio/mpeg`)
* FLAC (`audio/flac`)
* Vorbis in OGG container (`audio/ogg`)
* Opus in OGG container (`audio/ogg` or `audio/opus`)
* WAV (`audio/wav`)
* AAC in M4A container (`audio/mp4`)
* ALAC in M4A container (`audio/mp4`)
* M4B (`audio/m4b`)
* AAC (`audio/aac`)
* AIFF (`audio/aiff`)
* AU (`audio/basic`)
* CAF (`audio/x-caf`)

_Note: The audio formats supported vary depending on the browser. Most recents versions of Chrome, Firefox and Edge should be able to play all the formats listed above. All browsers should be able to play at least the MP3 files._

### Detail

The modern web browsers ship with a wide variety of built-in audio codecs which can be used directly via the standard HTML5 audio API. Still, there is no browser which could natively play all the formats listed above. For those formats not supported natively, the Music app utilizes the Aurora.js javascript library which is able to play most of the formats listed above, excluding only the OGG containers. On the other hand, Aurora.js may not be able to play all the individual files of the supported formats and is very limited in features (no seeking, no adjusting of playback speed).

_Note: In order to be playable in the Music app, the file type has to be mapped to a MIME type `audio/*` on your cloud instance. Neither ownCloud nor Nextcloud has these mappings by default for the file types AAC, AIFF, AU, or CAF. To add these mappings, run:_

	./occ music:register-mime-types

## Usage hints

Normally, the Music app detects any new audio files in the filesystem on application start and scans metadata from those to its database tables when the user clicks the prompt. The Music app also detects file removals and modifications on the background and makes the required database changes automatically.

If the database would somehow get corrupted, the user can force it to be rebuilt from scratch by opening the Settings (at the bottom of the left pane) and clicking the option "Reset music collection".



#### Scan music files

Scan all audio files not already indexed in the database. Extract metadata from those and insert it to the database. Target either specified user(s) or user group(s) or all users.

	./occ music:scan USERNAME1 USERNAME2 ...
	./occ music:scan --group=USERGROUP1 --group==USERGROUP2 ...
	./occ music:scan --all

All the above commands can be combined with the `--debug` switch, which enables debug output and shows the memory usage of each scan step.

You can also supply the option `--rescan` to scan also the files which are already part of the collection. This might be necessary, if some file update has been missed by the app because of some bug or because the admin has temporarily disabled the app.

Lastly, you can give option `--clean-obsolete` to make the process check all the previously scanned files, and clean up those which are no longer found. Again, this is usually handled automatically, but manually running the command could be necessary on some special cases.



#### Reset cache

Music app caches some results for performance reasons. Normally, there should be no reason to reset this cache manually, but it might be desiredable e.g. when running performance tests. Target either specified user(s) or user group(s) or all users.

	./occ music:reset-cache USERNAME1 USERNAME2 ...
	./occ music:reset-cache --group=USERGROUP1 --group==USERGROUP2 ...
	./occ music:reset-cache --all




#### Authentication

Ampache and Subsonic don't use your ownCloud password for authentication. Instead, you need to use a specifically generated APIKEY with them.
The APIKEY is generated through the Music app settings accessible from the link at the bottom of the left pane within the app. The generated APIKEY is shown upon creation but it is impossible to retrieve it at later time. If you forget or misplace the key, you can always delete it and generate a new one.

When you create the APIKEY, the application shows also the username you should use on your Ampache/Subsonic client. Typically, this is your ownCloud login name but it may also be an UUID in case you have set up LDAP authentication.


### Installation

The Music app can be installed using the App Management available on the web UI of ownCloud or Nextcloud for the admin user.

After installation, you may want to select a specific sub-folder containing your music files through the settings of the application. This can be useful to prevent unwanted audio files to be included in the music library.

### Known issues

#### Huge music collections

The application's scalability for large music collections has gradually improved as new versions have been released. Still, if the collection is large enough, the application may fail to load. The maximum number of tracks supported depends on your server but should be more than 50'000. Also, when there are tens of thousands of tracks, you may notice slow down of the web UI.

#### Translations

There exist partial translations for the Music app for many languages, but most of them are very much incomplete. In the past, the application was translated at https://www.transifex.com/owncloud-org/owncloud/ and the resource still exists there. However, large majoriry of the strings used in the app have not been picked by Transifex for many years now, and hence the translations from Transifex cannot be actually used. The root cause is disparity in the localization mechanisms used in the Music app and on ownCloud in general, and bridging the gap would require some support from ownCloud core team. This is probably never going to happen, see https://central.owncloud.org/t/owncloud-music-app-translations/14881. For now, you may contribute translations as normal pull requests, by following the instructions from https://github.com/owncloud/music/issues/671#issuecomment-782746463.

#### SMB shares

The Music app may be unable to extract metadata of the files residing on a SMB share. This is because, on some system configurations, it is not possible to use `fseek()` function to seek within the remote files on the SMB share. The `getID3` library used for metadata extraction depends on `fseek()` and will fail on such systems. If the metadata extraction fails, the Music app falls back to deducing the track names from the file names and the album names from the folder names. Whether or not the probelm exists on a system, may depend on the details of the SMB support library on the host computer and the remote computer providing the share.

## Development

### Build frontend bundle

All the frontend javascript sources of the Music app, including the used vendor libraries, are bundled into a single file for deployment using webpack. This bundle file is `dist/webpack.app.js`. Similarly, all the style files of the Music app are bundled into `dist/webpack.app.css`. Downloading the vendor libraries and generating these bundles requires the `npm` utility, and happens by running:

	cd build
	npm install --deps
	npm run build

The command above builds the minified production version of the bundle. To build the development version, use

	npm run build-dev

To automatically regenerate the development mode bundles whenever the source .js/.css files change, use

	npm run watch


### `/api/log`

Allows to log a message to ownCloud defined log system.

	POST /api/log

Parameters:

	{
		"message": "The message to log"
	}

Response:

	{
		"success": true
	}


### `/api/collection`

Returns all artists with nested albums and each album with nested tracks. Each track carries a file ID which can be used to obtain the file path with `/api/file/{fileId}/path`. The front-end converts the path into playable WebDAV link like this: `OC.linkToRemoteBase('webdav') + path`.

	GET /api/collection

Response:

	[
		{
			"id": 2,
			"name": "Blind Guardian",
			"albums": [
				{
					"name": "Nightfall in Middle-Earth",
					"year": 1998,
					"disk" : 1,
					"cover": "/index.php/apps/music/api/album/16/cover",
					"id": 16,
					"tracks": [
						{
							"title": "A Dark Passage",
							"number": 21,
							"artistId": 2,
							"files": {
								"audio/mpeg": 1001
							},
							"id": 202
						},
						{
							"title": "Battle of Sudden Flame",
							"number": 12,
							"artistId": 2,
							"files": {
								"audio/mpeg": 1002
							},
							"id": 212
						}
					]
				}
			]
		},
		{
			"id": 3,
			"name": "blink-182",
			"albums": [
				{
					"name": "Stay Together for the Kids",
					"year": 2002,
					"disk" : 1,
					"cover": "/index.php/apps/music/api/album/22/cover",
					"id": 22,
					"tracks": [
						{
							"title": "Stay Together for the Kids",
							"number": 1,
							"artistId": 3,
							"files": {
								"audio/ogg": 1051
							},
							"id": 243
						},
						{
							"title": "The Rock Show (live)",
							"number": 2,
							"artistId": 3,
							"files": {
								"audio/ogg": 1052
							},
							"id": 244
						}
					]
				}
			]
		}
	]

### Creating APIKEY for Subsonic/Ampache

The endpoint `/api/settings/userkey/generate` may be used to programatically generate a random password to be used with an Ampache or a Subsonic client. The endpoint expects two parameters, `length` and `description` (both optional) and returns a JSON response.
Please note that the minimum password length is 10 characters. The HTTP return codes represent also the status of the request.

```
POST /api/settings/userkey/generate
```

Parameters:

```
{
	"length": <length>,
	"description": <description>
}
```

Response (success):

```
HTTP/1.1 201 Created

{
	"id": <userkey_id>,
	"password": <random_password>,
	"description": <description>
}
```

Response (error - no description provided):

```
HTTP/1.1 400 Bad request

{
	"message": "Please provide a description"
}
```

Response (error - error while saving password):

```
HTTP/1.1 500 Internal Server Error

{
	"message": "Error while saving the credentials"
}
```

