// ==UserScript==
// @name         Game of Bombs - Real Camera Zoom
// @namespace    http://tampermonkey.net/
// @version      2.0.0
// @description  Changes the game's actual camera scale so world copies/culling expand correctly.
// @match        https://gameofbombs.com/*
// @match        http://gameofbombs.com/*
// @run-at       document-start
// @grant        none
// ==/UserScript==

(() => {
    'use strict';

    if (window.__GOB_REAL_CAMERA_ZOOM_V2__) return;
    window.__GOB_REAL_CAMERA_ZOOM_V2__ = true;

    const STORAGE_KEY = 'gob_real_camera_zoom_v2';

    // Lower = zoom out
    // Higher = zoom in
    const MIN_ZOOM = 0.40;
    const MAX_ZOOM = 1.80;
    const STEP = 0.05;

    let zoom = parseFloat(localStorage.getItem(STORAGE_KEY));

    if (!Number.isFinite(zoom)) {
        zoom = 1;
    }

    zoom = Math.max(MIN_ZOOM, Math.min(MAX_ZOOM, zoom));

    let camera = null;

    let zoomText = null;
    let statusText = null;

    // ============================================================
    // CAMERA IDENTIFICATION
    // ============================================================

    function isNumber(v) {
        return typeof v === 'number' && Number.isFinite(v);
    }

    function looksLikeCamera(obj) {
        if (!obj || typeof obj !== 'object') {
            return false;
        }

        /*
            Camera discovered from game source:

            c = camera X
            d = camera Y
            s = camera scale

            e/f = viewport dimensions

            n = f / s
            k = e / s

            o/q/p/r = world-space viewport bounds
        */

        const required = [
            'a', 'b',
            'c', 'd',
            'e', 'f',
            's',
            'n', 'k',
            'o', 'p',
            'q', 'r'
        ];

        for (const key of required) {
            if (!(key in obj)) {
                return false;
            }

            if (!isNumber(obj[key])) {
                return false;
            }
        }

        if (obj.s <= 0 || obj.s > 10) {
            return false;
        }

        if (obj.e <= 100 || obj.f <= 100) {
            return false;
        }

        /*
            Confirm its fields approximately obey the game's
            camera equations.
        */

        const expectedN = obj.f / obj.s;
        const expectedK = obj.e / obj.s;

        if (Math.abs(obj.n - expectedN) > 5) {
            return false;
        }

        if (Math.abs(obj.k - expectedK) > 5) {
            return false;
        }

        return true;
    }

    // ============================================================
    // SAFE OBJECT GRAPH SEARCH
    // ============================================================

    function findCameraFromRoot(root, maxDepth = 7) {
        if (!root) return null;

        const visited = new WeakSet();

        const queue = [{
            value: root,
            depth: 0
        }];

        let checked = 0;

        while (queue.length) {
            const item = queue.shift();

            const value = item.value;
            const depth = item.depth;

            if (
                !value ||
                (
                    typeof value !== 'object' &&
                    typeof value !== 'function'
                )
            ) {
                continue;
            }

            if (visited.has(value)) {
                continue;
            }

            visited.add(value);

            checked++;

            if (checked > 30000) {
                break;
            }

            try {
                if (looksLikeCamera(value)) {
                    return value;
                }
            } catch {}

            if (depth >= maxDepth) {
                continue;
            }

            let keys;

            try {
                keys = Object.keys(value);
            } catch {
                continue;
            }

            /*
                Prevent huge arrays / DOM trees from exploding
                the search.
            */

            if (keys.length > 250) {
                keys = keys.slice(0, 250);
            }

            for (const key of keys) {
                /*
                    Skip properties that are known to lead to massive
                    browser object graphs.
                */

                if (
                    key === 'parentNode' ||
                    key === 'parentElement' ||
                    key === 'children' ||
                    key === 'childNodes' ||
                    key === 'ownerDocument' ||
                    key === 'document' ||
                    key === 'window'
                ) {
                    continue;
                }

                let next;

                try {
                    next = value[key];
                } catch {
                    continue;
                }

                if (
                    next &&
                    (
                        typeof next === 'object' ||
                        typeof next === 'function'
                    )
                ) {
                    queue.push({
                        value: next,
                        depth: depth + 1
                    });
                }
            }
        }

        return null;
    }

    function getSearchRoots() {
        const roots = [];

        const layer0 =
            document.getElementById('layer0');

        const layer1 =
            document.getElementById('layer1');

        /*
            GWT commonly stores its Java object on __listener.
        */

        for (const canvas of [layer0, layer1]) {
            if (!canvas) continue;

            if (canvas.__listener) {
                roots.push(canvas.__listener);
            }

            let p = canvas.parentElement;

            let levels = 0;

            while (p && levels < 6) {
                if (p.__listener) {
                    roots.push(p.__listener);
                }

                p = p.parentElement;
                levels++;
            }
        }

        /*
            Some useful game objects may also be exposed globally.
            Only inspect likely ones instead of crawling the entire
            Window object.
        */

        const interestingNames =
            Object.getOwnPropertyNames(window)
                .filter(name =>
                    /game|view|camera|client|world|plane|canvas/i
                        .test(name)
                );

        for (const name of interestingNames) {
            try {
                const value = window[name];

                if (
                    value &&
                    (
                        typeof value === 'object' ||
                        typeof value === 'function'
                    )
                ) {
                    roots.push(value);
                }
            } catch {}
        }

        return roots;
    }

    function findRealCamera() {
        if (camera && looksLikeCamera(camera)) {
            return camera;
        }

        const roots = getSearchRoots();

        for (const root of roots) {
            const found =
                findCameraFromRoot(root, 8);

            if (found) {
                camera = found;

                console.log(
                    '[GOB Zoom] Real camera found:',
                    camera
                );

                console.log(
                    '[GOB Zoom] Camera dimensions:',
                    camera.e,
                    camera.f
                );

                return camera;
            }
        }

        return null;
    }

    // ============================================================
    // CAMERA UPDATE
    // ============================================================

    function applyCameraScale(cam, scale) {
        /*
            This recreates the game's own ez(camera,x,y,scale)
            function found in app.nocache.js.

            Original:

            a.c = b;
            a.d = c;
            a.s = d;
            a.n = a.f / d;
            a.k = a.e / d;

            a.o = a.c - a.a/a.s - a.n/2;
            a.q = a.d - a.b/a.s - a.k/2;

            a.p = a.c - a.a/a.s + a.n/2;
            a.r = a.d - a.b/a.s + a.k/2;
        */

        cam.s = scale;

        cam.n = cam.f / scale;
        cam.k = cam.e / scale;

        cam.o =
            cam.c -
            cam.a / scale -
            cam.n / 2;

        cam.q =
            cam.d -
            cam.b / scale -
            cam.k / 2;

        cam.p =
            cam.c -
            cam.a / scale +
            cam.n / 2;

        cam.r =
            cam.d -
            cam.b / scale +
            cam.k / 2;
    }

    function applyZoom() {
        const cam = findRealCamera();

        if (!cam) {
            if (statusText) {
                statusText.textContent =
                    'finding camera';
            }

            return false;
        }

        applyCameraScale(
            cam,
            zoom
        );

        if (statusText) {
            statusText.textContent =
                'camera linked';
        }

        return true;
    }

    // ============================================================
    // ZOOM
    // ============================================================

    function clampZoom(v) {
        return Math.max(
            MIN_ZOOM,
            Math.min(
                MAX_ZOOM,
                Math.round(v * 100) / 100
            )
        );
    }

    function setZoom(v) {
        zoom = clampZoom(v);

        localStorage.setItem(
            STORAGE_KEY,
            String(zoom)
        );

        applyZoom();

        updateUI();

        console.log(
            '[GOB Zoom]',
            Math.round(zoom * 100) + '%'
        );
    }

    function zoomOut() {
        setZoom(
            zoom - STEP
        );
    }

    function zoomIn() {
        setZoom(
            zoom + STEP
        );
    }

    function resetZoom() {
        setZoom(1);
    }

    // ============================================================
    // KEEP CAMERA SCALE SYNCHRONIZED
    // ============================================================

    /*
        The game may update the camera each frame or when your
        character moves.

        Reapply the actual camera scale at a low cost.
    */

    let lastScale = null;

    function cameraLoop() {
        const cam = findRealCamera();

        if (cam) {
            if (
                Math.abs(cam.s - zoom) >
                0.0001
            ) {
                applyCameraScale(
                    cam,
                    zoom
                );
            }

            if (lastScale !== cam.s) {
                lastScale = cam.s;
            }
        }

        requestAnimationFrame(
            cameraLoop
        );
    }

    // ============================================================
    // UI
    // ============================================================

    function updateUI() {
        if (zoomText) {
            zoomText.textContent =
                Math.round(zoom * 100) + '%';
        }
    }

    function makeButton(
        text,
        title,
        callback
    ) {
        const button =
            document.createElement('button');

        button.textContent = text;
        button.title = title;

        button.style.cssText = `
            width:30px;
            height:28px;

            padding:0;

            border:1px solid #555;
            border-radius:5px;

            background:#222;
            color:#fff;

            font-size:16px;

            cursor:pointer;
        `;

        button.addEventListener(
            'click',
            event => {
                event.preventDefault();
                event.stopPropagation();

                callback();
            }
        );

        return button;
    }

    function createUI() {
        if (!document.body) return;

        if (
            document.getElementById(
                'gob-real-camera-zoom-ui'
            )
        ) {
            return;
        }

        const panel =
            document.createElement('div');

        panel.id =
            'gob-real-camera-zoom-ui';

        panel.style.cssText = `
            position:fixed;
            right:15px;
            bottom:15px;

            z-index:2147483647;

            display:flex;
            align-items:center;
            gap:6px;

            padding:7px 9px;

            background:
                rgba(20,20,20,.93);

            border:
                1px solid #555;

            border-radius:
                8px;

            color:white;

            font:
                13px Arial,sans-serif;

            user-select:none;
        `;

        const title =
            document.createElement('span');

        title.textContent =
            'Zoom';

        title.style.fontWeight =
            'bold';

        const minus =
            makeButton(
                '−',
                'Zoom out [',
                zoomOut
            );

        zoomText =
            document.createElement('span');

        zoomText.style.cssText = `
            min-width:42px;
            text-align:center;
            font-family:monospace;
        `;

        const plus =
            makeButton(
                '+',
                'Zoom in ]',
                zoomIn
            );

        const reset =
            makeButton(
                '↺',
                'Reset',
                resetZoom
            );

        statusText =
            document.createElement('span');

        statusText.style.cssText = `
            min-width:80px;
            font-size:10px;
            color:#aaa;
        `;

        statusText.textContent =
            'finding camera';

        panel.appendChild(title);
        panel.appendChild(minus);
        panel.appendChild(zoomText);
        panel.appendChild(plus);
        panel.appendChild(reset);
        panel.appendChild(statusText);

        document.body.appendChild(
            panel
        );

        updateUI();
    }

    // ============================================================
    // HOTKEYS
    // ============================================================

    window.addEventListener(
        'keydown',
        event => {
            const active =
                document.activeElement;

            if (
                active &&
                (
                    active.tagName ===
                        'INPUT' ||
                    active.tagName ===
                        'TEXTAREA' ||
                    active.isContentEditable
                )
            ) {
                return;
            }

            if (event.key === '[') {
                event.preventDefault();
                zoomOut();
            }

            else if (event.key === ']') {
                event.preventDefault();
                zoomIn();
            }

            else if (
                event.key === '\\'
            ) {
                event.preventDefault();
                resetZoom();
            }
        },
        true
    );

    // ============================================================
    // START
    // ============================================================

    function start() {
        createUI();

        requestAnimationFrame(
            cameraLoop
        );

        /*
            Retry after the match UI loads.
        */

        const finder =
            setInterval(() => {
                if (findRealCamera()) {
                    applyZoom();

                    clearInterval(
                        finder
                    );
                }
            }, 500);
    }

    if (
        document.readyState ===
        'loading'
    ) {
        document.addEventListener(
            'DOMContentLoaded',
            start,
            {
                once:true
            }
        );
    } else {
        start();
    }

})();
