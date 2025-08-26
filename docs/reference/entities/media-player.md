---
sidebar_position: 20
description: "Entities of domain: media_player"
---

# Media Player

The `MediaPlayer` interface represents media player entities in Home Assistant. When you define a field of type 
`MediaPlayer` in your SmartBean class and annotate it with `@Entity`, the framework automatically injects an object that
lets you control the corresponding media player in Home Assistant. This provides a type-safe way to interact with your
media players.

````java
@Entity("media_player.speaker_livingroom")
private MediaPlayer mediaPlayer;
````

## State

The media players's state is represented by the `MediaPlayer.State` enum, which mirrors the possible states in Home
Assistant: `OFF`, `ON`, `IDLE`, `PLAYING`, `PAUSED`, `STANDBY`, `BUFFERING`, `UNKNOWN` and `UNAVAILABLE`. You can query
the media players's current state using these methods:

| Method               | Description                                                |
|----------------------|------------------------------------------------------------|
| `getState()`         | Get the current state of the media player.                 |
| `getStateAsString()` | Returns the original state from the Home Assistant entity. |

## Attributes

The following attributes of a media player can be accessed through simple getter methods:

| HA attribute      | Method               | Description                                                                                                                                                                                                                                                                                                     |
|-------------------|----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `friendly_name`   | `getFriendlyName()`  | Friendly name of the entity.                                                                                                                                                                                                                                                                                    |
| `icon`            | `getIcon()`          | Icon of the entity.                                                                                                                                                                                                                                                                                             |
| `media_*`         | `getMedia()`         | Current media playing on this media player. This covers all attributes describing the media in a single `Media` object. This is for example `media_title` or `media_episode`, see [Home Assistant documentation](https://developers.home-assistant.io/docs/core/entity/media-player/#properties) for more info. |
| `shuffle`         | `isShuffle()`        | Returns whether shuffle mode is enabled. If this media player doesn't support shuffle functionality, the method returns `null`. To avoid `null` returns, use `isShuffle(boolean)` instead.                                                                                                                      |
| `repeat`          | `getRepeat()`        | Current aktive repeat mode, `ALL`, `OFF` or `ONE`, can also be `null`, if not supported by this media player.                                                                                                                                                                                                   |
| `is_volume_muted` | `isMuted()`          | Returns whether media player is muted. If this media player doesn't support mute functionality, the method returns `null`. To avoid `null` returns, use `isMuted(boolean)` instead.                                                                                                                             |
| `volume_level`    | `getVolume()`        | Current volume level, a `double` value between 0 ans 1. Returns `-1` if current volume is unknown.                                                                                                                                                                                                              |
| `volume_level`    | `getVolumePercent()` | Current volume level in percent, a `int` value between 0 ans 100. Returns `-1` if current volume is unknown.                                                                                                                                                                                                    |

You can access any additional attributes that are not directly supported through the `getAttributes()` method.

## Services

The `MediaPlayer` interface provides several methods to control your media player through Home Assistant services. Here are the
supported operations:

| HA service                          | Method                  | Description                                                             |
|-------------------------------------|-------------------------|-------------------------------------------------------------------------|
| `media_player.shuffle_set`          | `setShuffle(boolean)`   | Sets the shuffle mode.                                                  |
| `media_player.repeat_set`           | `setRepeat(RepeatMode)` | Sets the repeat mode to `ALL`, `OFF` or `ONE`.                          |
| `media_player.volume_mute`          | `mute()`                | Mutes the media player.                                                 |
| `media_player.volume_mute`          | `unmute()`              | Unmutes the media player.                                               |
| `media_player.volume_set`           | `setVolume(double)`     | Sets the volume level to a value between 0 and 1.                       |
| `media_player.volume_set`           | `setVolume(int)`        | Sets the volume level to a value between 0 and 100.                     |
| `media_player.volume_up`            | `volumeUp()`            | Increases the volume.                                                   |
| `media_player.volume_down`          | `volumeDown()`          | Decreases the volume.                                                   |
| `media_player.media_play`           | `play()`                | Plays the current media (resume after pause).                           |
| `media_player.media_pause`          | `pause()`               | Pause current media playing.                                            |
| `media_player.media_stop`           | `stop()`                | Stop playing.                                                           |
| `media_player.media_next_track`     | `next()`                | Select next track.                                                      |
| `media_player.media_previous_track` | `previous()`            | Select previous track.                                                  |
| `media_player.play_media`           | `playMedia(...)`        | See [section below](#play-media).                                       |
| `media_player.select_source`        | `selectSource(String)`  | Selects th input source, the possible values are media player specific. |

### Play media

The `playMedia()` method invokes the `media_player.play_media` service to play media content on the media player device.

Required parameters are:
- `contentId` (String): Identifies the media to be played. The format varies depending on the content source.
- `contentType`: Specified using the `MediaPlayer.ContentType` enum.

Additional options can be configured through a `MediaPlayerAttr` object. The interface provides convenient factory 
methods to create these attribute objects.

Example usage:

````java
public class ASampleBean implements SmartBean {

  @Entity("media_player.speaker_livingroom")
  private MediaPlayer mediaPlayer;

  public void someBeanMethod() {
    mediaPlayer.playMedia(
        "x-file-cifs://my-music-server.local/music/a_song.mp3",
        MediaPlayer.ContentType.MUSIC,
        MediaPlayerAttr.enqueue(EnqueueMode.ADD),
        MediaPlayerAttr.extra(atts -> atts.setAttribute("title", "Song title"))
    );
  }
}
````

## Media Player Groups

Home Assistant allows certain media players to be grouped together, enabling synchronized content playback across all
devices within the group. This functionality is provided through the `media_player.join` and `media_player.unjoin` 
services.

SmartBeans abstracts this feature through the `MediaPlayerGroup` concept. Any `MediaPlayer` instance can initiate a 
group by calling `createGroup()`. The resulting `MediaPlayerGroup` object provides methods to manage group membership 
through `join()` and `unjoin()` operations.

When a media player joins the group, its content is automatically synchronized with the group's current playback. 
Subsequently, any operations performed on the group's initiating `MediaPlayer` will affect all members of the group 
simultaneously.

````java
public class ASampleBean implements SmartBean {

  @Entity("media_player.speaker_livingroom")
  private MediaPlayer speakerLivingroom;

  @Entity("media_player.speaker_kitchen")
  private MediaPlayer speakerKitchen;

  public void someBeanMethod() {
    MediaPlayerGroup group = speakerLivingroom.createGroup();
    group.join(speakerKitchen);
    //content played in the living room is now also played in the kitchen
  }
}
````

## Access entities programmatically

In addition to the annotation-based approach, you can programmatically access media players using the `getMediaPlayer()` 
method of the `SmartBeans` API. You might prefer this programmatic approach over annotations for example when the entity
ID is dynamically generated through business logic and cannot be determined at compile time.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  public void someBeanMethod() {
    MediaPlayer mediaPlayer = sb.getMediaPLayer("media_player.speaker_kitchen");
    mediaPlayer.setVolumePercent(50);
  }
}
````

import UncachedNote from './_note_uncached_entities.md';

<UncachedNote />