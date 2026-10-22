// ==UserScript==
// @name         Game of Bombs - Cyberpunk Reskin + Blue Explosions
// @namespace    http://tampermonkey.net/
// @version      3.1
// @description  Cyberpunk tileset/atlas/items reskin + blue explosion recolor for gameofbombs.com
// @author       You
// @match        *://gameofbombs.com/*
// @grant        GM_xmlhttpRequest
// @connect      raw.githubusercontent.com
// @run-at       document-start
// ==/UserScript==

(function () {
    'use strict';

    // ─── Config ───────────────────────────────────────────────────────────────

    const REPLACEMENTS = [
        {
            pattern: /tileset\.png/i,
            url: 'https://raw.githubusercontent.com/AnthonyMANMAN/Scripts/main/Reskins/Cyberpunk/tileset-cyberpunk.png',
            blob: null
        },
        {
            pattern: /atlas6\.png/i,
            url: 'https://raw.githubusercontent.com/AnthonyMANMAN/Scripts/main/Reskins/Cyberpunk/atlas-cyberpunk.png',
            blob: null
        },
        {
            pattern: /items\.png/i,
            url: 'https://raw.githubusercontent.com/AnthonyMANMAN/Scripts/main/Reskins/Cyberpunk/items-cyberpunk.png',
            blob: null
        }
    ];

    const EXPLOSION_PATTERN = /explosion\.png/i;
    const HUE_ROTATE_DEG    = 200;
    const SATURATION        = 1.4;

    // ─── Save originals BEFORE any patching ──────────────────────────────────

    const nativeSrc     = Object.getOwnPropertyDescriptor(HTMLImageElement.prototype, 'src');
    const OriginalImage = window.Image;

    // ─── Preload cyberpunk images as blobs ────────────────────────────────────

    REPLACEMENTS.forEach(entry => {
        GM_xmlhttpRequest({
            method: 'GET',
            url: entry.url,
            responseType: 'blob',
            onload(res) {
                entry.blob = URL.createObjectURL(res.response);
                console.log('[GOB] Blob ready:', entry.url);
            },
            onerror(err) {
                console.error('[GOB] Fetch failed:', entry.url, err);
            }
        });
    });

    // ─── Color conversion helpers ─────────────────────────────────────────────

    function rgbToHsl(r, g, b) {
        r /= 255; g /= 255; b /= 255;
        const max = Math.max(r, g, b), min = Math.min(r, g, b);
        const l = (max + min) / 2;
        if (max === min) return [0, 0, l];
        const d = max - min;
        const s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
        let h;
        switch (max) {
            case r: h = ((g - b) / d + (g < b ? 6 : 0)) / 6; break;
            case g: h = ((b - r) / d + 2) / 6; break;
            case b: h = ((r - g) / d + 4) / 6; break;
        }
        return [h, s, l];
    }

    function hslToRgb(h, s, l) {
        if (s === 0) { const v = Math.round(l * 255); return [v, v, v]; }
        const q = l < 0.5 ? l * (1 + s) : l + s - l * s;
        const p = 2 * l - q;
        const hue2rgb = (p, q, t) => {
            if (t < 0) t += 1; if (t > 1) t -= 1;
            if (t < 1/6) return p + (q - p) * 6 * t;
            if (t < 1/2) return q;
            if (t < 2/3) return p + (q - p) * (2/3 - t) * 6;
            return p;
        };
        return [
            Math.round(hue2rgb(p, q, h + 1/3) * 255),
            Math.round(hue2rgb(p, q, h)       * 255),
            Math.round(hue2rgb(p, q, h - 1/3) * 255)
        ];
    }

    // ─── Explosion recolor ────────────────────────────────────────────────────

    function recolorExplosion(originalUrl, targetImg) {
        const tempImg = new OriginalImage();
        tempImg.crossOrigin = 'anonymous';

        tempImg.onload = () => {
            const canvas = document.createElement('canvas');
            canvas.width  = tempImg.naturalWidth;
            canvas.height = tempImg.naturalHeight;
            const ctx = canvas.getContext('2d');
            ctx.drawImage(tempImg, 0, 0);

            const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
            const d = imageData.data;
            for (let i = 0; i < d.length; i += 4) {
                if (d[i + 3] === 0) continue;
                const [h, s, l] = rgbToHsl(d[i], d[i + 1], d[i + 2]);
                const [nr, ng, nb] = hslToRgb(
                    (h + HUE_ROTATE_DEG / 360) % 1,
                    Math.min(1, s * SATURATION),
                    l
                );
                d[i] = nr; d[i + 1] = ng; d[i + 2] = nb;
            }
            ctx.putImageData(imageData, 0, 0);
            nativeSrc.set.call(targetImg, canvas.toDataURL('image/png'));
        };

        tempImg.onerror = () => {
            console.warn('[GOB] Explosion recolor failed, using original');
            nativeSrc.set.call(targetImg, originalUrl);
        };

        nativeSrc.set.call(tempImg, originalUrl);
    }

    // ─── Single unified src interceptor ──────────────────────────────────────

    Object.defineProperty(HTMLImageElement.prototype, 'src', {
        get: nativeSrc.get,
        set(value) {
            if (typeof value !== 'string') {
                nativeSrc.set.call(this, value);
                return;
            }

            // 1. Cyberpunk asset swaps
            const replacement = REPLACEMENTS.find(e => e.pattern.test(value));
            if (replacement) {
                const target = replacement.blob || replacement.url;
                console.log(`[GOB] Swapping: ${value} → ${target}`);
                nativeSrc.set.call(this, target);
                return;
            }

            // 2. Blue explosion recolor
            if (EXPLOSION_PATTERN.test(value)) {
                recolorExplosion(value, this);
                return;
            }

            // 3. Pass through
            nativeSrc.set.call(this, value);
        },
        configurable: true
    });

    console.log('[GOB] Cyberpunk reskin (tileset + atlas + items) + blue explosions active.');
})();
