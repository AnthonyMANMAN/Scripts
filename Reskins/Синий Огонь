// ==UserScript==
// @name         Game of Bombs - Blue Explosion
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  Recolors the bomb explosion sprite to blue in Game of Bombs
// @author       You
// @match        https://gameofbombs.com/*
// @grant        none
// @run-at       document-start
// ==/UserScript==

(function () {
    'use strict';

    // --- Config ---
    // Hue rotation in degrees. 0=original (orange/red), ~200=blue
    const HUE_ROTATE_DEG = 200;
    // Optionally boost saturation (1 = no change, 1.5 = 50% more)
    const SATURATION = 1.4;

    const EXPLOSION_URL_PATTERN = /explosion\.png/i;

    // Recolor an Image element using canvas and return a new object URL
    function recolorImage(img) {
        const canvas = document.createElement('canvas');
        canvas.width = img.naturalWidth;
        canvas.height = img.naturalHeight;
        const ctx = canvas.getContext('2d');

        // Draw the original image
        ctx.drawImage(img, 0, 0);

        // Get pixel data
        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const data = imageData.data;

        // Convert each pixel's hue
        for (let i = 0; i < data.length; i += 4) {
            const r = data[i], g = data[i + 1], b = data[i + 2], a = data[i + 3];
            if (a === 0) continue; // skip transparent pixels

            // Convert RGB -> HSL
            const [h, s, l] = rgbToHsl(r, g, b);

            // Rotate hue and adjust saturation
            const newH = (h + HUE_ROTATE_DEG / 360) % 1;
            const newS = Math.min(1, s * SATURATION);

            // Convert back to RGB
            const [nr, ng, nb] = hslToRgb(newH, newS, l);
            data[i] = nr;
            data[i + 1] = ng;
            data[i + 2] = nb;
        }

        ctx.putImageData(imageData, 0, 0);
        return canvas.toDataURL('image/png');
    }

    // --- Color conversion helpers ---
    function rgbToHsl(r, g, b) {
        r /= 255; g /= 255; b /= 255;
        const max = Math.max(r, g, b), min = Math.min(r, g, b);
        let h, s, l = (max + min) / 2;
        if (max === min) {
            h = s = 0;
        } else {
            const d = max - min;
            s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
            switch (max) {
                case r: h = ((g - b) / d + (g < b ? 6 : 0)) / 6; break;
                case g: h = ((b - r) / d + 2) / 6; break;
                case b: h = ((r - g) / d + 4) / 6; break;
            }
        }
        return [h, s, l];
    }

    function hslToRgb(h, s, l) {
        let r, g, b;
        if (s === 0) {
            r = g = b = l;
        } else {
            const hue2rgb = (p, q, t) => {
                if (t < 0) t += 1;
                if (t > 1) t -= 1;
                if (t < 1/6) return p + (q - p) * 6 * t;
                if (t < 1/2) return q;
                if (t < 2/3) return p + (q - p) * (2/3 - t) * 6;
                return p;
            };
            const q = l < 0.5 ? l * (1 + s) : l + s - l * s;
            const p = 2 * l - q;
            r = hue2rgb(p, q, h + 1/3);
            g = hue2rgb(p, q, h);
            b = hue2rgb(p, q, h - 1/3);
        }
        return [Math.round(r * 255), Math.round(g * 255), Math.round(b * 255)];
    }

    // --- Intercept Image loading ---
    const OriginalImage = window.Image;

    window.Image = function (width, height) {
        const img = new OriginalImage(width, height);
        const originalSrcDescriptor = Object.getOwnPropertyDescriptor(HTMLImageElement.prototype, 'src');

        let _src = '';

        Object.defineProperty(img, 'src', {
            get() { return _src; },
            set(value) {
                _src = value;
                if (EXPLOSION_URL_PATTERN.test(value)) {
                    // Load the real image first, then recolor it
                    const tempImg = new OriginalImage();
                    tempImg.crossOrigin = 'anonymous';
                    tempImg.onload = () => {
                        const recolored = recolorImage(tempImg);
                        originalSrcDescriptor.set.call(img, recolored);
                    };
                    tempImg.onerror = () => {
                        // Fallback: load original if recolor fails
                        originalSrcDescriptor.set.call(img, value);
                    };
                    tempImg.src = value;
                } else {
                    originalSrcDescriptor.set.call(img, value);
                }
            },
            configurable: true
        });

        return img;
    };

    // Copy prototype so instanceof checks still work
    window.Image.prototype = OriginalImage.prototype;

    console.log('[Blue Explosion] Userscript loaded — explosions will be recolored blue.');
})();
