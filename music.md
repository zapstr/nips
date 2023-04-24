# NIP-XX
# Music Recording

`draft` `optional` `author:pablof7z` `author:bu5hm4nn`

Make music tracks first class citizens on nostr.

## Definitions

 - **publisher**: the pubkey that publishes the music recording event to nostr
 - **client**: the software a listener uses to access the music recording

## Music Recording kind: 31337

```JSON
{
  "kind": 31337,
  "tags": [
    [ "subject", "My sunset walk" ],
    [ "p", "<pubkey1>", "author" ],
    [ "p", "<pubkey2>", "author" ],
    [ "media", "audio/ogg", "https://zapstr.com/m/owkRl2sx.ogg", "Low" ],
    [ "media", "audio/mpeg", "https://zapstr.com/m/kd923ksV.mp3", "HiRes" ],
    [ "media", "audio/flac", "https://zapstr.com/m/Bj9S2s8j.flac", "Lossless" ],
    [ "cover", "<url>" ],
    [ "zap", "humble-piano-9382@zapstr.com", "lud16" ],
    [ "zap-play", "21000000" ],
    [ "sat-stream", "1000000", "PT1M" ],
    [ "c", "Instrumental" ],
    [ "c", "Ambient Music" ],
    [ "e", "<eventid1>", "component" ],
    [ "duration", "PT4M5S" ]
  ],
  "content": "Reflection on a sunset walk played on acoustic piano",
  "id": "<event-id>",
  "pubkey": "<pubkey>",
  "sig": "<sig>",
  "created_at": <timestamp>,
}
```

## Content

Content might be empty or a string describing the track (*abstract*).

### Tags

#### p

The `p` tags reference the authors of this music track. Clients SHOULD show them in the order in which they appear in the tags. Clients CAN make the author names a clickable link to the author's profile to open in a general nostr client, or alternatively to create a view with other tracks by this author.

TODO: Complete the `p` types (`author`, etc.)

#### e

```JSON
  "tags": [
  	[ "e", "<event>", "component" ]
  ]
```

Additional to the standard use of `e` tags in nostr, a reference to a `"component"` event allows linking to other nostr events that have been used in the composition of this music recording. For example this allows the linking of 'stems' used within this recording (TODO: link to stems NIP).

TODO: Define the `e` types (`component`, `reply`, `root`, etc.)

#### media

```JSON
  "tags": [
  	[ "media", "<mime-type>", "<url>", "<quality>" ],
  	[ "media", "<mime-type-2>", "<url-2>", "<quality-2>" ]
  ]
```

 - `<mime-type>`: MUST be specified, SHOULD follow a spec, and is used by clients to select the decoding strategy, or to choose an appropriate file to download.
 - `<url>`: the url for this media file
 - `<quality>`: an optional quality description, like `"Hi-Res"` or `"Lossless"`

Publishers can repeat the `media` tag to list multiple files to suite different client capabilities and bandwidth choices.

#### duration (optional)

The duration of the track ([Sch](https://schema.org/duration)) in [ISO 8601 format](https://en.wikipedia.org/wiki/ISO_8601#Durations). this allows playlists and players to show the time before downloading the media file.

#### c (optional)

```JSON
  "tags": [
    [ "c", "<genre1>" ],
    [ "c", "<genre2>" ],
  ]
```

 - `c`: (optional) a category or genre for this track. Will be indexed by relays.

Repeating the `c` tag allows specifying multiple categories or genres.

#### zap (optional)

Allow the publisher to specify a target for zaps related to this music recording.

```JSON
  "tags": [
  	[ "zap", "<address>", "<type>" ]
  ]
```

 - `zap`: see [NIP-57](57.md#appendix-g-zap-tag-on-zapped-event) for the specification of the zap tag.

If a zap tag is not supplied, clients will try to zap the publishers profile LNURL instead.

#### zap-play (optional)

Allow the publisher to suggest an amount that COULD be zapped when the user plays the file.

```JSON
  "tags": [
  	[ "zap-play", "<amount>" ]
  ]
```
 - `<amount>`: the suggested amount in millisats the publisher suggests to pay when the client plays the song

Clients are free to let the user override this value, however they SHOULD let the user see the publisher suggestion.

#### sat-stream (optional)

Allow the publisher to suggest an amount that COULD be streamed per minute that the client plays the music recording. Streaming sats does not use nostr zap messaging.

```JSON
  "tags": [
  	[ "sat-stream", "<amount>", "<interval>" ]
  ]
```

 - `<amount>`: the amount in millisats the publisher suggests to pay per interval
 - `<interval>`: a duration in seconds of the suggested frequency of zap intervals.

Clients are free to let the user override this value, however they SHOULD let the user see the publisher suggestion.

### Zapping music flow

Clients use the `zap`, `zap-play` and `sat-stream` tags to determine the options they can offer a user to give value back to the artist and collaborators.

If the publisher has specified a `zap` tag it SHOULD take precedence over zapping the publishers profile (for example if the publisher has specified a split target to distribute funds to collaborators).

The client should try to support giving back in this order:

1. **streaming sats** if a `sat-stream` tag is present. (allowing override amounts, and lower frequencies).
2. **zap on play** regardless of the presence of the `zap-play` tag but taking it into consideration (allowing overrides)
3. **zap like** sending a zap to the `zap` target or the publisher profile if the `zap` tag is empty or fails

The **zap on play** and **zap like** methods use the regular NIP-57 zap mechanism.

### Purchase music flow (TODO)

Artists might choose to require payment for their tracks; this might be kept for a separate NIP but this NIP should consider that related use case.
