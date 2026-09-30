// ==UserScript==
// @name         Game of Bombs — Русская панель с настройкой клавиш
// @namespace    http://tampermonkey.net/
// @version      2.0.4
// @description  Белая тема, русская панель, настраиваемые клавиши EN/RU, автоцепь, удержание и телепорт (заморозка сети). Без canvas drawImage перехвата.
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
        autoChainMs: 20,
        wsUi: false
    };

    const log = (...args) => console.log('[GOB Unified]', ...args);

    // ============================================================
    // 1) WHITE CSS THEME
    // ============================================================
    const THEME_CSS = `
:root {
  --y1: #ffffff;
  --y2: #d9a6ff;
}

/* profile page */
.wrap-inner:before { border-color: var(--y1); }
.wrap-inner h2 { color: var(--y1); }
.wrap-inner .page-profile .half:first-child { border-right-color: var(--y1); }
.wrap-inner .page-profile .half:last-child { border-left-color: var(--y1); }
.wrap-inner .page-profile .half .ref { border-bottom-color: var(--y1); }
.wrap-inner .page-profile .half a { color: var(--y1); }
.wrap-inner .page-profile .half .referral-link-container { border-color: var(--y1); }
.wrap-inner .page-profile .half .referral-link-container button.referrals { color: var(--y1); border-right-color: var(--y1); }
.wrap-inner .page-profile ul.player-statistics li { color: var(--y1); }

/* left menu */
.left-menu ul .tab .padd-left a { color: var(--y1); }
.left-menu ul .tab .padd-left .selected { box-shadow: inset -6px 0 0 var(--y1); }
.left-menu ul .tab:hover .padd-left { box-shadow: inset -6px 0 0 var(--y1); }

/* out_butt / top buttons */
.out_butt .butt_left_menu { border-color: var(--y1); color: var(--y1); }
.out_butt .resize-dropdown .list-resize:before { border-color: var(--y1); }

/* info-user / balance bar */
.info-user { color: var(--y1); }
.info-user .balance { box-shadow: 0 4px 0 0 #181818, 0px 6px 0 0 #292929, 0 7px 0 0 var(--y1), 0 13px 0 0 #181818; }

.txt_gold { color: var(--y1); }
.button_style { color: var(--y1); }
.button_style:before { border-color: var(--y2); }
.colorGold { color: var(--y1); }

/* yellow scrollbar */
.perfect-scrollbar.yellow-scroll .ps-scrollbar-y-rail { border-left-color: var(--y1); }
.perfect-scrollbar.yellow-scroll .ps-scrollbar-y-rail:hover .ps-scrollbar-y,
.perfect-scrollbar.yellow-scroll .ps-scrollbar-y-rail.in-scrolling .ps-scrollbar-y,
.perfect-scrollbar.yellow-scroll .ps-scrollbar-y-rail.hover .ps-scrollbar-y {
  background-color: var(--y1);
  box-shadow: 0px 0px 0px 1px #3d3d3d inset, 0px 0px 0px 2px var(--y1) inset, 0px 0px 0px 3px #3d3d3d inset;
}
.perfect-scrollbar.yellow-scroll .ps-scrollbar-y-rail .ps-scrollbar-y {
  box-shadow: 0px 0px 0px 1px #3d3d3d inset, 0px 0px 0px 2px var(--y1) inset;
}

.border-gold-light:before { border-color: var(--y2); }
.border-gold:before { border-color: var(--y1); }

.register_form input:focus { color: var(--y1); outline-color: var(--y2); }

.div-border-Ydark { border-color: var(--y1); }
.bordGold { border-color: var(--y1); }

input, select { color: var(--y1); }

/* character/skins constructor */
.character_wrapper .ul-container .tabs-game li:first-child { border-left-color: var(--y1); }
.character_wrapper .ul-container .tabs-game li:last-child { border-right-color: var(--y1); }
.character_wrapper .shop-outer:before, .character_wrapper .price-outer:before { border-color: var(--y1); }
.character_wrapper .shop-top { border-bottom-color: var(--y1); }
.character_wrapper .shop-outer, .character_wrapper .price-outer { color: var(--y1); }
.character_wrapper .skin_prev:before { color: var(--y1); }
.character_wrapper .ul-price .elemShopName { color: var(--y1); }

/* rankings */
.page-rankings ul.rankings-by-country li.selected { color: var(--y1); }
.page-rankings ul.page-tabs.rankings-by-period li:hover,
.page-rankings ul.page-tabs.rankings-by-sort li:hover,
.page-rankings ul.page-tabs.rankings-by-filter li:hover,
.page-rankings ul.page-tabs.rankings-by-tour li:hover { color: var(--y1); }
.page-rankings ul.page-tabs.rankings-by-period li.selected,
.page-rankings ul.page-tabs.rankings-by-sort li.selected,
.page-rankings ul.page-tabs.rankings-by-filter li.selected,
.page-rankings ul.page-tabs.rankings-by-tour li.selected { color: var(--y1) !important; }

/* payment / shop */
.payment { color: var(--y2); }
.payment .out-balance { border-bottom-color: var(--y1); }
.payment .out_box .one_box { border-color: var(--y1); }
.payment .out_box .summ { border-color: var(--y1); }
.payment .out_box .summ input, .payment .out_box .summ select { color: var(--y1); }
.payment .out_box .box { border-color: var(--y1); color: var(--y1); }

.tutor_popup { border-color: var(--y1); color: var(--y2); }
.tutoial_arrow_label { color: var(--y1); }

.scr h2 { color: var(--y1); }
.buy-slot-modal h3 { color: var(--y1); }

.list-change-but li.select { background-color: var(--y1); }
.list-change-but li.select input { outline-color: var(--y1); }

.shop .peek-inside:hover { color: var(--y2); }
.shop a.gold_chest:hover { background-color: var(--y1); }
.shop .out_box .box .number { color: var(--y1); border-color: var(--y1); }
.shop .out_box .box .perk-time { color: var(--y1); }
.shop .bg-chest.gold_chest { color: var(--y1); }
.shop .skins-select .skins-top { border-bottom-color: var(--y1); }
.shop .skins-select .skins-top .bord-gold:before { border-color: var(--y1); }
.shop .skins-select .skins-preview { border-color: var(--y1); }
.shop .out_box .name-basic-shop { border-bottom-color: var(--y1); }
.shop .out_box .name-basic-shop ul.list-level li.active-level { color: var(--y1); border-left-color: var(--y1); border-right-color: var(--y1); }
.shop .out_box.advanced-shop .name-advanced-shop h2 { background-color: var(--y1); }
.shop .out_box.advanced-shop .things-advanced-shop li { border-bottom-color: var(--y1); }

/* server/room select */
#server-select ul.server-list li.selected:before,
.room-select ul.server-list li.selected:before,
#server-select ul.room-list li.selected:before,
.room-select ul.room-list li.selected:before { border-color: var(--y1); }
#server-select ul.server-list li.selected h2,
.room-select ul.server-list li.selected h2,
#server-select ul.room-list li.selected h2,
.room-select ul.room-list li.selected h2 { color: var(--y1); }
.out-servers ul.server-list li h2 { color: var(--y2); }
ul.room-list li h2 { color: var(--y2); }
ul.room-list li .n_player { color: var(--y1); }
ul.room-list li .time_left { color: var(--y1); border-bottom-color: var(--y1); }
ul.room-list li .txt-room { color: var(--y1); border-top-color: var(--y1); }

/* modals */
.out_modal .modal:before, .out_modal_light .modal:before { border-color: var(--y1); }
.out_modal .modal h2.borb_bot, .out_modal_light .modal h2.borb_bot { border-bottom-color: var(--y1); }

.goal-icon-list .goal_border { border-color: var(--y1); }

/* tasks / donations */
.one-task { border-color: var(--y1); }
.one-task .progress { border-color: var(--y1); }
.one-task .txt_donated { color: var(--y1); }
ul.task-tab li { color: var(--y1); }
.list-donators li.main-donator .num { color: var(--y1); }

.wiki .settings_input { border-color: var(--y1); }
.wiki a:hover, .wiki a:active { color: var(--y2); }

/* chat */
#chat #dropDownChat ul li:hover { color: var(--y2); }
#chat .misc-top.misc-top, #chat .misc-bottom.misc-top { border-bottom-color: var(--y1); }
#chat .misc-top h1, #chat .misc-bottom h1 { color: var(--y1); }
#chat ul.chat-list li a { color: var(--y1); }
#chat ul.chat-list li b.my-msg { color: var(--y1); }
#chat ul.chat-list li b.nickname { color: var(--y2); }
.chatmsg .mentions-autocomplete-list ul li b { color: var(--y1); }

/* scoreboard */
#scoreboard-new .perks2-list, #scoreboard-new .perk-details { border-color: var(--y1); }
#scoreboard-new .perks2-list .title-ul, #scoreboard-new .perk-details .title-ul { color: var(--y1); }
#scoreboard-new .perks2-list .out-bord-gold .power:before, #scoreboard-new .perk-details .out-bord-gold .power:before,
#scoreboard-new .perks2-list .out-bord-gold .time:before, #scoreboard-new .perk-details .out-bord-gold .time:before,
#scoreboard-new .perks2-list .out-bord-gold .perk:before, #scoreboard-new .perk-details .out-bord-gold .perk:before { border-top-color: var(--y1); }
#scoreboard-new .perks2-list .out-bord-gold .bord-gold-li.down-illustration:before, #scoreboard-new .perk-details .out-bord-gold .bord-gold-li.down-illustration:before,
#scoreboard-new .perks2-list .out-bord-gold .bord-gold-li.down-illustration:after, #scoreboard-new .perk-details .out-bord-gold .bord-gold-li.down-illustration:after { border-left-color: var(--y1); }
#scoreboard-new .perks2-list .out-bord-gold .bord-gold-li.down-illustration .perk2-count, #scoreboard-new .perk-details .out-bord-gold .bord-gold-li.down-illustration .perk2-count { color: var(--y1); border-color: var(--y1); }
#scoreboard-new ul.pagination { border-top-color: var(--y1); border-bottom-color: var(--y1); }
#scoreboard-new ul.pagination li.count { border-left-color: var(--y1); border-right-color: var(--y1); }
#scoreboard-new ul.pagination button { color: var(--y1); }
#scoreboard-new .perks .title { border-top-color: var(--y1); border-bottom-color: var(--y1); }
#scoreboard-new #perks div.perks-list-wrap div.event-block p.no-event,
#scoreboard-new #bonus-perks div.perks-list-wrap div.event-block p.no-event { color: var(--y1); }
#scoreboard-new #perks .perks2-list .list-inventory .perk2-count,
#scoreboard-new #bonus-perks .perks2-list .list-inventory .perk2-count { color: var(--y1); border-color: var(--y1); }
#scoreboard-new #perks .perks2-list li.perks2-item .perk2-count.count-time,
#scoreboard-new #bonus-perks .perks2-list li.perks2-item .perk2-count.count-time { color: var(--y1); }
#scoreboard-new #perks .perk-details span.perk-timed, #scoreboard-new #bonus-perks .perk-details span.perk-timed,
#scoreboard-new #perks .perks-item span.perk-timed, #scoreboard-new #bonus-perks .perks-item span.perk-timed { color: var(--y1); }
#scoreboard-new #perks .perk-details .perk-count.count-time, #scoreboard-new #bonus-perks .perk-details .perk-count.count-time,
#scoreboard-new #perks .perks-item .perk-count.count-time, #scoreboard-new #bonus-perks .perks-item .perk-count.count-time { color: var(--y1); }
#scoreboard-new #scores ul#list-players li.highlighted { background-color: var(--y1) !important; }
#scoreboard-new #scores .headings { border-bottom-color: var(--y1); }
#scoreboard-new #scores .headings span { color: var(--y1); }
#scoreboard-new #scores .headings input { border-color: var(--y1); }
#scoreboard-new #scores #list-thumb { outline-color: var(--y1); }
#scoreboard-new #scores #list-thumb:hover, #scoreboard-new #scores #list-thumb:active { background-color: var(--y1); }
#scoreboard-new #squads .settings .search-squads { border-color: var(--y1); }
#scoreboard-new #squads .settings .create-squad { color: var(--y1); }
#scoreboard-new #squads ul.squads li.squad.my header { background-color: var(--y2); }
#scoreboard-new #squads ul.squads li.squad.my header i.lock { background-color: var(--y2); }
#scoreboard-new #squads ul.squads li.squad.my ol.players, #scoreboard-new #squads ul.squads li.squad.my div.players { border-color: var(--y2); }
#scoreboard-new #squads ul.squads li.squad.leader header i.lock { background-color: var(--y1); }
#scoreboard-new #squads ul.squads li.squad.leader header i.lock:hover { background-color: var(--y1); }
#scoreboard-new .register_tab { color: var(--y1); }
#scoreboard-new .register_tab input:focus { color: var(--y1); outline-color: var(--y2); }

#bonus-board .bonus-box:before { border-color: var(--y1); }
#bonus-board .bonus-box .bonus-header-daily, #bonus-board .bonus-box .bonus-header-kills-total,
#bonus-board .bonus-box .bonus-header-kills-round, #bonus-board .bonus-box .bonus-header-gold-total,
#bonus-board .bonus-box .bonus-header-rounds { color: var(--y2); }

.tabs-game.blue-select-color { color: var(--y1); }
.tabs-game li { border-left-color: var(--y1); border-bottom-color: var(--y1); }
.tabs-game li:hover, .tabs-game li.selected { color: var(--y1); border-color: var(--y1); }

.out-slider button.src { color: var(--y1); }
.out-slider .navigation-slider li { background-color: var(--y2); }
.out-slider .navigation-slider li.select { background-color: var(--y1); }

/* death / end-round screens */
#scoreboard-death h2.title { border-bottom-color: var(--y1); }
.list-loss li .how { color: var(--y1); }
.list-loss li .how span:first-child { border-right-color: var(--y1); }
ul.list-powerups li .count { color: var(--y1); }
ul.list-tasks li { border-color: var(--y1); }
ul.list-tasks li .desc title { color: var(--y1); }
.death-skins { border-top-color: var(--y1); }
.death-skins .skin-outer .activ_skin { color: var(--y1); }
.death-buttoms { border-top-color: var(--y1); }
.death_button { color: var(--y1); border-color: var(--y1); }
#scoreboard-end-round h2.title { border-bottom-color: var(--y1); }
#scoreboard-end-round .death-buttoms { border-top-color: var(--y1); }
#scoreboard-end-round ul.list-tasks { border-left-color: var(--y1); }
#scoreboard-end-round #end-round-scores { border-right-color: var(--y1); }
#scoreboard-end-round #end-round-scores ul#end-round-list-players li.highlighted { background-color: var(--y1) !important; }
#scoreboard-end-round #end-round-scores .headings { border-bottom-color: var(--y1); }
#scoreboard-end-round #end-round-scores .headings span { color: var(--y1); }
#scoreboard-end-round #end-round-scores .headings input { border-color: var(--y1); }
#scoreboard-end-round #end-round-scores #end-round-list-thumb { outline-color: var(--y1); }
#scoreboard-end-round #end-round-scores #end-round-list-thumb:hover, #scoreboard-end-round #end-round-scores #end-round-list-thumb:active { background-color: var(--y1); }

.new-rank .level-name { color: var(--y1); }

ul.list-players li span.nickname { color: var(--y2); }

.quest-items .help-item { border-color: var(--y1); }

/* landing / login */
.class_landing { color: var(--y1); }
.class_landing:before { border-color: var(--y1); }
.class_landing .form_land2.form3 { border-color: var(--y1); }
.class_landing .form_land2 input:focus { border-color: var(--y1); color: var(--y1); }
.class_landing .form_land2 input.tibia-code { color: var(--y1); }
.class_landing .form_land2 a { color: var(--y1); }
.class_landing .logo-txt { color: var(--y2); }

.region-list li { border-color: var(--y1); }
.reg-refresh { border-color: var(--y1); }
.reg-game { border-color: var(--y1); }

.out-chests .box .bonus-perk-count { color: var(--y1); border-color: var(--y1); }

.simple-shop-elem {
  border-top-color: var(--y1);
  border-bottom-color: var(--y1);
  border-left-color: var(--y1);
  border-right-color: var(--y1);
}

.table-log a { color: var(--y1); }

/* jPlayer audio widget skin */
.jp-audio, .jp-audio-stream, .jp-video { color: var(--y1); }
.jp-audio-stream a { color: var(--y1); }

/* hide the animated background canvas */
#bg { display: none !important; }
`;

    function injectTheme() {
        const style = document.createElement('style');
        style.id = '__gobUnifiedWhiteTheme';
        style.textContent = THEME_CSS;
        (document.head || document.documentElement).appendChild(style);
    }
    injectTheme();

    // ============================================================
    // 2) IMAGE REPLACEMENT
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
                console.warn('[GOB Unified] Tileset preload failed; using direct URL fallback.', err);
            });
    }
    preloadTileset();

    function buildWhiteExplosion(sourceUrl) {
        const key = absoluteUrl(sourceUrl);
        if (whiteExplosionCache.has(key)) return whiteExplosionCache.get(key);

        const promise = new Promise((resolve, reject) => {
            const img = new NativeImage();

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

                    buildWhiteExplosion(stringValue).then(whiteUrl => {
                        if (whiteUrl) {
                            nativeImageSrcSet.call(target, whiteUrl);
                        } else {
                            nativeImageSrcSet.call(target, stringValue);
                        }
                    });

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
    // 3) НАСТРАИВАЕМЫЕ КЛАВИШИ И РУССКАЯ ПАНЕЛЬ
    // ============================================================
    const STORAGE_KEY = '__gobUnifiedRuKeysV2';

    const defaults = {
        actionKey: 'KeyK',
        chainHotkey: 'KeyJ',
        stickHotkey: 'KeyL',
        teleportHotkey: 'KeyZ'
    };

    const keyInfo = new Map();
    const en = "qwertyuiop[]asdfghjkl;\'zxcvbnm,.";
    const ru = 'йцукенгшщзхъфывапролджэячсмитьбю';
    const punctuation = {
        '[': 'BracketLeft',
        ']': 'BracketRight',
        ';': 'Semicolon',
        "'": 'Quote',
        ',': 'Comma',
        '.': 'Period'
    };

    for (let i = 0; i < en.length; i++) {
        const code = punctuation[en[i]] || 'Key' + en[i].toUpperCase();
        keyInfo.set(code, {
            key: en[i],
            ru: ru[i],
            keyCode: punctuation[en[i]]
                ? ({BracketLeft:219,BracketRight:221,Semicolon:186,Quote:222,Comma:188,Period:190})[code]
                : en[i].toUpperCase().charCodeAt(0)
        });
    }

    for (let i = 0; i <= 9; i++) {
        keyInfo.set('Digit' + i, { key: String(i), keyCode: 48 + i });
    }

    for (let i = 1; i <= 12; i++) {
        keyInfo.set('F' + i, { key: 'F' + i, keyCode: 111 + i });
    }

    for (const [code, key, keyCode] of [
        ['Space',' ',32],
        ['Enter','Enter',13],
        ['Tab','Tab',9],
        ['Backspace','Backspace',8],
        ['Escape','Escape',27],
        ['ArrowLeft','ArrowLeft',37],
        ['ArrowUp','ArrowUp',38],
        ['ArrowRight','ArrowRight',39],
        ['ArrowDown','ArrowDown',40],
        ['Minus','-',189],
        ['Equal','=',187],
        ['Backquote','`',192],
        ['Backslash','\\',220],
        ['Delete','Delete',46],
        ['Home','Home',36],
        ['End','End',35],
        ['PageUp','PageUp',33],
        ['PageDown','PageDown',34]
    ]) {
        keyInfo.set(code, {
            key,
            keyCode,
            ru: code === 'Backquote' ? 'ё' : undefined
        });
    }

    function eventCode(event) {
        if (keyInfo.has(event.code)) return event.code;

        const key = String(event.key || '').toLowerCase();

        for (const [code, info] of keyInfo) {
            if (info.key.toLowerCase() === key || info.ru === key) {
                return code;
            }
        }

        return null;
    }

    function keyLabel(code) {
        const info = keyInfo.get(code);
        if (!info) return '—';

        const labels = {
            Space: 'Пробел',
            Enter: 'Enter',
            Tab: 'Tab',
            Backspace: 'Backspace',
            Escape: 'Esc',
            ArrowLeft: '←',
            ArrowRight: '→',
            ArrowUp: '↑',
            ArrowDown: '↓'
        };

        return (labels[code] || info.key.toUpperCase()) +
            (info.ru ? ' / ' + info.ru.toUpperCase() : '');
    }

    let bindings = { ...defaults };

    try {
        const saved = JSON.parse(localStorage.getItem(STORAGE_KEY) || '{}');

        if (!saved.actionKey) {
            saved.actionKey = saved.chainKey || saved.stickKey;
        }

        for (const name of Object.keys(defaults)) {
            if (keyInfo.has(saved[name])) {
                bindings[name] = saved[name];
            }
        }
    } catch {}

    let autoChainActive = false;
    let autoChainInterval = null;
    let pulseReleaseTimer = null;
    let pulseHeldCode = null;
    let kStuck = false;

    let featureUi = null;
    let menuOpenButton = null;
    let menuHidden = false;

    try {
        menuHidden = localStorage.getItem(STORAGE_KEY + ':hidden') === '1';
    } catch {}

    function setMenuHidden(hidden) {
        menuHidden = Boolean(hidden);

        if (menuHidden && captureName) {
            captureName = null;

            if (settingsNotice) {
                settingsNotice.textContent = 'Изменение отменено.';
            }

            updateFeatureUi();
        }

        if (featureUi) {
            featureUi.style.display = menuHidden ? 'none' : 'block';
        }

        if (menuOpenButton) {
            menuOpenButton.style.display = menuHidden ? 'block' : 'none';
        }

        try {
            localStorage.setItem(
                STORAGE_KEY + ':hidden',
                menuHidden ? '1' : '0'
            );
        } catch {}
    }

    let chainUiBtn;
    let kStickUiBtn;
    let teleportUiBtn;
    let queueFeatureLabel;
    let captureName = null;

    const capturedCodes = new Set();
    const bindingButtons = new Map();
    const hotkeyBadges = new Map();
    const syntheticKeys = new WeakSet();

    let settingsNotice = null;

    function isTypingTarget(target) {
        return !!target && (
            target.isContentEditable ||
            !!target.closest?.('input,textarea,select,[contenteditable="true"]')
        );
    }

    function sendKey(type, code) {
        const info = keyInfo.get(code);
        if (!info) return;

        const event = new KeyboardEvent(type, {
            key: info.key,
            code,
            keyCode: info.keyCode,
            which: info.keyCode,
            bubbles: true,
            cancelable: true
        });

        for (const prop of ['keyCode', 'which']) {
            if (event[prop] !== info.keyCode) {
                Object.defineProperty(event, prop, {
                    value: info.keyCode
                });
            }
        }

        syntheticKeys.add(event);
        document.dispatchEvent(event);
    }

    function releasePulse() {
        if (pulseReleaseTimer !== null) {
            clearTimeout(pulseReleaseTimer);
        }

        pulseReleaseTimer = null;

        if (pulseHeldCode) {
            sendKey('keyup', pulseHeldCode);
        }

        pulseHeldCode = null;
    }

    function autoChainPulse() {
        if (
            !autoChainActive ||
            captureName ||
            isTypingTarget(document.activeElement) ||
            document.hidden
        ) {
            return;
        }

        releasePulse();

        const code = bindings.actionKey;
        pulseHeldCode = code;

        sendKey('keydown', code);

        pulseReleaseTimer = setTimeout(
            releasePulse,
            10
        );
    }

    function stopAutoChain() {
        if (autoChainInterval !== null) {
            clearInterval(autoChainInterval);
        }

        autoChainInterval = null;
        releasePulse();
    }

    function toggleAutoChain() {
        autoChainActive = !autoChainActive;
        stopAutoChain();

        if (autoChainActive) {
            autoChainInterval = setInterval(
                autoChainPulse,
                CONFIG.autoChainMs
            );
        }

        updateFeatureUi();
        return autoChainActive;
    }

    function toggleKStick() {
        kStuck = !kStuck;

        if (!kStuck) {
            sendKey('keyup', bindings.actionKey);
        }

        updateFeatureUi();
        return kStuck;
    }

    function triggerTeleport() {
        return toggleWsFreeze();
    }

    function styleFeatureButton(button) {
        button.type = 'button';
        button.style.cssText =
            'padding:5px 8px;' +
            'border:1px solid rgba(217,166,255,.38);' +
            'border-radius:5px;' +
            'background:#202020;' +
            'color:#fff;' +
            'font:11px monospace;' +
            'cursor:pointer';
    }

    function setFeatureButtonState(button, active) {
        if (!button) return;

        button.textContent = active ? 'ВКЛ' : 'ВЫКЛ';
        button.style.background = active ? '#d9a6ff' : '#202020';
        button.style.color = active ? '#111' : '#fff';
    }

    function updateFeatureUi() {
        setFeatureButtonState(chainUiBtn, autoChainActive);
        setFeatureButtonState(kStickUiBtn, kStuck);
        setFeatureButtonState(teleportUiBtn, wsFrozen);

        if (queueFeatureLabel) {
            queueFeatureLabel.textContent =
                'Пакетов в очереди: ' + wsQueue.length;
        }

        for (const [name, button] of bindingButtons) {
            button.textContent =
                captureName === name
                    ? 'Нажмите клавишу…'
                    : keyLabel(bindings[name]);
        }

        for (const [name, badge] of hotkeyBadges) {
            badge.textContent = keyLabel(bindings[name]);
        }
    }

    function saveBindings() {
        try {
            localStorage.setItem(
                STORAGE_KEY,
                JSON.stringify(bindings)
            );

            settingsNotice.textContent =
                'Сохранено автоматически.';
        } catch {
            settingsNotice.textContent =
                'Не удалось сохранить. Настройки действуют до перезагрузки.';
        }
    }

    function createFeatureUi() {
        if (featureUi || !document.body) return;

        featureUi = document.createElement('div');
        featureUi.id = '__gobUnifiedFeatureUi';
        featureUi.lang = 'ru';
        featureUi.style.cssText =
            'position:fixed;' +
            'right:18px;' +
            'bottom:18px;' +
            'z-index:999999;' +
            'width:310px;' +
            'max-width:calc(100vw - 36px);' +
            'max-height:80vh;' +
            'overflow:auto;' +
            'box-sizing:border-box;' +
            'padding:12px;' +
            'border:1px solid rgba(217,166,255,.4);' +
            'border-radius:9px;' +
            'background:rgba(16,16,18,.96);' +
            'color:#fff;' +
            'font:12px/1.4 monospace;' +
            'box-shadow:0 5px 18px #0007';

        const title = document.createElement('div');
        title.textContent = 'УПРАВЛЕНИЕ GOB';
        title.style.cssText =
            'color:#d9a6ff;' +
            'font-weight:bold;' +
            'margin-bottom:8px';

        const header = document.createElement('div');
        header.style.cssText =
            'display:flex;' +
            'align-items:center;' +
            'justify-content:space-between;' +
            'gap:8px;' +
            'margin-bottom:8px';

        title.style.marginBottom = '0';

        const hideButton = document.createElement('button');
        styleFeatureButton(hideButton);
        hideButton.textContent = 'Скрыть';
        hideButton.title =
            'Скрыть панель. Горячие клавиши продолжат работать.';
        hideButton.addEventListener(
            'click',
            () => setMenuHidden(true)
        );

        header.append(title, hideButton);
        featureUi.append(header);

        menuOpenButton = document.createElement('button');
        menuOpenButton.id = '__gobUnifiedOpenMenu';
        styleFeatureButton(menuOpenButton);
        menuOpenButton.textContent = 'Меню';
        menuOpenButton.title =
            'Открыть панель управления';
        menuOpenButton.style.cssText +=
            ';position:fixed;' +
            'right:18px;' +
            'bottom:18px;' +
            'z-index:999999;' +
            'padding:8px 12px;' +
            'background:rgba(16,16,18,.96);' +
            'box-shadow:0 3px 12px #0007';

        menuOpenButton.addEventListener(
            'click',
            () => setMenuHidden(false)
        );

        document.body.append(menuOpenButton);

        function row(label, name, handler) {
            const element = document.createElement('div');
            element.style.cssText =
                'display:flex;' +
                'align-items:center;' +
                'gap:8px;' +
                'margin:6px 0';

            const text = document.createElement('span');
            text.textContent = label;
            text.style.flex = '1';

            const badge = document.createElement('span');
            badge.style.cssText =
                'color:#d9a6ff;' +
                'font-size:10px';

            hotkeyBadges.set(name, badge);

            const button = document.createElement('button');
            styleFeatureButton(button);
            button.addEventListener('click', handler);

            element.append(text, badge, button);
            featureUi.append(element);

            return button;
        }

        chainUiBtn = row(
            'Автоцепь',
            'chainHotkey',
            toggleAutoChain
        );

        kStickUiBtn = row(
            'Удержание клавиши',
            'stickHotkey',
            toggleKStick
        );

        teleportUiBtn = row(
            'Телепорт',
            'teleportHotkey',
            triggerTeleport
        );

        teleportUiBtn.title =
            'ВКЛ — заморозить исходящие пакеты; ВЫКЛ — отправить очередь.';

        queueFeatureLabel = document.createElement('div');
        queueFeatureLabel.style.cssText =
            'font-size:10px;' +
            'color:#aaa';

        featureUi.append(queueFeatureLabel);

        const flush = document.createElement('button');
        styleFeatureButton(flush);
        flush.textContent = 'Отправить очередь';
        flush.addEventListener(
            'click',
            flushWsQueue
        );

        featureUi.append(flush);

        const details = document.createElement('details');
        details.style.marginTop = '10px';

        const summary = document.createElement('summary');
        summary.textContent = 'Настройки клавиш';
        summary.style.cssText =
            'cursor:pointer;' +
            'color:#d9a6ff';

        details.append(summary);

        const help = document.createElement('p');
        help.textContent =
            'Нажмите кнопку, затем нужную клавишу. EN и RU работают по одной физической клавише. Esc — отмена. Удержание блокирует отпускание выбранной клавиши после её нажатия.';
        help.style.cssText =
            'color:#aaa;' +
            'font-size:10px';

        details.append(help);

        const names = {
            actionKey: 'Клавиша автоцепи и удержания',
            chainHotkey: 'Вкл/выкл автоцепь',
            stickHotkey: 'Вкл/выкл удержание',
            teleportHotkey: 'Вкл/выкл телепорт'
        };

        for (const [name, label] of Object.entries(names)) {
            const line = document.createElement('div');
            line.style.cssText =
                'display:flex;' +
                'align-items:center;' +
                'justify-content:space-between;' +
                'gap:8px;' +
                'margin:7px 0';

            const text = document.createElement('span');
            text.textContent = label;

            const button = document.createElement('button');
            styleFeatureButton(button);

            button.addEventListener('click', () => {
                captureName = name;
                releasePulse();
                settingsNotice.textContent =
                    'Нажмите нужную клавишу (Esc — отмена).';
                updateFeatureUi();
            });

            bindingButtons.set(name, button);

            line.append(text, button);
            details.append(line);
        }

        settingsNotice = document.createElement('div');
        settingsNotice.textContent =
            'Настройки сохраняются автоматически.';
        settingsNotice.style.cssText =
            'font-size:10px;' +
            'color:#aaa;' +
            'margin:8px 0';

        details.append(settingsNotice);

        const reset = document.createElement('button');
        styleFeatureButton(reset);
        reset.textContent = 'Сбросить клавиши';

        reset.addEventListener('click', () => {
            releasePulse();

            if (kStuck) {
                kStuck = false;
                sendKey('keyup', bindings.actionKey);
            }

            bindings = { ...defaults };
            captureName = null;

            saveBindings();
            updateFeatureUi();
        });

        details.append(reset);
        featureUi.append(details);

        document.body.append(featureUi);

        updateFeatureUi();
        setMenuHidden(menuHidden);
    }

    window.addEventListener('keydown', event => {
        if (syntheticKeys.has(event)) return;

        const code = eventCode(event);

        if (captureName) {
            event.preventDefault();
            event.stopImmediatePropagation();

            if (code) {
                capturedCodes.add(code);
            }

            if (event.repeat) return;

            if (code === 'Escape') {
                captureName = null;
                settingsNotice.textContent =
                    'Изменение отменено.';
                updateFeatureUi();
                return;
            }

            if (
                !code ||
                event.ctrlKey ||
                event.altKey ||
                event.metaKey
            ) {
                settingsNotice.textContent =
                    'Выберите одну клавишу без Ctrl, Alt или Win.';
                return;
            }

            const name = captureName;

            const conflict = Object.keys(bindings).find(other =>
                other !== name &&
                (
                    (
                        name.endsWith('Hotkey') &&
                        other.endsWith('Hotkey') &&
                        bindings[other] === code
                    ) ||
                    (
                        name.endsWith('Hotkey') &&
                        other === 'actionKey' &&
                        bindings[other] === code
                    ) ||
                    (
                        name === 'actionKey' &&
                        other.endsWith('Hotkey') &&
                        bindings[other] === code
                    )
                )
            );

            if (conflict) {
                settingsNotice.textContent =
                    'Клавиша уже используется. Выберите другую.';
                return;
            }

            releasePulse();

            if (name === 'actionKey' && kStuck) {
                kStuck = false;
                sendKey(
                    'keyup',
                    bindings.actionKey
                );
            }

            bindings[name] = code;
            captureName = null;

            saveBindings();
            updateFeatureUi();

            return;
        }

        if (
            isTypingTarget(event.target) ||
            event.ctrlKey ||
            event.altKey ||
            event.metaKey ||
            event.isComposing
        ) {
            return;
        }

        const handlers = {
            chainHotkey: toggleAutoChain,
            stickHotkey: toggleKStick,
            teleportHotkey: triggerTeleport
        };

        for (const [name, handler] of Object.entries(handlers)) {
            if (code === bindings[name]) {
                event.preventDefault();
                event.stopImmediatePropagation();

                if (!event.repeat) {
                    handler();
                }

                return;
            }
        }
    }, true);

    window.addEventListener('keyup', event => {
        if (syntheticKeys.has(event)) return;

        const code = eventCode(event);

        if (
            capturedCodes.delete(code) ||
            captureName
        ) {
            event.preventDefault();
            event.stopImmediatePropagation();
            return;
        }

        if (isTypingTarget(event.target)) return;

        if (
            kStuck &&
            code === bindings.actionKey
        ) {
            event.preventDefault();
            event.stopImmediatePropagation();
            return;
        }

        if (
            Object.keys(bindings).some(
                name =>
                    name.endsWith('Hotkey') &&
                    bindings[name] === code
            )
        ) {
            event.preventDefault();
            event.stopImmediatePropagation();
        }
    }, true);

    window.addEventListener(
        'blur',
        releasePulse
    );

    window.toggleAutoChain = toggleAutoChain;
    window.toggleKStick = toggleKStick;

    // ============================================================
    // 4) WEBSOCKET FREEZE / TELEPORT
    // ============================================================
    let wsFrozen = false;
    let wsQueue = [];

    const nativeWebSocketSend =
        WebSocket.prototype.send;

    function directWsSend(ws, data) {
        return nativeWebSocketSend.call(
            ws,
            data
        );
    }

    WebSocket.prototype.send = function (data) {
        if (!wsFrozen) {
            return directWsSend(
                this,
                data
            );
        }

        wsQueue.push({
            ws: this,
            data
        });

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
            if (
                packet.ws.readyState ===
                WebSocket.OPEN
            ) {
                try {
                    directWsSend(
                        packet.ws,
                        packet.data
                    );
                } catch (err) {
                    console.warn(
                        '[GOB Unified] Failed to flush a queued packet.',
                        err
                    );
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

        log(
            'WS Freeze:',
            wsFrozen ? 'ВКЛ' : 'ВЫКЛ'
        );

        return wsFrozen;
    }

    function toggleWsFreeze() {
        return setWsFrozen(
            !wsFrozen
        );
    }

    function updateWsUi() {
        updateFeatureUi();

        if (!wsUi) return;

        wsQueueLabel.textContent =
            'Пакетов в очереди: ' +
            wsQueue.length;

        wsFreezeBtn.textContent =
            wsFrozen ? 'ВКЛ' : 'ВЫКЛ';

        wsFreezeBtn.style.background =
            wsFrozen ? '#ffffff' : '#222';

        wsFreezeBtn.style.color =
            wsFrozen ? '#000000' : '#ffffff';

        wsFreezeBtn.style.borderColor =
            wsFrozen ? '#ffffff' : '#555';
    }

    if (document.readyState === 'loading') {
        document.addEventListener(
            'DOMContentLoaded',
            createFeatureUi,
            { once: true }
        );
    } else {
        createFeatureUi();
    }

    window.toggleWsFreeze = toggleWsFreeze;
    window.flushWsQueue = flushWsQueue;

    // ============================================================
    // STATUS + CLEANUP
    // ============================================================
    window.getUnifiedModStatus = () => ({
        autoChain: autoChainActive,
        kStick: kStuck,
        wsFrozen,
        wsQueuedPackets: wsQueue.length,
        tilesetCached: Boolean(tilesetBlobUrl)
    });

    window.addEventListener('beforeunload', () => {
        stopAutoChain();

        // Не отправляем очередь при закрытии страницы.
        wsQueue.length = 0;
    });

    log('Загружено.');
    log(
        'Клавиши настраиваются в панели. По умолчанию: J/О — автоцепь, L/Д — удержание, Z/Я — телепорт (заморозка сети).'
    );
    log(
        'Canvas drawImage interception, bomb timer, blast overlay, world grid and block tracking removed.'
    );
    log(
        'Console: getUnifiedModStatus()'
    );
})();
