# uBlock Origin filter to block autoplay videos in TechCrunch

Every TechCrunch article autoplays a video in the middle of the post.
Hiding it isn't enough, because the player keeps running in the background.

The video is a JW Player embed, and the page also loads Connatix's floating
ad player. Blocking their scripts stops the video from loading at all, and the
cosmetic rules hide the empty box it leaves behind.

These work in uBlock Origin on Firefox (the full MV2 version, not uBO Lite).
Paste them into *Dashboard → My filters* and click *Apply changes*:

````
! TechCrunch: stop the autoplaying JW Player video
||content.jwplatform.com/libraries/$script,domain=techcrunch.com
||ssl.p.jwpcdn.com^$script,domain=techcrunch.com
||connatix.com^$third-party,domain=techcrunch.com
techcrunch.com##.wp-block-techcrunch-jw-player-embed
techcrunch.com##.jw-player-inline-promo
````

If TechCrunch switches video vendors, the `##` rules will probably keep hiding
the box, but you'll need a new network rule to stop the audio.
