# Music Recording

Make music tracks first class citizens on nostr.

##### Definitions

 - **publisher**: the pubkey that publishes the music recording event to nostr
 - **client**: the software a listener uses to access the music recording

### Why a separate kind?

Clients should be able to request specific media kinds from relays.

### Music Recording kind: 31337

```JSON
{
  "kind": 31337,
  "name": "My sunset walk",
  "duration": "PT4M5S",
  "abstract": "Reflection on a sunset walk played on acoustic piano",
  "byArtist": "Humble Piano",
  "inAlbum": "Reflections",
  "tags": [
    [ "media", "audio/ogg", "https://zapstr.com/owkRl2sx.ogg"],
    [ "media", "audio/mpeg", "https://zapstr.com/kd923ksV.mp3"],
    [ "album", "<album-playlist-id>", "<relay-url"> ],
    [ "zap", "humble-piano-9382@zapstr.com", "lud16" ],
    [ "zap-play", "21000000" ],
    [ "sat-stream", "1000000", "PT1M" ],
    [ "c": "Instrumental" ],
    [ "c", "Ambient Music" ],
  ],
  ],
  "id": "<event-id>",
  "pubkey": "<pubkey>",
  "sig": "<sig>",
  "created_at": 1679790774,
}
```

#### Fields

Fields marked (Sch) are taken from https://schema.org/MusicRecording

 - `name`: title of the track (Sch)
 - `duration`: the duration of the track ([Sch](https://schema.org/duration)) in [ISO 8601 format](https://en.wikipedia.org/wiki/ISO_8601#Durations). this allows playlists and players to show the time before downloading the media file.
 - `abstract`: (optional) short description of the track (Sch)
 - `byArtist`: (optional) artist of the track (Sch).
	if not present the publisher of the event should be displayed as the artist.
 - `inAlbum`: (optional) an album name (Sch)
 
#### Tags

**media**

```JSON
  "tags": [
  	[ "media", "<mime-type>", "<url>" ],
  	[ "media", "<mime-type-2>", "<url-2>" ]
  ]
```

 - `<mime-type>`: MUST be specified, MUST follow [IANA spec](https://www.iana.org/assignments/media-types/media-types.xhtml#audio), and is used by clients to select the decoding strategy, or to choose an appropriate file to download.
 - `<url>`: the url for this media file
 - `<quality>`: an optional quality description, like `"Hi-Res"` or `"Lossless"`

Publishers can repeat the `media` tag to list multiple files to suite different client capabilities and bandwidth choices.
 
**album** (optional)
 
```JSON
  "tags": [
  	[ "album", "<album-playlist-id>", "<relay-url>" ]
  	[ "album", "<album-playlist-id>", "<relay-url-2>" ]
  ]

```

 - `<album-playlist-id>`: this field contains a nostr event ID of an album playlist (kind:31338) that this track is a part of. this allows client apps to dynamically link to the album. once the album event is known to the client, it takes precendence over the `inAlbum` field.
 - `<relay-url>`: a preferred relay where the playlist can be found.

Repeating the `album` tag allows specifying alternative relays.

**c** (optional)

```JSON
  "tags": [
    [ "c", "<genre1>" ],
    [ "c", "<genre2>" ],
  ]
```

 - `c`: (optional) a category or genre for this track. Will be indexed by relays.

Repeating the `c` tag allows specifying multiple categories or genres.

**zap** (optional)

Allow the publisher to specify a target for zaps related to this music recording.

```JSON
  "tags": [
  	[ "zap", "<address>", "<type>" ]
  ]

```

 - `zap`: see [NIP-57](57.md#appendix-g-zap-tag-on-zapped-event) for the specification of the zap tag.

If a zap tag is not supplied, clients will try to zap the publishers profile LNURL instead.

**zap-play** (optional)

Allow the publisher to suggest an amount that COULD be zapped when the user plays the file.

```JSON
  "tags": [
  	[ "zap-play", "<amount>" ]
  ]
```
 - `<amount>`: the suggested amount in millisats the publisher suggests to pay when the client plays the song

Clients are free to let the user override this value, however they SHOULD let the user see the publisher suggestion.

**sat-stream** (optional)

Allow the publisher to suggest an amount that COULD be streamed per minute that the client plays the music recording. Streaming sats does not use nostr zap messaging.

```JSON
  "tags": [
  	[ "sat-stream", "<amount>", "<interval>" ]
  ]
```

 - `<amount>`: the amount in millisats the publisher suggests to pay per interval
 - `<interval>`: an [ISO 8601 duration](https://en.wikipedia.org/wiki/ISO_8601#Durations) string like `"PT1M"` or `"PT10S"` that describes the suggested frequency of streaming sats. The duration must be in minute `"PTxxM"` or second `"PTxxS"` increments. The client MUST NOT stream sats at a faster frequency than specified by the publisher.

Clients are free to let the user override this value, however they SHOULD let the user see the publisher suggestion.

### Zapping music

Clients use the `zap`, `zap-play` and `sat-stream` tags to determine the options they can offer a user to give value back to the artist and collaborators.

If the publisher has specified a `zap` tag it SHOULD take precedence over zapping the publishers profile (for example if the publisher has specified a split target to distribute funds to collaborators).

The client should try to support giving back in this order:

1. streaming sats per duration according to `sat-stream` tag.
2. sending a zap on play of the file taking the suggested amount in `zap-play` into consideration (but allowing overrides).
3. sending a zap to the `zap` target or the publisher profile if the `zap` tag is empty or fails
