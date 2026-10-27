# Playable AVIF reference

Apple Mail and Outlook on iPhone receive Playable videos as animated AVIF images rather than video files. These files are for checking that playback and for making AVIFs by hand.

## Files

| File | What it is |
| --- | --- |
| `standard-snippet-braze.html` | The standard inline snippet for the demo video, with Braze's user token `{{${user_id}}}` |
| `standard-snippet-marketo.html` | The same snippet with Marketo's token `{{lead.Id}}` |
| `lag-check-email-braze.html` | A test email: the snippet plus two timers, for checking on a phone that the video keeps real time |
| `lag-check-email-marketo.html` | The same email with Marketo's token |

## Using the lag-check email

1. Paste it into the ESP's HTML editor. In Braze, turn inline CSS off. Send yourself a test.
2. Open it on an iPhone and stay in Mail. Note the timer each time the video starts over. The gaps should equal the clip's length (20 seconds for the demo video), and the video should never slow down.
3. Timer 1 is built with SVG animation, which survives CSS inlining. Timer 2 is CSS. If only one counts, use that one.

To check another video, replace everything from `<!-- Playable Video` to `<!-- end Playable Video -->` with that video's snippet.

The demo video's AVIF has a thin pink strip along the top. It's a marker that shows the AVIF is what's playing.

## Keyframes: one every second

An animated AVIF with only one keyframe plays slower and slower the longer it runs. The phone has to rebuild each frame starting from the last keyframe. One keyframe per second keeps playback in real time.

Count a file's keyframes (ffprobe comes with ffmpeg). Expect about one per second of clip:

```bash
ffprobe -v error -select_streams v:1 -skip_frame nokey -show_entries frame=pts_time -of csv=p=0 3.avif | wc -l
```

## Making an AVIF with ffmpeg

```bash
ffmpeg -i video.mp4 -an -vf "fps=11,scale=848:-2:flags=lanczos" -c:v libsvtav1 -preset 6 -crf 35 -g 11 -pix_fmt yuv420p -loop 0 out.avif
```

- Keep `fps=` and `-g` the same number, so there is one keyframe per second.
- `scale=848:-2` sets the width and keeps the shape. Never scale above the source size.
- `-crf 35` sets the quality: lower is better and bigger.
- `-loop 0` loops forever.
- Needs ffmpeg 6 or later, built with SVT-AV1.
