# userscripts for [violentmonkey](https://violentmonkey.github.io/)

[FrankerFaceZ](https://www.frankerfacez.com/) - Twitch extension, activate BTTV/7TV etc. under Addons. No need for BTTv or 7TV extension

[Youtube HD Premium](https://greasyfork.org/en/scripts/498145-youtube-hd-premium) - Automatically takes the highest resolution set for the video incl. Youtube Premium bitrate

[Return Youtube Dislike](https://www.returnyoutubedislike.com/)

[Youtube Non-Round Design](https://greasyfork.org/en/scripts/453802-youtube-non-rounded-design)

[Bypass All Shortlinks Debloated](https://codeberg.org/Amm0ni4/bypass-all-shortlinks-debloated)

[Always use AV1 Youtube](https://greasyfork.org/en/scripts/466127-use-youtube-av1)


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
