// ==UserScript==
// @name         Purple Theme Override (Static CSS)
// @namespace    http://tampermonkey.net/
// @version      3.1
// @description  Overrides the site's gold/yellow theme colors with purple via a real static stylesheet, built directly from the site's own CSS selectors.
// @match        *://*/*
// @run-at       document-start
// @grant        GM_addStyle
// ==/UserScript==

(function () {
  'use strict';

  const css = `
:root {
  --y1: #9b30ff; /* replaces #ffd21f / #ffd31f / #ffd733 / #ffff00 / gold / yellow */
  --y2: #d9a6ff; /* replaces #fceaa2 / #ffe28f / #ffe9aa (lighter gold accents) */
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

  if (typeof GM_addStyle === 'function') {
    GM_addStyle(css);
  } else {
    const style = document.createElement('style');
    style.textContent = css;
    (document.head || document.documentElement).appendChild(style);
  }
})();
