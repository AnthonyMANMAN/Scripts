// ==UserScript==
// @name         Game of Bombs - World Grid + Sprite/Tile Tracker
// @namespace    http://tampermonkey.net/
// @version      1.6.0
// @description  World-aligned grid, real 8x8 terrain tile tracking, and full sprite labels.
// @match        https://gameofbombs.com/*
// @match        http://gameofbombs.com/*
// @run-at       document-start
// @grant        none
// ==/UserScript==

(() => {
    'use strict';

    if (window.__GOB_WORLD_DEBUG_OVERLAY__) return;
    window.__GOB_WORLD_DEBUG_OVERLAY__ = true;

    // ============================================================
    // SETTINGS
    // ============================================================

    const TILE_SIZE = 32;
    const WORLD_WRAP_TILES = 32;

    const SETTINGS = {
        grid: true,
        coords: true,
        labels: true,
        tileLabels: true,
        spriteLabels: true,
        boxes: false,
        minimumSize: 8,
        maxSpriteLabels: 1000,
        maxTileLabels: 1800
    };

    // ============================================================
    // STATE
    // ============================================================

    let layer0 = null;
    let layer1 = null;

    let overlay = null;
    let overlayCtx = null;

    let worldMatrix = null;

    const frameDraws = {
        layer0: [],
        layer1: []
    };

    const blockPositions = Object.create(null);

    // ============================================================
    // SPRITE DATABASE
    // ============================================================

    const knownSpriteNames = new Set([
        // Core groups / atlases
        'items',
        'icons',
        'perks',
        'font',
        'main',
        'glyph',
        'atlas',
        'atlas-hud',
        'icons-hud',

        // Terrain
        'tiles',
        'tiles-winter',
        'tiles-moon',

        // Bombs / effects / objects
        'bomb64x64',
        'bomb32x32',
        'bomb_skin',
        'bomb_skin2',
        'mine',
        'wick',
        'explosion',
        'regen',
        'blast',
        'turret',
        'ball',
        'snowball',
        'sweets',
        'shield',
        'portal',
        'portal_strips_items',
        'boost-anim',
        'stun',
        'tangled',
        'gammaProjectile',
        'squad_pick',
        'big_arrow',
        'arrows',
        'signs',
        'hpbar',

        // Characters / mobs
        'default',
        'default2',
        'flying',
        'slowpoke',
        'jetpack',
        'nyan',
        'nyan_chest',
        'langolier',
        'mimic_1',
        'mimic_3',
        'robot',
        'orb',
        'jelly'
    ]);
    const runtimeSprites = [];

    // Game's real terrain children from:
    // sprites.get("tiles").childs
    // sprites.get("tiles-winter").childs
    // sprites.get("tiles-moon").childs
    const realTileFrames = [];

    let spriteAddHooked = false;
    let rebuildTimer = null;

    // When the game's tile renderer builds a CanvasPattern, it first draws
    // one child tile into a tiny offscreen canvas and then calls createPattern.
    // These maps let us carry the real tile name through that process.
    const lastTileDrawByCanvas = new WeakMap();
    const patternTileMeta = new WeakMap();
    const offscreenTileCells = new WeakMap();

    // ============================================================
    // HELPERS
    // ============================================================

    function mod(n, m) {
        return ((n % m) + m) % m;
    }

    function copyMatrix(m) {
        return {
            a: m.a,
            b: m.b,
            c: m.c,
            d: m.d,
            e: m.e,
            f: m.f
        };
    }

    function transformPoint(m, x, y) {
        return {
            x: m.a * x + m.c * y + m.e,
            y: m.b * x + m.d * y + m.f
        };
    }

    function inversePoint(m, x, y) {
        const det = m.a * m.d - m.b * m.c;

        if (Math.abs(det) < 0.000001) {
            return { x: 0, y: 0 };
        }

        const px = x - m.e;
        const py = y - m.f;

        return {
            x: (m.d * px - m.c * py) / det,
            y: (-m.b * px + m.a * py) / det
        };
    }

    function isGameLayer(canvas) {
        return canvas && (canvas.id === 'layer0' || canvas.id === 'layer1');
    }

    function isTerrainName(name) {
        return /grass|deep|sand|dirt|rock|field|hole|bridge|tile|button|wall|base|brick|bush|box|logs|cactus|chest|metal|gate|tunnel|tube|spike|flag|teleport|recycler|tomb|catapult|crystal|lava|goo|surf|barrel|floor|tough|diamond|gold|silver|skeleton|well|arena_spawn|arena_castle/i.test(name || '');
    }

    // ============================================================
    // SPRITE DEFINITION COLLECTION
    // ============================================================

    function collectDefinition(def) {
        if (!def) return;

        if (Array.isArray(def)) {
            for (const item of def) {
                collectDefinition(item);
            }
            return;
        }

        if (typeof def !== 'object') return;

        if (typeof def.name === 'string' && def.name.length) {
            knownSpriteNames.add(def.name);
        }

        if (Array.isArray(def.rows)) {
            for (const row of def.rows) {
                if (!Array.isArray(row)) continue;

                for (const entry of row) {
                    if (typeof entry === 'string' && entry.length) {
                        knownSpriteNames.add(entry);
                    } else if (
                        entry &&
                        typeof entry === 'object' &&
                        typeof entry.name === 'string' &&
                        entry.name.length
                    ) {
                        knownSpriteNames.add(entry.name);
                    }
                }
            }
        }

        if (Array.isArray(def.frames)) {
            for (const frame of def.frames) {
                if (
                    frame &&
                    typeof frame === 'object' &&
                    typeof frame.name === 'string' &&
                    frame.name.length
                ) {
                    knownSpriteNames.add(frame.name);
                }
            }
        }

        if (Array.isArray(def.childs)) {
            collectDefinition(def.childs);
        }
    }

    // ============================================================
    // REAL TILE INDEX
    // ============================================================

    function rebuildRealTileFrames() {
        const sprites = window.sprites;

        if (!sprites || typeof sprites.get !== 'function') {
            return;
        }

        realTileFrames.length = 0;

        const groups = [
            'tiles',
            'tiles-winter',
            'tiles-moon'
        ];

        for (const groupName of groups) {
            let group;

            try {
                group = sprites.get(groupName);
            } catch {
                continue;
            }

            if (!group || !Array.isArray(group.childs)) {
                continue;
            }

            for (let index = 0; index < group.childs.length; index++) {
                const child = group.childs[index];

                if (!child) continue;

                const image =
                    child.texture?.element ||
                    group.texture?.element ||
                    null;

                if (!image) continue;

                realTileFrames.push({
                    group: groupName,
                    index,

                    name:
                        cleanDisplayName(
                            child.name ||
                            `${groupName}[${index}]`
                        ),

                    x:
                        Number(child.x) || 0,

                    y:
                        Number(child.y) || 0,

                    width:
                        Number(child.frameWidth) ||
                        Number(group.frameWidth) ||
                        32,

                    height:
                        Number(child.frameHeight) ||
                        Number(group.frameHeight) ||
                        32,

                    image
                });
            }
        }
    }

    function identifyRealTile(image, sx, sy, sw, sh) {
        if (!realTileFrames.length) {
            try { rebuildRealTileFrames(); } catch {}
        }

        for (const tile of realTileFrames) {
            if (tile.image !== image) continue;

            if (
                Math.abs(sx - tile.x) < 1 &&
                Math.abs(sy - tile.y) < 1 &&
                Math.abs(sw - tile.width) < 1 &&
                Math.abs(sh - tile.height) < 1
            ) {
                return tile;
            }
        }

        // Tiles can be registered after our previous index pass.
        try { rebuildRealTileFrames(); } catch {}

        for (const tile of realTileFrames) {
            if (tile.image !== image) continue;
            if (
                Math.abs(sx - tile.x) < 1 &&
                Math.abs(sy - tile.y) < 1 &&
                Math.abs(sw - tile.width) < 1 &&
                Math.abs(sh - tile.height) < 1
            ) return tile;
        }

        return null;
    }

    // ============================================================
    // GENERIC RUNTIME SPRITE INDEX
    // ============================================================

    function cleanDisplayName(name) {
        if (!name) return '';

        return String(name)
            // Remove terrain-group attachment/prefix from labels.
            .replace(/^tiles-(?:winter|moon)-/i, '')
            .replace(/^tiles-/i, '')
            .replace(/^tiles(?:-winter|-moon)?[:/]/i, '');
    }

    function addRuntimeSpriteRecord(sprite, fallbackName = '') {
        if (!sprite || typeof sprite !== 'object') return;

        let image = null;

        try {
            image =
                sprite.texture?.element ||
                sprite.image ||
                sprite.element ||
                null;
        } catch {}

        const rawName =
            (
                typeof sprite.name === 'string' &&
                sprite.name
            )
                ? sprite.name
                : fallbackName;

        if (rawName) {
            knownSpriteNames.add(rawName);
        }

        /*
            IMPORTANT:
            Group children (items/bomb/ball/etc.) are often not retrievable
            reliably with sprites.get(childName). Index the actual child
            objects directly instead.
        */
        if (image && rawName) {
            runtimeSprites.push({
                name:
                    cleanDisplayName(rawName),

                rawName,

                sprite,
                image,

                x:
                    Number(sprite.x) || 0,

                y:
                    Number(sprite.y) || 0,

                frameWidth:
                    Number(sprite.frameWidth) ||
                    Number(sprite.width) ||
                    0,

                frameHeight:
                    Number(sprite.frameHeight) ||
                    Number(sprite.frameWidth) ||
                    Number(sprite.height) ||
                    0,

                renderWidth:
                    Number(sprite.renderWidth) || 0,

                renderHeight:
                    Number(sprite.renderHeight) || 0,

                frames:
                    Array.isArray(sprite.frames)
                        ? sprite.frames
                        : null
            });
        }

        if (Array.isArray(sprite.childs)) {
            for (const child of sprite.childs) {
                if (!child) continue;

                /*
                    Some child objects inherit the parent texture while others
                    already have it attached. Supply it temporarily in the
                    record if needed without mutating the game object.
                */
                let childImage = null;

                try {
                    childImage =
                        child.texture?.element ||
                        sprite.texture?.element ||
                        image ||
                        null;
                } catch {}

                const childName =
                    (
                        typeof child.name === 'string' &&
                        child.name
                    )
                        ? child.name
                        : '';

                if (childName) {
                    knownSpriteNames.add(childName);
                }

                if (childImage && childName) {
                    runtimeSprites.push({
                        name:
                            cleanDisplayName(childName),

                        rawName:
                            childName,

                        sprite:
                            child,

                        image:
                            childImage,

                        x:
                            Number(child.x) || 0,

                        y:
                            Number(child.y) || 0,

                        frameWidth:
                            Number(child.frameWidth) ||
                            Number(sprite.frameWidth) ||
                            0,

                        frameHeight:
                            Number(child.frameHeight) ||
                            Number(child.frameWidth) ||
                            Number(sprite.frameHeight) ||
                            Number(sprite.frameWidth) ||
                            0,

                        renderWidth:
                            Number(child.renderWidth) ||
                            Number(sprite.renderWidth) ||
                            0,

                        renderHeight:
                            Number(child.renderHeight) ||
                            Number(sprite.renderHeight) ||
                            0,

                        frames:
                            Array.isArray(child.frames)
                                ? child.frames
                                : null
                    });
                }

                // Nested groups are rare but supported.
                if (Array.isArray(child.childs)) {
                    addRuntimeSpriteRecord(
                        child,
                        childName
                    );
                }
            }
        }
    }

    function harvestSpriteNames() {
        const sprites = window.sprites;
        if (!sprites) return;

        const arr = sprites.arr;

        if (arr && typeof arr === 'object') {
            for (const name of Object.keys(arr)) {
                if (name) {
                    knownSpriteNames.add(name);
                }
            }
        }
    }

    function rebuildSpriteIndex() {
        const sprites = window.sprites;

        if (!sprites) {
            return;
        }

        harvestSpriteNames();
        runtimeSprites.length = 0;

        const seenObjects =
            new WeakSet();

        function addUnique(sprite, name) {
            if (
                !sprite ||
                typeof sprite !== 'object'
            ) {
                return;
            }

            /*
                Do not skip the same child if reached through a different name
                until its direct record has been added.
            */
            if (!seenObjects.has(sprite)) {
                seenObjects.add(sprite);
                addRuntimeSpriteRecord(sprite, name);
            }
        }

        /*
            Best source: the sprite manager's actual registry. This includes
            groups and their real child objects, which is what fixes bomb,
            ball, wick, items, etc.
        */
        const arr = sprites.arr;

        if (arr && typeof arr === 'object') {
            for (const name of Object.keys(arr)) {
                try {
                    addUnique(
                        arr[name],
                        name
                    );
                } catch {}
            }
        }

        /*
            Fallback for managers where some sprites are exposed only through
            get(name).
        */
        if (typeof sprites.get === 'function') {
            for (const name of knownSpriteNames) {
                if (!name) continue;

                try {
                    addUnique(
                        sprites.get(name),
                        name
                    );
                } catch {}
            }
        }

        rebuildRealTileFrames();
    }

    function scheduleSpriteIndexRebuild() {
        clearTimeout(rebuildTimer);

        rebuildTimer = setTimeout(() => {
            rebuildSpriteIndex();
        }, 0);

        // Some sprite children/textures become usable slightly later.
        setTimeout(rebuildSpriteIndex, 100);
        setTimeout(rebuildSpriteIndex, 500);
    }

    function identifySprite(image, sx, sy, sw, sh) {
        let best = null;
        let bestScore = Infinity;

        for (const record of runtimeSprites) {
            if (record.image !== image) continue;

            const fw =
                Number(record.frameWidth) || 0;

            const fh =
                Number(record.frameHeight) || 0;

            // ----------------------------------------------------
            // Explicit frame list
            // ----------------------------------------------------

            if (record.frames) {
                for (
                    let i = 0;
                    i < record.frames.length;
                    i++
                ) {
                    const frame =
                        record.frames[i];

                    if (!frame) continue;

                    const fx =
                        Number(frame.x) || 0;

                    const fy =
                        Number(frame.y) || 0;

                    const frameW =
                        Number(frame.width) ||
                        fw;

                    const frameH =
                        Number(frame.height) ||
                        fh;

                    if (
                        Math.abs(sx - fx) < 0.75 &&
                        Math.abs(sy - fy) < 0.75 &&
                        (
                            !frameW ||
                            Math.abs(sw - frameW) < 1.25
                        ) &&
                        (
                            !frameH ||
                            Math.abs(sh - frameH) < 1.25
                        )
                    ) {
                        const score =
                            (frameW || 9999) *
                            (frameH || 9999);

                        if (score < bestScore) {
                            bestScore = score;

                            best = {
                                name:
                                    cleanDisplayName(
                                        record.name
                                    ),

                                frame: i
                            };
                        }
                    }
                }
            }

            if (!fw || !fh) continue;

            // ----------------------------------------------------
            // Exact single-frame child
            // ----------------------------------------------------

            const exact =
                Math.abs(sx - record.x) < 0.75 &&
                Math.abs(sy - record.y) < 0.75 &&
                Math.abs(sw - fw) < 1.25 &&
                Math.abs(sh - fh) < 1.25;

            if (exact) {
                /*
                    Strongly prefer a named child over a big generic parent
                    atlas such as "atlas" or "items".
                */
                const genericParent =
                    /^(atlas|items|icons|main|tiles|perks)$/i
                        .test(record.rawName || record.name);

                const score =
                    fw * fh +
                    (genericParent ? 1000000 : 0);

                if (score < bestScore) {
                    bestScore = score;

                    best = {
                        name:
                            cleanDisplayName(
                                record.name
                            ),

                        frame: null
                    };
                }

                continue;
            }

            // ----------------------------------------------------
            // Multi-frame strip/grid resource
            // ----------------------------------------------------

            if (
                sx >= record.x - 0.75 &&
                sy >= record.y - 0.75
            ) {
                const relX =
                    sx - record.x;

                const relY =
                    sy - record.y;

                const gx =
                    relX / fw;

                const gy =
                    relY / fh;

                const gridX =
                    Math.abs(
                        gx -
                        Math.round(gx)
                    );

                const gridY =
                    Math.abs(
                        gy -
                        Math.round(gy)
                    );

                if (
                    gridX < 0.025 &&
                    gridY < 0.025 &&
                    Math.abs(sw - fw) < 1.25 &&
                    Math.abs(sh - fh) < 1.25
                ) {
                    const genericParent =
                        /^(atlas|items|icons|main|tiles|perks)$/i
                            .test(record.rawName || record.name);

                    const score =
                        fw * fh +
                        (genericParent ? 1000000 : 0);

                    if (score < bestScore) {
                        bestScore = score;

                        best = {
                            name:
                                cleanDisplayName(
                                    record.name
                                ),

                            frame:
                                Math.round(gx) +
                                ',' +
                                Math.round(gy)
                        };
                    }
                }
            }
        }

        return best;
    }

    // ============================================================
    // HOOK SPRITE MANAGER
    // ============================================================

    function hookSpriteManager() {
        if (spriteAddHooked) return true;

        const sprites = window.sprites;

        if (!sprites || typeof sprites.add !== 'function') {
            return false;
        }

        const originalAdd = sprites.add;

        if (originalAdd.__gobTrackerHook) {
            spriteAddHooked = true;
            return true;
        }

        function hookedAdd(definition, ...rest) {
            try {
                collectDefinition(definition);
            } catch {}

            const result =
                originalAdd.call(
                    this,
                    definition,
                    ...rest
                );

            // Build immediately because the game constructs tile children
            // synchronously during sprites.add().
            try {
                rebuildSpriteIndex();
            } catch {}

            scheduleSpriteIndexRebuild();

            return result;
        }

        hookedAdd.__gobTrackerHook = true;
        hookedAdd.__original = originalAdd;

        sprites.add = hookedAdd;

        spriteAddHooked = true;

        // The manager may already contain definitions if the hook attached late.
        scheduleSpriteIndexRebuild();

        console.log('[GOB Overlay] sprites.add hooked');

        return true;
    }

    // ============================================================
    // WORLD MATRIX
    // ============================================================

    function considerWorldMatrix(ctx, width, height) {
        if (ctx.canvas.id !== 'layer0') return;

        const m = ctx.getTransform();

        // World camera should be a positive uniform scale with no rotation.
        if (
            Math.abs(m.b) > 0.001 ||
            Math.abs(m.c) > 0.001
        ) {
            return;
        }

        if (m.a <= 0 || m.d <= 0) return;

        if (
            Math.abs(
                Math.abs(m.a) -
                Math.abs(m.d)
            ) > 0.01
        ) {
            return;
        }

        // Ignore obvious identity/UI transforms.
        if (
            Math.abs(m.e) < 20 &&
            Math.abs(m.f) < 20 &&
            Math.abs(m.a - 1) < 0.001
        ) {
            return;
        }

        if (width >= 16 && height >= 16) {
            worldMatrix = copyMatrix(m);
        }
    }

    // ============================================================
    // TILE/BLOCK DATABASE
    // ============================================================

    function storeBlock(name, x, y, width, height, extra = {}) {
        const centerX = x + width / 2;
        const centerY = y + height / 2;

        const tileX =
            Math.floor(centerX / TILE_SIZE);

        const tileY =
            Math.floor(centerY / TILE_SIZE);

        const wrappedX =
            mod(tileX, WORLD_WRAP_TILES);

        const wrappedY =
            mod(tileY, WORLD_WRAP_TILES);

        const key =
            wrappedX + ',' + wrappedY;

        blockPositions[key] = {
            tileX: wrappedX,
            tileY: wrappedY,

            worldTileX: tileX,
            worldTileY: tileY,

            x,
            y,
            width,
            height,

            name,

            lastSeen:
                performance.now(),

            ...extra
        };
    }

    // ============================================================
    // DRAW CAPTURE
    // ============================================================

    function captureDraw(ctx, image, args) {
        const canvas = ctx.canvas;
        const id = canvas?.id || '';

        let sx = 0;
        let sy = 0;

        let sw =
            image?.width ||
            image?.videoWidth ||
            0;

        let sh =
            image?.height ||
            image?.videoHeight ||
            0;

        let dx = 0;
        let dy = 0;

        let dw = sw;
        let dh = sh;

        if (args.length === 2) {
            dx = args[0];
            dy = args[1];
        }
        else if (args.length === 4) {
            dx = args[0];
            dy = args[1];
            dw = args[2];
            dh = args[3];
        }
        else if (args.length === 8) {
            sx = args[0];
            sy = args[1];
            sw = args[2];
            sh = args[3];
            dx = args[4];
            dy = args[5];
            dw = args[6];
            dh = args[7];
        }
        else {
            return;
        }

        if (
            !Number.isFinite(dx) ||
            !Number.isFinite(dy) ||
            !Number.isFinite(dw) ||
            !Number.isFinite(dh)
        ) return;

        let identified = null;

        if (args.length === 8) {
            const realTile = identifyRealTile(image, sx, sy, sw, sh);

            if (realTile) {
                identified = {
                    name: realTile.name,
                    frame: realTile.index,
                    tile: true,
                    group: realTile.group
                };

                // gj() paints one real tile into a tiny temp canvas before
                // createPattern(). Keep the name attached to that canvas.
                if (!isGameLayer(canvas)) {
                    lastTileDrawByCanvas.set(canvas, {
                        name: realTile.name,
                        frame: realTile.index,
                        group: realTile.group,
                        sx,
                        sy,
                        sw,
                        sh
                    });
                }
            }

            if (!identified) {
                identified = identifySprite(image, sx, sy, sw, sh);
            }
        }

        // Offscreen chunk canvases are expanded later when layer0 draws them.
        if (!isGameLayer(canvas)) return;

        const matrix = copyMatrix(ctx.getTransform());

        considerWorldMatrix(
            ctx,
            Math.abs(dw),
            Math.abs(dh)
        );

        // The terrain renderer ij() draws a whole cached 8x8 chunk canvas to
        // layer0. Expand that chunk back into its individual named tile cells.
        if (
            id === 'layer0' &&
            image instanceof HTMLCanvasElement
        ) {
            const cells = offscreenTileCells.get(image);

            if (cells && cells.size) {
                const sourceX = args.length === 8 ? sx : 0;
                const sourceY = args.length === 8 ? sy : 0;
                const sourceW = args.length === 8 ? sw : image.width;
                const sourceH = args.length === 8 ? sh : image.height;

                const scaleX = sourceW ? dw / sourceW : 1;
                const scaleY = sourceH ? dh / sourceH : 1;

                for (const cell of cells.values()) {
                    if (
                        cell.x + cell.w <= sourceX ||
                        cell.y + cell.h <= sourceY ||
                        cell.x >= sourceX + sourceW ||
                        cell.y >= sourceY + sourceH
                    ) continue;

                    const tileDX = dx + (cell.x - sourceX) * scaleX;
                    const tileDY = dy + (cell.y - sourceY) * scaleY;
                    const tileDW = cell.w * scaleX;
                    const tileDH = cell.h * scaleY;

                    const record = {
                        layer: 'layer0',
                        image,
                        sx: cell.sx,
                        sy: cell.sy,
                        sw: cell.sw,
                        sh: cell.sh,
                        dx: tileDX,
                        dy: tileDY,
                        dw: tileDW,
                        dh: tileDH,
                        matrix: copyMatrix(matrix),
                        name: cell.name,
                        frame: cell.frame,
                        tile: true,
                        group: cell.group,
                        chunkTile: true
                    };

                    frameDraws.layer0.push(record);

                    storeBlock(
                        cell.name,
                        tileDX,
                        tileDY,
                        tileDW,
                        tileDH,
                        {
                            group: cell.group,
                            frame: cell.frame,
                            chunkTile: true
                        }
                    );
                }

                // Do not add a useless label for the entire chunk canvas.
                return;
            }
        }

        let filename = '';

        try {
            if (image && typeof image.src === 'string') {
                filename = image.src.split('/').pop().split('?')[0];
            }
        } catch {}

        const record = {
            layer: id,
            image,
            sx,
            sy,
            sw,
            sh,
            dx,
            dy,
            dw,
            dh,
            matrix,
            name:
                identified?.name ||
                (
                    /^(?:atlas\d*|items|tileset(?:-winter|-moon)?)\.png$/i.test(filename)
                        ? 'sprite'
                        : filename
                ) ||
                'image',
            frame: identified?.frame ?? null,
            tile: !!identified?.tile
        };

        frameDraws[id].push(record);

        if (
            id === 'layer0' &&
            identified?.tile &&
            Math.abs(dw) >= 16 &&
            Math.abs(dw) <= 64 &&
            Math.abs(dh) >= 16 &&
            Math.abs(dh) <= 64
        ) {
            storeBlock(
                record.name,
                dx,
                dy,
                dw,
                dh,
                {
                    group: identified.group,
                    frame: identified.frame,
                    directTileDraw: true
                }
            );
        }
    }

    // ============================================================
    // PATTERN-TILE CAPTURE
    // ============================================================

    function getOffscreenCellMap(canvas) {
        let map = offscreenTileCells.get(canvas);
        if (!map) {
            map = new Map();
            offscreenTileCells.set(canvas, map);
        }
        return map;
    }

    function removeOffscreenCellsInRect(canvas, x, y, w, h) {
        const map = offscreenTileCells.get(canvas);
        if (!map || !map.size) return;

        const left = Math.min(x, x + w);
        const right = Math.max(x, x + w);
        const top = Math.min(y, y + h);
        const bottom = Math.max(y, y + h);

        for (const [key, cell] of map) {
            const cLeft = Math.min(cell.x, cell.x + cell.w);
            const cRight = Math.max(cell.x, cell.x + cell.w);
            const cTop = Math.min(cell.y, cell.y + cell.h);
            const cBottom = Math.max(cell.y, cell.y + cell.h);

            if (
                cRight > left &&
                cLeft < right &&
                cBottom > top &&
                cTop < bottom
            ) {
                map.delete(key);
            }
        }
    }

    function recordPatternFill(ctx, x, y, w, h) {
        let fillStyle;

        try {
            fillStyle = ctx.fillStyle;
        } catch {
            return;
        }

        if (!fillStyle || typeof fillStyle !== 'object') return;

        const tile = patternTileMeta.get(fillStyle);
        if (!tile) return;

        const canvas = ctx.canvas;
        const id = canvas?.id || '';

        // This is the key terrain path: ij() renders the 8x8 world chunk into
        // an offscreen canvas using tile CanvasPatterns. Save each local cell.
        if (id !== 'layer0' && id !== 'layer1') {
            const map = getOffscreenCellMap(canvas);

            const signX = w < 0 ? -1 : 1;
            const signY = h < 0 ? -1 : 1;
            const absW = Math.abs(w);
            const absH = Math.abs(h);

            const cols = Math.max(1, Math.ceil(absW / TILE_SIZE));
            const rows = Math.max(1, Math.ceil(absH / TILE_SIZE));

            for (let cx = 0; cx < cols; cx++) {
                for (let cy = 0; cy < rows; cy++) {
                    const cellX = x + cx * TILE_SIZE * signX;
                    const cellY = y + cy * TILE_SIZE * signY;
                    const cellW = Math.min(TILE_SIZE, absW - cx * TILE_SIZE) * signX;
                    const cellH = Math.min(TILE_SIZE, absH - cy * TILE_SIZE) * signY;

                    const key = `${Math.round(cellX)},${Math.round(cellY)}`;

                    map.set(key, {
                        x: cellX,
                        y: cellY,
                        w: cellW,
                        h: cellH,
                        name: tile.name,
                        frame: tile.frame,
                        group: tile.group,
                        sx: tile.sx,
                        sy: tile.sy,
                        sw: tile.sw,
                        sh: tile.sh
                    });
                }
            }

            return;
        }

        // Direct world pattern fills are uncommon but still supported.
        if (id !== 'layer0') return;

        const matrix = copyMatrix(ctx.getTransform());
        considerWorldMatrix(ctx, Math.abs(w), Math.abs(h));

        const signX = w < 0 ? -1 : 1;
        const signY = h < 0 ? -1 : 1;
        const absW = Math.abs(w);
        const absH = Math.abs(h);
        const cols = Math.max(1, Math.ceil(absW / TILE_SIZE));
        const rows = Math.max(1, Math.ceil(absH / TILE_SIZE));

        for (let cx = 0; cx < cols; cx++) {
            for (let cy = 0; cy < rows; cy++) {
                const cellX = x + cx * TILE_SIZE * signX;
                const cellY = y + cy * TILE_SIZE * signY;
                const cellW = Math.min(TILE_SIZE, absW - cx * TILE_SIZE) * signX;
                const cellH = Math.min(TILE_SIZE, absH - cy * TILE_SIZE) * signY;

                const record = {
                    layer: 'layer0',
                    image: null,
                    sx: tile.sx,
                    sy: tile.sy,
                    sw: tile.sw,
                    sh: tile.sh,
                    dx: cellX,
                    dy: cellY,
                    dw: cellW,
                    dh: cellH,
                    matrix: copyMatrix(matrix),
                    name: tile.name,
                    frame: tile.frame,
                    tile: true,
                    group: tile.group,
                    patternTile: true
                };

                frameDraws.layer0.push(record);
                storeBlock(tile.name, cellX, cellY, cellW, cellH, {
                    group: tile.group,
                    frame: tile.frame,
                    patternTile: true
                });
            }
        }
    }

    // ============================================================
    // CANVAS PROTOTYPE HOOKS
    // ============================================================

    const canvasProto =
        CanvasRenderingContext2D.prototype;

    const originalDrawImage =
        canvasProto.drawImage;

    const originalClearRect =
        canvasProto.clearRect;

    const originalCreatePattern =
        canvasProto.createPattern;

    const originalFillRect =
        canvasProto.fillRect;

    canvasProto.drawImage =
        function(image, ...args) {

            try {
                captureDraw(
                    this,
                    image,
                    args
                );
            } catch (error) {
                console.error(
                    '[GOB Overlay] draw capture error',
                    error
                );
            }

            return originalDrawImage.apply(
                this,
                [
                    image,
                    ...args
                ]
            );
        };

    canvasProto.createPattern =
        function(image, repetition) {

            const pattern =
                originalCreatePattern.call(
                    this,
                    image,
                    repetition
                );

            try {
                if (
                    pattern &&
                    image instanceof HTMLCanvasElement
                ) {
                    const tile =
                        lastTileDrawByCanvas.get(
                            image
                        );

                    if (tile) {
                        patternTileMeta.set(
                            pattern,
                            { ...tile }
                        );
                    }
                }
            } catch {}

            return pattern;
        };

    canvasProto.fillRect =
        function(x, y, w, h) {

            try {
                recordPatternFill(
                    this,
                    x,
                    y,
                    w,
                    h
                );
            } catch (error) {
                console.error(
                    '[GOB Overlay] pattern capture error',
                    error
                );
            }

            return originalFillRect.call(
                this,
                x,
                y,
                w,
                h
            );
        };

    canvasProto.clearRect =
        function(x, y, w, h) {

            const canvas = this.canvas;
            const id = canvas?.id || '';

            if (
                (
                    id === 'layer0' ||
                    id === 'layer1'
                ) &&
                x === 0 &&
                y === 0 &&
                w >= canvas.width - 2 &&
                h >= canvas.height - 2
            ) {
                frameDraws[id].length = 0;
            }
            else if (
                canvas instanceof HTMLCanvasElement &&
                id !== 'layer0' &&
                id !== 'layer1'
            ) {
                removeOffscreenCellsInRect(canvas, x, y, w, h);
            }

            return originalClearRect.call(
                this,
                x,
                y,
                w,
                h
            );
        };

    // ============================================================
    // OVERLAY CREATION / ALIGNMENT
    // ============================================================

    function findLayers() {
        layer0 =
            document.getElementById(
                'layer0'
            );

        layer1 =
            document.getElementById(
                'layer1'
            );

        return !!(
            layer0 &&
            layer1
        );
    }

    function createOverlay() {
        if (!findLayers()) {
            return false;
        }

        if (
            overlay &&
            overlay.isConnected
        ) {
            return true;
        }

        overlay =
            document.createElement(
                'canvas'
            );

        overlay.id =
            'gob-world-debug-overlay';

        overlay.style.cssText = `
            position: fixed;
            z-index: 2147483000;
            pointer-events: none;
            background: transparent;
        `;

        document.body.appendChild(
            overlay
        );

        overlayCtx =
            overlay.getContext('2d');

        syncOverlay();

        console.log(
            '[GOB Overlay] overlay created'
        );

        return true;
    }

    function syncOverlay() {
        if (
            !overlay ||
            !layer1
        ) {
            return;
        }

        const rect =
            layer1.getBoundingClientRect();

        overlay.style.left =
            rect.left + 'px';

        overlay.style.top =
            rect.top + 'px';

        overlay.style.width =
            rect.width + 'px';

        overlay.style.height =
            rect.height + 'px';

        if (
            overlay.width !==
            layer1.width
        ) {
            overlay.width =
                layer1.width;
        }

        if (
            overlay.height !==
            layer1.height
        ) {
            overlay.height =
                layer1.height;
        }
    }

    // ============================================================
    // WORLD GRID
    // ============================================================

    function drawWorldGrid() {
        if (
            !SETTINGS.grid ||
            !worldMatrix ||
            !overlayCtx ||
            !overlay
        ) {
            return;
        }

        const ctx =
            overlayCtx;

        const width =
            overlay.width;

        const height =
            overlay.height;

        const p1 =
            inversePoint(
                worldMatrix,
                0,
                0
            );

        const p2 =
            inversePoint(
                worldMatrix,
                width,
                height
            );

        const worldLeft =
            Math.min(
                p1.x,
                p2.x
            );

        const worldRight =
            Math.max(
                p1.x,
                p2.x
            );

        const worldTop =
            Math.min(
                p1.y,
                p2.y
            );

        const worldBottom =
            Math.max(
                p1.y,
                p2.y
            );

        const startTileX =
            Math.floor(
                worldLeft /
                TILE_SIZE
            ) - 1;

        const endTileX =
            Math.ceil(
                worldRight /
                TILE_SIZE
            ) + 1;

        const startTileY =
            Math.floor(
                worldTop /
                TILE_SIZE
            ) - 1;

        const endTileY =
            Math.ceil(
                worldBottom /
                TILE_SIZE
            ) + 1;

        ctx.save();

        ctx.strokeStyle =
            'rgba(255,0,0,0.50)';

        ctx.lineWidth = 1;

        for (
            let tileX = startTileX;
            tileX <= endTileX;
            tileX++
        ) {
            const worldX =
                tileX *
                TILE_SIZE;

            const top =
                transformPoint(
                    worldMatrix,
                    worldX,
                    worldTop -
                        TILE_SIZE
                );

            const bottom =
                transformPoint(
                    worldMatrix,
                    worldX,
                    worldBottom +
                        TILE_SIZE
                );

            ctx.beginPath();

            ctx.moveTo(
                top.x,
                top.y
            );

            ctx.lineTo(
                bottom.x,
                bottom.y
            );

            ctx.stroke();
        }

        for (
            let tileY = startTileY;
            tileY <= endTileY;
            tileY++
        ) {
            const worldY =
                tileY *
                TILE_SIZE;

            const left =
                transformPoint(
                    worldMatrix,
                    worldLeft -
                        TILE_SIZE,
                    worldY
                );

            const right =
                transformPoint(
                    worldMatrix,
                    worldRight +
                        TILE_SIZE,
                    worldY
                );

            ctx.beginPath();

            ctx.moveTo(
                left.x,
                left.y
            );

            ctx.lineTo(
                right.x,
                right.y
            );

            ctx.stroke();
        }

        if (SETTINGS.coords) {
            ctx.font =
                'bold 10px monospace';

            ctx.textBaseline =
                'top';

            ctx.fillStyle =
                'rgba(255,255,0,0.95)';

            ctx.strokeStyle =
                'rgba(0,0,0,0.85)';

            ctx.lineWidth = 3;

            for (
                let tileX = startTileX;
                tileX <= endTileX;
                tileX++
            ) {
                for (
                    let tileY = startTileY;
                    tileY <= endTileY;
                    tileY++
                ) {
                    const wrappedX =
                        mod(
                            tileX,
                            WORLD_WRAP_TILES
                        );

                    const wrappedY =
                        mod(
                            tileY,
                            WORLD_WRAP_TILES
                        );

                    const point =
                        transformPoint(
                            worldMatrix,

                            tileX *
                                TILE_SIZE +
                                2,

                            tileY *
                                TILE_SIZE +
                                2
                        );

                    const text =
                        wrappedX +
                        ',' +
                        wrappedY;

                    ctx.strokeText(
                        text,
                        point.x,
                        point.y
                    );

                    ctx.fillText(
                        text,
                        point.x,
                        point.y
                    );
                }
            }
        }

        ctx.restore();
    }

    // ============================================================
    // LABELS
    // ============================================================

    function drawLabel(draw) {
        if (
            !SETTINGS.labels ||
            !overlayCtx
        ) {
            return;
        }

        const terrain =
            draw.tile ||
            draw.patternTile ||
            isTerrainName(draw.name);

        if (
            terrain &&
            !SETTINGS.tileLabels
        ) {
            return;
        }

        if (
            !terrain &&
            !SETTINGS.spriteLabels
        ) {
            return;
        }

        if (
            Math.abs(draw.dw) <
                SETTINGS.minimumSize ||
            Math.abs(draw.dh) <
                SETTINGS.minimumSize
        ) {
            return;
        }

        const m =
            draw.matrix;

        const center =
            transformPoint(
                m,

                draw.dx +
                    draw.dw / 2,

                draw.dy +
                    draw.dh / 2
            );

        const corner1 =
            transformPoint(
                m,
                draw.dx,
                draw.dy
            );

        const corner2 =
            transformPoint(
                m,

                draw.dx +
                    draw.dw,

                draw.dy +
                    draw.dh
            );

        const boxX =
            Math.min(
                corner1.x,
                corner2.x
            );

        const boxY =
            Math.min(
                corner1.y,
                corner2.y
            );

        const boxW =
            Math.abs(
                corner2.x -
                corner1.x
            );

        const boxH =
            Math.abs(
                corner2.y -
                corner1.y
            );

        if (
            center.x < -100 ||
            center.y < -100 ||
            center.x >
                overlay.width + 100 ||
            center.y >
                overlay.height + 100
        ) {
            return;
        }

        const ctx =
            overlayCtx;

        ctx.save();

        if (SETTINGS.boxes) {
            ctx.strokeStyle =
                terrain
                    ? 'rgba(255,220,0,0.85)'
                    : 'rgba(0,255,255,0.75)';

            ctx.lineWidth = 1;

            ctx.strokeRect(
                boxX,
                boxY,
                boxW,
                boxH
            );
        }

        const text =
            cleanDisplayName(draw.name) || 'unknown';

        ctx.font =
            terrain
                ? 'bold 9px monospace'
                : 'bold 11px monospace';

        ctx.textAlign =
            'center';

        ctx.textBaseline =
            'bottom';

        const metrics =
            ctx.measureText(text);

        const padding = 2;

        const textWidth =
            metrics.width +
            padding * 2;

        const textHeight =
            terrain ? 11 : 13;

        const tx =
            center.x;

        // Terrain label inside the tile so it doesn't fight with
        // coordinates at the top-left.
        const ty =
            terrain
                ? center.y + textHeight / 2
                : boxY - 2;

        ctx.fillStyle =
            'rgba(0,0,0,0.72)';

        ctx.fillRect(
            tx -
                textWidth / 2,

            ty -
                textHeight,

            textWidth,
            textHeight
        );

        ctx.fillStyle =
            terrain
                ? '#ffff66'
                : '#ffffff';

        ctx.fillText(
            text,
            tx,
            ty - 1
        );

        ctx.restore();
    }

    // ============================================================
    // OVERLAY FRAME
    // ============================================================

    function renderOverlay() {
        requestAnimationFrame(
            renderOverlay
        );

        if (!createOverlay()) {
            return;
        }

        syncOverlay();

        if (
            !overlayCtx ||
            !overlay
        ) {
            return;
        }

        overlayCtx.setTransform(
            1,
            0,
            0,
            1,
            0,
            0
        );

        overlayCtx.clearRect(
            0,
            0,
            overlay.width,
            overlay.height
        );

        drawWorldGrid();

        /*
            IMPORTANT:
            Draw moving/object sprites FIRST so terrain labels can never
            consume the entire label budget.

            At zoomed-out views, layer0 may contain hundreds/thousands of
            terrain records. Previously those exhausted maxLabels before
            layer1 (bombs, ball, wick, players, etc.) was reached.
        */

        let spriteLabels = 0;
        let tileLabels = 0;

        // --------------------------------------------------------
        // NORMAL OBJECTS / SPRITES FIRST
        // --------------------------------------------------------

        for (const draw of frameDraws.layer1) {
            if (
                spriteLabels >=
                SETTINGS.maxSpriteLabels
            ) {
                break;
            }

            drawLabel(draw);
            spriteLabels++;
        }

        // Some non-terrain objects can also be drawn on layer0.
        for (const draw of frameDraws.layer0) {
            const terrain =
                draw.tile ||
                draw.patternTile ||
                isTerrainName(draw.name);

            if (terrain) {
                continue;
            }

            if (
                spriteLabels >=
                SETTINGS.maxSpriteLabels
            ) {
                break;
            }

            drawLabel(draw);
            spriteLabels++;
        }

        // --------------------------------------------------------
        // TERRAIN AFTER OBJECTS
        // --------------------------------------------------------

        for (const draw of frameDraws.layer0) {
            const terrain =
                draw.tile ||
                draw.patternTile ||
                isTerrainName(draw.name);

            if (!terrain) {
                continue;
            }

            if (
                tileLabels >=
                SETTINGS.maxTileLabels
            ) {
                break;
            }

            drawLabel(draw);
            tileLabels++;
        }
    }

    // ============================================================
    // COMMANDS
    // ============================================================

    window.toggleWorldGrid =
        function() {
            SETTINGS.grid =
                !SETTINGS.grid;

            console.log(
                'World grid:',
                SETTINGS.grid
                    ? 'ON'
                    : 'OFF'
            );
        };

    window.toggleWorldCoords =
        function() {
            SETTINGS.coords =
                !SETTINGS.coords;

            console.log(
                'Coordinates:',
                SETTINGS.coords
                    ? 'ON'
                    : 'OFF'
            );
        };

    window.toggleSpriteLabels =
        function() {
            SETTINGS.spriteLabels =
                !SETTINGS.spriteLabels;

            console.log(
                'Sprite labels:',
                SETTINGS.spriteLabels
                    ? 'ON'
                    : 'OFF'
            );
        };

    window.toggleTileLabels =
        function() {
            SETTINGS.tileLabels =
                !SETTINGS.tileLabels;

            console.log(
                'Tile labels:',
                SETTINGS.tileLabels
                    ? 'ON'
                    : 'OFF'
            );
        };

    window.toggleAllLabels =
        function() {
            SETTINGS.labels =
                !SETTINGS.labels;

            console.log(
                'All labels:',
                SETTINGS.labels
                    ? 'ON'
                    : 'OFF'
            );
        };

    window.toggleObjectBoxes =
        function() {
            SETTINGS.boxes =
                !SETTINGS.boxes;

            console.log(
                'Object boxes:',
                SETTINGS.boxes
                    ? 'ON'
                    : 'OFF'
            );
        };

    window.getBlockAt =
        function(tileX, tileY) {

            const wrappedX =
                mod(
                    tileX,
                    WORLD_WRAP_TILES
                );

            const wrappedY =
                mod(
                    tileY,
                    WORLD_WRAP_TILES
                );

            return (
                blockPositions[
                    wrappedX +
                    ',' +
                    wrappedY
                ] ||
                null
            );
        };

    window.getAllBlocks =
        function() {
            return Object.values(
                blockPositions
            );
        };

    window.getBlocksInArea =
        function(
            startX,
            startY,
            endX,
            endY
        ) {
            const output = [];

            for (
                let x = startX;
                x <= endX;
                x++
            ) {
                for (
                    let y = startY;
                    y <= endY;
                    y++
                ) {
                    const block =
                        window.getBlockAt(
                            x,
                            y
                        );

                    if (block) {
                        output.push(
                            block
                        );
                    }
                }
            }

            return output;
        };

    window.getLabelCounts =
        function() {
            const layer0Terrain =
                frameDraws.layer0.filter(
                    draw =>
                        draw.tile ||
                        draw.patternTile ||
                        isTerrainName(draw.name)
                ).length;

            const layer0Objects =
                frameDraws.layer0.length -
                layer0Terrain;

            const layer1Objects =
                frameDraws.layer1.length;

            const result = {
                layer0Terrain,
                layer0Objects,
                layer1Objects,
                maxTileLabels:
                    SETTINGS.maxTileLabels,
                maxSpriteLabels:
                    SETTINGS.maxSpriteLabels
            };

            console.table(result);

            return result;
        };

    window.getCurrentDraws =
        function() {
            return {
                layer0:
                    frameDraws.layer0
                        .slice(),

                layer1:
                    frameDraws.layer1
                        .slice()
            };
        };

    window.debugSpriteTracker =
        function() {
            const names =
                runtimeSprites.map(
                    r => r.name
                );

            const wanted = [
                'bomb',
                'bomb64x64',
                'ball',
                'wick',
                'mine',
                'snowball',
                'power',
                'atomic',
                'shield'
            ];

            const found = {};

            for (const name of wanted) {
                found[name] =
                    names.includes(name);
            }

            console.table(found);

            console.log(
                '[GOB Overlay] runtime sprite records:',
                runtimeSprites.length
            );

            console.log(
                '[GOB Overlay] examples:',
                names.slice(0, 100)
            );

            return {
                found,
                count:
                    runtimeSprites.length,
                names
            };
        };

    window.getTrackedSprites =
        function() {
            return runtimeSprites.map(
                record => ({
                    name:
                        record.name,

                    frameWidth:
                        record.frameWidth,

                    frameHeight:
                        record.frameHeight
                })
            );
        };

    window.getTrackedTiles =
        function() {
            return realTileFrames.map(
                tile => ({
                    group:
                        tile.group,

                    index:
                        tile.index,

                    name:
                        tile.name,

                    x:
                        tile.x,

                    y:
                        tile.y,

                    width:
                        tile.width,

                    height:
                        tile.height
                })
            );
        };

    // ============================================================
    // EARLY SPRITE HOOK
    // ============================================================

    /*
        IMPORTANT:
        This starts immediately at document-start, BEFORE DOMContentLoaded.

        loadSprites() normally runs very early. Waiting until DOMContentLoaded
        misses the sprites.add(...) calls and causes normal objects to fall back
        to atlas filenames.
    */

    const earlySpriteHookInterval =
        setInterval(() => {
            try {
                if (hookSpriteManager()) {
                    clearInterval(
                        earlySpriteHookInterval
                    );

                    // Populate direct sprites/groups immediately.
                    rebuildSpriteIndex();

                    // Textures/children can finish resolving slightly later.
                    setTimeout(rebuildSpriteIndex, 50);
                    setTimeout(rebuildSpriteIndex, 250);
                    setTimeout(rebuildSpriteIndex, 1000);
                }
            } catch {}
        }, 1);

    // ============================================================
    // START
    // ============================================================

    // Install the sprite hook immediately at document-start. Waiting for
    // DOMContentLoaded is too late: loadSprites() may already have run.
    const spriteHookInterval =
        setInterval(() => {
            if (hookSpriteManager()) {
                clearInterval(spriteHookInterval);
                try { rebuildSpriteIndex(); } catch {}
            }
        }, 10);

    try { hookSpriteManager(); } catch {}

    function start() {
        // Rebuild periodically because some assets/textures load later.
        setInterval(() => {
            try {
                rebuildSpriteIndex();
            } catch {}
        }, 3000);

        // Game canvases can be recreated between rooms.
        setInterval(() => {
            findLayers();

            if (
                !overlay ||
                !overlay.isConnected
            ) {
                createOverlay();
            }
        }, 500);

        requestAnimationFrame(
            renderOverlay
        );

        console.log(
            '[GOB Overlay] World grid + real tile tracker active'
        );

        console.log(
            'Commands:'
        );

        console.log(
            'toggleWorldGrid()'
        );

        console.log(
            'toggleWorldCoords()'
        );

        console.log(
            'toggleSpriteLabels()'
        );

        console.log(
            'toggleTileLabels()'
        );

        console.log(
            'toggleAllLabels()'
        );

        console.log(
            'toggleObjectBoxes()'
        );

        console.log(
            'getBlockAt(x,y)'
        );

        console.log(
            'getAllBlocks()'
        );

        console.log(
            'getBlocksInArea(x1,y1,x2,y2)'
        );

        console.log(
            'getCurrentDraws()'
        );

        console.log(
            'getTrackedSprites()'
        );

        console.log(
            'debugSpriteTracker()'
        );

        console.log(
            'getTrackedTiles()'
        );
    }

    if (
        document.readyState ===
        'loading'
    ) {
        document.addEventListener(
            'DOMContentLoaded',
            start,
            {
                once: true
            }
        );
    }

    else {
        start();
    }

})();
