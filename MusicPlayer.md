# Watermelon Framework

## MusicPlayer Usage

### Dir
```
java
└ [packages]
  └ domain
    └ [domain_name]
      └ viewmodel
        └ [player_view_model] // for Play related view models
        ...
      ...
    ...
  ...
```

### Player init

The player that can be controlled has to implement `Playable` interface.
Also the player can be ExoPlayer(google) or MediaPlayer(Android) and it will called by PlayManager.

The PlayManager is global player that only one.


```java
public class PlayerViewModel extends MyBaseViewModel implements Playable<Music> {
    //...
    @Override
    public void mediaPlay(Context context, Music music, TextureView view) {
        MusicPlayer mediaPlayer 
                = new ExoPlayerWrapper(music.getMusicUrl(), 10f, context);
                // = new MediaPlayerWrapper(music.getMusicUrl(), 10f, context);
        PlayManager.getInstance().setPlayer(mediaPlayer);
        PlayManager.getInstance().startPlayer();
    }

    @Override
    public void mediaStop() {
        PlayManager.getInstance().stopPlayer();
    }

    @Override
    public boolean isPlay() {
        return PlayManager.getInstance().isPlaying();
    }

    @Override
    public long getCurrentPosition() {
        return PlayManager.getInstance().getCurrentPosition();
    }
}
```