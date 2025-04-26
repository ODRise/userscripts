### Collection of userscripts

<details>
  <summary>Wikipedia Desktop to Mobile Redirect</summary>
  
    // ==UserScript==
    // @name         Wikipedia Desktop to Mobile Redirect
    // @namespace    http://tampermonkey.net/
    // @version      v1
    // @author       OD
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

</details>

<details>
  <summary>Youtube - Always use AV1</summary>
  
    // ==UserScript==
    // @name                Use YouTube AV1
    // @description         Use AV1 for video playback on YouTube with enhanced features and compatibility
    // @namespace           http://tampermonkey.net/
    // @version             v1
    // @author              OD
    // @match               https://www.youtube.com/*
    // @match               https://www.youtube.com/embed/*
    // @match               https://www.youtube-nocookie.com/embed/*
    // @exclude             https://www.youtube.com/live_chat*
    // @exclude             https://www.youtube.com/live_chat_replay*
    // @exclude             /^https?://\S+\.(txt|png|jpg|jpeg|gif|xml|svg|manifest|log|ini)[^\/]*$/
    // @icon                https://www.google.com/s2/favicons?sz=64&domain=youtube.com
    // @grant               none
    // @run-at              document-start
    // @license             MIT
    // @unwrap
    // @allFrames           true
    // @inject-into         page
    // ==/UserScript==
    
    /**
     * Enhanced YouTube AV1 Userscript
     *
     * This script forces YouTube to use the AV1 video codec for playback, which can provide
     * better video quality at the same bitrate compared to older codecs like VP9 or H.264.
     *
     * Improvements in this version:
     * - Better error handling and browser compatibility checks
     * - Improved code structure and documentation
     * - More efficient promise handling
     * - Enhanced debugging capabilities
     * - Better performance through optimized code
     */
    (function () {
      'use strict';
    
      // Configuration options
      const CONFIG = {
        debug: false,           // Set to true to enable detailed console logging
        forceAV1: true,         // Set to false to disable AV1 forcing (for testing)
        preferenceValue: '8192' // YouTube's internal value for AV1 preference
      };
    
      // Logger utility for better debugging
      const logger = {
        debug: function(...args) {
          if (CONFIG.debug) console.debug("YouTube-AV1:", ...args);
        },
        info: function(...args) {
          console.info("YouTube-AV1:", ...args);
        },
        warn: function(...args) {
          console.warn("YouTube-AV1:", ...args);
        },
        error: function(...args) {
          console.error("YouTube-AV1:", ...args);
        }
      };
    
      logger.info("Script injected");
    
      /**
       * Safely get the native Promise implementation
       * This avoids issues with modified Promise objects in some browsers
       * @returns {PromiseConstructor} A reliable Promise constructor
       */
      function getSafePromise() {
        try {
          // Use the Promise from an async function to avoid hacked global Promise
          return (async () => {})().constructor;
        } catch (e) {
          logger.warn("Failed to get safe Promise, falling back to global Promise", e);
          return window.Promise;
        }
      }
    
      /**
       * Configure YouTube experiment flags to optimize for AV1
       * This modifies YouTube's internal configuration to prefer AV1
       */
      function configureYouTubeFlags() {
        logger.debug("Configuring YouTube experiment flags");
    
        let firstConfigDetected = true;
        let intervalId = 0;
        const { setInterval, clearInterval, setTimeout } = window;
    
        const updateFlags = () => {
          // Check if YouTube config exists
          const ytConfig = (window.ytcfg && window.ytcfg.data_) ? window.ytcfg.data_ : null;
          if (!ytConfig) return;
    
          const isFirstConfig = firstConfigDetected;
          firstConfigDetected = false;
    
          // Update experiment flags to optimize for AV1
          const flagSets = [ytConfig.EXPERIMENT_FLAGS, ytConfig.EXPERIMENTS_FORCED_FLAGS];
    
          for (const flags of flagSets) {
            if (!flags) continue;
    
            // Disable flags that would prevent AV1 from being selected
            flags.html5_disable_av1_hdr = false;
            flags.html5_prefer_hbr_vp9_over_av1 = false;
            flags.html5_account_onesie_format_selection_during_format_filter = false;
    
            // Additional flags that might help with AV1 selection
            if (flags.hasOwnProperty('html5_prefer_av1_over_vp9_for_hdr')) {
              flags.html5_prefer_av1_over_vp9_for_hdr = true;
            }
          }
    
          // Set up a mutation observer to detect when YouTube updates its configuration
          if (isFirstConfig) {
            let observer = new MutationObserver(() => {
              observer.disconnect();
              observer.takeRecords();
              observer = null;
    
              setTimeout(() => {
                if (intervalId) {
                  clearInterval.call(window, intervalId);
                  intervalId = 0;
                }
                updateFlags();
              });
            });
    
            observer.observe(document, { subtree: true, childList: true });
          }
        };
    
        // Start checking for YouTube config
        intervalId = setInterval.call(window, updateFlags, 100);
      }
    
      /**
       * Configure the browser to report AV1 support
       * This overrides the browser's media type checking functions to report AV1 compatibility
       */
      function configureBrowserFormatSupport() {
        logger.debug("Configuring browser to report AV1 support");
    
        /**
         * Tests if a MIME type is an AV1 video type
         * @param {string} type - MIME type string to test
         * @returns {boolean|undefined} true if AV1, undefined if not AV1 (to defer to original checker)
         */
        function isAV1VideoType(type) {
          if (typeof type !== 'string' || !type.startsWith('video/')) {
            return undefined;
          }
    
          // Check for AV1 codec in the MIME type
          if (type.includes('av01') && /codecs[\x20-\x7F]+\bav01\b/.test(type)) {
            return true;
          } else if (type.includes('av1') && /codecs[\x20-\x7F]+\bav1\b/.test(type)) {
            return true;
          }
    
          return undefined;
        }
    
        /**
         * Creates a modified type checker that reports AV1 support
         * @param {Function} originalChecker - The original type checking function
         * @param {boolean} useStringResult - Whether to return "probably" instead of true
         * @returns {Function} A modified type checker function
         */
        function createModifiedTypeChecker(originalChecker, useStringResult) {
          return function(type) {
            let result = undefined;
    
            // Handle undefined input
            if (type === undefined) {
              result = false;
            } else {
              // Check if this is an AV1 type
              result = isAV1VideoType(type);
            }
    
            // If not an AV1 type or undefined input, defer to original checker
            if (result === undefined) {
              result = originalChecker.apply(this, arguments);
            } else if (useStringResult) {
              // Convert boolean to string result if needed
              result = result ? "probably" : "";
            }
    
            logger.debug("Type check:", type, "→", result);
            return result;
          };
        }
    
        // Override HTMLVideoElement.canPlayType()
        try {
          const videoProto = (HTMLVideoElement || 0).prototype;
          if (videoProto && typeof videoProto.canPlayType === 'function') {
            videoProto.canPlayType = createModifiedTypeChecker(videoProto.canPlayType, true);
            logger.debug("Successfully overrode HTMLVideoElement.canPlayType");
          }
        } catch (e) {
          logger.error("Failed to override HTMLVideoElement.canPlayType", e);
        }
    
        // Override MediaSource.isTypeSupported()
        try {
          const mediaSource = window.MediaSource;
          if (mediaSource && typeof mediaSource.isTypeSupported === 'function') {
            mediaSource.isTypeSupported = createModifiedTypeChecker(mediaSource.isTypeSupported, false);
            logger.debug("Successfully overrode MediaSource.isTypeSupported");
          }
        } catch (e) {
          logger.error("Failed to override MediaSource.isTypeSupported", e);
        }
      }
    
      /**
       * Set the YouTube AV1 preference in localStorage
       * This tells YouTube that the user prefers AV1 codec
       */
      function setYouTubeAV1Preference() {
        logger.debug("Setting YouTube AV1 preference");
    
        const prefKey = 'yt-player-av1-pref';
        const prefValue = CONFIG.preferenceValue;
    
        try {
          // Try to define a property that always returns the AV1 preference value
          Object.defineProperty(localStorage.constructor.prototype, prefKey, {
            get() {
              if (this === localStorage) return prefValue;
              return this.getItem(prefKey);
            },
            set(newValue) {
              this.setItem(prefKey, newValue);
              return true;
            },
            enumerable: true,
            configurable: true
          });
    
          logger.debug("Successfully defined localStorage property for AV1 preference");
        } catch (e) {
          // Fall back to directly setting the value if property definition fails
          try {
            localStorage[prefKey] = prefValue;
            logger.debug("Set AV1 preference directly in localStorage");
          } catch (e2) {
            logger.error("Failed to set AV1 preference in localStorage", e2);
          }
        }
    
        // Verify the preference was set correctly
        if (localStorage[prefKey] !== prefValue) {
          logger.warn("AV1 preference verification failed - script may not work correctly");
        }
      }
    
      /**
       * Enable AV1 on YouTube by applying all necessary configurations
       */
      function enableAV1() {
        if (!CONFIG.forceAV1) {
          logger.info("AV1 forcing is disabled in configuration");
          return;
        }
    
        logger.info("Enabling AV1 for YouTube");
    
        // Set the YouTube AV1 preference
        setYouTubeAV1Preference();
    
        // Configure browser to report AV1 support
        configureBrowserFormatSupport();
    
        // Configure YouTube experiment flags
        configureYouTubeFlags();
    
        logger.info("AV1 enabled successfully");
      }
    
      /**
       * Check if the browser supports AV1 decoding
       * @returns {Promise<boolean>} Promise resolving to true if AV1 is supported
       */
      function checkAV1Support() {
        logger.debug("Checking browser AV1 support");
    
        // Use a safe Promise implementation
        const SafePromise = getSafePromise();
    
        // If mediaCapabilities API is not available, assume no support
        if (!navigator.mediaCapabilities || typeof navigator.mediaCapabilities.decodingInfo !== 'function') {
          logger.warn("MediaCapabilities API not available");
          return SafePromise.resolve(false);
        }
    
        try {
          // Test AV1 decoding capability
          return navigator.mediaCapabilities.decodingInfo({
            type: "file",
            video: {
              contentType: "video/mp4; codecs=av01.0.05M.08.0.110.05.01.06.0",
              height: 1080,
              width: 1920,
              framerate: 30,
              bitrate: 2826848,
            },
            audio: {
              contentType: "audio/webm; codecs=opus",
              channels: "2.1",
              samplerate: 44100,
              bitrate: 255236,
            }
          }).then(result => {
            const supported = !!(result && result.supported && result.smooth);
            logger.info("Browser AV1 support:", supported ? "Yes" : "No");
            return supported;
          }).catch(error => {
            logger.error("Error checking AV1 support:", error);
            return false;
          });
        } catch (e) {
          logger.error("Exception checking AV1 support:", e);
          return SafePromise.resolve(false);
        }
      }
    
      /**
       * Main initialization function
       */
      function initialize() {
        // Check if browser supports AV1
        checkAV1Support().then(supported => {
          if (supported) {
            // Browser supports AV1, enable it
            enableAV1();
          } else {
            // Browser doesn't support AV1, show warning
            logger.warn("Your browser does not support AV1. Consider using the latest version of Google Chrome, Microsoft Edge, or Mozilla Firefox for AV1 support.");
          }
        });
      }
    
      // Start the script
      initialize();
    
    })();
</details>

<details>
  <summary>Youtube - Automatically sets YouTube playback speed to 2.5x, excluding live streams.</summary>
  
      // ==UserScript==
      // @name         YouTube Default Playback Speed
      // @namespace    https://youtube.com/
      // @version      v1
      // @author       OD
      // @description  Automatically sets YouTube playback speed to 2.5x, excluding live streams.
      // @match        https://www.youtube.com/*
      // @grant        none
      // ==/UserScript==
      
      (function () {
          'use strict';
      
          const DEFAULT_SPEED = 1.25;
      
          function isLiveStream() {
              // Check for the "LIVE" badge; YouTube uses a specific indicator for live streams
              const liveBadge = document.querySelector('.ytp-live-badge');
              return liveBadge && liveBadge.offsetParent !== null;
          }
      
          function setPlaybackSpeed(video) {
              if (video && !isLiveStream() && video.playbackRate !== DEFAULT_SPEED) {
                  video.playbackRate = DEFAULT_SPEED;
                  console.log(`Playback speed set to ${DEFAULT_SPEED}x`);
              }
          }
      
          function handleNewVideo() {
              const video = document.querySelector('video.html5-main-video');
              if (video) {
                  setPlaybackSpeed(video);
              }
          }
      
          function initObservers() {
              // Observe DOM changes for dynamically loaded pages (YouTube's SPA behavior)
              const observer = new MutationObserver(() => {
                  handleNewVideo();
              });
      
              observer.observe(document.body, { childList: true, subtree: true });
      
              // Handle YouTube's navigation events (yt-navigate-finish)
              window.addEventListener('yt-navigate-finish', () => {
                  setTimeout(() => {
                      handleNewVideo();
                  }, 500); // Delay to ensure the video element is loaded
              });
          }
      
          // Initial setup for the video on page load
          window.addEventListener('load', () => {
              setTimeout(() => {
                  handleNewVideo();
              }, 1000);
          });
      
          // Initialize observers
          initObservers();
      })();

</details>

<details>
  <summary>Youtube - Automatically switches to high quality (selectable) inkl. Yt Premium</summary>
  
      // ==UserScript==
      // @name                Youtube HD Premium
      // @icon                https://www.youtube.com/img/favicon_48.png
      // @version             v1
      // @author              OD
      // @match               *://www.youtube.com/*
      // @match               *://m.youtube.com/*
      // @match               *://www.youtube-nocookie.com/*
      // @exclude             *://www.youtube.com/live_chat*
      // @grant               GM.getValue
      // @grant               GM.setValue
      // @grant               GM.deleteValue
      // @grant               GM.listValues
      // @grant               GM.registerMenuCommand
      // @grant               GM.unregisterMenuCommand
      // @grant               GM.notification
      // @grant               GM_getValue
      // @grant               GM_setValue
      // @grant               GM_deleteValue
      // @grant               GM_listValues
      // @grant               GM_registerMenuCommand
      // @grant               GM_unregisterMenuCommand
      // @grant               GM_notification
      // @license             MIT
      // @description         Automatically switches to high quality (1080p+)
      
      // ==/UserScript==
      
      /*jshint esversion: 11 */
      
      (function () {
          "use strict";
      
          // -------------------------------
          // Default settings (for storage key "settings")
          // -------------------------------
          const DEFAULT_SETTINGS = {
              targetResolution: "hd1080", // Default to 1080p
              expandMenu: false,
              debug: false,
              forceHD: true // New option to force minimum 1080p when available
          };
      
          // -------------------------------
          // Translations - English only
          // -------------------------------
          const TRANSLATIONS = {
              tampermonkeyOutdatedAlertMessage: "It looks like you're using an older version of Tampermonkey that might cause menu issues. For the best experience, please update to version 5.4.6224 or later.",
              qualityMenu: 'Quality Menu',
              autoModeName: 'Optimized Auto',
              debug: 'DEBUG',
              forceHD: 'Force HD (min 1080p)'
          };
      
          // Keep only HD resolutions (1080p and above)
          const QUALITIES = {
              highres: 4320,
              hd2160: 2160,
              hd1440: 1440,
              hd1080: 1080,
              auto: 0
          };
      
          const PREMIUM_INDICATOR_LABEL = "Premium";
      
          // -------------------------------
          // Global variables
          // -------------------------------
          let userSettings = { ...DEFAULT_SETTINGS };
          let useCompatibilityMode = false;
          let menuItems = [];
          let moviePlayer = null;
          let isOldTampermonkey = false;
          const updatedVersions = {
              Tampermonkey: '5.4.624',
          };
          let isScriptRecentlyUpdated = false;
          let retryCount = 0;
          const MAX_RETRIES = 3;
      
          // -------------------------------
          // GM FUNCTION OVERRIDES
          // -------------------------------
          const GMCustomRegisterMenuCommand = useCompatibilityMode ? GM_registerMenuCommand : GM.registerMenuCommand;
          const GMCustomUnregisterMenuCommand = useCompatibilityMode ? GM_unregisterMenuCommand : GM.unregisterMenuCommand;
          const GMCustomGetValue = useCompatibilityMode ? GM_getValue : GM.getValue;
          const GMCustomSetValue = useCompatibilityMode ? GM_setValue : GM.setValue;
          const GMCustomDeleteValue = useCompatibilityMode ? GM_deleteValue : GM.deleteValue;
          const GMCustomListValues = useCompatibilityMode ? GM_listValues : GM.listValues;
          const GMCustomNotification = useCompatibilityMode ? GM_notification : GM.notification;
      
          // -------------------------------
          // Debug logging helper
          // -------------------------------
          function debugLog(message) {
              if (!userSettings.debug) return;
              const stack = new Error().stack;
              const stackLines = stack.split("\n");
              const callerLine = stackLines[2] ? stackLines[2].trim() : "Line not found";
              if (!message.endsWith(".")) {
                  message += ".";
              }
              console.log(`[YTHD DEBUG] ${message} Function called ${callerLine}`);
          }
      
          // -------------------------------
          // Video quality functions
          // -------------------------------
          function setResolution(force = false) {
              try {
                  if (!moviePlayer) throw "Movie player not found.";
                  const videoQualityData = moviePlayer.getAvailableQualityData();
                  const currentPlaybackQuality = moviePlayer.getPlaybackQuality();
                  const currentQualityLabel = moviePlayer.getPlaybackQualityLabel();
                  if (!videoQualityData.length) throw "Quality options missing.";
      
                  // Get available quality levels
                  const availableQualityLevels = moviePlayer.getAvailableQualityLevels();
      
                  // Check if HD quality is available
                  const hasHDQuality = availableQualityLevels.some(q => QUALITIES[q] >= 1080);
      
                  if (userSettings.targetResolution === 'auto') {
                      if (!currentPlaybackQuality || !currentQualityLabel) throw "Unable to determine current playback quality.";
      
                      const currentQualityValue = QUALITIES[currentPlaybackQuality] || 0;
                      const isHDQuality = currentQualityValue >= 1080;
                      const isPremiumQuality = currentQualityLabel.trim().endsWith(PREMIUM_INDICATOR_LABEL);
      
                      // If Force HD is enabled, check if we need to upgrade quality
                      if (userSettings.forceHD && hasHDQuality && !isHDQuality) {
                          // Find the lowest HD quality available
                          const hdQuality = availableQualityLevels.find(q => QUALITIES[q] >= 1080);
                          if (hdQuality) {
                              const premiumData = videoQualityData.find(q =>
                                  q.quality === hdQuality &&
                                  q.qualityLabel.trim().endsWith(PREMIUM_INDICATOR_LABEL) &&
                                  q.isPlayable
                              );
                              moviePlayer.setPlaybackQualityRange(hdQuality, hdQuality, premiumData?.formatId);
                              debugLog(`Force HD enabled: Setting quality to [${hdQuality}${premiumData ? " Premium" : ""}]`);
                              return;
                          }
                      }
      
                      const isOptimalQuality = videoQualityData.filter(q => q.quality == currentPlaybackQuality).length <= 1 ||
                          isPremiumQuality;
                      if (!isOptimalQuality) moviePlayer.loadVideoById(moviePlayer.getVideoData().video_id);
                      debugLog(`Setting quality to: [${TRANSLATIONS.autoModeName}]`);
                  } else {
                      let resolvedTarget = findNextAvailableQuality(userSettings.targetResolution, availableQualityLevels);
                      if (!resolvedTarget) {
                          // Retry mechanism if target resolution is not found
                          if (retryCount < MAX_RETRIES) {
                              retryCount++;
                              debugLog(`Resolution not found, retrying... (${retryCount}/${MAX_RETRIES})`);
                              setTimeout(() => setResolution(force), 1000);
                              return;
                          } else {
                              debugLog("Max retries reached, using best available quality.");
                              resolvedTarget = availableQualityLevels[0] || 'auto';
                              retryCount = 0;
                          }
                      } else {
                          retryCount = 0;
                      }
      
                      const premiumData = videoQualityData.find(q =>
                          q.quality === resolvedTarget &&
                          q.qualityLabel.trim().endsWith(PREMIUM_INDICATOR_LABEL) &&
                          q.isPlayable
                      );
                      moviePlayer.setPlaybackQualityRange(resolvedTarget, resolvedTarget, premiumData?.formatId);
                      debugLog(`Setting quality to: [${resolvedTarget}${premiumData ? " Premium" : ""}]`);
                  }
              } catch (error) {
                  debugLog("Did not set resolution. " + error);
                  // Retry on errors if we haven't exceeded max retries
                  if (retryCount < MAX_RETRIES) {
                      retryCount++;
                      debugLog(`Error encountered, retrying... (${retryCount}/${MAX_RETRIES})`);
                      setTimeout(() => setResolution(force), 1000);
                  } else {
                      retryCount = 0;
                  }
              }
          }
      
          function findNextAvailableQuality(target, availableQualities) {
              const targetValue = QUALITIES[target];
              return availableQualities
                  .map(q => ({ quality: q, value: QUALITIES[q] || 0 }))
                  .sort((a, b) => b.value - a.value) // Sort by highest to lowest
                  .find(q => q.value <= targetValue)?.quality;
          }
      
          function processNewPage() {
              debugLog('Processing new page...');
              moviePlayer = document.querySelector('#movie_player');
      
              // If immediately available, set the resolution
              if (moviePlayer) {
                  setResolution();
              } else {
                  // If not available yet, try again after a short delay
                  debugLog('Movie player not found, will retry...');
                  setTimeout(() => {
                      moviePlayer = document.querySelector('#movie_player');
                      if (moviePlayer) {
                          setResolution();
                      } else {
                          debugLog('Movie player still not found after delay');
                      }
                  }, 2000);
              }
      
              // Also listen for new video data to ensure resolution is applied
              document.addEventListener('yt-navigate-finish', () => {
                  setTimeout(() => {
                      if (!moviePlayer) moviePlayer = document.querySelector('#movie_player');
                      setResolution();
                  }, 1000);
              }, { once: true });
          }
      
          // -------------------------------
          // Menu functions
          // -------------------------------
          function processMenuOptions(options, callback) {
              Object.values(options).forEach(option => {
                  if (!option.alwaysShow && !userSettings.expandMenu && !isOldTampermonkey) return;
                  if (option.items) {
                      option.items.forEach(item => callback(item));
                  } else {
                      callback(option);
                  }
              });
          }
      
          // The menu callbacks now use the helper "updateSetting" to update the stored settings.
          function showMenuOptions() {
              const shouldAutoClose = isOldTampermonkey;
              removeMenuOptions();
              const menuExpandButton = isOldTampermonkey ? {} : {
                  expandMenu: {
                      alwaysShow: true,
                      label: () => `${TRANSLATIONS.qualityMenu} ${userSettings.expandMenu ? "🔼" : "🔽"}`,
                      menuId: "menuExpandBtn",
                      handleClick: async function () {
                          userSettings.expandMenu = !userSettings.expandMenu;
                          await updateSetting('expandMenu', userSettings.expandMenu);
                          showMenuOptions();
                      },
                  },
              };
              const menuOptions = {
                  ...menuExpandButton,
                  qualities: {
                      items: Object.entries(QUALITIES).map(([label, resolution]) => ({
                          label: () => `${resolution === 0 ? TRANSLATIONS.autoModeName : resolution + 'p'} ${label === userSettings.targetResolution ? "✅" : ""}`,
                          menuId: label,
                          handleClick: async function () {
                              if (userSettings.targetResolution === label) return;
                              userSettings.targetResolution = label;
                              await updateSetting('targetResolution', label);
                              setResolution(true);
                              showMenuOptions();
                          },
                      })),
                  },
                  forceHD: {
                      label: () => `${TRANSLATIONS.forceHD} ${userSettings.forceHD ? "✅" : ""}`,
                      menuId: "forceHDBtn",
                      handleClick: async function () {
                          userSettings.forceHD = !userSettings.forceHD;
                          await updateSetting('forceHD', userSettings.forceHD);
                          setResolution(true);
                          showMenuOptions();
                      },
                  },
                  debug: {
                      label: () => `${TRANSLATIONS.debug} ${userSettings.debug ? "✅" : ""}`,
                      menuId: "debugBtn",
                      handleClick: async function () {
                          userSettings.debug = !userSettings.debug;
                          await updateSetting('debug', userSettings.debug);
                          showMenuOptions();
                      },
                  },
              };
      
              processMenuOptions(menuOptions, (item) => {
                  GMCustomRegisterMenuCommand(item.label(), item.handleClick, {
                      id: item.menuId,
                      autoClose: shouldAutoClose,
                  });
                  menuItems.push(item.menuId);
              });
          }
      
          function removeMenuOptions() {
              while (menuItems.length) {
                  GMCustomUnregisterMenuCommand(menuItems.pop());
              }
          }
      
          // -------------------------------
          // GreaseMonkey / Tampermonkey version checks
          // -------------------------------
          function compareVersions(v1, v2) {
              try {
                  if (!v1 || !v2) throw "Invalid version string.";
                  if (v1 === v2) return 0;
                  const parts1 = v1.split('.').map(Number);
                  const parts2 = v2.split('.').map(Number);
                  const len = Math.max(parts1.length, parts2.length);
                  for (let i = 0; i < len; i++) {
                      const num1 = parts1[i] || 0;
                      const num2 = parts2[i] || 0;
                      if (num1 > num2) return 1;
                      if (num1 < num2) return -1;
                  }
                  return 0;
              } catch (error) {
                  throw ("Error comparing versions: " + error);
              }
          }
      
          function hasGreasyMonkeyAPI() {
              if (typeof GM !== 'undefined') return true;
              if (typeof GM_info !== 'undefined') {
                  useCompatibilityMode = true;
                  debugLog("Running in compatibility mode.");
                  return true;
              }
              return false;
          }
      
          function CheckTampermonkeyUpdated() {
              if (GM_info.scriptHandler === "Tampermonkey" &&
                  compareVersions(GM_info.version, updatedVersions.Tampermonkey) !== 1) {
                  isOldTampermonkey = true;
                  if (isScriptRecentlyUpdated) {
                      GMCustomNotification({
                          text: TRANSLATIONS.tampermonkeyOutdatedAlertMessage,
                          timeout: 15000
                      });
                  }
              }
          }
      
          // -------------------------------
          // Storage helper functions
          // -------------------------------
      
          /**
           * Load user settings from the "settings" key.
           * Ensures that only keys existing in DEFAULT_SETTINGS are kept.
           * If no stored settings are found, defaults are used.
           */
          async function loadUserSettings() {
              try {
                  const storedSettings = await GMCustomGetValue('settings', {});
      
                  // Check for the legacy versions without forceHD option
                  if (storedSettings && !('forceHD' in storedSettings)) {
                      storedSettings.forceHD = DEFAULT_SETTINGS.forceHD;
                  }
      
                  userSettings = Object.keys(DEFAULT_SETTINGS).reduce((accumulator, key) => {
                      accumulator[key] = storedSettings.hasOwnProperty(key) ? storedSettings[key] : DEFAULT_SETTINGS[key];
                      return accumulator;
                  }, {});
      
                  // Migration: If stored resolution is below 1080p, upgrade it
                  const storedResolution = storedSettings.targetResolution;
                  if (storedResolution && !QUALITIES.hasOwnProperty(storedResolution)) {
                      userSettings.targetResolution = 'hd1080';
                      debugLog(`Migrated resolution from ${storedResolution} to hd1080`);
                  }
      
                  await GMCustomSetValue('settings', userSettings);
                  debugLog(`Loaded user settings: ${JSON.stringify(userSettings)}.`);
              } catch (error) {
                  throw error;
              }
          }
      
          // Update one setting in the stored settings.
          async function updateSetting(key, value) {
              try {
                  let currentSettings = await GMCustomGetValue('settings', DEFAULT_SETTINGS);
                  currentSettings[key] = value;
                  await GMCustomSetValue('settings', currentSettings);
              } catch (error) {
                  debugLog("Error updating setting: " + error);
              }
          }
      
          async function updateScriptInfo() {
              try {
                  const oldScriptInfo = await GMCustomGetValue('scriptInfo', null);
                  debugLog(`Previous script info: ${JSON.stringify(oldScriptInfo)}`);
                  const newScriptInfo = {
                      version: getScriptVersionFromMeta(),
                      lastRun: new Date().toISOString()
                  };
                  await GMCustomSetValue('scriptInfo', newScriptInfo);
      
                  if (!oldScriptInfo || compareVersions(newScriptInfo.version, oldScriptInfo?.version) !== 0) {
                      isScriptRecentlyUpdated = true;
      
                      // Show update notification
                      GMCustomNotification({
                          text: "YouTube HD Premium has been updated to remove resolutions below 1080p and add Force HD option",
                          title: "YouTube HD Premium Updated",
                          timeout: 10000
                      });
                  }
                  debugLog(`Updated script info: ${JSON.stringify(newScriptInfo)}`);
              } catch (error) {
                  debugLog("Error updating script info: " + error);
              }
          }
      
          // Cleanup any leftover keys from previous versions.
          async function cleanupOldStorage() {
              try {
                  const allowedKeys = ['settings', 'scriptInfo'];
                  const keys = await GMCustomListValues();
                  for (const key of keys) {
                      if (!allowedKeys.includes(key)) {
                          await GMCustomDeleteValue(key);
                          debugLog(`Deleted leftover key: ${key}`);
                      }
                  }
              } catch (error) {
                  debugLog("Error cleaning up old storage keys: " + error);
              }
          }
      
          // -------------------------------
          // Script metadata extraction
          // -------------------------------
          function getScriptVersionFromMeta() {
              const meta = GM_info.scriptMetaStr;
              const versionMatch = meta.match(/@version\s+([^\r\n]+)/);
              return versionMatch ? versionMatch[1].trim() : null;
          }
      
          // -------------------------------
          // Enhanced event handling
          // -------------------------------
          function addEventListeners() {
              if (window.location.hostname === "m.youtube.com") {
                  window.addEventListener('state-navigateend', processNewPage, true);
              } else {
                  window.addEventListener('yt-player-updated', processNewPage, true);
                  window.addEventListener('yt-page-data-updated', processNewPage, true);
      
                  // Additional event listeners for better reliability
                  window.addEventListener('yt-player-state-change', () => {
                      if (moviePlayer && moviePlayer.getPlayerState() === 1) { // Playing state
                          setResolution();
                      }
                  }, true);
              }
      
              // Add mutation observer to detect player changes
              const observer = new MutationObserver((mutations) => {
                  for (let mutation of mutations) {
                      if (mutation.addedNodes.length) {
                          const player = document.querySelector('#movie_player');
                          if (player && player !== moviePlayer) {
                              moviePlayer = player;
                              debugLog('Player detected via mutation observer');
                              setResolution();
                              break;
                          }
                      }
                  }
              });
      
              observer.observe(document.body, {
                  childList: true,
                  subtree: true
              });
          }
      
          async function initialize() {
              try {
                  if (!hasGreasyMonkeyAPI()) throw "Did not detect valid Grease Monkey API";
                  await cleanupOldStorage();
                  await loadUserSettings();
                  await updateScriptInfo();
                  CheckTampermonkeyUpdated();
              } catch (error) {
                  debugLog(`Error loading user settings: ${error}. Loading with default settings.`);
              }
              if (window.self === window.top) {
                  processNewPage(); // Ensure initial resolution update on first load.
                  window.addEventListener('yt-navigate-finish', addEventListeners, { once: true });
                  showMenuOptions();
              } else {
                  window.addEventListener('loadstart', processNewPage, true);
                  // Also process for embedded videos immediately
                  processNewPage();
              }
          }
      
          // -------------------------------
          // Entry Point
          // -------------------------------
          initialize();
      })();

</details>

<details>
  <summary>Unround Everything Everywhere</summary>
  
      // ==UserScript==
      // @name         Unround Everything Everywhere
      // @namespace    RM
      // @version      2.0.0
      // @description  Forces zero border-radius and attempts to disable clipping/masking for square corners.
      // @license      CC0 - Public Domain
      // @grant        GM_addStyle
      // @run-at       document-start
      // @match        *://*/*
      // ==/UserScript==
      
      (function() {
          'use strict';
      
          let css = "";
      
          // --- Global and Aggressive Unrounding ---
          // Applies border-radius: 0 to almost all elements and their pseudo-elements.
          // Excludes specific elements like radio buttons to preserve functionality.
          // Also attempts to remove clip-path globally, which often rounds images/avatars.
          css += `
            *:not(#u#n#r#o#u#n#d):not(input[type="radio" i]):not(input[type="radio" i] + label){
              &,
              &::before,
              &::after {
                border-radius: 0 !important;
              }
            }
      
            /* Attempt to disable circular clipping globally - common for avatars */
            *:not(svg *) { /* Avoid applying this inside SVGs where it might break things */
               clip-path: none !important;
               -webkit-clip-path: none !important; /* For older WebKit browsers */
            }
      
            /* Specifically target image elements */
            img {
              border-radius: 0 !important;
              clip-path: none !important;
              -webkit-clip-path: none !important;
            }
          `;
      
          // --- Specific Fixes for SVG Elements ---
          // Targets SVG elements commonly used for icons or avatars.
          css += `
            /* Unround rect elements within SVG */
            svg rect {
              rx: 0 !important;
              ry: 0 !important;
            }
      
            /* Try to disable masks/clip-paths specifically on SVG image elements */
            svg image {
               mask: none !important;
               clip-path: none !important;
            }
      
            /* Remove border-radius from SVG elements themselves if applied */
            svg {
              border-radius: 0 !important;
            }
      
            /* NOTE: Aggressively modifying generic SVG <circle> or <path> elements
               can easily break icons and logos. Site-specific overrides below are
               generally safer for complex SVG modifications. */
          `;
      
          // --- Compatibility Layer for Border Widths ---
          // Attempts to reset border-width for form elements.
          css += `
            @layer i_miss_true_user_origin_level_stylesheets {
              :where(input, button, select) {
                border-width: 1px;
              }
            }
          `;
      
          // --- Site-Specific Overrides ---
          // Apply fixes for specific websites known to use complex rounding methods.
      
          // Facebook & Workplace
          if ((location.hostname === "facebook.com" || location.hostname.endsWith(".facebook.com"))) {
              css += `
                /* Square masks used for avatars/status indicators */
                :root#facebook svg[role] > mask[id]:first-child + g[mask]:last-child {
                  mask: none !important;
                  /* Hide the original circle outline */
                  & circle { opacity: 0.1 !important; }
                  /* Add a squared outline using the image element */
                  & image:has(~ circle[stroke="var(--accent)"]) {
                    outline: 2px solid var(--accent);
                    outline-offset: 2px;
                    & ~ circle { display: none; } /* Ensure the original circle is hidden */
                  }
                }
      
                /* Facebook Logo - Unrounding the container and the "f" shape */
                :root#facebook svg[viewBox="0 0 36 36"] path {
                  /* Unround the main background circle to a square */
                  &[d="M20.181 35.87C29.094 34.791 36 27.202 36 18c0-9.941-8.059-18-18-18S0 8.059 0 18c0 8.442 5.811 15.526 13.652 17.471L14 34h5.5l.681 1.87Z"] ,
                  &[d="M15 35.8C6.5 34.3 0 26.9 0 18 0 8.1 8.1 0 18 0s18 8.1 18 18c0 8.9-6.5 16.3-15 17.8l-1-.8h-4l-1 .8z"] {
                    d: path("M0 0 H 36 V 36 H 0 Z"); /* Replace circle path with square path */
                  }
                  /* Unround the bottom of the "f" shape */
                  &[d="M13.651 35.471v-11.97H9.936V18h3.715v-2.37c0-6.127 2.772-8.964 8.784-8.964 1.138 0 3.103.223 3.91.446v4.983c-.425-.043-1.167-.065-2.081-.065-2.952 0-4.09 1.116-4.09 4.025V18h5.883l-1.008 5.5h-4.867v12.37a18.183 18.183 0 0 1-6.53-.399Z"],
                  &[d="M25 23l.8-5H21v-3.5c0-1.4.5-2.5 2.7-2.5H26V7.4c-1.3-.2-2.7-.4-4-.4-4.1 0-7 2.5-7 7v4h-4.5v5H15    V 35 h 6    V23h4z"] {
                    d: path("M25 23l.8-5H21v-3.5c0-1.4.5-2.5 2.7-2.5H26V7.4c-1.3-.2-2.7-.4-4-.4-4.1 0-7 2.5-7 7v4h-4.5v5H15 V 36 h 6 V23h4z"); /* Adjust path to be straight at the bottom */
                  }
                }
              `;
          }
      
          // Twitter / X
          if ((location.hostname === "twitter.com" || location.hostname.endsWith(".twitter.com")) || (location.hostname === "x.com" || location.hostname.endsWith(".x.com"))) {
              css += `
                /* Disable clip-path used for avatars (already targeted globally, but keep for specificity) */
                [style*='clip-path: url("#circle-hw-shapeclip-clipconfig")'],
                [style*='clip-path: circle'], /* More general clip-path targeting */
                [style*='-webkit-clip-path: circle'] {
                  clip-path: none !important;
                  -webkit-clip-path: none !important;
                }
      
                /* Target potential container elements */
                div[data-testid="UserAvatar-Container"] > div > div {
                   border-radius: 0 !important;
                   clip-path: none !important;
                   -webkit-clip-path: none !important;
                }
              `;
          }
      
          // WhatsApp Web
          if ((location.hostname === "web.whatsapp.com" || location.hostname.endsWith(".web.whatsapp.com"))) {
              css += `
                /* Target profile picture elements specifically */
                div[data-testid="chat-list-item-container"] img, /* Chat list */
                div[data-testid="conversation-header"] img, /* Conversation header */
                div[data-testid="status-v3-avatar-sender"] img, /* Status avatar */
                div[data-testid="profile-viewer-photo"] img, /* Profile picture viewer */
                custom-status-modal img, /* Status modal */
                div[role="button"] > div > div > img[draggable="false"] /* Other potential avatar locations */
                {
                  border-radius: 0 !important;
                  clip-path: none !important;
                  -webkit-clip-path: none !important;
                }
      
                /* Fixes for specific SVG backgrounds/shapes in WhatsApp */
                svg:has(path.background) {
                  background-color: rgba(var(--white-rgb),.16); /* Restore background color */
                }
                svg:has(path.background) g path{
                  opacity: .3; /* Restore opacity */
                }
                svg:has(path.background) path.background {
                  display: none; /* Hide the original background path */
                }
              `;
          }
      
          // Google / Chromium
          if ((location.hostname === "google.com" || location.hostname.endsWith(".google.com")) || (location.hostname === "chromium.org" || location.hostname.endsWith(".chromium.org"))) {
              css += `
                /* Google Account profile pictures */
                img.gb_Ia.gbii, /* Top bar avatar */
                img[src*='googleusercontent.com/a/'], /* Common avatar URL pattern */
                img[srcset*='googleusercontent.com/a/'],
                img[aria-label*='Profile picture'],
                img[alt*='Profile picture'],
                img[jsname] /* Generic attribute often on profile pics */
                {
                   border-radius: 0 !important;
                   clip-path: none !important;
                   -webkit-clip-path: none !important;
                }
      
                /* Google One avatar four-colour outline - replacing curved paths with straight lines */
                path[d="M4.02,28.27C2.73,25.8,2,22.98,2,20c0-2.87,0.68-5.59,1.88-8l-1.72-1.04C0.78,13.67,0,16.75,0,20c0,3.31,0.8,6.43,2.23,9.18L4.02,28.27z"][fill="#F6AD01"] {
                  d: path("M 2 2 v 36 l 2 -2 V 4 Z"); /* Left segment */
                }
                path[d="M32.15,33.27C28.95,36.21,24.68,38,20,38c-6.95,0-12.98-3.95-15.99-9.73l-1.79,0.91C5.55,35.61,12.26,40,20,40c5.2,0,9.93-1.98,13.48-5.23L32.15,33.27z"][fill="#249A41"] {
                  d: path("M 2 38 h 36 l -2 -2 H 4 Z"); /* Bottom segment */
                }
                path[d="M33.49,34.77C37.49,31.12,40,25.85,40,20c0-5.86-2.52-11.13-6.54-14.79l-1.37,1.46C35.72,9.97,38,14.72,38,20c0,5.25-2.26,9.98-5.85,13.27L33.49,34.77z"][fill="#3174F1"] {
                  d: path("M 38 2 v 36 l -2 -2 V 4 Z"); /* Right segment */
                }
                path[d="M20,2c4.65,0,8.89,1.77,12.09,4.67l1.37-1.46C29.91,1.97,25.19,0,20,0l0,0C12.21,0,5.46,4.46,2.16,10.96L3.88,12C6.83,6.08,12.95,2,20,2"][fill="#E92D18"] {
                  d: path("M 2 2 h 36 l -2 2 H 4 Z"); /* Top segment */
                }
              `;
          }
      
          // Spotify - Corrected Hostname Check
          // Targeting open.spotify.com (web player) and accounts.spotify.com
          if ((location.hostname === "open.spotify.com" || location.hostname.endsWith(".open.spotify.com")) || (location.hostname === "accounts.spotify.com" || location.hostname.endsWith(".accounts.spotify.com"))) {
              css += `
                /* Spotify profile pictures / artist images */
                img[data-testid="user-widget-avatar"], /* User avatar top right */
                figure[data-testid="user-widget-avatar"] img,
                img._6fa2d5efe1456f4e563658938a08641f-scss, /* Common class pattern */
                div[data-testid="avatar"] img, /* Playlist/Artist avatars */
                figure[data-testid="image-loading-indicator"] img, /* Generic image loader */
                div[data-encore-id="avatar"] img /* Encore UI avatars */
                {
                   border-radius: 0 !important;
                   clip-path: none !important;
                   -webkit-clip-path: none !important;
                }
      
                /* Specific path overrides for Spotify icons (e.g., checkmarks, circles) */
                /* Checkmark in circle */
                path[d="M1 12C1 5.925 5.925 1 12 1s11 4.925 11 11-4.925 11-11 11S1 18.075 1 12zm16.398-2.38a1 1 0 0 0-1.414-1.413l-6.011 6.01-1.894-1.893a1 1 0 0 0-1.414 1.414l3.308 3.308 7.425-7.425z"] {
                  d: path("M 4 4 H 20 V 20 H 4 Z M1 12m16.398-2.38a1 1 0 0 0-1.414-1.413l-6.011 6.01-1.894-1.893a1 1 0 0 0-1.414 1.414l3.308 3.308 7.425-7.425z");
                }
                /* Another checkmark in circle */
                path[d="M0 8a8 8 0 1 1 16 0A8 8 0 0 1 0 8zm11.748-1.97a.75.75 0 0 0-1.06-1.06l-4.47 4.47-1.405-1.406a.75.75 0 1 0-1.061 1.06l2.466 2.467 5.53-5.53z"] {
                  d: path("M 2 2 H 14 V 14 H 2 Z M0 8m11.748-1.97a.75.75 0 0 0-1.06-1.06l-4.47 4.47-1.405-1.406a.75.75 0 1 0-1.061 1.06l2.466 2.467 5.53-5.53z");
                }
                /* Circle outline */
                path[d="M11.999 3a9 9 0 1 0 0 18 9 9 0 0 0 0-18zm-11 9c0-6.075 4.925-11 11-11s11 4.925 11 11-4.925 11-11 11-11-4.925-11-11z"] {
                  d: path("M 2 2 H 22 V 22 H 2 Z"); /* Unround circle outline */
                  stroke-width: 2px;
                  fill: none;
                  stroke: color-mix(in srgb,currentcolor, transparent);
                }
                /* Smaller circle outline */
                path[d="M8 1.5a6.5 6.5 0 1 0 0 13 6.5 6.5 0 0 0 0-13zM0 8a8 8 0 1 1 16 0A8 8 0 0 1 0 8z"] {
                  d: path("M 1 1 H 15 V 15 H 1 Z"); /* Unround smaller circle outline */
                  stroke-width: 1.25px;
                  fill: none;
                  stroke: color-mix(in srgb,currentcolor, transparent);
                }
              `;
          }
      
          // Microsoft Copilot
          if ((location.hostname === "copilot.microsoft.com" || location.hostname.endsWith(".copilot.microsoft.com"))) {
              css += `
                /* Disable clip-path or other properties used for "squircle" shapes */
                [class*="squircle"],
                cib-serp-feedback ::part(avatar-image), /* Target shadow parts */
                cib-participant-avatar ::part(avatar-image)
                 {
                  clip-path: none !important;
                  -webkit-clip-path: none !important;
                  border-radius: 0 !important; /* Ensure border-radius is also zeroed */
                }
              `;
          }
      
          // Seznam Zprávy
          if ((location.hostname === "seznamzpravy.cz" || location.hostname.endsWith(".seznamzpravy.cz"))) {
              css += `
                /* Disable masks potentially used for squircle images */
                svg > defs:has([id*="squircle"]) ~ image[mask]:only-of-type {
                  mask: none !important;
                }
                /* Target images directly */
                img.atm-image-placeholder__image {
                   border-radius: 0 !important;
                   clip-path: none !important;
                   -webkit-clip-path: none !important;
                }
              `;
          }
      
          // --- Inject CSS ---
          // Uses GM_addStyle if available (Tampermonkey, Greasemonkey) or falls back
          // to creating a <style> element and appending it to the document head.
          if (typeof GM_addStyle !== "undefined") {
              GM_addStyle(css);
          } else {
              const styleNode = document.createElement("style");
              styleNode.setAttribute("type", "text/css"); // Good practice
              styleNode.appendChild(document.createTextNode(css));
              // Wait for head or body to be available
              const observer = new MutationObserver((mutations, obs) => {
                  const head = document.querySelector("head");
                  if (head) {
                      head.appendChild(styleNode);
                      obs.disconnect(); // Stop observing once appended
                  } else if (document.body) { // Fallback to body if head isn't found quickly
                      document.body.insertAdjacentElement('afterbegin', styleNode);
                      obs.disconnect();
                  }
              });
              // Start observing the document structure
              observer.observe(document.documentElement, { childList: true, subtree: true });
          }
      })();



</details>

