// ==UserScript==
// @name         Omega02 Skin Patcher
// @namespace    http://tampermonkey.net/
// @version      1.2
// @description  Patches Omega02 skin with custom image and soul anim
// @author       You
// @match        https://gameofbombs.com/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    const newUrl = 'https://raw.githubusercontent.com/Kix34/Scripts/refs/heads/main/Omega021.png';

    function patchSkin() {
        if (typeof sprites === 'undefined' || !sprites._dynlist) return false;

        const spr = sprites._dynlist.find(s => s.name === 'Omega02');
        if (!spr) return false;

        spr.src = newUrl;

        if (spr.texture) {
            spr.texture.src = newUrl;
        } else {
            spr.texture = { src: newUrl, element: null, width: 0, height: 0 };
        }

        Object.assign(spr.skin, {
            renderWidth       : 80,
            renderHeight      : 80,
            animSpeed         : 2400,
            animMirrorLeft    : false,
            publicFrame       : [2, 0],
            animCount         : 4,
            animCountSide     : 4,
            frameWidth        : 55,
            frameHeight       : 55,
            soulWidth         : 55,
            lookLut           : [0, 1, 2, 3, 4, 5, 6, 7],
            idleAnim          : true,
            idleAnimSpeed     : 4000,
            idleCol           : [0, 1, 2, 3],
            idleAnimCount     : 4,
            idleAnimCountSide : 4,
            hasSoul           : true
        });

        // mirror top-level render values too
        spr.renderWidth  = 80;
        spr.renderHeight = 80;

        const img = new Image();
        img.crossOrigin = 'anonymous';
        img.onload = () => {
            spr.texture.width  = img.width;
            spr.texture.height = img.height;
            try {
                spr.texture.element = img;
            } catch(e) {
                console.warn('[Omega02] Could not set texture.element:', e);
            }
            spr.loaded = true;
            console.log(`[Omega02] Patched! ${img.width}x${img.height}`);
        };
        img.onerror = () => console.warn('[Omega02] Image failed to load — check CORS');
        img.src = newUrl;

        return true;
    }

    const interval = setInterval(() => {
        if (patchSkin()) clearInterval(interval);
    }, 500);

})();
