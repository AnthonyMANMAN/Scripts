// ==UserScript==
// @name         Game of Bombs — JP1 100 Guest WebSocket Swarm
// @namespace    http://tampermonkey.net/
// @version      1.3.0
// @description  Ultra-fast JP1 guest swarm using native cookie-less fetch; current tab session stays untouched.
// @match        https://gameofbombs.com/*
// @match        http://gameofbombs.com/*
// @run-at       document-start
// @grant        none
// ==/UserScript==

(() => {
    'use strict';

    if (window.__GOB_JP1_GUEST_SWARM__) return;
    Object.defineProperty(window, '__GOB_JP1_GUEST_SWARM__', {
        value: true,
        configurable: false,
        writable: false
    });

    // ============================================================
    // CONFIG
    // ============================================================

    const CONFIG = {
        defaultGuests: 100,
        roomId: 'JP1',
        version: '0.1.32',

        // Same guest flow as the working WebSocket version.
        guestApi: 'https://gameofbombs.com/api/v3/user/guest',

        // Limit guest-session HTTP creation bursts.
        guestConcurrency: 100,

        // Delay between starting clients.
        startStaggerMs: 0,

        // Reconnect delay.
        reconnectMinMs: 500,
        reconnectMaxMs: 1500,

        // Full character scans are relatively expensive.
        preSpawnScanEvery: 3,
        postSpawnScanEvery: 15,

        missingScansBeforeRespawn: 5,

        reportEveryMs: 1000
    };

    // ============================================================
    // PROTOCOL
    // ============================================================

    const MSG_MAP = 6;
    const MSG_STATE = 7;
    const MSG_CLIENT_ORDERS = 9;
    const MSG_SLOWMO = 17;

    const ORDER_VISION = 0;
    const ORDER_RESPAWN = 1;

    const UNIT_CHARACTER = 7;
    const UNIT_REGEN_TERRAIN = 8;

    function i32(bytes, offset) {
        return new DataView(
            bytes.buffer,
            bytes.byteOffset,
            bytes.byteLength
        ).getInt32(offset, true);
    }

    function u16(bytes, offset) {
        return new DataView(
            bytes.buffer,
            bytes.byteOffset,
            bytes.byteLength
        ).getUint16(offset, true);
    }

    function parseInitialHandshake(data) {
        if (
            data.length >= 16 &&
            data[4] === MSG_SLOWMO &&
            data[7] === MSG_MAP
        ) {
            return {
                streamId: i32(data, 0),
                playerId: i32(data, 8),
                tick: i32(data, 12)
            };
        }
        return null;
    }

    function parseStateTick(data) {
        if (
            data.length >= 9 &&
            data[4] === MSG_STATE
        ) {
            return {
                streamId: i32(data, 0),
                tick: i32(data, 5)
            };
        }
        return null;
    }

    function bytesToWireString(bytes) {
        let out = '';
        const CHUNK = 0x8000;

        for (let i = 0; i < bytes.length; i += CHUNK) {
            out += String.fromCharCode(
                ...bytes.subarray(i, i + CHUNK)
            );
        }

        return out;
    }

    async function frameToBytes(frame) {
        if (typeof frame === 'string') {
            const out = new Uint8Array(frame.length);
            for (let i = 0; i < frame.length; i++) {
                out[i] = frame.charCodeAt(i) & 0xff;
            }
            return out;
        }

        if (frame instanceof ArrayBuffer) {
            return new Uint8Array(frame);
        }

        if (ArrayBuffer.isView(frame)) {
            return new Uint8Array(
                frame.buffer,
                frame.byteOffset,
                frame.byteLength
            );
        }

        if (frame instanceof Blob) {
            return new Uint8Array(
                await frame.arrayBuffer()
            );
        }

        return new Uint8Array(0);
    }

    // ============================================================
    // CHARACTER DECODER
    // ============================================================

    function scanCharacter(data, playerId) {
        const end = data.length - 8;

        for (let offset = 117; offset < end; offset++) {
            if (data[offset + 2] !== UNIT_CHARACTER) continue;
            if (u16(data, offset + 3) !== playerId) continue;

            let terrainOffset = null;
            const lower = Math.max(6, offset - 512);

            for (
                let candidate = offset - 110;
                candidate > lower;
                candidate--
            ) {
                if (
                    candidate < 7 ||
                    candidate + 109 >= data.length
                ) {
                    continue;
                }

                const location = candidate - 7;
                const packedTerrain = data[candidate + 7];

                if (
                    data[candidate + 2] === UNIT_REGEN_TERRAIN &&
                    u16(data, candidate + 3) === 0 &&
                    data[location] === (packedTerrain & 0x1f) &&
                    data[candidate + 108] === 255 &&
                    data[candidate + 109] === 0
                ) {
                    terrainOffset = candidate;
                    break;
                }
            }

            if (terrainOffset === null) continue;

            const locationOffset = terrainOffset - 7;

            const chunkX = u16(data, locationOffset + 1);
            const chunkY = u16(data, locationOffset + 3);

            const packed = data[offset + 7];

            const localX =
                data[offset + 5] +
                ((packed & 0x40) << 2);

            const localY =
                data[offset + 6] +
                ((packed & 0x80) << 1);

            const fixedX = chunkX * 256 + localX;
            const fixedY = chunkY * 256 + localY;

            return {
                unitId: u16(data, offset),
                playerId,
                plane: packed & 0x1f,
                fixedX,
                fixedY,
                tileX: fixedX >> 5,
                tileY: fixedY >> 5,
                chunkX,
                chunkY
            };
        }

        return null;
    }

    // ============================================================
    // ORDER BUILDERS
    // ============================================================

    function visionOrder(playerId, nextTick, character) {
        const buffer = new ArrayBuffer(31);
        const view = new DataView(buffer);

        view.setUint8(0, ORDER_VISION);
        view.setUint16(1, 0, true);
        view.setUint16(3, playerId, true);
        view.setInt32(5, playerId, true);
        view.setInt32(9, nextTick, true);

        let values;

        if (!character) {
            values = [0, 0, 0, 0, 0, 0, 0, 0];
        } else {
            values = [
                character.plane,
                Math.max(0, character.tileX - 1),
                character.tileY,
                character.unitId,
                2,
                2,
                4,
                4
            ];
        }

        let offset = 13;

        for (const value of values) {
            const safe = Math.max(
                0,
                Math.min(65535, value | 0)
            );

            view.setUint16(
                offset,
                safe,
                true
            );

            offset += 2;
        }

        view.setUint8(29, 0);
        view.setUint8(30, 0);

        return new Uint8Array(buffer);
    }

    function respawnOrder(playerId) {
        const buffer = new ArrayBuffer(7);
        const view = new DataView(buffer);

        view.setUint8(0, ORDER_RESPAWN);
        view.setUint16(1, 0, true);
        view.setUint16(3, playerId, true);
        view.setUint8(5, 0);
        view.setUint8(6, 1);

        return new Uint8Array(buffer);
    }

    function clientOrders(streamId, tick, orders) {
        let total = 10;

        for (const order of orders) {
            total += order.length;
        }

        const buffer = new ArrayBuffer(total);
        const view = new DataView(buffer);
        const bytes = new Uint8Array(buffer);

        view.setUint8(0, MSG_CLIENT_ORDERS);
        view.setInt32(1, streamId, true);
        view.setInt32(5, tick, true);
        view.setUint8(9, orders.length);

        let offset = 10;

        for (const order of orders) {
            bytes.set(order, offset);
            offset += order.length;
        }

        return bytes;
    }

    // ============================================================
    // GUEST SESSION
    // ============================================================

    function randomGuestName() {
        return 'newbie' + String(
            Math.floor(10000 + Math.random() * 90000)
        );
    }

    async function isolatedGuestRequest() {
        // Native browser request, but explicitly detached from the current
        // Game of Bombs login session:
        //
        // credentials:'omit'
        //   - does not send the tab's cookies
        //   - does not accept/store Set-Cookie from this response
        //
        // This is much faster than GM_xmlhttpRequest while keeping guest
        // creation separate from the current tab's account/session.
        const response = await fetch(
            CONFIG.guestApi,
            {
                method: 'POST',
                credentials: 'omit',
                cache: 'no-store',
                redirect: 'follow',
                headers: {
                    'Content-Type': 'application/json',
                    'Accept': 'application/json'
                },
                body: JSON.stringify({
                    username: randomGuestName(),
                    referrer: ''
                })
            }
        );

        if (!response.ok) {
            throw new Error(
                `Guest HTTP ${response.status}`
            );
        }

        return await response.json();
    }

    async function createGuestSession() {
        // IMPORTANT:
        // Native fetch is used with credentials:'omit', so it neither sends
        // the current tab's cookies nor stores guest-session cookies.
        const payload =
            await isolatedGuestRequest();

        const user = payload.user;

        if (!user || !user.token) {
            throw new Error(
                'Guest API returned no user/token'
            );
        }

        let selected = null;
        let region = '';

        for (const server of user.servers || []) {
            for (const room of server.rooms || []) {
                if (
                    String(room.id || '').toUpperCase() ===
                    CONFIG.roomId
                ) {
                    selected = room;
                    region = String(
                        server.id || ''
                    ).toUpperCase();
                    break;
                }
            }

            if (selected) break;
        }

        if (!selected) {
            const available = [];

            for (const server of user.servers || []) {
                for (const room of server.rooms || []) {
                    if (room && room.id) {
                        available.push(
                            String(room.id)
                        );
                    }
                }
            }

            throw new Error(
                `${CONFIG.roomId} was not returned by guest API; ` +
                `available=${available.join(',')}`
            );
        }

        if (selected.cantEnter) {
            throw new Error(
                `${CONFIG.roomId} cannot be entered: ` +
                String(selected.cantPlayReason || '')
            );
        }

        if (!selected.domain) {
            throw new Error(
                `${CONFIG.roomId} has no domain`
            );
        }

        return {
            userId: Number(user.id),
            nickname:
                user.nickname ||
                user.username ||
                'guest',
            token: String(user.token),
            roomId: CONFIG.roomId,
            region,
            domain: String(selected.domain)
        };
    }

    function websocketUrl(session) {
        const params =
            'room=' +
            encodeURIComponent(session.roomId) +
            '&play' +
            '&auth=b' +
            '&version=' +
            encodeURIComponent(CONFIG.version) +
            '&token=' +
            encodeURIComponent(session.token);

        return (
            'wss://' +
            session.domain +
            '/?' +
            params
        );
    }

    // ============================================================
    // BOT
    // ============================================================

    class GuestBot {
        constructor(index, swarm) {
            this.index = index;
            this.swarm = swarm;

            this.ws = null;
            this.session = null;

            this.streamId = null;
            this.playerId = null;
            this.character = null;

            this.respawnSent = false;
            this.missingScans = 0;
            this.scanCounter = 0;

            this.connected = false;
            this.spawned = false;

            this.rx = 0;
            this.tx = 0;
            this.states = 0;
            this.errors = 0;

            this.dead = false;
            this.reconnectTimer = null;
        }

        resetRuntime() {
            this.streamId = null;
            this.playerId = null;
            this.character = null;

            this.respawnSent = false;
            this.missingScans = 0;
            this.scanCounter = 0;

            this.spawned = false;
        }

        shouldScanCharacter() {
            this.scanCounter++;

            const every = this.character
                ? CONFIG.postSpawnScanEvery
                : CONFIG.preSpawnScanEvery;

            if (this.scanCounter >= every) {
                this.scanCounter = 0;
                return true;
            }

            return false;
        }

        send(bytes) {
            if (
                !this.ws ||
                this.ws.readyState !== WebSocket.OPEN
            ) {
                return;
            }

            // Match the existing Python protocol transport:
            // arbitrary bytes encoded as a JS string.
            this.ws.send(
                bytesToWireString(bytes)
            );

            this.tx++;
        }

        async connect() {
            if (
                this.dead ||
                !this.swarm.running
            ) {
                return;
            }

            this.resetRuntime();

            try {
                if (!this.session) {
                    this.session =
                        await this.swarm.withGuestSlot(
                            () => createGuestSession()
                        );

                    this.swarm.sessionsCreated++;
                }

                if (
                    this.dead ||
                    !this.swarm.running
                ) {
                    return;
                }

                const ws = new WebSocket(
                    websocketUrl(this.session)
                );

                this.ws = ws;
                ws.binaryType = 'arraybuffer';

                ws.onopen = () => {
                    this.connected = true;
                };

                ws.onmessage = async event => {
                    if (
                        this.dead ||
                        !this.swarm.running
                    ) {
                        return;
                    }

                    try {
                        const data =
                            await frameToBytes(
                                event.data
                            );

                        this.rx++;

                        const handshake =
                            parseInitialHandshake(
                                data
                            );

                        if (handshake) {
                            this.streamId =
                                handshake.streamId;

                            this.playerId =
                                handshake.playerId;

                            return;
                        }

                        const state =
                            parseStateTick(
                                data
                            );

                        if (!state) return;

                        this.states++;

                        if (this.streamId === null) {
                            this.streamId =
                                state.streamId;
                        }

                        if (
                            state.streamId !==
                                this.streamId ||
                            this.playerId === null
                        ) {
                            return;
                        }

                        if (
                            this.shouldScanCharacter()
                        ) {
                            const character =
                                scanCharacter(
                                    data,
                                    this.playerId
                                );

                            if (character) {
                                this.character =
                                    character;

                                this.missingScans = 0;
                                this.spawned = true;
                            } else if (
                                this.character
                            ) {
                                this.missingScans++;
                            }
                        }

                        const orders = [
                            visionOrder(
                                this.playerId,
                                state.tick + 1,
                                this.character
                            )
                        ];

                        if (!this.respawnSent) {
                            orders.push(
                                respawnOrder(
                                    this.playerId
                                )
                            );

                            this.respawnSent = true;
                        } else if (
                            this.character &&
                            this.missingScans >=
                                CONFIG.missingScansBeforeRespawn
                        ) {
                            orders.push(
                                respawnOrder(
                                    this.playerId
                                )
                            );

                            this.character = null;
                            this.spawned = false;
                            this.missingScans = 0;
                            this.scanCounter = 0;
                        }

                        // Intentionally NO movement order.
                        // Intentionally NO bomb order.

                        this.send(
                            clientOrders(
                                this.streamId,
                                state.tick,
                                orders
                            )
                        );
                    } catch (error) {
                        this.errors++;
                        this.swarm.lastError =
                            `Bot ${this.index}: ` +
                            String(
                                error?.message ||
                                error
                            );
                    }
                };

                ws.onerror = () => {
                    this.errors++;
                };

                ws.onclose = event => {
                    this.connected = false;
                    this.spawned = false;

                    if (
                        this.dead ||
                        !this.swarm.running
                    ) {
                        return;
                    }

                    // Authentication failures should get
                    // a fresh guest session.
                    if (
                        event.code === 1008 ||
                        event.code === 4001 ||
                        event.code === 4003
                    ) {
                        this.session = null;
                    }

                    const delay =
                        CONFIG.reconnectMinMs +
                        Math.random() *
                        (
                            CONFIG.reconnectMaxMs -
                            CONFIG.reconnectMinMs
                        );

                    this.reconnectTimer =
                        setTimeout(
                            () => this.connect(),
                            delay
                        );
                };
            } catch (error) {
                this.errors++;

                this.swarm.lastError =
                    `Bot ${this.index}: ` +
                    String(
                        error?.message ||
                        error
                    );

                const delay =
                    CONFIG.reconnectMinMs +
                    Math.random() *
                    (
                        CONFIG.reconnectMaxMs -
                        CONFIG.reconnectMinMs
                    );

                this.reconnectTimer =
                    setTimeout(
                        () => this.connect(),
                        delay
                    );
            }
        }

        stop() {
            this.dead = true;
            this.connected = false;
            this.spawned = false;

            if (this.reconnectTimer) {
                clearTimeout(
                    this.reconnectTimer
                );
                this.reconnectTimer = null;
            }

            if (this.ws) {
                try {
                    this.ws.close();
                } catch (_) {}

                this.ws = null;
            }
        }
    }

    // ============================================================
    // SWARM
    // ============================================================

    class Swarm {
        constructor() {
            this.running = false;
            this.bots = [];

            this.activeGuestJobs = 0;
            this.guestWaiters = [];

            this.startedAt = 0;
            this.lastError = '';
            this.sessionsCreated = 0;

            this.reportTimer = null;
        }

        async withGuestSlot(fn) {
            if (
                this.activeGuestJobs >=
                CONFIG.guestConcurrency
            ) {
                await new Promise(resolve => {
                    this.guestWaiters.push(
                        resolve
                    );
                });
            }

            this.activeGuestJobs++;

            try {
                return await fn();
            } finally {
                this.activeGuestJobs--;

                const next =
                    this.guestWaiters.shift();

                if (next) next();
            }
        }

        start(count) {
            if (this.running) {
                this.stop();
            }

            count = Math.max(
                1,
                Math.min(
                    1000,
                    Number(count) || 100
                )
            );

            this.running = true;
            this.startedAt = Date.now();
            this.lastError = '';
            this.sessionsCreated = 0;
            this.bots = [];

            // Start every client immediately. Guest creation itself is
            // cookie-less and fully asynchronous. Browser/server transport
            // limits still provide natural backpressure.
            for (let i = 0; i < count; i++) {
                const bot =
                    new GuestBot(
                        i + 1,
                        this
                    );

                this.bots.push(bot);

                Promise.resolve().then(() => {
                    if (this.running) {
                        bot.connect();
                    }
                });
            }

            if (this.reportTimer) {
                clearInterval(
                    this.reportTimer
                );
            }

            this.reportTimer =
                setInterval(
                    () => updatePanel(),
                    CONFIG.reportEveryMs
                );

            updatePanel();
        }

        stop() {
            this.running = false;

            for (const bot of this.bots) {
                bot.stop();
            }

            this.bots = [];
            this.activeGuestJobs = 0;

            while (
                this.guestWaiters.length
            ) {
                const next =
                    this.guestWaiters.shift();

                try {
                    next();
                } catch (_) {}
            }

            if (this.reportTimer) {
                clearInterval(
                    this.reportTimer
                );

                this.reportTimer = null;
            }

            updatePanel();
        }

        stats() {
            let connected = 0;
            let spawned = 0;
            let rx = 0;
            let tx = 0;
            let states = 0;
            let errors = 0;

            for (const bot of this.bots) {
                connected += bot.connected
                    ? 1
                    : 0;

                spawned += bot.spawned
                    ? 1
                    : 0;

                rx += bot.rx;
                tx += bot.tx;
                states += bot.states;
                errors += bot.errors;
            }

            return {
                total: this.bots.length,
                connected,
                spawned,
                rx,
                tx,
                states,
                errors
            };
        }
    }

    const swarm = new Swarm();

    console.log(
        '[JP1 Swarm] Guest HTTP uses credentials:omit; current tab session is untouched.'
    );

    // ============================================================
    // SMALL UI
    // ============================================================

    let panel;
    let countInput;
    let statusText;
    let errorText;
    let startButton;
    let stopButton;

    function installPanel() {
        if (
            document.getElementById(
                'gob-jp1-swarm-panel'
            )
        ) {
            return;
        }

        panel = document.createElement(
            'div'
        );

        panel.id =
            'gob-jp1-swarm-panel';

        panel.style.cssText = `
            position: fixed;
            right: 14px;
            bottom: 14px;
            z-index: 2147483647;
            width: 260px;
            padding: 12px;
            border-radius: 10px;
            background: rgba(20,20,24,.94);
            border: 1px solid rgba(255,255,255,.15);
            color: #fff;
            font: 12px/1.4 Arial,sans-serif;
            box-shadow: 0 8px 30px rgba(0,0,0,.35);
        `;

        const title =
            document.createElement('div');

        title.textContent =
            'JP1 Guest Swarm';

        title.style.cssText =
            'font-weight:700;font-size:14px;margin-bottom:8px;';

        const row =
            document.createElement('div');

        row.style.cssText =
            'display:flex;gap:6px;margin-bottom:8px;';

        countInput =
            document.createElement('input');

        countInput.type = 'number';
        countInput.min = '1';
        countInput.max = '1000';
        countInput.value =
            String(CONFIG.defaultGuests);

        countInput.style.cssText =
            'width:64px;background:#111;color:#fff;border:1px solid #555;border-radius:5px;padding:5px;';

        startButton =
            document.createElement('button');

        startButton.textContent =
            'Start';

        stopButton =
            document.createElement('button');

        stopButton.textContent =
            'Stop';

        for (
            const button of [
                startButton,
                stopButton
            ]
        ) {
            button.style.cssText =
                'flex:1;border:1px solid #666;border-radius:5px;background:#2a2a30;color:white;padding:5px;cursor:pointer;';
        }

        startButton.onclick = () => {
            swarm.start(
                countInput.value
            );
        };

        stopButton.onclick = () => {
            swarm.stop();
        };

        row.append(
            countInput,
            startButton,
            stopButton
        );

        statusText =
            document.createElement('div');

        errorText =
            document.createElement('div');

        errorText.style.cssText =
            'margin-top:6px;color:#ffb7b7;word-break:break-word;';

        const note =
            document.createElement('div');

        note.textContent =
            'Native cookie-less fetch • tab login untouched';

        note.style.cssText =
            'margin-top:7px;opacity:.65;font-size:11px;';

        panel.append(
            title,
            row,
            statusText,
            errorText,
            note
        );

        (
            document.body ||
            document.documentElement
        ).appendChild(panel);

        updatePanel();
    }

    function updatePanel() {
        if (!statusText) return;

        const stats = swarm.stats();

        if (!swarm.running) {
            statusText.textContent =
                'Stopped';

            errorText.textContent = '';
            return;
        }

        const uptime = Math.floor(
            (
                Date.now() -
                swarm.startedAt
            ) / 1000
        );

        const creating =
            swarm.activeGuestJobs;

        statusText.textContent =
            `Sessions ${swarm.sessionsCreated}/${stats.total}` +
            ` | Connected ${stats.connected}/${stats.total}` +
            `\nSpawned ${stats.spawned}/${stats.total}` +
            ` | Creating ${creating}` +
            ` | Errors ${stats.errors}` +
            ` | ${uptime}s`;

        statusText.style.whiteSpace =
            'pre-line';

        errorText.textContent =
            swarm.lastError
                ? `Last: ${swarm.lastError}`
                : '';
    }

    if (
        document.readyState ===
        'loading'
    ) {
        document.addEventListener(
            'DOMContentLoaded',
            installPanel,
            { once: true }
        );
    } else {
        installPanel();
    }

    // Handy console control too.
    window.GOB_JP1_SWARM = {
        start: count => swarm.start(
            count || CONFIG.defaultGuests
        ),
        stop: () => swarm.stop(),
        stats: () => swarm.stats()
    };
})();
