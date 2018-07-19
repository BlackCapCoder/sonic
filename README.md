Added ability to read input from STDIN using underscore.


Downloading, speeding up, and playing audio from youtube in mpv can be done in a single pipeline like so:
```bash
SPEED="2.5"

youtube-dl -f bestaudio "$1" --quiet -o - \
  | ffmpeg -loglevel panic -i - -f wav - \
  | sox -t wav - -t wav - \
  | sonic -c -s $SPEED _ _ \
  | mpv -
```

Downloading, speeding up, and playing a video from youtube in vlc can be done like so:
```bash
SPEED="2.5"

youtube-dl -f bestvideo "$1" --quiet -o - \
  | ffmpeg -i <( youtube-dl -f bestaudio "$1" --quiet -o - \
               | ffmpeg -loglevel panic -i - -f wav - \
               | sox -t wav - -t wav - \
               | sonic -c -s 2.0 _ _
               ) -i - \
      -loglevel panic \
      -preset ultrafast \
      -filter_complex "[1:v]setpts=$(lua -e "print(1/$SPEED)")*PTS[v];[0:a]atempo=1.0[a]" \
      -map "[v]" -map "[a]" \
      -f mpeg - \
  | cvlc -
```

The reason I pipe the output of ffmpeg to sox is that:
1) sonic cannot parse the output from ffmpeg
2) sox cannot read webm files, which is what `bestaudio` usually is
