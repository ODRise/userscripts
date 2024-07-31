# userscripts für [violentmonkey](https://violentmonkey.github.io/)

[FrankerFaceZ](https://www.frankerfacez.com/) - Twitch Erweiterung, dort unter Addons BTTV/7TV etc aktivieren

[Youtube HD Premium](https://greasyfork.org/en/scripts/498145-youtube-hd-premium) - Nimmt automatisch die höchste Auflösung für das Video

[Return Youtube Dislike](https://www.returnyoutubedislike.com/)

[Youtube Non-Round Design](https://greasyfork.org/en/scripts/453802-youtube-non-rounded-design) - bin kein freund davon wenn alles rund ist

[Bypass All Shortlinks Debloated](https://codeberg.org/Amm0ni4/bypass-all-shortlinks-debloated) - überspringt alle bekannten URL Shortener

[Perplexity Model Selection](https://greasyfork.org/en/scripts/490634-perplexity-model-selection) - Fügt einen Menu bei der Texteingabe zur Auswahl des Models bei Perplexity hinzu

[Reddit Old Redirect](https://greasyfork.org/en/scripts/471477-reddit-old-redirect) - Leitet alle reddit.com urls auf old.reddit.com um

[Fix Old Reddit Image Links](https://github.com/abdurazaaqmohammed/userscripts?tab=readme-ov-file#fix-old-reddit-image-links)

[Undiscord](https://github.com/victornpb/undiscord) - Delete all messages in a Discord channel or DM


Wikipedia Desktop to mobile redirect

```// ==UserScript==
// @name         Wikipedia Desktop to Mobile Redirect
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  Redirect desktop Wikipedia to mobile version
// @match        https://*.wikipedia.org/*
// @exclude      https://*.m.wikipedia.org/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';
    let currentURL = window.location.href;
    let mobileURL = currentURL.replace(/\/\/(\w+)\.wikipedia\.org/, '//$1.m.wikipedia.org');
    window.location.replace(mobileURL);
})();
