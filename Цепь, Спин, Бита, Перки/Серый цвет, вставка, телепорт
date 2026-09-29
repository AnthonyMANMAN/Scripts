// ==UserScript==
// @name         Game of Bombs - Unified White Mod Pack
// @namespace    http://tampermonkey.net/
// @version      1.1.0
// @description  Unified white theme/fire, autumn tileset, optimized bomb/grid overlay, compact feature UI, Auto Chain, K-Stick, Skin Cycler, teleport shortcut, and optional WS freeze.
// @author       You
// @match        https://gameofbombs.com/*
// @match        http://gameofbombs.com/*
// @run-at       document-start
// @grant        none
// ==/UserScript==

(() => {
    'use strict';

    if (window.__GOB_UNIFIED_WHITE_MOD__) return;
    Object.defineProperty(window, '__GOB_UNIFIED_WHITE_MOD__', {
        value: true,
        configurable: false,
        enumerable: false
    });

    // ============================================================
    // CONFIG
    // ============================================================
    const CONFIG = {
        tilesetOriginal: 'tileset.png?v=6',
        tilesetReplacement: 'https://raw.githubusercontent.com/AnthonyMANMAN/Scripts/main/Reskins/dark1.png',
        explosionPattern: /explosion\.png/i,
        bombFuseSeconds: 4.3,
        skinCycleMs: 100,
        autoChainMs: 20,
        wsUi: false
    };

    const log = (...args) => console.log('[GOB Unified]', ...args);

    // ============================================================
    // 1) WHITE CSS THEME
    //    Your original CSS is preserved; the two accent variables
    //    are changed to pure white.
    // ============================================================
    const THEME_CSS = "\n:root {\n  --y1: #ffffff; /* replaces #ffd21f / #ffd31f / #ffd733 / #ffff00 / gold / yellow */\n  --y2: #d9a6ff; /* replaces #fceaa2 / #ffe28f / #ffe9aa (lighter gold accents) */\n}\n\n/* profile page */\n.wrap-inner:before { border-color: var(--y1); }\n.wrap-inner h2 { color: var(--y1); }\n.wrap-inner .page-profile .half:first-child { border-right-color: var(--y1); }\n.wrap-inner .page-profile .half:last-child { border-left-color: var(--y1); }\n.wrap-inner .page-profile .half .ref { border-bottom-color: var(--y1); }\n.wrap-inner .page-profile .half a { color: var(--y1); }\n.wrap-inner .page-profile .half .referral-link-container { border-color: var(--y1); }\n.wrap-inner .page-profile .half .referral-link-container button.referrals { color: var(--y1); border-right-color: var(--y1); }\n.wrap-inner .page-profile ul.player-statistics li { color: var(--y1); }\n\n/* left menu */\n.left-menu ul .tab .padd-left a { color: var(--y1); }\n.left-menu ul .tab .padd-left .selected { box-shadow: inset -6px 0 0 var(--y1); }\n.left-menu ul .tab:hover .padd-left { box-shadow: inset -6px 0 0 var(--y1); }\n\n/* out_butt / top buttons */\n.out_butt .butt_left_menu { border-color: var(--y1); color: var(--y1); }\n.out_butt .resize-dropdown .list-resize:before { border-color: var(--y1); }\n\n/* info-user / balance bar */\n.info-user { color: var(--y1); }\n.info-user .balance { box-shadow: 0 4px 0 0 #181818, 0px 6px 0 0 #292929, 0 7px 0 0 var(--y1), 0 13px 0 0 #181818; }\n\n.txt_gold { color: var(--y1); }\n.button_style { color: var(--y1); }\n.button_style:before { border-color: var(--y2); }\n.colorGold { color: var(--y1); }\n\n/* yellow scrollbar */\n.perfect-scrollbar.yellow-scroll .ps-scrollbar-y-rail { border-left-color: var(--y1); }\n.perfect-scrollbar.yellow-scroll .ps-scrollbar-y-rail:hover .ps-scrollbar-y,\n.perfect-scrollbar.yellow-scroll .ps-scrollbar-y-rail.in-scrolling .ps-scrollbar-y,\n.perfect-scrollbar.yellow-scroll .ps-scrollbar-y-rail.hover .ps-scrollbar-y {\n  background-color: var(--y1);\n  box-shadow: 0px 0px 0px 1px #3d3d3d inset, 0px 0px 0px 2px var(--y1) inset, 0px 0px 0px 3px #3d3d3d inset;\n}\n.perfect-scrollbar.yellow-scroll .ps-scrollbar-y-rail .ps-scrollbar-y {\n  box-shadow: 0px 0px 0px 1px #3d3d3d inset, 0px 0px 0px 2px var(--y1) inset;\n}\n\n.border-gold-light:before { border-color: var(--y2); }\n.border-gold:before { border-color: var(--y1); }\n\n.register_form input:focus { color: var(--y1); outline-color: var(--y2); }\n\n.div-border-Ydark { border-color: var(--y1); }\n.bordGold { border-color: var(--y1); }\n\ninput, select { color: var(--y1); }\n\n/* character/skins constructor */\n.character_wrapper .ul-container .tabs-game li:first-child { border-left-color: var(--y1); }\n.character_wrapper .ul-container .tabs-game li:last-child { border-right-color: var(--y1); }\n.character_wrapper .shop-outer:before, .character_wrapper .price-outer:before { border-color: var(--y1); }\n.character_wrapper .shop-top { border-bottom-color: var(--y1); }\n.character_wrapper .shop-outer, .character_wrapper .price-outer { color: var(--y1); }\n.character_wrapper .skin_prev:before { color: var(--y1); }\n.character_wrapper .ul-price .elemShopName { color: var(--y1); }\n\n/* rankings */\n.page-rankings ul.rankings-by-country li.selected { color: var(--y1); }\n.page-rankings ul.page-tabs.rankings-by-period li:hover,\n.page-rankings ul.page-tabs.rankings-by-sort li:hover,\n.page-rankings ul.page-tabs.rankings-by-filter li:hover,\n.page-rankings ul.page-tabs.rankings-by-tour li:hover { color: var(--y1); }\n.page-rankings ul.page-tabs.rankings-by-period li.selected,\n.page-rankings ul.page-tabs.rankings-by-sort li.selected,\n.page-rankings ul.page-tabs.rankings-by-filter li.selected,\n.page-rankings ul.page-tabs.rankings-by-tour li.selected { color: var(--y1) !important; }\n\n/* payment / shop */\n.payment { color: var(--y2); }\n.payment .out-balance { border-bottom-color: var(--y1); }\n.payment .out_box .one_box { border-color: var(--y1); }\n.payment .out_box .summ { border-color: var(--y1); }\n.payment .out_box .summ input, .payment .out_box .summ select { color: var(--y1); }\n.payment .out_box .box { border-color: var(--y1); color: var(--y1); }\n\n.tutor_popup { border-color: var(--y1); color: var(--y2); }\n.tutoial_arrow_label { color: var(--y1); }\n\n.scr h2 { color: var(--y1); }\n.buy-slot-modal h3 { color: var(--y1); }\n\n.list-change-but li.select { background-color: var(--y1); }\n.list-change-but li.select input { outline-color: var(--y1); }\n\n.shop .peek-inside:hover { color: var(--y2); }\n.shop a.gold_chest:hover { background-color: var(--y1); }\n.shop .out_box .box .number { color: var(--y1); border-color: var(--y1); }\n.shop .out_box .box .perk-time { color: var(--y1); }\n.shop .bg-chest.gold_chest { color: var(--y1); }\n.shop .skins-select .skins-top { border-bottom-color: var(--y1); }\n.shop .skins-select .skins-top .bord-gold:before { border-color: var(--y1); }\n.shop .skins-select .skins-preview { border-color: var(--y1); }\n.shop .out_box .name-basic-shop { border-bottom-color: var(--y1); }\n.shop .out_box .name-basic-shop ul.list-level li.active-level { color: var(--y1); border-left-color: var(--y1); border-right-color: var(--y1); }\n.shop .out_box.advanced-shop .name-advanced-shop h2 { background-color: var(--y1); }\n.shop .out_box.advanced-shop .things-advanced-shop li { border-bottom-color: var(--y1); }\n\n/* server/room select */\n#server-select ul.server-list li.selected:before,\n.room-select ul.server-list li.selected:before,\n#server-select ul.room-list li.selected:before,\n.room-select ul.room-list li.selected:before { border-color: var(--y1); }\n#server-select ul.server-list li.selected h2,\n.room-select ul.server-list li.selected h2,\n#server-select ul.room-list li.selected h2,\n.room-select ul.room-list li.selected h2 { color: var(--y1); }\n.out-servers ul.server-list li h2 { color: var(--y2); }\nul.room-list li h2 { color: var(--y2); }\nul.room-list li .n_player { color: var(--y1); }\nul.room-list li .time_left { color: var(--y1); border-bottom-color: var(--y1); }\nul.room-list li .txt-room { color: var(--y1); border-top-color: var(--y1); }\n\n/* modals */\n.out_modal .modal:before, .out_modal_light .modal:before { border-color: var(--y1); }\n.out_modal .modal h2.borb_bot, .out_modal_light .modal h2.borb_bot { border-bottom-color: var(--y1); }\n\n.goal-icon-list .goal_border { border-color: var(--y1); }\n\n/* tasks / donations */\n.one-task { border-color: var(--y1); }\n.one-task .progress { border-color: var(--y1); }\n.one-task .txt_donated { color: var(--y1); }\nul.task-tab li { color: var(--y1); }\n.list-donators li.main-donator .num { color: var(--y1); }\n\n.wiki .settings_input { border-color: var(--y1); }\n.wiki a:hover, .wiki a:active { color: var(--y2); }\n\n/* chat */\n#chat #dropDownChat ul li:hover { color: var(--y2); }\n#chat .misc-top.misc-top, #chat .misc-bottom.misc-top { border-bottom-color: var(--y1); }\n#chat .misc-top h1, #chat .misc-bottom h1 { color: var(--y1); }\n#chat ul.chat-list li a { color: var(--y1); }\n#chat ul.chat-list li b.my-msg { color: var(--y1); }\n#chat ul.chat-list li b.nickname { color: var(--y2); }\n.chatmsg .mentions-autocomplete-list ul li b { color: var(--y1); }\n\n/* scoreboard */\n#scoreboard-new .perks2-list, #scoreboard-new .perk-details { border-color: var(--y1); }\n#scoreboard-new .perks2-list .title-ul, #scoreboard-new .perk-details .title-ul { color: var(--y1); }\n#scoreboard-new .perks2-list .out-bord-gold .power:before, #scoreboard-new .perk-details .out-bord-gold .power:before,\n#scoreboard-new .perks2-list .out-bord-gold .time:before, #scoreboard-new .perk-details .out-bord-gold .time:before,\n#scoreboard-new .perks2-list .out-bord-gold .perk:before, #scoreboard-new .perk-details .out-bord-gold .perk:before { border-top-color: var(--y1); }\n#scoreboard-new .perks2-list .out-bord-gold .bord-gold-li.down-illustration:before, #scoreboard-new .perk-details .out-bord-gold .bord-gold-li.down-illustration:before,\n#scoreboard-new .perks2-list .out-bord-gold .bord-gold-li.down-illustration:after, #scoreboard-new .perk-details .out-bord-gold .bord-gold-li.down-illustration:after { border-left-color: var(--y1); }\n#scoreboard-new .perks2-list .out-bord-gold .bord-gold-li.down-illustration .perk2-count, #scoreboard-new .perk-details .out-bord-gold .bord-gold-li.down-illustration .perk2-count { color: var(--y1); border-color: var(--y1); }\n#scoreboard-new ul.pagination { border-top-color: var(--y1); border-bottom-color: var(--y1); }\n#scoreboard-new ul.pagination li.count { border-left-color: var(--y1); border-right-color: var(--y1); }\n#scoreboard-new ul.pagination button { color: var(--y1); }\n#scoreboard-new .perks .title { border-top-color: var(--y1); border-bottom-color: var(--y1); }\n#scoreboard-new #perks div.perks-list-wrap div.event-block p.no-event,\n#scoreboard-new #bonus-perks div.perks-list-wrap div.event-block p.no-event { color: var(--y1); }\n#scoreboard-new #perks .perks2-list .list-inventory .perk2-count,\n#scoreboard-new #bonus-perks .perks2-list .list-inventory .perk2-count { color: var(--y1); border-color: var(--y1); }\n#scoreboard-new #perks .perks2-list li.perks2-item .perk2-count.count-time,\n#scoreboard-new #bonus-perks .perks2-list li.perks2-item .perk2-count.count-time { color: var(--y1); }\n#scoreboard-new #perks .perk-details span.perk-timed, #scoreboard-new #bonus-perks .perk-details span.perk-timed,\n#scoreboard-new #perks .perks-item span.perk-timed, #scoreboard-new #bonus-perks .perks-item span.perk-timed { color: var(--y1); }\n#scoreboard-new #perks .perk-details .perk-count.count-time, #scoreboard-new #bonus-perks .perk-details .perk-count.count-time,\n#scoreboard-new #perks .perks-item .perk-count.count-time, #scoreboard-new #bonus-perks .perks-item .perk-count.count-time { color: var(--y1); }\n#scoreboard-new #scores ul#list-players li.highlighted { background-color: var(--y1) !important; }\n#scoreboard-new #scores .headings { border-bottom-color: var(--y1); }\n#scoreboard-new #scores .headings span { color: var(--y1); }\n#scoreboard-new #scores .headings input { border-color: var(--y1); }\n#scoreboard-new #scores #list-thumb { outline-color: var(--y1); }\n#scoreboard-new #scores #list-thumb:hover, #scoreboard-new #scores #list-thumb:active { background-color: var(--y1); }\n#scoreboard-new #squads .settings .search-squads { border-color: var(--y1); }\n#scoreboard-new #squads .settings .create-squad { color: var(--y1); }\n#scoreboard-new #squads ul.squads li.squad.my header { background-color: var(--y2); }\n#scoreboard-new #squads ul.squads li.squad.my header i.lock { background-color: var(--y2); }\n#scoreboard-new #squads ul.squads li.squad.my ol.players, #scoreboard-new #squads ul.squads li.squad.my div.players { border-color: var(--y2); }\n#scoreboard-new #squads ul.squads li.squad.leader header i.lock { background-color: var(--y1); }\n#scoreboard-new #squads ul.squads li.squad.leader header i.lock:hover { background-color: var(--y1); }\n#scoreboard-new .register_tab { color: var(--y1); }\n#scoreboard-new .register_tab input:focus { color: var(--y1); outline-color: var(--y2); }\n\n#bonus-board .bonus-box:before { border-color: var(--y1); }\n#bonus-board .bonus-box .bonus-header-daily, #bonus-board .bonus-box .bonus-header-kills-total,\n#bonus-board .bonus-box .bonus-header-kills-round, #bonus-board .bonus-box .bonus-header-gold-total,\n#bonus-board .bonus-box .bonus-header-rounds { color: var(--y2); }\n\n.tabs-game.blue-select-color { color: var(--y1); }\n.tabs-game li { border-left-color: var(--y1); border-bottom-color: var(--y1); }\n.tabs-game li:hover, .tabs-game li.selected { color: var(--y1); border-color: var(--y1); }\n\n.out-slider button.src { color: var(--y1); }\n.out-slider .navigation-slider li { background-color: var(--y2); }\n.out-slider .navigation-slider li.select { background-color: var(--y1); }\n\n/* death / end-round screens */\n#scoreboard-death h2.title { border-bottom-color: var(--y1); }\n.list-loss li .how { color: var(--y1); }\n.list-loss li .how span:first-child { border-right-color: var(--y1); }\nul.list-powerups li .count { color: var(--y1); }\nul.list-tasks li { border-color: var(--y1); }\nul.list-tasks li .desc title { color: var(--y1); }\n.death-skins { border-top-color: var(--y1); }\n.death-skins .skin-outer .activ_skin { color: var(--y1); }\n.death-buttoms { border-top-color: var(--y1); }\n.death_button { color: var(--y1); border-color: var(--y1); }\n#scoreboard-end-round h2.title { border-bottom-color: var(--y1); }\n#scoreboard-end-round .death-buttoms { border-top-color: var(--y1); }\n#scoreboard-end-round ul.list-tasks { border-left-color: var(--y1); }\n#scoreboard-end-round #end-round-scores { border-right-color: var(--y1); }\n#scoreboard-end-round #end-round-scores ul#end-round-list-players li.highlighted { background-color: var(--y1) !important; }\n#scoreboard-end-round #end-round-scores .headings { border-bottom-color: var(--y1); }\n#scoreboard-end-round #end-round-scores .headings span { color: var(--y1); }\n#scoreboard-end-round #end-round-scores .headings input { border-color: var(--y1); }\n#scoreboard-end-round #end-round-scores #end-round-list-thumb { outline-color: var(--y1); }\n#scoreboard-end-round #end-round-scores #end-round-list-thumb:hover, #scoreboard-end-round #end-round-scores #end-round-list-thumb:active { background-color: var(--y1); }\n\n.new-rank .level-name { color: var(--y1); }\n\nul.list-players li span.nickname { color: var(--y2); }\n\n.quest-items .help-item { border-color: var(--y1); }\n\n/* landing / login */\n.class_landing { color: var(--y1); }\n.class_landing:before { border-color: var(--y1); }\n.class_landing .form_land2.form3 { border-color: var(--y1); }\n.class_landing .form_land2 input:focus { border-color: var(--y1); color: var(--y1); }\n.class_landing .form_land2 input.tibia-code { color: var(--y1); }\n.class_landing .form_land2 a { color: var(--y1); }\n.class_landing .logo-txt { color: var(--y2); }\n\n.region-list li { border-color: var(--y1); }\n.reg-refresh { border-color: var(--y1); }\n.reg-game { border-color: var(--y1); }\n\n.out-chests .box .bonus-perk-count { color: var(--y1); border-color: var(--y1); }\n\n.simple-shop-elem {\n  border-top-color: var(--y1);\n  border-bottom-color: var(--y1);\n  border-left-color: var(--y1);\n  border-right-color: var(--y1);\n}\n\n.table-log a { color: var(--y1); }\n\n/* jPlayer audio widget skin */\n.jp-audio, .jp-audio-stream, .jp-video { color: var(--y1); }\n.jp-audio-stream a { color: var(--y1); }\n\n/* hide the animated background canvas */\n#bg { display: none !important; }\n";

    function injectTheme() {
        const style = document.createElement('style');
        style.id = '__gobUnifiedWhiteTheme';
        style.textContent = THEME_CSS;
        (document.head || document.documentElement).appendChild(style);
    }
    injectTheme();

    // ============================================================
    // 2) IMAGE REPLACEMENT
    //    - tileset -> autumn tileset
    //    - explosion -> cached white version
    //
    // One shared src patch is much safer than stacking several
    // independent HTMLImageElement/Image overrides.
    // ============================================================
    const imageSrcDescriptor = Object.getOwnPropertyDescriptor(HTMLImageElement.prototype, 'src');
    const nativeImageSrcGet = imageSrcDescriptor && imageSrcDescriptor.get;
    const nativeImageSrcSet = imageSrcDescriptor && imageSrcDescriptor.set;
    const NativeImage = window.Image;

    let tilesetBlobUrl = null;
    let tilesetFetchStarted = false;
    const whiteExplosionCache = new Map();

    function absoluteUrl(url) {
        try {
            return new URL(url, location.href).href;
        } catch {
            return String(url);
        }
    }

    function preloadTileset() {
        if (tilesetFetchStarted) return;
        tilesetFetchStarted = true;

        fetch(CONFIG.tilesetReplacement, { cache: 'force-cache', mode: 'cors' })
            .then(r => {
                if (!r.ok) throw new Error('HTTP ' + r.status);
                return r.blob();
            })
            .then(blob => {
                tilesetBlobUrl = URL.createObjectURL(blob);
                log('Autumn tileset cached.');
            })
            .catch(err => {
                // The raw GitHub URL is still usable directly as a fallback.
                console.warn('[GOB Unified] Tileset preload failed; using direct URL fallback.', err);
            });
    }
    preloadTileset();

    function buildWhiteExplosion(sourceUrl) {
        const key = absoluteUrl(sourceUrl);
        if (whiteExplosionCache.has(key)) return whiteExplosionCache.get(key);

        const promise = new Promise((resolve, reject) => {
            const img = new NativeImage();

            // Only request anonymous CORS for genuinely cross-origin URLs.
            try {
                const u = new URL(key);
                if (u.origin !== location.origin) img.crossOrigin = 'anonymous';
            } catch {}

            img.onload = () => {
                try {
                    const w = img.naturalWidth || img.width;
                    const h = img.naturalHeight || img.height;
                    if (!w || !h) throw new Error('Explosion image has no size');

                    const canvas = document.createElement('canvas');
                    canvas.width = w;
                    canvas.height = h;
                    const ctx = canvas.getContext('2d', { willReadFrequently: false });

                    // Draw once, then replace every visible pixel with white
                    // while preserving the sprite's original alpha.
                    ctx.drawImage(img, 0, 0);
                    ctx.globalCompositeOperation = 'source-in';
                    ctx.fillStyle = '#ffffff';
                    ctx.fillRect(0, 0, w, h);
                    ctx.globalCompositeOperation = 'source-over';

                    resolve(canvas.toDataURL('image/png'));
                } catch (err) {
                    reject(err);
                }
            };
            img.onerror = () => reject(new Error('Explosion image failed to load'));
            nativeImageSrcSet.call(img, key);
        }).catch(err => {
            console.warn('[GOB Unified] White explosion conversion failed; original sprite will be used.', err);
            return null;
        });

        whiteExplosionCache.set(key, promise);
        return promise;
    }

    if (nativeImageSrcSet && nativeImageSrcGet) {
        Object.defineProperty(HTMLImageElement.prototype, 'src', {
            configurable: true,
            enumerable: imageSrcDescriptor.enumerable,
            get: nativeImageSrcGet,
            set(value) {
                const stringValue = String(value ?? '');

                if (stringValue.includes(CONFIG.tilesetOriginal)) {
                    nativeImageSrcSet.call(this, tilesetBlobUrl || CONFIG.tilesetReplacement);
                    return;
                }

                if (CONFIG.explosionPattern.test(stringValue)) {
                    const target = this;

                    // Start the conversion once; do not block the game thread.
                    buildWhiteExplosion(stringValue).then(whiteUrl => {
                        if (whiteUrl) {
                            nativeImageSrcSet.call(target, whiteUrl);
                        } else {
                            nativeImageSrcSet.call(target, stringValue);
                        }
                    });

                    // Do not load the orange frame first; this avoids a visible flash.
                    return;
                }

                nativeImageSrcSet.call(this, value);
            }
        });
    }

    window.addEventListener('beforeunload', () => {
        if (tilesetBlobUrl) URL.revokeObjectURL(tilesetBlobUrl);
    }, { once: true });

    // ============================================================
    // 3) OPTIMIZED WORLD GRID + BOMB TIMER / BLAST OVERLAY
    // ============================================================
    let layer0Ctx = null;
    let layer1Ctx = null;

    let showWorldGrid = false;
    let showBombBoxes = false;

    const blockPositions = Object.create(null);
    const bombPositions = Object.create(null);
    const shadowPositions = Object.create(null);

    const PROTO_PATCH_FLAG = '__gobUnifiedCanvasPatched';
    const MATCH_RADIUS = 60;

    let frameCount = 0;
    let nextBombId = 1;
    let overlayScheduled = false;

    // One tiny RAF counter is enough for per-frame bomb matching.
    const frameTick = () => {
        frameCount++;
        requestAnimationFrame(frameTick);
    };
    requestAnimationFrame(frameTick);

    function scheduleBombOverlay() {
        if (overlayScheduled) return;
        overlayScheduled = true;

        requestAnimationFrame(() => {
            overlayScheduled = false;
            drawBombBoxes();
        });
    }

    function drawWorldGrid() {
        if (!layer0Ctx || !showWorldGrid) return;

        const transform = layer0Ctx.getTransform();
        if (Math.abs(transform.e) < 10 && Math.abs(transform.f) < 10) return;

        const scale = transform.a || 1;
        const tileSize = 32;
        const worldLeft = -transform.e / scale;
        const worldTop = -transform.f / scale;
        const worldRight = worldLeft + layer0Ctx.canvas.width / scale;
        const worldBottom = worldTop + layer0Ctx.canvas.height / scale;

        const startX = Math.floor(worldLeft / tileSize) * tileSize;
        const startY = Math.floor(worldTop / tileSize) * tileSize;

        layer0Ctx.save();
        layer0Ctx.strokeStyle = 'rgba(255,255,255,0.42)';
        layer0Ctx.lineWidth = 1;
        layer0Ctx.beginPath();

        for (let x = startX; x <= worldRight + tileSize; x += tileSize) {
            layer0Ctx.moveTo(x, worldTop - tileSize);
            layer0Ctx.lineTo(x, worldBottom + tileSize);
        }
        for (let y = startY; y <= worldBottom + tileSize; y += tileSize) {
            layer0Ctx.moveTo(worldLeft - tileSize, y);
            layer0Ctx.lineTo(worldRight + tileSize, y);
        }
        layer0Ctx.stroke();

        layer0Ctx.fillStyle = 'rgba(255,255,255,0.9)';
        layer0Ctx.font = 'bold 10px monospace';

        for (let x = startX; x <= worldRight; x += tileSize) {
            for (let y = startY; y <= worldBottom; y += tileSize) {
                const tileX = Math.floor(x / tileSize);
                const tileY = Math.floor(y / tileSize);
                const wrappedX = ((tileX % 32) + 32) % 32;
                const wrappedY = ((tileY % 32) + 32) % 32;
                layer0Ctx.fillText(wrappedX + ',' + wrappedY, x + 2, y + 12);
            }
        }

        layer0Ctx.restore();
    }

    function handleLayer0DrawImage(ctx, image, args) {
        let worldX, worldY, width, height;

        if (args.length === 2) {
            [worldX, worldY] = args;
            width = image.width;
            height = image.height;
        } else if (args.length === 4) {
            [worldX, worldY, width, height] = args;
        } else if (args.length === 8) {
            worldX = args[4];
            worldY = args[5];
            width = args[6];
            height = args[7];
        } else {
            return;
        }

        if (width >= 16 && width <= 64 && height >= 16 && height <= 64) {
            const tileX = Math.floor(worldX / 32);
            const tileY = Math.floor(worldY / 32);
            const wrappedX = ((tileX % 32) + 32) % 32;
            const wrappedY = ((tileY % 32) + 32) % 32;

            blockPositions[wrappedX + ',' + wrappedY] = {
                x: worldX,
                y: worldY,
                width,
                height,
                tileX: wrappedX,
                tileY: wrappedY,
                image: image && image.src ? image.src : 'unknown'
            };
        }
    }

    let lastCamE = null;
    let lastCamF = null;
    let camSampledFrame = -1;
    let camMove = 0;

    function compensateForCamera(ctx) {
        if (camSampledFrame === frameCount) return;

        const t = ctx.getTransform();
        if (t.e === 0 && t.f === 0) return;

        camSampledFrame = frameCount;

        if (lastCamE !== null && (t.e !== lastCamE || t.f !== lastCamF)) {
            const dE = t.e - lastCamE;
            const dF = t.f - lastCamF;

            for (const key in bombPositions) {
                bombPositions[key].x += dE;
                bombPositions[key].y += dF;
            }
            camMove = Math.hypot(dE, dF);
        } else {
            camMove = 0;
        }

        lastCamE = t.e;
        lastCamF = t.f;
    }

    function drawBombBoxes() {
        if (!layer1Ctx) return;

        const keys = Object.keys(bombPositions);
        if (keys.length === 0) return;

        const now = performance.now();

        layer1Ctx.save();
        layer1Ctx.lineWidth = 3;
        layer1Ctx.font = 'bold 12px Arial';
        layer1Ctx.textAlign = 'center';
        layer1Ctx.textBaseline = 'middle';

        for (const key of keys) {
            const bomb = bombPositions[key];

            const elapsed = (now - bomb.placedTime) / 1000;
            const remaining = Math.max(0, CONFIG.bombFuseSeconds - elapsed);

            if (remaining <= 0 || now - bomb.lastSeen > 250) {
                delete bombPositions[key];
                continue;
            }

            const bombCenterX = bomb.x + bomb.w / 2;
            const bombCenterY = bomb.y + bomb.h / 2;
            const boxSize = 32;

            if (showBombBoxes) {
                layer1Ctx.strokeStyle = '#ffffff';
                layer1Ctx.setLineDash([]);
                layer1Ctx.strokeRect(
                    bombCenterX - boxSize / 2,
                    bombCenterY - boxSize / 2,
                    boxSize,
                    boxSize
                );
            }

            const timerX = bombCenterX + boxSize / 2 - 10;
            const timerY = bombCenterY - boxSize / 2 + 10;

            layer1Ctx.fillStyle = 'rgba(0,0,0,0.72)';
            layer1Ctx.beginPath();
            layer1Ctx.arc(timerX, timerY, 10, 0, Math.PI * 2);
            layer1Ctx.fill();

            layer1Ctx.fillStyle = '#ffffff';
            layer1Ctx.fillText(remaining.toFixed(1), timerX, timerY);

            if (remaining <= 1) {
                layer1Ctx.save();
                layer1Ctx.globalAlpha = 1;
                layer1Ctx.strokeStyle = '#ffffff';
                layer1Ctx.lineWidth = 4;
                layer1Ctx.setLineDash([8, 6]);

                const blastLen = 96;
                layer1Ctx.beginPath();
                layer1Ctx.moveTo(bombCenterX, bombCenterY - blastLen);
                layer1Ctx.lineTo(bombCenterX, bombCenterY + blastLen);
                layer1Ctx.moveTo(bombCenterX - blastLen, bombCenterY);
                layer1Ctx.lineTo(bombCenterX + blastLen, bombCenterY);
                layer1Ctx.stroke();
                layer1Ctx.restore();
            }
        }

        layer1Ctx.restore();
    }

    let lastFillWasGrey = false;
    let lastGlobalAlpha = 1;

    function handleLayer1DrawImage(ctx, image, args) {
        if (
            args.length !== 8 ||
            args[0] !== 0 ||
            args[1] !== 0 ||
            args[2] !== 64 ||
            args[3] !== 64
        ) {
            return;
        }

        const x = args[4];
        const y = args[5];
        const w = args[6];
        const h = args[7];
        const currentTime = performance.now();

        const screenCenterX = x + w / 2;
        const screenCenterY = y + h / 2;

        const isShadow = lastFillWasGrey && w >= 50 && w <= 70 && h >= 50 && h <= 70;
        const isGreyOutline = (lastFillWasGrey && !isShadow) || lastGlobalAlpha < 0.8;

        if (lastFillWasGrey) lastFillWasGrey = false;
        lastGlobalAlpha = 1;

        if (isShadow) {
            const shadowKey = 'shadow_' + Math.round(screenCenterX / 8) + '_' + Math.round(screenCenterY / 8);
            shadowPositions[shadowKey] = {
                x: screenCenterX,
                y: screenCenterY,
                lastSeen: currentTime
            };

            for (const key in shadowPositions) {
                if (currentTime - shadowPositions[key].lastSeen > 250) {
                    delete shadowPositions[key];
                }
            }
            return;
        }

        if (isGreyOutline) return;

        const radius = MATCH_RADIUS + camMove;
        let existingKey = null;
        let minDist = Infinity;
        let duplicate = false;

        for (const key in bombPositions) {
            const bomb = bombPositions[key];
            const bombCenterX = bomb.x + bomb.w / 2;
            const bombCenterY = bomb.y + bomb.h / 2;

            if (bomb.lastFrame === frameCount) {
                if (
                    Math.abs(screenCenterX - bombCenterX) < 6 &&
                    Math.abs(screenCenterY - bombCenterY) < 6
                ) {
                    duplicate = true;
                    break;
                }
                continue;
            }

            const predictedX = bombCenterX + (bomb.lastDX || 0);
            const predictedY = bombCenterY + (bomb.lastDY || 0);
            const dist = Math.hypot(
                screenCenterX - predictedX,
                screenCenterY - predictedY
            );

            if (dist < radius && dist < minDist) {
                minDist = dist;
                existingKey = key;
            }
        }

        if (duplicate) return;

        if (existingKey) {
            const bomb = bombPositions[existingKey];
            const prevCenterX = bomb.x + bomb.w / 2;
            const prevCenterY = bomb.y + bomb.h / 2;

            bomb.lastDX = screenCenterX - prevCenterX;
            bomb.lastDY = screenCenterY - prevCenterY;
            bomb.x = x;
            bomb.y = y;
            bomb.w = w;
            bomb.h = h;
            bomb.lastSeen = currentTime;
            bomb.lastFrame = frameCount;
        } else {
            bombPositions['bomb_' + nextBombId++] = {
                x, y, w, h,
                placedTime: currentTime,
                lastSeen: currentTime,
                lastFrame: frameCount,
                lastDX: 0,
                lastDY: 0
            };
        }

        scheduleBombOverlay();
    }

    function patchCanvasPrototypeOnce() {
        const proto = CanvasRenderingContext2D.prototype;
        if (proto[PROTO_PATCH_FLAG]) return;

        Object.defineProperty(proto, PROTO_PATCH_FLAG, {
            value: true,
            configurable: false
        });

        const originalDrawImage = proto.drawImage;
        const originalFillRect = proto.fillRect;
        const originalSave = proto.save;
        const originalRestore = proto.restore;
        const alphaDesc = Object.getOwnPropertyDescriptor(proto, 'globalAlpha');
        const saveCounts = new WeakMap();

        if (alphaDesc && alphaDesc.get && alphaDesc.set) {
            Object.defineProperty(proto, 'globalAlpha', {
                configurable: true,
                enumerable: alphaDesc.enumerable,
                get() {
                    return alphaDesc.get.call(this);
                },
                set(value) {
                    if (this.canvas && this.canvas.id === 'layer1') {
                        lastGlobalAlpha = value;
                    }
                    alphaDesc.set.call(this, value);
                }
            });
        }

        proto.fillRect = function (...args) {
            if (this.canvas && this.canvas.id === 'layer1') {
                const fillStyle = this.fillStyle;
                if (typeof fillStyle === 'string') {
                    const lower = fillStyle.toLowerCase();
                    const hex = lower.match(/^#([0-9a-f]{6})$/);

                    lastFillWasGrey =
                        lower.includes('grey') ||
                        lower.includes('gray') ||
                        Boolean(
                            hex &&
                            hex[1].slice(0, 2) === hex[1].slice(2, 4) &&
                            hex[1].slice(2, 4) === hex[1].slice(4, 6)
                        );
                }
            }

            return originalFillRect.apply(this, args);
        };

        proto.save = function (...args) {
            if (this.canvas && this.canvas.id === 'layer0') {
                saveCounts.set(this, (saveCounts.get(this) || 0) + 1);
            }
            return originalSave.apply(this, args);
        };

        proto.restore = function (...args) {
            if (this.canvas && this.canvas.id === 'layer0') {
                const count = saveCounts.get(this) || 0;
                if (count > 0 && showWorldGrid) drawWorldGrid();
                saveCounts.set(this, Math.max(0, count - 1));
            }
            return originalRestore.apply(this, args);
        };

        proto.drawImage = function (image, ...args) {
            const result = originalDrawImage.apply(this, [image, ...args]);

            if (this.canvas) {
                if (this.canvas.id === 'layer0') {
                    compensateForCamera(this);
                    handleLayer0DrawImage(this, image, args);
                } else if (this.canvas.id === 'layer1') {
                    handleLayer1DrawImage(this, image, args);
                }
            }

            return result;
        };
    }

    function hookCanvas(canvas) {
        if (!(canvas instanceof HTMLCanvasElement)) return;

        const ctx = canvas.getContext('2d');
        if (!ctx) return;

        if (canvas.id === 'layer0' && layer0Ctx !== ctx) {
            layer0Ctx = ctx;
            log('World grid canvas hooked.');
        }

        if (canvas.id === 'layer1' && layer1Ctx !== ctx) {
            layer1Ctx = ctx;
            log('Bomb overlay canvas hooked.');
        }
    }

    function discoverCanvases(root = document) {
        if (root instanceof HTMLCanvasElement) hookCanvas(root);
        if (root.querySelectorAll) {
            root.querySelectorAll('canvas').forEach(hookCanvas);
        }
    }

    patchCanvasPrototypeOnce();

    function startCanvasDiscovery() {
        discoverCanvases(document);

        const target = document.documentElement;
        if (!target) return;

        const observer = new MutationObserver(records => {
            for (const record of records) {
                for (const node of record.addedNodes) {
                    if (node.nodeType === Node.ELEMENT_NODE) {
                        discoverCanvases(node);
                    }
                }
            }
        });

        observer.observe(target, { childList: true, subtree: true });
    }

    if (document.documentElement) {
        startCanvasDiscovery();
    } else {
        document.addEventListener('DOMContentLoaded', startCanvasDiscovery, { once: true });
    }

    window.toggleWorldGrid = () => {
        showWorldGrid = !showWorldGrid;
        log('World grid:', showWorldGrid ? 'ON' : 'OFF');
        return showWorldGrid;
    };

    window.toggleBombBoxes = () => {
        showBombBoxes = !showBombBoxes;
        log('Bomb boxes:', showBombBoxes ? 'ON' : 'OFF');
        return showBombBoxes;
    };

    window.getBlockAt = (tileX, tileY) => {
        const wrappedX = ((tileX % 32) + 32) % 32;
        const wrappedY = ((tileY % 32) + 32) % 32;
        return blockPositions[wrappedX + ',' + wrappedY] || null;
    };

    window.getAllBlocks = () => Object.values(blockPositions);

    window.getBlocksInArea = (startX, startY, endX, endY) => {
        const blocks = [];
        for (let x = startX; x <= endX; x++) {
            for (let y = startY; y <= endY; y++) {
                const block = window.getBlockAt(x, y);
                if (block) blocks.push(block);
            }
        }
        return blocks;
    };

    // ============================================================
    // 4) AUTO CHAIN + K-STICK + SKIN CYCLER
    // ============================================================
    let autoChainActive = false;
    let autoChainInterval = null;

    let kStuck = false;
    let preventKKeyUp = null;

    let skinCyclerActive = false;
    let skinCyclerInterval = null;
    let skinCyclerCurrentIndex = 0;
    let skinCyclerSkinsList = [];
    let skinCyclerScope = null;

    let featureUi = null;
    let chainUiBtn = null;
    let kStickUiBtn = null;
    let skinUiBtn = null;
    let teleportUiBtn = null;

    function isTypingTarget(target) {
        return !!target && (
            target.tagName === 'INPUT' ||
            target.tagName === 'TEXTAREA' ||
            target.isContentEditable
        );
    }

    function autoChainPulse() {
        if (!autoChainActive) return;

        const down = new KeyboardEvent('keydown', {
            key: 'k',
            code: 'KeyK',
            keyCode: 75,
            which: 75,
            bubbles: true
        });

        const up = new KeyboardEvent('keyup', {
            key: 'k',
            code: 'KeyK',
            keyCode: 75,
            which: 75,
            bubbles: true
        });

        document.dispatchEvent(down);
        setTimeout(() => {
            if (autoChainActive) document.dispatchEvent(up);
        }, 10);
    }

    function stopAutoChain() {
        if (autoChainInterval) {
            clearInterval(autoChainInterval);
            autoChainInterval = null;
        }
    }

    function toggleAutoChain() {
        autoChainActive = !autoChainActive;

        if (autoChainActive) {
            stopAutoChain();
            autoChainInterval = setInterval(autoChainPulse, CONFIG.autoChainMs);
        } else {
            stopAutoChain();
        }

        log('Auto Chain:', autoChainActive ? 'ON' : 'OFF');
        updateFeatureUi();
        return autoChainActive;
    }

    function toggleKStick() {
        if (!kStuck) {
            preventKKeyUp = event => {
                if (event.key?.toLowerCase() === 'k' || event.code === 'KeyK') {
                    event.preventDefault();
                    event.stopImmediatePropagation();
                }
            };

            document.addEventListener('keyup', preventKKeyUp, true);
            kStuck = true;
        } else {
            if (preventKKeyUp) {
                document.removeEventListener('keyup', preventKKeyUp, true);
                preventKKeyUp = null;
            }
            kStuck = false;
        }

        log('K-Stick:', kStuck ? 'BLOCKED' : 'UNBLOCKED');
        updateFeatureUi();
        return kStuck;
    }

    function initializeSkinCycler() {
        const angular = window.angular;
        if (!angular) return false;

        const skinsElement = document.querySelector('[ng-repeat*="skin in group.skins"]');
        if (!skinsElement) return false;

        let scope = angular.element(skinsElement).scope();
        if (!scope) return false;

        let parent = scope;
        while (parent && !parent.group) parent = parent.$parent;

        if (!parent || !parent.group || !Array.isArray(parent.group.skins)) {
            return false;
        }

        skinCyclerSkinsList = parent.group.skins;

        // Find and cache an activation scope ONCE, not every 100 ms.
        let activationScope = parent;
        while (
            activationScope &&
            typeof activationScope.activateSkin !== 'function'
        ) {
            activationScope = activationScope.$parent;
        }

        if (!activationScope) {
            const activationElement = document.querySelector('[ng-click*="activateSkin"]');
            if (activationElement) {
                activationScope = angular.element(activationElement).scope();
            }
        }

        if (!activationScope || typeof activationScope.activateSkin !== 'function') {
            return false;
        }

        skinCyclerScope = activationScope;
        return skinCyclerSkinsList.length > 0;
    }

    function activateSkin(skin) {
        const scope = skinCyclerScope;
        if (!scope || scope.$$destroyed || typeof scope.activateSkin !== 'function') {
            skinCyclerScope = null;
            if (!initializeSkinCycler()) return false;
        }

        const targetScope = skinCyclerScope;
        const run = () => {
            if (typeof targetScope.selectSkin === 'function') {
                targetScope.selectSkin(skin);
            }
            targetScope.activateSkin(skin);
        };

        // Avoid "$digest already in progress" errors.
        if (targetScope.$root && targetScope.$root.$$phase) {
            run();
        } else if (typeof targetScope.$apply === 'function') {
            targetScope.$apply(run);
        } else {
            run();
        }

        return true;
    }

    function cycleSkin() {
        if (!skinCyclerActive) return;

        if (!skinCyclerSkinsList.length && !initializeSkinCycler()) {
            stopSkinCycler();
            log('Skin Cycler: no skins found. Open the skins page once first.');
            return;
        }

        if (skinCyclerSkinsList.length <= 1) return;

        skinCyclerCurrentIndex++;
        if (skinCyclerCurrentIndex >= skinCyclerSkinsList.length) {
            skinCyclerCurrentIndex = 1;
        }

        activateSkin(skinCyclerSkinsList[skinCyclerCurrentIndex]);
    }

    function startSkinCycler() {
        if (skinCyclerActive) return true;

        if (!initializeSkinCycler()) {
            log('Skin Cycler: no skins found. Open the skins page once first.');
            return false;
        }

        skinCyclerActive = true;
        skinCyclerInterval = setInterval(cycleSkin, CONFIG.skinCycleMs);
        log('Skin Cycler: ON');
        updateFeatureUi();
        return true;
    }

    function stopSkinCycler() {
        skinCyclerActive = false;

        if (skinCyclerInterval) {
            clearInterval(skinCyclerInterval);
            skinCyclerInterval = null;
        }

        updateFeatureUi();
        return false;
    }

    function toggleSkinCycler() {
        if (skinCyclerActive) {
            stopSkinCycler();
            log('Skin Cycler: OFF');
            return false;
        }
        return startSkinCycler();
    }


    function triggerTeleport() {
        // The attached script does not contain teleport movement logic.
        // This button triggers the common T shortcut so an existing teleport
        // handler in the page / another module can receive it without polling.
        const down = new KeyboardEvent('keydown', {
            key: 'z',
            code: 'KeyZ',
            keyCode: 84,
            which: 84,
            bubbles: true
        });
        const up = new KeyboardEvent('keyup', {
            key: 'z',
            code: 'KeyZ',
            keyCode: 84,
            which: 84,
            bubbles: true
        });
        document.dispatchEvent(down);
        document.dispatchEvent(up);

        if (teleportUiBtn) {
            teleportUiBtn.textContent = 'SENT';
            teleportUiBtn.style.background = '#ffffff';
            teleportUiBtn.style.color = '#111111';
            setTimeout(updateFeatureUi, 220);
        }
    }

    function styleFeatureButton(button) {
        button.type = 'button';
        button.style.cssText = [
            "min-width:58px",
            "height:25px",
            "padding:0 9px",
            "border:1px solid rgba(255,255,255,.28)",
            "border-radius:6px",
            "background:#202020",
            "color:#f5f5f5",
            "font:600 10px/1 monospace",
            "letter-spacing:.04em",
            "cursor:pointer",
            "transition:background .12s ease,border-color .12s ease,color .12s ease,transform .08s ease"
        ].join(';');
        button.addEventListener('mouseenter', () => {
            button.style.borderColor = '#d9a6ff';
        });
        button.addEventListener('mouseleave', updateFeatureUi);
        button.addEventListener('mousedown', () => {
            button.style.transform = 'translateY(1px)';
        });
        button.addEventListener('mouseup', () => {
            button.style.transform = 'translateY(0)';
        });
    }

    function setFeatureButtonState(button, active, activeText, inactiveText) {
        if (!button) return;
        button.textContent = active ? activeText : inactiveText;
        button.style.background = active ? '#ffffff' : '#202020';
        button.style.color = active ? '#111111' : '#f5f5f5';
        button.style.borderColor = active ? '#d9a6ff' : 'rgba(255,255,255,.28)';
        button.style.boxShadow = active ? '0 0 0 1px rgba(217,166,255,.16)' : 'none';
    }

    function updateFeatureUi() {
        setFeatureButtonState(chainUiBtn, autoChainActive, 'ON', 'OFF');
        setFeatureButtonState(kStickUiBtn, kStuck, 'ON', 'OFF');
        setFeatureButtonState(skinUiBtn, skinCyclerActive, 'ON', 'OFF');

        if (teleportUiBtn && teleportUiBtn.textContent !== 'SENT') {
            teleportUiBtn.textContent = 'GO';
            teleportUiBtn.style.background = '#202020';
            teleportUiBtn.style.color = '#f5f5f5';
            teleportUiBtn.style.borderColor = 'rgba(255,255,255,.28)';
            teleportUiBtn.style.boxShadow = 'none';
        }
    }

    function createFeatureUi() {
        if (featureUi || !document.body) return;

        featureUi = document.createElement('div');
        featureUi.id = '__gobUnifiedFeatureUi';
        featureUi.style.cssText = [
            "position:fixed",
            "right:18px",
            "bottom:18px",
            "z-index:999999",
            "width:214px",
            "padding:10px",
            "border:1px solid rgba(255,255,255,.23)",
            "border-radius:9px",
            "background:rgba(16,16,18,.94)",
            "color:#ffffff",
            "font:11px/1.2 monospace",
            "box-shadow:0 5px 18px rgba(0,0,0,.42)",
            "backdrop-filter:blur(4px)",
            "user-select:none"
        ].join(';');

        const header = document.createElement('div');
        header.style.cssText = 'display:flex;align-items:center;justify-content:space-between;margin:0 1px 8px;padding-bottom:7px;border-bottom:1px solid rgba(217,166,255,.22)';

        const title = document.createElement('span');
        title.textContent = 'GOB CONTROLS';
        title.style.cssText = 'font-weight:700;letter-spacing:.11em;font-size:10px;color:#ffffff';

        const accent = document.createElement('span');
        accent.textContent = 'WHITE';
        accent.style.cssText = 'font-size:9px;letter-spacing:.08em;color:#d9a6ff';
        header.append(title, accent);

        function makeRow(name, key, button, clickHandler) {
            const row = document.createElement('div');
            row.style.cssText = 'display:flex;align-items:center;justify-content:space-between;gap:10px;min-height:31px';

            const left = document.createElement('div');
            left.style.cssText = 'display:flex;align-items:center;gap:7px;min-width:0';

            const keyBox = document.createElement('span');
            keyBox.textContent = key;
            keyBox.style.cssText = 'display:inline-flex;align-items:center;justify-content:center;width:21px;height:19px;border:1px solid rgba(217,166,255,.38);border-radius:4px;color:#d9a6ff;font-size:9px;font-weight:700;background:rgba(217,166,255,.06)';

            const label = document.createElement('span');
            label.textContent = name;
            label.style.cssText = 'white-space:nowrap;color:#eeeeee';

            left.append(keyBox, label);
            styleFeatureButton(button);
            button.addEventListener('click', clickHandler);
            row.append(left, button);
            return row;
        }

        chainUiBtn = document.createElement('button');
        kStickUiBtn = document.createElement('button');
        skinUiBtn = document.createElement('button');
        teleportUiBtn = document.createElement('button');

        featureUi.append(
            header,
            makeRow('Auto Chain', 'J', chainUiBtn, toggleAutoChain),
            makeRow('K-Stick', 'L', kStickUiBtn, toggleKStick),
            makeRow('Skin Cycle', 'H', skinUiBtn, toggleSkinCycler),
            makeRow('Teleport', 'Z', teleportUiBtn, triggerTeleport)
        );

        document.body.appendChild(featureUi);
        updateFeatureUi();
    }

    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', createFeatureUi, { once: true });
    } else {
        createFeatureUi();
    }

    document.addEventListener('keydown', event => {
        if (isTypingTarget(event.target)) return;
        if (event.ctrlKey || event.altKey || event.metaKey) return;

        const key = event.key?.toLowerCase();

        if (key === 'j' && !event.repeat) {
            event.preventDefault();
            toggleAutoChain();
        } else if (key === 'l' && !event.repeat) {
            event.preventDefault();
            toggleKStick();
        } else if (key === 'h' && !event.repeat) {
            event.preventDefault();
            toggleSkinCycler();
        }
    }, true);

    window.toggleAutoChain = toggleAutoChain;
    window.toggleKStick = toggleKStick;
    window.toggleSkinCycler = toggleSkinCycler;

    // ============================================================
    // 5) OPTIONAL WEBSOCKET FREEZE
    //
    // OFF = direct native send, no queue, no deliberate network delay.
    // ON  = packets are intentionally withheld until released.
    // ============================================================
    let wsFrozen = false;
    let wsQueue = [];
    const nativeWebSocketSend = WebSocket.prototype.send;

    function directWsSend(ws, data) {
        return nativeWebSocketSend.call(ws, data);
    }

    WebSocket.prototype.send = function (data) {
        if (!wsFrozen) {
            return directWsSend(this, data);
        }

        wsQueue.push({ ws: this, data });
        updateWsUi();
    };

    let wsUi = null;
    let wsFreezeBtn = null;
    let wsQueueLabel = null;

    function flushWsQueue() {
        if (!wsQueue.length) {
            updateWsUi();
            return;
        }

        const pending = wsQueue;
        wsQueue = [];

        for (const packet of pending) {
            if (packet.ws.readyState === WebSocket.OPEN) {
                try {
                    directWsSend(packet.ws, packet.data);
                } catch (err) {
                    console.warn('[GOB Unified] Failed to flush a queued packet.', err);
                }
            }
        }

        updateWsUi();
    }

    function setWsFrozen(next) {
        wsFrozen = Boolean(next);

        if (!wsFrozen) {
            flushWsQueue();
        }

        updateWsUi();
        log('WS Freeze:', wsFrozen ? 'ON' : 'OFF');
        return wsFrozen;
    }

    function toggleWsFreeze() {
        return setWsFrozen(!wsFrozen);
    }

    function updateWsUi() {
        if (!wsUi) return;

        wsQueueLabel.textContent = 'Queue: ' + wsQueue.length + ' packet(s)';
        wsFreezeBtn.textContent = wsFrozen ? 'ON' : 'OFF';
        wsFreezeBtn.style.background = wsFrozen ? '#ffffff' : '#222';
        wsFreezeBtn.style.color = wsFrozen ? '#000000' : '#ffffff';
        wsFreezeBtn.style.borderColor = wsFrozen ? '#ffffff' : '#555';
    }

    function createWsUi() {
        if (!CONFIG.wsUi || wsUi || !document.body) return;

        wsUi = document.createElement('div');
        wsUi.id = '__gobUnifiedWsUi';
        wsUi.style.cssText = [
            "position:fixed",
            "right:20px",
            "bottom:20px",
            "z-index:999999",
            "min-width:220px",
            "padding:12px 14px",
            "display:flex",
            "flex-direction:column",
            "gap:9px",
            "border:1px solid #555",
            "border-radius:9px",
            "background:rgba(18,18,18,.94)",
            "color:#fff",
            "font:12px/1.3 monospace",
            "box-shadow:0 4px 18px rgba(0,0,0,.55)",
            "user-select:none",
        ].join(';');

        const title = document.createElement('div');
        title.textContent = 'WS OUTGOING';
        title.style.cssText = 'font-size:10px;letter-spacing:.12em;color:#aaa';

        const row = document.createElement('div');
        row.style.cssText = 'display:flex;align-items:center;justify-content:space-between;gap:12px';

        const label = document.createElement('span');
        label.textContent = 'Freeze [Z]';

        wsFreezeBtn = document.createElement('button');
        wsFreezeBtn.type = 'button';
        wsFreezeBtn.style.cssText = [
            "min-width:54px",
            "padding:4px 10px",
            "border:1px solid #555",
            "border-radius:5px",
            "background:#222",
            "color:#fff",
            "font:12px monospace",
            "cursor:pointer",
        ].join(';');
        wsFreezeBtn.addEventListener('click', toggleWsFreeze);

        row.append(label, wsFreezeBtn);

        wsQueueLabel = document.createElement('div');
        wsQueueLabel.style.color = '#aaa';

        const flushButton = document.createElement('button');
        flushButton.type = 'button';
        flushButton.textContent = 'Flush Queue';
        flushButton.style.cssText = [
            "padding:5px",
            "border:1px solid #555",
            "border-radius:5px",
            "background:#222",
            "color:#fff",
            "font:11px monospace",
            "cursor:pointer",
        ].join(';');
        flushButton.addEventListener('click', flushWsQueue);

        wsUi.append(title, row, wsQueueLabel, flushButton);
        document.body.appendChild(wsUi);
        updateWsUi();
    }

    if (CONFIG.wsUi) {
        if (document.readyState === 'loading') {
            document.addEventListener('DOMContentLoaded', createWsUi, { once: true });
        } else {
            createWsUi();
        }
    }

    document.addEventListener('keyup', event => {
        if (isTypingTarget(event.target)) return;
        if (event.ctrlKey || event.altKey || event.metaKey) return;

        const key = event.key?.toLowerCase();
        if (key === 'z' || key === 'м') {
            event.preventDefault();
            toggleWsFreeze();
        }
    }, true);

    window.toggleWsFreeze = toggleWsFreeze;
    window.flushWsQueue = flushWsQueue;

    // ============================================================
    // STATUS + CLEANUP
    // ============================================================
    window.getUnifiedModStatus = () => ({
        autoChain: autoChainActive,
        kStick: kStuck,
        skinCycler: skinCyclerActive,
        worldGrid: showWorldGrid,
        bombBoxes: showBombBoxes,
        wsFrozen,
        wsQueuedPackets: wsQueue.length,
        tilesetCached: Boolean(tilesetBlobUrl)
    });

    window.addEventListener('beforeunload', () => {
        stopAutoChain();
        stopSkinCycler();

        if (preventKKeyUp) {
            document.removeEventListener('keyup', preventKKeyUp, true);
            preventKKeyUp = null;
        }

        // Do not intentionally flush on unload. Let the socket/page close normally.
        wsQueue.length = 0;
    });

    log('Loaded.');
    log('J = Auto Chain | L = K-Stick | H = Skin Cycler | Z = WS Freeze');
    log('Console: toggleWorldGrid(), toggleBombBoxes(), getUnifiedModStatus()');
})();
