// ==UserScript==
// @name         Game of Bombs - Intelligent Tile Bot
// @namespace    http://tampermonkey.net/
// @version      5.2.0
// @description  Source-driven bot that traces only the active on-screen world, keeps separate worlds separate, predicts full-fuse fire, and retains stable movement/anti-idle planning.
// @match        https://gameofbombs.com/*
// @match        http://gameofbombs.com/*
// @run-at       document-start
// @grant        none
// ==/UserScript==

(() => {
    'use strict';

    /*
        v5 EARLY BombModel tap
        ======================

        The compiled BombModel prototype is initialized in this exact order:

            _.He = ...
            _.xb = function() { return 'Bomb{ strength=' + this.s ... }
            _.q = 0
            _.r = 0
            _.s = 0
            _.t = 0

        The previous v4.x attempt waited for `.s = 0` itself. That proved too
        brittle in the live page. v5 catches the EARLIER `.q = 0` assignment,
        identifies the prototype by its already-installed xb()/He methods,
        and installs the `s` accessor before the next `_.s = 0` line executes.

        After BombModel is found, Object.prototype.q is immediately restored.
        All unrelated `.q = ...` writes before that point are converted into
        normal own data properties, so they keep normal JS behavior.
    */
    (function installBombModelTapV5() {
        const VERSION =
            '5.2.0';

        /*
            If an older userscript is still enabled, stop its active controller.
            Its animation frame may continue to exist, but disabled old bots do
            not send movement.
        */
        try {
            if (
                typeof window.gobBotStop ===
                    'function'
            ) {
                window.gobBotStop();
            }
        }
        catch {}

        window.__GOB_ACTIVE_BOT_VERSION__ =
            VERSION;

        const state = {
            version:
                VERSION,
            installedAt:
                performance.now(),
            prototypeCaptured:
                false,
            capturedAt:
                0,
            records:
                [],
            models:
                new Set(),
            lastError:
                null
        };

        window.__GOB_BOMB_MODEL_TAP_V5__ =
            state;

        /*
            Clean up the old v4.x temporary Object.prototype.s hook if another
            copy left it behind and it is configurable.
        */
        try {
            const staleS =
                Object.getOwnPropertyDescriptor(
                    Object.prototype,
                    's'
                );

            if (
                staleS &&
                staleS.configurable &&
                window.__GOB_BOMB_STRENGTH_HOOK_STATE__
            ) {
                delete Object.prototype.s;
            }
        }
        catch {}

        const existingQ =
            Object.getOwnPropertyDescriptor(
                Object.prototype,
                'q'
            );

        if (existingQ) {
            state.lastError =
                'Object.prototype already owns q; early prototype tap not installed';

            return;
        }

        const fallbackQ =
            new WeakMap();

        function recordModel(
            model,
            strength
        ) {
            try {
                const value =
                    Number(
                        strength
                    );

                if (
                    !Number.isFinite(value) ||
                    value <= 0
                ) {
                    return;
                }

                state.models.add(
                    model
                );

                let record =
                    null;

                for (
                    let i =
                        state.records.length - 1;
                    i >= 0;
                    i--
                ) {
                    if (
                        state.records[i].model ===
                            model
                    ) {
                        record =
                            state.records[i];

                        break;
                    }
                }

                if (!record) {
                    record = {
                        model,
                        firstCapturedAt:
                            performance.now(),
                        lastCapturedAt:
                            performance.now(),
                        strength:
                            value
                    };

                    state.records.push(
                        record
                    );
                }
                else {
                    record.lastCapturedAt =
                        performance.now();

                    record.strength =
                        value;
                }

                if (
                    state.records.length >
                        160
                ) {
                    state.records.splice(
                        0,
                        state.records.length -
                            160
                    );
                }
            }
            catch (error) {
                state.lastError =
                    String(
                        error?.message ||
                        error
                    );
            }
        }

        try {
            Object.defineProperty(
                Object.prototype,
                'q',
                {
                    configurable:
                        true,
                    enumerable:
                        false,

                    get() {
                        return fallbackQ.has(
                            this
                        )
                            ? fallbackQ.get(
                                this
                            )
                            : undefined;
                    },

                    set(value) {
                        let isBombPrototype =
                            false;

                        try {
                            isBombPrototype =
                                value === 0 &&
                                typeof this?.xb ===
                                    'function' &&
                                typeof this?.He ===
                                    'function' &&
                                Function.prototype
                                    .toString
                                    .call(
                                        this.xb
                                    )
                                    .includes(
                                        'Bomb{ strength='
                                    );
                        }
                        catch {}

                        if (
                            isBombPrototype
                        ) {
                            const bombPrototype =
                                this;

                            const strengths =
                                new WeakMap();

                            /*
                                q itself must remain a normal own data property
                                on BombModel.prototype after the discovery.
                            */
                            Object.defineProperty(
                                bombPrototype,
                                'q',
                                {
                                    configurable:
                                        true,
                                    enumerable:
                                        true,
                                    writable:
                                        true,
                                    value
                                }
                            );

                            /*
                                Install before the compiled code reaches
                                `_.s = 0`.
                            */
                            Object.defineProperty(
                                bombPrototype,
                                's',
                                {
                                    configurable:
                                        true,
                                    enumerable:
                                        true,

                                    get() {
                                        return strengths.has(
                                            this
                                        )
                                            ? strengths.get(
                                                this
                                            )
                                            : 0;
                                    },

                                    set(nextValue) {
                                        strengths.set(
                                            this,
                                            nextValue
                                        );

                                        if (
                                            this !==
                                                bombPrototype
                                        ) {
                                            recordModel(
                                                this,
                                                nextValue
                                            );
                                        }
                                    }
                                }
                            );

                            state.prototypeCaptured =
                                true;

                            state.capturedAt =
                                performance.now();

                            try {
                                delete Object.prototype.q;
                            }
                            catch {}

                            return;
                        }

                        /*
                            Normal semantics for unrelated objects.
                        */
                        Object.defineProperty(
                            this,
                            'q',
                            {
                                configurable:
                                    true,
                                enumerable:
                                    true,
                                writable:
                                    true,
                                value
                            }
                        );
                    }
                }
            );
        }
        catch (error) {
            state.lastError =
                String(
                    error?.message ||
                    error
                );
        }
    })();

    if (
        window.__GOB_SOURCE_DRIVEN_BOT_V5__ ===
            '5.2.0'
    ) {
        return;
    }

    window.__GOB_SOURCE_DRIVEN_BOT_V5__ =
        '5.2.0';

    // ============================================================
    // SETTINGS
    // ============================================================

    const TILE_SIZE = 32;
    const WORLD_WRAP_TILES = 32;

    const SETTINGS = {
        grid: false,
        coords: false,
        labels: false,
        tileLabels: false,
        spriteLabels: false,
        boxes: false,
        minimumSize: 8,
        maxSpriteLabels: 0,
        maxTileLabels: 0
    };

    // ============================================================
    // STATE
    // ============================================================

    let layer0 = null;
    let layer1 = null;

    let overlay = null;
    let overlayCtx = null;

    let worldMatrix = null;
    let worldMatrixSeenAt = 0;
    let worldMatrixLayer = null;
    let worldMatrixSource = null;
    let lastTerrainMatrix = null;
    let lastTerrainMatrixSeenAt = 0;

    const frameDraws = {
        layer0: [],
        layer1: []
    };

    // Legacy frame list retained for compatibility/debug commands.
    const framePickups = [];

    // Exact gameplay observations. These are populated from known atlas source
    // rectangles rather than generic sprite-name guesses.
    const exactWorldObservations = {
        pickups: new Map(),
        bombs: new Map(),
        blasts: new Map()
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

    let spriteRectCache = new WeakMap();

    function spriteRectKey(sx, sy, sw, sh) {
        return (
            Math.round(sx * 10) + ':' +
            Math.round(sy * 10) + ':' +
            Math.round(sw * 10) + ':' +
            Math.round(sh * 10)
        );
    }

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

    function isCanvasActivelyVisible(
        canvas
    ) {
        if (
            !canvas ||
            !canvas.isConnected
        ) {
            return false;
        }

        let style =
            null;

        try {
            style =
                getComputedStyle(
                    canvas
                );
        }
        catch {}

        if (
            style &&
            (
                style.display ===
                    'none' ||
                style.visibility ===
                    'hidden' ||
                Number(
                    style.opacity
                ) <=
                    0.01
            )
        ) {
            return false;
        }

        const rect =
            canvas.getBoundingClientRect();

        if (
            rect.width <
                8 ||
            rect.height <
                8
        ) {
            return false;
        }

        return (
            rect.right >
                0 &&
            rect.bottom >
                0 &&
            rect.left <
                window.innerWidth &&
            rect.top <
                window.innerHeight
        );
    }

    function transformedDrawBounds(
        matrix,
        x,
        y,
        w,
        h
    ) {
        const points = [
            transformPoint(
                matrix,
                x,
                y
            ),
            transformPoint(
                matrix,
                x + w,
                y
            ),
            transformPoint(
                matrix,
                x,
                y + h
            ),
            transformPoint(
                matrix,
                x + w,
                y + h
            )
        ];

        return {
            left:
                Math.min(
                    ...points.map(
                        p => p.x
                    )
                ),
            right:
                Math.max(
                    ...points.map(
                        p => p.x
                    )
                ),
            top:
                Math.min(
                    ...points.map(
                        p => p.y
                    )
                ),
            bottom:
                Math.max(
                    ...points.map(
                        p => p.y
                    )
                )
        };
    }

    function worldDrawIsActivelyOnScreen(
        ctx,
        x,
        y,
        w,
        h,
        margin =
            32
    ) {
        const canvas =
            ctx?.canvas;

        if (
            !isGameLayer(
                canvas
            ) ||
            !isCanvasActivelyVisible(
                canvas
            )
        ) {
            return false;
        }

        const bounds =
            transformedDrawBounds(
                copyMatrix(
                    ctx.getTransform()
                ),
                x,
                y,
                w,
                h
            );

        return (
            bounds.right >=
                -margin &&
            bounds.bottom >=
                -margin &&
            bounds.left <=
                canvas.width +
                    margin &&
            bounds.top <=
                canvas.height +
                    margin
        );
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
        spriteRectCache = new WeakMap();

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

    function scanSpriteSlow(image, sx, sy, sw, sh) {
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


    function identifySprite(image, sx, sy, sw, sh) {
        if (!image) return null;

        let map = spriteRectCache.get(image);

        if (!map) {
            map = new Map();
            spriteRectCache.set(image, map);
        }

        const key = spriteRectKey(sx, sy, sw, sh);

        if (map.has(key)) {
            return map.get(key);
        }

        const found =
            scanSpriteSlow(
                image,
                sx,
                sy,
                sw,
                sh
            );

        map.set(key, found || null);

        return found;
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



    function considerWorldMatrix(
        ctx,
        width,
        height,
        source =
            'generic'
    ) {
        const id =
            ctx?.canvas?.id ||
            '';

        if (
            id !== 'layer0' &&
            id !== 'layer1'
        ) {
            return;
        }

        if (
            width < 16 ||
            height < 16
        ) {
            return;
        }

        const m =
            ctx.getTransform();

        if (
            !Number.isFinite(m.a) ||
            !Number.isFinite(m.d) ||
            !Number.isFinite(m.e) ||
            !Number.isFinite(m.f)
        ) {
            return;
        }

        if (
            Math.abs(m.b) >
                0.001 ||
            Math.abs(m.c) >
                0.001 ||
            m.a <= 0 ||
            m.d <= 0 ||
            Math.abs(
                Math.abs(m.a) -
                Math.abs(m.d)
            ) >
                0.01
        ) {
            return;
        }

        /*
            IMPORTANT:

            Never take the display matrix from an arbitrary sprite draw.

            Individual game objects can temporarily call:
                save()
                translate(objectX, objectY)
                scale(...)
                drawImage(...)
                restore()

            ctx.getTransform() during that draw therefore contains BOTH the
            camera transform and the object's local transform. v5.0 sometimes
            captured exactly that and used it as the overlay camera, which is
            why fire boxes could shift, scale incorrectly, or jump around.

            v5.1 only accepts matrices explicitly marked as terrain/world-base
            matrices. Terrain chunk draws happen under the plane/camera
            transform but before object-local transforms.
        */
        if (
            source !==
                'terrain-chunk' &&
            source !==
                'terrain-pattern'
        ) {
            return;
        }

        lastTerrainMatrix =
            copyMatrix(m);

        lastTerrainMatrixSeenAt =
            performance.now();

        worldMatrix =
            copyMatrix(m);

        worldMatrixSeenAt =
            lastTerrainMatrixSeenAt;

        worldMatrixLayer =
            id;

        worldMatrixSource =
            source;
    }

    // ============================================================
    // TILE/BLOCK DATABASE
    // ============================================================


    function storeBlock(
        name,
        x,
        y,
        width,
        height,
        extra =
            {}
    ) {
        const centerX =
            x +
            width /
                2;

        const centerY =
            y +
            height /
                2;

        const tileX =
            Math.floor(
                centerX /
                TILE_SIZE
            );

        const tileY =
            Math.floor(
                centerY /
                TILE_SIZE
            );

        blockPositions[
            tileKey(
                tileX,
                tileY
            )
        ] = {
            tileX,
            tileY,
            worldTileX:
                tileX,
            worldTileY:
                tileY,
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
            // Exact known gameplay atlases first. This is intentionally
            // independent of the runtime sprite registry.
            identified =
                identifyDirectGameplayFrame(
                    image,
                    sx,
                    sy,
                    sw,
                    sh
                );

            const realTile =
                !identified
                    ? identifyRealTile(image, sx, sy, sw, sh)
                    : null;

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

        // The terrain renderer ij() draws a whole cached 8x8 chunk canvas to
        // layer0. Expand that chunk back into its individual named tile cells.
        if (
            id === 'layer0' &&
            image instanceof HTMLCanvasElement
        ) {
            const cells = offscreenTileCells.get(image);

            if (cells && cells.size) {
                /*
                    This is the authoritative camera/world matrix. The cached
                    terrain chunk itself is drawn directly in world space, so
                    no bomb/player-local translate/scale contaminates it.
                */
                considerWorldMatrix(
                    ctx,
                    Math.abs(dw),
                    Math.abs(dh),
                    'terrain-chunk'
                );

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

                    if (
                        !worldDrawIsActivelyOnScreen(
                            ctx,
                            tileDX,
                            tileDY,
                            tileDW,
                            tileDH,
                            8
                        )
                    ) {
                        continue;
                    }

                    const record = {
                        layer: 'layer0',
                        onScreen: true,
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

            onScreen:
                worldDrawIsActivelyOnScreen(
                    ctx,
                    dx,
                    dy,
                    dw,
                    dh
                ),
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
            tile: !!identified?.tile,
            kind: identified?.kind || null,
            direct: !!identified?.direct
        };

        // Feed the bot's exact world model before any ordinary draw caps.
        observeExactWorldDraw(
            record,
            filename
        );

        if (
            record.onScreen === true &&
            (id === 'layer0' || id === 'layer1') &&
            isPowerupName(record.name) &&
            Math.abs(dw) >= 18 &&
            Math.abs(dw) <= 48 &&
            Math.abs(dh) >= 18 &&
            Math.abs(dh) <= 48
        ) {
            framePickups.push({
                ...record,
                pickup: true
            });
        }

        const botRelevantName =
            String(record.name || '').toLowerCase();

        const botRelevantFile =
            String(filename || '').toLowerCase();

        const looksLikeBombOrExplosion =
            /bomb|mine|explosion|blast|wick/.test(botRelevantName) ||
            /bomb|mine|explosion|blast|wick/.test(botRelevantFile);

        const looksLikePlayerSprite =
            /default|character|player|flying|slowpoke|jetpack|nyan|private_|skin/.test(botRelevantName) ||
            /character|player|private_|skin|nyan/.test(botRelevantFile);

        if (id === 'layer1') {
            if (frameDraws.layer1.length < 350) {
                frameDraws.layer1.push(record);
            }
        }
        else if (
            id === 'layer0' &&
            (
                looksLikeBombOrExplosion ||
                looksLikePlayerSprite
            )
        ) {
            if (frameDraws.layer0.length < 120) {
                frameDraws.layer0.push(record);
            }
        }

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

        considerWorldMatrix(
            ctx,
            Math.abs(w),
            Math.abs(h),
            'terrain-pattern'
        );

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

                if (
                    !worldDrawIsActivelyOnScreen(
                        ctx,
                        cellX,
                        cellY,
                        cellW,
                        cellH,
                        8
                    )
                ) {
                    continue;
                }

                const record = {
                    layer: 'layer0',
                    onScreen: true,
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

                if (id === 'layer1') {
                    framePickups.length = 0;
                }
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
        // Visible labels/grid intentionally disabled.
        // Tracking continues through the canvas hooks above.

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
            return (
                blockPositions[
                    canonicalTileKey(
                        tileX,
                        tileY
                    )
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
    // INTELLIGENT TILE BOT V4
    // ============================================================

    /*
        Design goals:
        - The bot never chooses a direction directly from "what looks good".
          It chooses a persistent objective and a complete path.
        - It changes direction only at tile centers.
        - Bomb danger is time-dependent, not a permanent red cross.
        - Own bomb range is exact from the bot's tracked power value.
        - Unknown-player bomb range/timing is treated conservatively and shown
          separately because their player power is not present in drawImage().
        - Pickups are detected from the real items.png source cells used by
          ItemUnit, not from generic atlas labels.
    */

    const ITEM_ROWS = [
        ['bomb', 'detonator'],
        ['power', 'bomb_super', 'atomic'],
        ['scate', 'kick', 'bat'],
        ['shield', 'heart', 'surprise'],
        ['random', 'disease', 'jelly'],
        ['jetpack'],
        ['', '', 'arena_cup'],
        ['silver', 'gold', 'diamond'],
        ['muffin', 'key', 'plutonium']
    ];

    const PICKUP_PRIORITY = {
        power: 140,
        bomb: 132,
        scate: 126,
        shield: 105,
        heart: 100,
        detonator: 94,
        bomb_super: 92,
        atomic: 90,
        kick: 82,
        bat: 78,
        jetpack: 75,
        surprise: 68,
        random: 62,
        jelly: 58,
        key: 50,
        plutonium: 45,
        gold: 0,
        diamond: 0,
        silver: 0,
        muffin: 0
    };

    const WALKABLE_EXACT = new Set([
        'grass1',
        'grass2',
        'grass3',
        'field',
        'dirt',
        'sand',
        'rocky1',
        'rocky_hole',
        'dirty_hole',

        'hole1',
        'hole2',
        'hole3',
        'hole4',

        'tile',
        'tile2',
        'tile3',
        'tile4',
        'left',
        'up',
        'right',
        'down',

        'button_off',
        'button_on',

        'bridge_h',
        'bridge_v',
        'bridge_metal_v',
        'bridge_metal_h',
        'bridge_off',
        'bridge_on',
        'deep_bridge',

        'tile_red',
        'tile_blue',
        'tile_yellow',
        'tile_purple',

        'gate_opened',
        'tunnel',

        'portalGreen',
        'portalBlue',
        'portalOrange',
        'portalRed',

        'teleport0_on',
        'teleport1_on',
        'teleport2_on',
        'teleport0_off',
        'teleport1_off',
        'teleport2_off',

        'arena_spawn_point0',
        'arena_spawn_point1',
        'arena_spawn_point2'
    ]);

    const CARDINALS = [
        {
            dx: 1,
            dy: 0,
            key: 'd'
        },
        {
            dx: -1,
            dy: 0,
            key: 'a'
        },
        {
            dx: 0,
            dy: 1,
            key: 's'
        },
        {
            dx: 0,
            dy: -1,
            key: 'w'
        }
    ];

    const BOT = {
        enabled: false,

        // Rules supplied by the user.
        fuseMs: 4000,

        // app.nocache.js:
        //   StandardBombUnit -> f = this.s; a.f = f;
        //   OP(a,b) -> a.s = b
        // so BombModel.s is the directional blast length.
        power: 2,

        // We cannot know another player's power from the sprite alone.
        // Foreign bombs therefore use a conservative radius and are marked
        // uncertain rather than pretending the estimate is exact.
        unknownBombRadius: 6,
        unknownBombMaxRemainingMs: 4000,

        // Actual fire lingers briefly after the explosion event.
        firePreMs: 140,
        firePostMs: 520,

        // World observation TTLs. Keeping these short prevents ghost bombs.
        pickupTtlMs: 220,
        bombVisualTtlMs: 420,
        blastVisualTtlMs: 220,

        bombConfirmMs: 850,

        /*
            Foreign-bomb anticipation.

            BmStandardBombView computes:
                stage = floor((40 - N) / 13), clamped to 0..2

            With the normal ~4 s / 40-count fuse this corresponds roughly to:
                stage 0: 2.8..4.0 s remaining
                stage 1: 1.5..2.7 s remaining
                stage 2: 0.0..1.4 s remaining

            These are used as timing BOUNDS, not fake exact times.
        */
        foreignFuseStageBoundsMs: [
            [2800, 4000],
            [1500, 2700],
            [0, 1400]
        ],

        // Movement.
        centerTolerancePx: 4.5,
        reachedTolerancePx: 4.0,
        movementSpeedPxPerSec: 175,
        lastMotionSample: null,

        currentKey: null,
        keyDownAt: 0,


        // Goal/path state.
        plan: null,

        /*
            Anti-idle is purposefully low priority. It does not override
            pickups, block breaking, or danger escape.
        */
        idleMoveAfterMs: 4200,
        lastPhysicalMoveAt: 0,
        lastPhysicalPose: null,
        lastRepositionAt: 0,
        recentRepositionTargets: [],

        planSerial: 0,
        drive: null,
        state: 'idle',
        replanReason: '',

        // Bomb state.
        bombs: new Map(),
        nextBombId: 1,
        pendingOwnBomb: null,
        escapeBombId: null,

        // Pickup bookkeeping.
        pendingPickupCollection: null,
        bombCapacityPickups: 0,
        speedPickups: 0,

        // Player tracking.
        lastPlayerWorld: null,
        lastPlayerCandidateName: null,

        // app.nocache.js exposes a public API object with:
        // getPlayerInfo(), appInputKeys(mask), appInputBomb().
        // It is closure-local in the compiled file, so discover the object
        // from the page/Angular object graph at runtime and cache it.
        gameApi: null,
        gameApiLastProbeAt: 0,
        playerInfo: null,
        playerInfoAt: 0,
        powerSource: 'fallback',
        bombSuperCount: 0,

        // Productive-objective memory. The bot keeps working on a chosen
        // block through bomb/escape cycles instead of wandering to a new one.
        focusBlock: null,
        revealWaitUntil: 0,
        revealTile: null,

        // Bomb diagnostics / removal history.
        recentBombEvents: [],

        /*
            Prevent a temporarily missed own-bomb render from being recreated
            as a yellow "foreign" bomb on the same tile.
        */
        recentOwnBombTiles: new Map(),

        // Exact BombModel.s captures from the early runtime hook.
        bombModelStrengthMatches: new Map(),
        lastBombModelCapturePruneAt: 0,

        // Loop timing.
        thinkEveryMs: 50,
        lastThinkAt: 0,

        stuckKeyCheckEveryMs: 900,
        lastStuckKeyCheck: 0,

        autoRespawn: true,
        respawnCheckEveryMs: 350,
        lastRespawnCheck: 0,

        // Render cache.
        lastSchedule: null,
        dangerOverlay: null,
        dangerCtx: null,
        lastOverlayDrawAt: 0,

        debug: false
    };


    // ============================================================
    // SOURCE-BACKED GAME API BRIDGE
    // ============================================================

    /*
        app.nocache.js exposes one public API object whose prototype contains:
          getPlayerInfo()
          appInputKeys(mask)
          appInputBomb()

        getPlayerInfo() returns:
          { alive, tileX, tileY, ...all named inventory/stat values... }

        qS() maps movement bits as:
          1 = right
          2 = up
          4 = left
          8 = down

        IMPORTANT:
        Xd() treats bit 16 as a SPECIAL stop-mode flag:
            c = (b & 16) != 0
            c && (b = 0)
            ...
            cd(..., c ? 2 : 1)

        In the live game that special mode corresponds to the Q-style
        movement stop behavior. It must NOT be used as a normal key release.
        A normal neutral movement state is appInputKeys(0).

        The compiled `Iy` variable itself is closure-local, so the userscript
        cannot name it directly. Instead we locate the same object from values
        exposed to page code / Angular scopes and cache it.
    */

    const GAME_MOVE_MASK = {
        d: 1,
        w: 2,
        a: 4,
        s: 8
    };

    function isGameApiObject(value) {
        return !!value &&
            (typeof value === 'object' || typeof value === 'function') &&
            typeof value.getPlayerInfo === 'function' &&
            typeof value.appInputKeys === 'function' &&
            typeof value.appInputBomb === 'function';
    }

    function probeObjectGraphForGameApi(root, maxDepth = 2, maxObjects = 300) {
        if (!root || (typeof root !== 'object' && typeof root !== 'function')) {
            return null;
        }

        const queue = [{ value: root, depth: 0 }];
        const seen = new WeakSet();
        let visited = 0;

        while (queue.length && visited++ < maxObjects) {
            const node = queue.shift();
            const value = node.value;

            if (!value || (typeof value !== 'object' && typeof value !== 'function')) {
                continue;
            }

            if (seen.has(value)) {
                continue;
            }

            seen.add(value);

            if (isGameApiObject(value)) {
                return value;
            }

            if (node.depth >= maxDepth) {
                continue;
            }

            let keys = [];

            try {
                keys = Object.getOwnPropertyNames(value);
            } catch {
                continue;
            }

            // Avoid traversing enormous browser/DOM graphs.
            if (keys.length > 180) {
                keys = keys.slice(0, 180);
            }

            for (const key of keys) {
                if (
                    key === 'window' ||
                    key === 'self' ||
                    key === 'parent' ||
                    key === 'top' ||
                    key === 'frames' ||
                    key === 'document'
                ) {
                    continue;
                }

                let child;

                try {
                    child = value[key];
                } catch {
                    continue;
                }

                if (
                    child &&
                    (typeof child === 'object' || typeof child === 'function')
                ) {
                    queue.push({
                        value: child,
                        depth: node.depth + 1
                    });
                }
            }
        }

        return null;
    }

    function discoverGameApi(force = false) {
        const now = performance.now();

        if (
            BOT.gameApi &&
            isGameApiObject(BOT.gameApi)
        ) {
            return BOT.gameApi;
        }

        if (
            !force &&
            now - BOT.gameApiLastProbeAt < 1000
        ) {
            return null;
        }

        BOT.gameApiLastProbeAt = now;

        // First: common direct page globals.
        for (const name of [
            'Game',
            'game',
            'gameApi',
            'GameApi',
            'api',
            'API'
        ]) {
            try {
                const value = window[name];

                if (isGameApiObject(value)) {
                    BOT.gameApi = value;
                    return value;
                }
            } catch {}
        }

        // Second: Angular scopes/controllers (the page uses ng-click).
        try {
            if (window.angular) {
                /*
                    app.nocache.js registers the exact public object as:
                        bombermine.service('Game', ... return Iy)
                    so injector.get('Game') is the preferred path.
                */
                const roots = [
                    document,
                    document.documentElement,
                    document.body,
                    document.querySelector('[ng-controller]'),
                    document.querySelector('.game'),
                    document.querySelector('#game')
                ].filter(Boolean);

                for (const element of roots) {
                    let wrapped;

                    try {
                        wrapped = window.angular.element(element);
                    } catch {
                        continue;
                    }

                    const possible = [];

                    try {
                        possible.push(wrapped.scope?.());
                    } catch {}

                    try {
                        possible.push(wrapped.isolateScope?.());
                    } catch {}

                    try {
                        const injector = wrapped.injector?.();

                        if (injector) {
                            for (const serviceName of [
                                'Game',
                                'game',
                                'GameService',
                                'gameService'
                            ]) {
                                try {
                                    possible.push(injector.get(serviceName));
                                } catch {}
                            }
                        }
                    } catch {}

                    for (const candidate of possible) {
                        const found =
                            probeObjectGraphForGameApi(
                                candidate,
                                3,
                                400
                            );

                        if (found) {
                            BOT.gameApi = found;
                            return found;
                        }
                    }
                }
            }
        } catch {}

        // Last resort: shallow scan of page globals.
        try {
            for (const key of Object.getOwnPropertyNames(window)) {
                let value;

                try {
                    value = window[key];
                } catch {
                    continue;
                }

                if (isGameApiObject(value)) {
                    BOT.gameApi = value;
                    return value;
                }
            }
        } catch {}

        return null;
    }

    function readAuthoritativePlayerInfo(force = false) {
        const now = performance.now();

        if (
            !force &&
            BOT.playerInfo &&
            now - BOT.playerInfoAt < 35
        ) {
            return BOT.playerInfo;
        }

        const api =
            discoverGameApi();

        if (!api) {
            return BOT.playerInfo;
        }

        try {
            const info =
                api.getPlayerInfo();

            if (
                info &&
                Number.isFinite(Number(info.tileX)) &&
                Number.isFinite(Number(info.tileY))
            ) {
                BOT.playerInfo = info;
                BOT.playerInfoAt = now;

                /*
                    DefaultBombFactory in app.nocache.js sets BombModel.s from
                    the local player's "power" stat. A bomb_super can add one
                    additional point for the next eligible bomb; when present
                    we include that one-point upper bound rather than using a
                    fixed guessed range.
                */
                const basePower =
                    Number(info.power);

                const superCount =
                    Number(info.bomb_super);

                if (
                    Number.isFinite(basePower) &&
                    basePower > 0
                ) {
                    /*
                        DefaultBombFactory uses the player's `power` stat as
                        BombModel.s. A bomb_super MAY add one point only when
                        its storage/eligibility condition passes:
                            bomb_super > YL(playerBombStorage)

                        getPlayerInfo() exposes the item count but not YL(...),
                        so `BOT.power` stays the exact base power. Before a new
                        bomb is visually confirmed we may use +1 only as a
                        conservative upper bound. Once the bomb is rendered,
                        bomb64x64 overlay frame 7 confirms q&4/bomb_super.
                    */
                    BOT.power =
                        Math.max(
                            1,
                            Math.floor(basePower)
                        );

                    BOT.bombSuperCount =
                        Number.isFinite(superCount)
                            ? Math.max(
                                0,
                                Math.floor(superCount)
                            )
                            : 0;

                    BOT.powerSource =
                        'getPlayerInfo.power';
                }

                return info;
            }
        } catch {}

        return BOT.playerInfo;
    }


    function sendGameMoveState(
        key
    ) {
        /*
            Intentionally disabled.

            appInputKeys() writes directly into the game's internal input
            state. Mixing that API with the page's real keyboard listeners can
            override physical key presses. v4.2 uses only synthetic keyboard
            events for movement, matching the successful v3.1 controller.
        */
        return false;
    }


    function sendGameStopState() {
        /*
            Intentionally disabled. Do not write appInputKeys(0) or
            appInputKeys(16) from the userscript.
        */
        return false;
    }




    function bombModelHookState() {
        return (
            window.__GOB_BOMB_MODEL_TAP_V5__ ||
            null
        );
    }


    function pruneBombModelStrengthCaptures(
        now =
            performance.now()
    ) {
        const state =
            bombModelHookState();

        if (
            !state ||
            !Array.isArray(
                state.records
            )
        ) {
            return;
        }

        if (
            now -
                BOT.lastBombModelCapturePruneAt <
            500
        ) {
            return;
        }

        BOT.lastBombModelCapturePruneAt =
            now;

        /*
            Do not discard a live model merely because its first strength write
            was several seconds ago. Keep it while N is positive; otherwise
            keep a small post-explosion history.
        */
        state.records =
            state.records.filter(
                record => {
                    const liveN =
                        Number(
                            record.model?.N
                        );

                    if (
                        Number.isFinite(
                            liveN
                        ) &&
                        liveN >
                            0
                    ) {
                        return true;
                    }

                    return (
                        now -
                            Number(
                                record.lastCapturedAt ??
                                record.firstCapturedAt ??
                                record.at ??
                                now
                            ) <
                        1800
                    );
                }
            );
    }



    function activeModelRenderOffset() {
        const info =
            BOT?.playerInfo;

        const render =
            BOT?.lastPlayerWorld;

        if (
            !info ||
            !render ||
            !Number.isFinite(
                Number(
                    info.tileX
                )
            ) ||
            !Number.isFinite(
                Number(
                    info.tileY
                )
            ) ||
            !Number.isFinite(
                Number(
                    render.x
                )
            ) ||
            !Number.isFinite(
                Number(
                    render.y
                )
            )
        ) {
            return {
                x: 0,
                y: 0,
                valid: false
            };
        }

        return {
            x:
                nearestTileIndex(
                    render.x
                ) -
                Number(
                    info.tileX
                ),
            y:
                nearestTileIndex(
                    render.y
                ) -
                Number(
                    info.tileY
                ),
            valid:
                true
        };
    }


    function capturedBombTile(
        record
    ) {
        const model =
            record?.model;

        if (!model) {
            return null;
        }

        const rawX =
            Number(
                model.fb?.e
            );

        const rawY =
            Number(
                model.fb?.f
            );

        if (
            !Number.isFinite(rawX) ||
            !Number.isFinite(rawY)
        ) {
            return null;
        }

        const modelX =
            rawX >>
            5;

        const modelY =
            rawY >>
            5;

        const offset =
            activeModelRenderOffset();

        const x =
            modelX +
            (
                offset.valid
                    ? offset.x
                    : 0
            );

        const y =
            modelY +
            (
                offset.valid
                    ? offset.y
                    : 0
            );

        return {
            x,
            y,
            canonicalX:
                x,
            canonicalY:
                y,
            modelX,
            modelY,
            rawX,
            rawY,
            offsetX:
                offset.x,
            offsetY:
                offset.y,
            offsetValid:
                offset.valid
        };
    }



    function findCapturedBombStrengthForTile(
        x,
        y,
        now =
            performance.now()
    ) {
        pruneBombModelStrengthCaptures(
            now
        );

        const state =
            bombModelHookState();

        if (
            !state ||
            !Array.isArray(
                state.records
            )
        ) {
            return null;
        }

        let best =
            null;

        let bestScore =
            Infinity;

        for (
            const record
            of state.records
        ) {
            const model =
                record.model;

            if (!model) {
                continue;
            }

            const strength =
                Number(
                    model.s
                );

            const liveN =
                Number(
                    model.N
                );

            if (
                !Number.isFinite(
                    strength
                ) ||
                strength <= 0 ||
                (
                    Number.isFinite(
                        liveN
                    ) &&
                    liveN <= 0
                )
            ) {
                continue;
            }

            const tile =
                capturedBombTile(
                    record
                );

            if (!tile) {
                continue;
            }

            const distance =
                manhattan(
                    x,
                    y,
                    tile.x,
                    tile.y
                );

            /*
                Do not match a bomb from another +/-32 world. Exact tile wins;
                one-tile tolerance is only for kicked/interpolating bombs.
            */
            if (
                distance > 1
            ) {
                continue;
            }

            let claimed =
                false;

            for (
                const bomb
                of BOT.bombs.values()
            ) {
                if (
                    bomb.modelRef ===
                        model
                ) {
                    claimed =
                        true;
                    break;
                }
            }

            if (claimed) {
                continue;
            }

            const score =
                distance *
                    10000 +
                (
                    Number.isFinite(
                        liveN
                    )
                        ? Math.abs(
                            40 -
                            liveN
                        )
                        : 50
                );

            if (
                score <
                    bestScore
            ) {
                bestScore =
                    score;
                best =
                    record;
            }
        }

        return best;
    }


    function applyCapturedBombStrength(
        bomb,
        capture,
        now =
            performance.now()
    ) {
        const model =
            capture?.model;

        if (
            !bomb ||
            !model
        ) {
            return false;
        }

        const strength =
            Math.floor(
                Number(
                    model.s
                )
            );

        if (
            !Number.isFinite(
                strength
            ) ||
            strength <= 0
        ) {
            return false;
        }

        bomb.modelRef =
            model;

        bomb.modelStrength =
            strength;

        bomb.radius =
            strength;

        bomb.rangeKnown =
            true;

        bomb.uncertain =
            false;

        bomb.modelCaptureAt =
            now;

        const modelTile =
            capturedBombTile(
                capture
            );

        if (modelTile) {
            bomb.canonicalX =
                modelTile.canonicalX;

            bomb.canonicalY =
                modelTile.canonicalY;

            const localized =
                localizeCanonicalTile(
                    modelTile.canonicalX,
                    modelTile.canonicalY
                );

            bomb.x =
                localized.x;

            bomb.y =
                localized.y;
        }

        const liveN =
            Number(
                model.N
            );

        if (
            Number.isFinite(
                liveN
            )
        ) {
            bomb.modelN =
                liveN;

            if (
                liveN >
                    0 &&
                liveN <=
                    40
            ) {
                const remaining =
                    liveN *
                    (
                        BOT.fuseMs /
                        40
                    );

                bomb.earliestExplodeAt =
                    now +
                    remaining -
                    120;

                bomb.latestExplodeAt =
                    now +
                    remaining +
                    160;

                bomb.fuseTimingSource =
                    'live-BombModel.N';
            }
        }

        pushRecentBombEvent(
            'exact-strength-captured',
            bomb,
            now,
            {
                strength,
                modelN:
                    bomb.modelN ??
                    null
            }
        );

        return true;
    }

    function predictedNextOwnBombRadius(
        info =
            BOT.playerInfo
    ) {
        const base =
            info &&
            Number(info.power) > 0
                ? Math.max(
                    1,
                    Math.floor(
                        Number(
                            info.power
                        )
                    )
                )
                : Math.max(
                    1,
                    BOT.power
                );

        /*
            Pre-confirm safety bound only. bomb_super adds one to strength
            only when the source's eligibility test passes. The public info
            object does not expose the PlayerBombStorage threshold, so do not
            call this exact. We resolve it after the actual bomb64x64 overlay
            frames are observed.
        */
        const possibleSuper =
            info &&
            Number(
                info.bomb_super
            ) > 0;

        return (
            base +
            (
                possibleSuper
                    ? 1
                    : 0
            )
        );
    }

    function confirmedOwnBombRadiusFromVisual(
        visual,
        basePower
    ) {
        const frames =
            Array.isArray(
                visual?.frames
            )
                ? visual.frames
                : [
                    visual?.frame
                ];

        /*
            BmStandardBombView overlays:
              frame 7 -> q&4  (bomb_super)
              frame 8 -> q&2
              frame 6 -> q&8

            DefaultBombFactory is the source that increments strength for
            q&4/bomb_super. Therefore frame 7 is the observable confirmation
            that THIS bomb got the +1 strength.
        */
        const hasBombSuperOverlay =
            frames.includes(
                7
            );

        return (
            Math.max(
                1,
                Math.floor(
                    Number(basePower) ||
                    BOT.power ||
                    1
                )
            ) +
            (
                hasBombSuperOverlay
                    ? 1
                    : 0
            )
        );
    }

    function sendGameBombInput() {
        const api =
            discoverGameApi();

        if (!api) {
            return false;
        }

        try {
            api.appInputBomb();
            return true;
        } catch {
            return false;
        }
    }

    // ============================================================
    // EXACT GAMEPLAY FRAME IDENTIFICATION
    // ============================================================

    function imageBaseName(image) {
        try {
            const src =
                image &&
                typeof image.src === 'string'
                    ? image.src
                    : '';

            return src
                .split('/')
                .pop()
                .split('?')[0]
                .toLowerCase();
        } catch {
            return '';
        }
    }

    function near(a, b, tolerance = 0.8) {
        return (
            Math.abs(a - b) <=
            tolerance
        );
    }

    /*
        This bypasses the unreliable generic sprite matcher for the two things
        the bot absolutely needs to get right:

        1. items.png is a fixed 32x32 grid:
           row0 bomb/detonator, row1 power/bomb_super/atomic, row2 scate/...
        2. bomb64x64 occupies the first 20 64x64 frames of atlas6.png
           (8 columns), as defined by the game's sprite file.
    */


    function isWickTextureImage(
        image
    ) {
        if (!image) {
            return false;
        }

        try {
            const sprites =
                window.sprites;

            if (
                !sprites ||
                typeof sprites.get !==
                    'function'
            ) {
                return false;
            }

            const wick =
                sprites.get(
                    'wick'
                );

            return (
                !!wick &&
                !!wick.texture &&
                wick.texture.element ===
                    image
            );
        }
        catch {
            return false;
        }
    }

    function inferFuseStageFromWick(
        record
    ) {
        /*
            app.nocache.js Hx():
                Ex = [6, 6, 5.5]
                Fx = [-14, -12, -10]

            BmStandardBombView:
                stage = floor((40 - N) / 13)
                wick at:
                    bombX + Ex[stage] * scaleX
                    bombY + Fx[stage] * scaleY

            Match this wick draw to the nearest currently rendered bomb and
            choose the stage whose expected offset best matches the observed
            wick center. This gives a coarse but source-derived fuse phase.
        */
        const wickX =
            record.dx +
            record.dw /
                2;

        const wickY =
            record.dy +
            record.dh /
                2;

        let best =
            null;

        for (
            const bomb
            of exactWorldObservations.bombs.values()
        ) {
            const age =
                performance.now() -
                bomb.lastSeenAt;

            if (
                age >
                180
            ) {
                continue;
            }

            const scaleX =
                Math.max(
                    0.35,
                    Math.min(
                        2.0,
                        Number(
                            bomb.renderScaleX
                        ) ||
                        1
                    )
                );

            const scaleY =
                Math.max(
                    0.35,
                    Math.min(
                        2.0,
                        Number(
                            bomb.renderScaleY
                        ) ||
                        scaleX
                    )
                );

            const expected = [
                {
                    stage: 0,
                    x:
                        bomb.worldX +
                        6 *
                            scaleX,
                    y:
                        bomb.worldY -
                        14 *
                            scaleY
                },
                {
                    stage: 1,
                    x:
                        bomb.worldX +
                        6 *
                            scaleX,
                    y:
                        bomb.worldY -
                        12 *
                            scaleY
                },
                {
                    stage: 2,
                    x:
                        bomb.worldX +
                        5.5 *
                            scaleX,
                    y:
                        bomb.worldY -
                        10 *
                            scaleY
                }
            ];

            for (
                const candidate
                of expected
            ) {
                const error =
                    Math.hypot(
                        wickX -
                            candidate.x,
                        wickY -
                            candidate.y
                    );

                if (
                    !best ||
                    error <
                        best.error
                ) {
                    best = {
                        bomb,
                        stage:
                            candidate.stage,
                        error
                    };
                }
            }
        }

        /*
            Stage offsets are only 2px apart vertically, so reject loose
            matches rather than assigning the wrong phase.
        */
        if (
            !best ||
            best.error >
                3.25
        ) {
            return null;
        }

        return best;
    }

    function observeBombWick(
        record,
        now =
            performance.now()
    ) {
        const match =
            inferFuseStageFromWick(
                record
            );

        if (!match) {
            return;
        }

        const bomb =
            match.bomb;

        const previousStage =
            Number.isInteger(
                bomb.fuseStage
            )
                ? bomb.fuseStage
                : null;

        /*
            Never allow an animation misread to move the fuse BACKWARD.
        */
        if (
            previousStage != null &&
            match.stage <
                previousStage
        ) {
            return;
        }

        if (
            previousStage !==
                match.stage
        ) {
            bomb.fuseStage =
                match.stage;

            bomb.fuseStageChangedAt =
                now;
        }

        bomb.fuseStageLastSeenAt =
            now;

        bomb.fuseStageError =
            match.error;
    }

    function stageTimingWindow(
        stage,
        now
    ) {
        const pair =
            BOT.foreignFuseStageBoundsMs[
                stage
            ];

        if (!pair) {
            return null;
        }

        return {
            earliest:
                now +
                pair[0],

            latest:
                now +
                pair[1]
        };
    }

    function refineForeignBombTimingFromVisual(
        bomb,
        visual,
        now
    ) {
        if (
            !bomb ||
            bomb.ours ||
            !visual ||
            !Number.isInteger(
                visual.fuseStage
            )
        ) {
            return;
        }

        const stage =
            visual.fuseStage;

        const window =
            stageTimingWindow(
                stage,
                now
            );

        if (!window) {
            return;
        }

        const previousStage =
            Number.isInteger(
                bomb.fuseStage
            )
                ? bomb.fuseStage
                : null;

        if (
            previousStage == null ||
            stage >
                previousStage
        ) {
            bomb.fuseStage =
                stage;

            bomb.fuseStageChangedAt =
                now;

            /*
                A freshly observed stage transition is much more informative:
                  entering stage 1 ~= N 27 ~= 2.7 s remaining
                  entering stage 2 ~= N 14 ~= 1.4 s remaining

                Keep a small tolerance for render/update cadence.
            */
            if (
                previousStage != null &&
                stage ===
                    previousStage +
                        1
            ) {
                const centerRemaining =
                    stage === 1
                        ? 2700
                        : (
                            stage === 2
                                ? 1400
                                : null
                        );

                if (
                    centerRemaining !=
                    null
                ) {
                    bomb.earliestExplodeAt =
                        now +
                        centerRemaining -
                        180;

                    bomb.latestExplodeAt =
                        now +
                        centerRemaining +
                        220;

                    bomb.fuseTimingSource =
                        'wick-transition';

                    return;
                }
            }
        }

        bomb.fuseStage =
            stage;

        bomb.fuseStageLastSeenAt =
            now;

        /*
            Intersect the existing timing estimate with the phase window.
            Seeing the same phase on later frames naturally tightens the
            explosion-time interval.
        */
        const oldEarliest =
            Number.isFinite(
                bomb.earliestExplodeAt
            )
                ? bomb.earliestExplodeAt
                : window.earliest;

        const oldLatest =
            Number.isFinite(
                bomb.latestExplodeAt
            )
                ? bomb.latestExplodeAt
                : window.latest;

        const nextEarliest =
            Math.max(
                oldEarliest,
                window.earliest
            );

        const nextLatest =
            Math.min(
                oldLatest,
                window.latest
            );

        if (
            nextEarliest <=
                nextLatest
        ) {
            bomb.earliestExplodeAt =
                nextEarliest;

            bomb.latestExplodeAt =
                nextLatest;
        }
        else {
            /*
                If the old first-seen estimate conflicts with direct visual
                phase evidence, trust the source-derived current phase.
            */
            bomb.earliestExplodeAt =
                window.earliest;

            bomb.latestExplodeAt =
                window.latest;
        }

        bomb.fuseTimingSource =
            'wick-stage';
    }

    function isExplosionTextureImage(
        image
    ) {
        if (!image) {
            return false;
        }

        try {
            const sprites =
                window.sprites;

            if (
                !sprites ||
                typeof sprites.get !==
                    'function'
            ) {
                return false;
            }

            const explosion =
                sprites.get(
                    'explosion'
                );

            return (
                !!explosion &&
                !!explosion.texture &&
                explosion.texture.element ===
                    image
            );
        }
        catch {
            return false;
        }
    }


    function rememberActualBlastCell(
        x,
        y,
        now,
        source =
            'explosion-render'
    ) {
        const canonicalX =
            canonicalTileCoord(
                x
            );

        const canonicalY =
            canonicalTileCoord(
                y
            );

        const key =
            tileKey(
                canonicalX,
                canonicalY
            );

        const previous =
            exactWorldObservations.blasts.get(
                key
            );

        exactWorldObservations.blasts.set(
            key,
            {
                key,
                canonicalX,
                canonicalY,
                renderX:
                    x,
                renderY:
                    y,
                x,
                y,
                firstSeenAt:
                    previous?.firstSeenAt ??
                    now,
                lastSeenAt:
                    now,
                source
            }
        );
    }

    function observeExplosionRenderRect(
        record,
        now =
            performance.now()
    ) {
        /*
            app.nocache.js BmExplosionView.Vv() can draw an entire run of fire
            with ONE stretched drawImage:
                horizontal destination = f*32 by j*32
                vertical destination   = j*32 by f*32

            Expand that observed destination rectangle back into the real 32px
            world cells. This is observed fire, not a guessed bomb radius.
        */
        const x0 =
            Math.min(
                record.dx,
                record.dx +
                    record.dw
            );

        const x1 =
            Math.max(
                record.dx,
                record.dx +
                    record.dw
            );

        const y0 =
            Math.min(
                record.dy,
                record.dy +
                    record.dh
            );

        const y1 =
            Math.max(
                record.dy,
                record.dy +
                    record.dh
            );

        const width =
            x1 -
            x0;

        const height =
            y1 -
            y0;

        if (
            !Number.isFinite(width) ||
            !Number.isFinite(height) ||
            width <= 0 ||
            height <= 0 ||
            width >
                TILE_SIZE *
                    20 ||
            height >
                TILE_SIZE *
                    20
        ) {
            return;
        }

        const horizontal =
            width >=
            height;

        const major =
            horizontal
                ? width
                : height;

        const count =
            Math.max(
                1,
                Math.round(
                    major /
                    TILE_SIZE
                )
            );

        /*
            The major dimension in Vv() is an integer number of 32px cells.
            If a helper/cap draw is slightly different, record just its center
            instead of inventing extra cells.
        */
        if (
            Math.abs(
                major -
                count *
                    TILE_SIZE
            ) >
            3.5
        ) {
            rememberActualBlastCell(
                nearestTileIndex(
                    (
                        x0 +
                        x1
                    ) /
                    2
                ),
                nearestTileIndex(
                    (
                        y0 +
                        y1
                    ) /
                    2
                ),
                now
            );

            return;
        }

        if (horizontal) {
            const tileY =
                nearestTileIndex(
                    (
                        y0 +
                        y1
                    ) /
                    2
                );

            for (
                let i = 0;
                i < count;
                i++
            ) {
                rememberActualBlastCell(
                    nearestTileIndex(
                        x0 +
                        (
                            i +
                            0.5
                        ) *
                        TILE_SIZE
                    ),
                    tileY,
                    now
                );
            }
        }
        else {
            const tileX =
                nearestTileIndex(
                    (
                        x0 +
                        x1
                    ) /
                    2
                );

            for (
                let i = 0;
                i < count;
                i++
            ) {
                rememberActualBlastCell(
                    tileX,
                    nearestTileIndex(
                        y0 +
                        (
                            i +
                            0.5
                        ) *
                        TILE_SIZE
                    ),
                    now
                );
            }
        }
    }


    function bombHasObservedFire(
        bomb,
        now =
            performance.now()
    ) {
        const bx =
            bomb.canonicalX ??
            canonicalTileCoord(
                bomb.x
            );

        const by =
            bomb.canonicalY ??
            canonicalTileCoord(
                bomb.y
            );

        for (
            const blast
            of exactWorldObservations.blasts.values()
        ) {
            if (
                now -
                    blast.lastSeenAt >
                700
            ) {
                continue;
            }

            if (
                logicalManhattan(
                    bx,
                    by,
                    blast.canonicalX ??
                        blast.x,
                    blast.canonicalY ??
                        blast.y
                ) <=
                    1
            ) {
                return true;
            }
        }

        return false;
    }

    function identifyDirectGameplayFrame(
        image,
        sx,
        sy,
        sw,
        sh
    ) {
        /*
            The real BmExplosionView uses sprites.get('explosion'), whose
            texture is dynamically rebuilt by ah(). Its source rectangle can
            span several frames, so identify it by texture object identity
            before ordinary filename/frame matching.
        */
        if (
            isExplosionTextureImage(
                image
            )
        ) {
            return {
                name:
                    'explosion',
                frame:
                    null,
                kind:
                    'actual-explosion-render',
                direct:
                    true
            };
        }

        if (
            isWickTextureImage(
                image
            )
        ) {
            return {
                name:
                    'wick',
                frame:
                    null,
                kind:
                    'bomb-wick',
                direct:
                    true
            };
        }

        const filename =
            imageBaseName(image);

        if (
            filename === 'items.png' &&
            near(sw, 32, 1.2) &&
            near(sh, 32, 1.2)
        ) {
            const col =
                Math.round(
                    sx / 32
                );

            const row =
                Math.round(
                    sy / 32
                );

            if (
                near(sx, col * 32, 1.2) &&
                near(sy, row * 32, 1.2)
            ) {
                const name =
                    ITEM_ROWS[row]?.[col] ||
                    '';

                if (name) {
                    return {
                        name,
                        frame:
                            row +
                            ',' +
                            col,
                        kind:
                            'pickup',
                        direct:
                            true
                    };
                }
            }
        }

        if (
            filename === 'atlas6.png' &&
            near(sw, 64, 1.2) &&
            near(sh, 64, 1.2)
        ) {
            const col =
                Math.round(
                    sx / 64
                );

            const row =
                Math.round(
                    sy / 64
                );

            const index =
                row * 8 +
                col;

            if (
                col >= 0 &&
                col < 8 &&
                row >= 0 &&
                index >= 0 &&
                index < 20 &&
                near(sx, col * 64, 1.2) &&
                near(sy, row * 64, 1.2)
            ) {
                return {
                    name:
                        'bomb64x64',
                    frame:
                        index,
                    kind:
                        'bomb',
                    direct:
                        true
                };
            }
        }

        if (
            filename === 'blast.png' &&
            near(sw, 32, 1.2) &&
            near(sh, 32, 1.2)
        ) {
            return {
                name:
                    'blast',
                frame:
                    Math.round(
                        sx / 32
                    ) +
                    ',' +
                    Math.round(
                        sy / 32
                    ),
                kind:
                    'blast',
                direct:
                    true
            };
        }

        return null;
    }

    function nearestTileIndex(worldCoord) {
        return Math.round(
            (
                worldCoord -
                TILE_SIZE / 2
            ) /
            TILE_SIZE
        );
    }

    function tileCenter(tile) {
        return (
            tile *
            TILE_SIZE +
            TILE_SIZE / 2
        );
    }


    function observationTileFromRecord(
        record
    ) {
        const cx =
            record.dx +
            record.dw /
                2;

        const cy =
            record.dy +
            record.dh /
                2;

        const renderX =
            nearestTileIndex(
                cx
            );

        const renderY =
            nearestTileIndex(
                cy
            );

        return {
            x:
                renderX,
            y:
                renderY,

            renderX,
            renderY,

            canonicalX:
                canonicalTileCoord(
                    renderX
                ),

            canonicalY:
                canonicalTileCoord(
                    renderY
                ),

            worldX:
                cx,
            worldY:
                cy
        };
    }

    function observeExactWorldDraw(
        record,
        filename
    ) {
        if (
            record.layer !==
                'layer0' &&
            record.layer !==
                'layer1'
        ) {
            return;
        }

        if (
            record.onScreen !==
                true
        ) {
            return;
        }

        const now =
            performance.now();

        if (
            record.kind ===
            'pickup'
        ) {
            const p =
                observationTileFromRecord(
                    record
                );

            const key =
                record.name +
                '@' +
                p.canonicalX +
                ',' +
                p.canonicalY;

            exactWorldObservations.pickups.set(
                key,
                {
                    key,
                    name:
                        record.name,
                    canonicalX:
                        p.canonicalX,
                    canonicalY:
                        p.canonicalY,
                    renderX:
                        p.renderX,
                    renderY:
                        p.renderY,
                    x:
                        p.renderX,
                    y:
                        p.renderY,
                    worldX:
                        p.worldX,
                    worldY:
                        p.worldY,
                    lastSeenAt:
                        now
                }
            );

            return;
        }

        if (
            record.kind ===
            'bomb'
        ) {
            /*
                BmBombView scales the bomb sprite over time (`cu()` / `du()`).
                The old 24..40px gate rejected legitimate bomb frames while
                they were scaled, which is why some foreign bombs were not
                noticed until very late in the fuse.

                The SOURCE rectangle already proves this is a bomb64x64 frame,
                so accept a much wider legitimate world-render scale range.
            */
            const renderW =
                Math.abs(
                    Number(record.dw) || 0
                );

            const renderH =
                Math.abs(
                    Number(record.dh) || 0
                );

            if (
                renderW < 10 ||
                renderW > 64 ||
                renderH < 10 ||
                renderH > 68
            ) {
                return;
            }

            const p =
                observationTileFromRecord(
                    record
                );

            const offset =
                Math.hypot(
                    p.worldX -
                        tileCenter(p.x),
                    p.worldY -
                        tileCenter(p.y)
                );

            /*
                renderShiftY=-2 for bomb64x64, so a real bomb center should be
                very close to the tile center. This removes false atlas hits.
            */
            if (offset > 16) {
                return;
            }

            const key =
                tileKey(
                    p.canonicalX,
                    p.canonicalY
                );

            const previous =
                exactWorldObservations.bombs.get(
                    key
                );

            const frameSet =
                new Set(
                    previous &&
                    Array.isArray(
                        previous.frames
                    )
                        ? previous.frames
                        : (
                            previous &&
                            previous.frame != null
                                ? [
                                    previous.frame
                                ]
                                : []
                        )
                );

            if (
                record.frame != null
            ) {
                frameSet.add(
                    record.frame
                );
            }

            exactWorldObservations.bombs.set(
                key,
                {
                    key,
                    canonicalX:
                        p.canonicalX,
                    canonicalY:
                        p.canonicalY,
                    renderX:
                        p.renderX,
                    renderY:
                        p.renderY,
                    x:
                        p.renderX,
                    y:
                        p.renderY,
                    worldX:
                        p.worldX,
                    worldY:
                        p.worldY,
                    frame:
                        record.frame,
                    frames:
                        [
                            ...frameSet
                        ],
                    firstSeenAt:
                        previous?.firstSeenAt ??
                        now,
                    lastSeenAt:
                        now,
                    renderScaleX:
                        renderW /
                        32,

                    renderScaleY:
                        renderH /
                        32,

                    fuseStage:
                        previous?.fuseStage ??
                        null,

                    fuseStageChangedAt:
                        previous?.fuseStageChangedAt ??
                        null,

                    fuseStageLastSeenAt:
                        previous?.fuseStageLastSeenAt ??
                        null,

                    fuseStageError:
                        previous?.fuseStageError ??
                        null,

                    confidence:
                        'bomb64x64-world'
                }
            );

            return;
        }

        if (
            record.kind ===
                'bomb-wick'
        ) {
            observeBombWick(
                record,
                now
            );

            return;
        }

        if (
            record.kind ===
                'actual-explosion-render'
        ) {
            observeExplosionRenderRect(
                record,
                now
            );

            return;
        }

        if (
            record.kind ===
            'blast'
        ) {
            const p =
                observationTileFromRecord(
                    record
                );

            rememberActualBlastCell(
                p.renderX,
                p.renderY,
                now,
                'legacy-blast'
            );
        }
    }

    function cleanupExactObservations(
        now
    ) {
        for (
            const [key, pickup]
            of exactWorldObservations.pickups
        ) {
            if (
                now -
                pickup.lastSeenAt >
                BOT.pickupTtlMs
            ) {
                exactWorldObservations.pickups.delete(
                    key
                );
            }
        }

        for (
            const [key, bomb]
            of exactWorldObservations.bombs
        ) {
            if (
                now -
                bomb.lastSeenAt >
                BOT.bombVisualTtlMs
            ) {
                exactWorldObservations.bombs.delete(
                    key
                );
            }
        }

        for (
            const [key, blast]
            of exactWorldObservations.blasts
        ) {
            if (
                now -
                blast.lastSeenAt >
                BOT.blastVisualTtlMs
            ) {
                exactWorldObservations.blasts.delete(
                    key
                );
            }
        }
    }



    function activePickups(
        now =
            performance.now()
    ) {
        cleanupExactObservations(
            now
        );

        return [
            ...exactWorldObservations.pickups.values()
        ];
    }



    function activeBombVisuals(
        now =
            performance.now()
    ) {
        cleanupExactObservations(
            now
        );

        return [
            ...exactWorldObservations.bombs.values()
        ];
    }

    function isPowerupName(name) {
        return (
            PICKUP_PRIORITY[
                String(name || '')
            ] != null
        );
    }

    // ============================================================
    // TERRAIN MODEL
    // ============================================================

    function normalizeTileName(name) {
        return cleanDisplayName(
            name || ''
        )
            .replace(
                /^-+|-+$/g,
                ''
            );
    }

    function isWalkableTileName(name) {
        name =
            normalizeTileName(
                name
            );

        if (!name) {
            return false;
        }

        if (
            WALKABLE_EXACT.has(
                name
            )
        ) {
            return true;
        }

        if (
            /^grass\d*$/i.test(
                name
            ) ||
            /^hole\d+$/i.test(
                name
            ) ||
            /^surf\d*$/i.test(
                name
            ) ||
            /^surf_hole\d+$/i.test(
                name
            ) ||
            /^surf_crater\d+$/i.test(
                name
            ) ||
            /^(hexTile|blueTile)\d*$/i.test(
                name
            )
        ) {
            return true;
        }

        return false;
    }

    function destructibleHP(name) {
        name =
            normalizeTileName(
                name
            );

        if (
            name ===
                'brick' ||
            name ===
                'brick-plain' ||
            name ===
                'blueBrick' ||
            name ===
                'blueBrick-plain'
        ) {
            return 1;
        }

        const tough =
            name.match(
                /^tough(\d+)(?:-t\d+)?$/i
            );

        if (tough) {
            return Math.max(
                1,
                Number(
                    tough[1]
                ) || 9
            );
        }

        return 0;
    }

    function classifyTile(name) {
        const hp =
            destructibleHP(
                name
            );

        if (hp > 0) {
            return {
                type:
                    'destructible',
                hp,
                walkable:
                    false
            };
        }

        if (
            isWalkableTileName(
                name
            )
        ) {
            return {
                type:
                    'walkable',
                hp: 0,
                walkable:
                    true
            };
        }

        return {
            type:
                'blocked',
            hp:
                Infinity,
            walkable:
                false
        };
    }

    function tileInfo(x, y) {
        const block =
            window.getBlockAt
                ? window.getBlockAt(
                    x,
                    y
                )
                : null;

        if (!block) {
            return {
                exists:
                    false,
                x,
                y,
                name:
                    '',
                type:
                    'unknown',
                hp:
                    Infinity,
                walkable:
                    false
            };
        }

        const name =
            normalizeTileName(
                block.name
            );

        return {
            exists:
                true,
            x,
            y,
            name,
            ...classifyTile(
                name
            ),
            raw:
                block
        };
    }

    function tileKey(x, y) {
        return (
            x +
            ',' +
            y
        );
    }


    function canonicalTileCoord(
        value
    ) {
        /*
            Separate worlds/planes can occupy coordinates 32 tiles apart.
            Treat the actual rendered tile as its identity. Never modulo it.
        */
        return Math.round(
            Number(value) ||
            0
        );
    }

    function canonicalTileKey(
        x,
        y
    ) {
        return tileKey(
            canonicalTileCoord(
                x
            ),
            canonicalTileCoord(
                y
            )
        );
    }


    function nearestWrappedAlias(
        coordinate,
        reference
    ) {
        /*
            v5.2 no longer aliases arbitrary +/-32 positions together.
        */
        return Math.round(
            Number(coordinate) ||
            0
        );
    }

    function plannerReferenceTile() {
        if (
            BOT &&
            BOT.lastPlayerWorld &&
            Number.isFinite(
                BOT.lastPlayerWorld.x
            ) &&
            Number.isFinite(
                BOT.lastPlayerWorld.y
            )
        ) {
            return {
                x:
                    nearestTileIndex(
                        BOT.lastPlayerWorld.x
                    ),
                y:
                    nearestTileIndex(
                        BOT.lastPlayerWorld.y
                    )
            };
        }

        if (
            BOT &&
            BOT.playerInfo &&
            Number.isFinite(
                Number(
                    BOT.playerInfo.tileX
                )
            ) &&
            Number.isFinite(
                Number(
                    BOT.playerInfo.tileY
                )
            )
        ) {
            return {
                x:
                    Number(
                        BOT.playerInfo.tileX
                    ),
                y:
                    Number(
                        BOT.playerInfo.tileY
                    )
            };
        }

        return {
            x: 0,
            y: 0
        };
    }

    function localizeCanonicalTile(
        canonicalX,
        canonicalY,
        reference =
            plannerReferenceTile()
    ) {
        return {
            x:
                nearestWrappedAlias(
                    canonicalX,
                    reference.x
                ),
            y:
                nearestWrappedAlias(
                    canonicalY,
                    reference.y
                )
        };
    }


    function sameLogicalTile(
        ax,
        ay,
        bx,
        by
    ) {
        return (
            Math.round(Number(ax)) ===
                Math.round(Number(bx)) &&
            Math.round(Number(ay)) ===
                Math.round(Number(by))
        );
    }


    function logicalManhattan(
        ax,
        ay,
        bx,
        by
    ) {
        return manhattan(
            Math.round(Number(ax)),
            Math.round(Number(ay)),
            Math.round(Number(bx)),
            Math.round(Number(by))
        );
    }


    function manhattan(
        ax,
        ay,
        bx,
        by
    ) {
        return (
            Math.abs(
                ax -
                bx
            ) +
            Math.abs(
                ay -
                by
            )
        );
    }

    // ============================================================
    // PLAYER POSITION
    // ============================================================

    function drawFilename(draw) {
        return imageBaseName(
            draw?.image
        );
    }

    function looksLikePlayerDraw(draw) {
        if (!draw) {
            return false;
        }

        if (
            draw.kind ===
                'pickup' ||
            draw.kind ===
                'bomb' ||
            draw.kind ===
                'blast'
        ) {
            return false;
        }

        const name =
            String(
                draw.name || ''
            ).toLowerCase();

        const file =
            drawFilename(
                draw
            );

        if (
            /bomb|mine|wick|explosion|blast|ball|snowball|item|font|icon|tile|wall|brick|bridge|turret|portal|shield/.test(
                name
            )
        ) {
            return false;
        }

        if (
            /character|player|default|flying|slowpoke|jetpack|nyan|private_|skin/.test(
                name
            ) ||
            /character_|nyan|private_|skin/.test(
                file
            )
        ) {
            return true;
        }

        const w =
            Math.abs(
                Number(
                    draw.dw
                ) || 0
            );

        const h =
            Math.abs(
                Number(
                    draw.dh
                ) || 0
            );

        /*
            Custom character resources are not guaranteed to carry one of the
            stock names, but their rendered footprint is still character-like.
        */
        return (
            w >= 30 &&
            w <= 48 &&
            h >= 30 &&
            h <= 56
        );
    }

    function getPlayerWorldPosition() {
        const canvas =
            layer1 ||
            layer0;

        if (!canvas) {
            return null;
        }

        const screenCX =
            canvas.width / 2;

        const screenCY =
            canvas.height / 2;

        let best = null;
        let bestScore =
            Infinity;

        for (
            const draw
            of frameDraws.layer1
        ) {
            if (
                !looksLikePlayerDraw(
                    draw
                )
            ) {
                continue;
            }

            const worldX =
                draw.dx +
                draw.dw / 2;

            const worldY =
                draw.dy +
                draw.dh / 2;

            const screen =
                transformPoint(
                    draw.matrix,
                    worldX,
                    worldY
                );

            let score =
                Math.hypot(
                    screen.x -
                        screenCX,
                    screen.y -
                        screenCY
                );

            const name =
                String(
                    draw.name || ''
                ).toLowerCase();

            const file =
                drawFilename(
                    draw
                );

            if (
                /character|default|flying|slowpoke|jetpack|nyan|private_|skin/.test(
                    name
                ) ||
                /character_|nyan|private_|skin/.test(
                    file
                )
            ) {
                score -= 80;
            }

            if (
                BOT.lastPlayerWorld
            ) {
                const jump =
                    Math.hypot(
                        worldX -
                            BOT.lastPlayerWorld.x,
                        worldY -
                            BOT.lastPlayerWorld.y
                    );

                score +=
                    Math.min(
                        220,
                        jump *
                        0.45
                    );
            }

            if (
                score <
                bestScore
            ) {
                bestScore =
                    score;

                best = {
                    x:
                        worldX,
                    y:
                        worldY,
                    source:
                        'player-draw',
                    name:
                        draw.name ||
                        file ||
                        'unknown'
                };
            }
        }

        if (
            best &&
            bestScore <
                250
        ) {
            BOT.lastPlayerWorld = {
                x:
                    best.x,
                y:
                    best.y
            };

            BOT.lastPlayerCandidateName =
                best.name;

            return best;
        }

        if (
            BOT.lastPlayerWorld &&
            !worldMatrix
        ) {
            return {
                ...BOT.lastPlayerWorld,
                source:
                    'last-known'
            };
        }

        if (!worldMatrix) {
            return null;
        }

        const fallback =
            inversePoint(
                worldMatrix,
                screenCX,
                screenCY
            );

        BOT.lastPlayerWorld = {
            x:
                fallback.x,
            y:
                fallback.y
        };

        return {
            ...fallback,
            source:
                'camera-fallback',
            name:
                'camera'
        };
    }


    function getPlayerPose() {
        const p =
            getPlayerWorldPosition();

        if (!p) {
            return null;
        }

        const tileX =
            nearestTileIndex(
                p.x
            );

        const tileY =
            nearestTileIndex(
                p.y
            );

        const centerX =
            tileCenter(
                tileX
            );

        const centerY =
            tileCenter(
                tileY
            );

        return {
            x:
                p.x,
            y:
                p.y,

            tileX,
            tileY,

            centerX,
            centerY,

            offsetX:
                p.x -
                centerX,
            offsetY:
                p.y -
                centerY,

            source:
                p.source,
            name:
                p.name
        };
    }

    function playerNearTileCenter(
        pose,
        tolerance =
            BOT.reachedTolerancePx
    ) {
        return (
            Math.abs(
                pose.offsetX
            ) <= tolerance &&
            Math.abs(
                pose.offsetY
            ) <= tolerance
        );
    }

    function updateMeasuredSpeed(
        pose,
        now
    ) {
        const previous =
            BOT.lastMotionSample;

        if (
            previous &&
            BOT.currentKey
        ) {
            const dt =
                now -
                previous.t;

            if (
                dt >= 20 &&
                dt <= 350
            ) {
                const distance =
                    Math.hypot(
                        pose.x -
                            previous.x,
                        pose.y -
                            previous.y
                    );

                if (
                    distance >=
                        0.2 &&
                    distance <=
                        28
                ) {
                    const speed =
                        distance /
                        (
                            dt /
                            1000
                        );

                    if (
                        speed >= 40 &&
                        speed <= 600
                    ) {
                        BOT.movementSpeedPxPerSec =
                            BOT.movementSpeedPxPerSec *
                            0.88 +
                            speed *
                            0.12;
                    }
                }
            }
        }

        BOT.lastMotionSample = {
            x:
                pose.x,
            y:
                pose.y,
            t:
                now
        };
    }

    function updateActivityWatchdog(
        pose,
        now
    ) {
        const previous =
            BOT.lastPhysicalPose;

        if (!previous) {
            BOT.lastPhysicalMoveAt =
                now;
        }
        else {
            const distance =
                Math.hypot(
                    pose.x -
                        previous.x,
                    pose.y -
                        previous.y
                );

            /*
                Ignore 32-tile renderer alias jumps. They are coordinate frame
                changes, not physical movement.
            */
            const aliasJump =
                Math.abs(
                    distance -
                    WORLD_WRAP_TILES *
                        TILE_SIZE
                ) <
                96;

            if (
                !aliasJump &&
                distance >
                    1.25
            ) {
                BOT.lastPhysicalMoveAt =
                    now;
            }
        }

        BOT.lastPhysicalPose = {
            x:
                pose.x,
            y:
                pose.y,
            tileX:
                pose.tileX,
            tileY:
                pose.tileY,
            t:
                now
        };
    }

    function pruneRepositionMemory(
        now
    ) {
        BOT.recentRepositionTargets =
            BOT.recentRepositionTargets.filter(
                item =>
                    now -
                        item.at <
                    18000
            );
    }

    function wasRecentRepositionTarget(
        x,
        y,
        now
    ) {
        pruneRepositionMemory(
            now
        );

        return BOT.recentRepositionTargets.some(
            item =>
                sameLogicalTile(
                    item.x,
                    item.y,
                    x,
                    y
                )
        );
    }

    function chooseSafeRepositionPlan(
        start,
        schedule,
        now
    ) {
        /*
            Not random wandering: pick a nearby safe tile that either opens
            access to destructibles or simply relocates out of a dead idle
            pocket. Productive goals always run before this.
        */
        const candidates =
            [];

        for (
            let dy = -6;
            dy <= 6;
            dy++
        ) {
            for (
                let dx = -6;
                dx <= 6;
                dx++
            ) {
                const distance =
                    Math.abs(dx) +
                    Math.abs(dy);

                if (
                    distance < 2 ||
                    distance > 6
                ) {
                    continue;
                }

                const x =
                    start.x +
                    dx;

                const y =
                    start.y +
                    dy;

                const info =
                    tileInfo(
                        x,
                        y
                    );

                if (
                    !info.exists ||
                    !info.walkable
                ) {
                    continue;
                }

                if (
                    !safeDuring(
                        schedule,
                        x,
                        y,
                        now,
                        now +
                            1500
                    )
                ) {
                    continue;
                }

                let openNeighbors =
                    0;

                let destructibleNeighbors =
                    0;

                for (
                    const dir
                    of CARDINALS
                ) {
                    const neighbor =
                        tileInfo(
                            x +
                                dir.dx,
                            y +
                                dir.dy
                        );

                    if (
                        neighbor.exists &&
                        neighbor.walkable
                    ) {
                        openNeighbors++;
                    }

                    if (
                        neighbor.type ===
                            'destructible'
                    ) {
                        destructibleNeighbors++;
                    }
                }

                const repeatPenalty =
                    wasRecentRepositionTarget(
                        x,
                        y,
                        now
                    )
                        ? 90
                        : 0;

                candidates.push({
                    x,
                    y,
                    score:
                        destructibleNeighbors *
                            45 +
                        openNeighbors *
                            8 +
                        distance *
                            2 -
                        repeatPenalty
                });
            }
        }

        candidates.sort(
            (
                a,
                b
            ) =>
                b.score -
                a.score
        );

        for (
            const target
            of candidates.slice(
                0,
                18
            )
        ) {
            const path =
                findPathToTarget(
                    start,
                    target,
                    schedule,
                    now,
                    {
                        maxSteps:
                            18,
                        maxNodes:
                            1400
                    }
                );

            if (
                !path ||
                path.length <
                    2
            ) {
                continue;
            }

            BOT.recentRepositionTargets.push({
                x:
                    target.x,
                y:
                    target.y,
                at:
                    now
            });

            BOT.lastRepositionAt =
                now;

            return {
                target,
                path
            };
        }

        return null;
    }


    function tileTravelMs() {
        return Math.max(
            110,
            Math.min(
                650,
                TILE_SIZE /
                    Math.max(
                        50,
                        BOT.movementSpeedPxPerSec
                    ) *
                    1000
            )
        );
    }

    // ============================================================
    // INPUT / MOVEMENT CONTROLLER
    // ============================================================


    function dispatchBotKey(
        key,
        type
    ) {
        const codeMap = {
            w:
                'KeyW',
            a:
                'KeyA',
            s:
                'KeyS',
            d:
                'KeyD',
            k:
                'KeyK'
        };

        const upper =
            key.toUpperCase();

        const options = {
            key,
            code:
                codeMap[key] ||
                upper,
            bubbles:
                true,
            cancelable:
                true,
            keyCode:
                upper.charCodeAt(
                    0
                ),
            which:
                upper.charCodeAt(
                    0
                )
        };

        document.dispatchEvent(
            new KeyboardEvent(
                type,
                options
            )
        );

        window.dispatchEvent(
            new KeyboardEvent(
                type,
                options
            )
        );
    }


    function releaseKey(key) {
        if (!key) {
            return;
        }

        dispatchBotKey(
            key,
            'keyup'
        );
    }



    function releaseMoveKey() {
        if (
            BOT.currentKey
        ) {
            releaseKey(
                BOT.currentKey
            );
        }

        BOT.currentKey =
            null;

        BOT.keyDownAt =
            0;
    }


    function releaseEveryBotKey(
        except =
            null
    ) {
        for (
            const key
            of [
                'w',
                'a',
                's',
                'd',
                'k'
            ]
        ) {
            if (
                key !==
                except
            ) {
                releaseKey(
                    key
                );
            }
        }
    }



    function holdMoveKey(
        key,
        now =
            performance.now()
    ) {
        if (
            !key ||
            !'wasd'.includes(
                key
            )
        ) {
            releaseMoveKey();
            return;
        }

        if (
            BOT.currentKey ===
            key
        ) {
            return;
        }

        releaseMoveKey();

        /*
            Clear stale opposite/perpendicular keys before every direction
            change. This is deliberately single-axis movement.
        */
        releaseEveryBotKey(
            key
        );

        BOT.currentKey =
            key;

        BOT.keyDownAt =
            now;

        dispatchBotKey(
            key,
            'keydown'
        );
    }



    function periodicStuckKeyCheck(
        now
    ) {
        if (
            now -
            BOT.lastStuckKeyCheck <
            BOT.stuckKeyCheckEveryMs
        ) {
            return;
        }

        BOT.lastStuckKeyCheck =
            now;

        if (
            !BOT.enabled
        ) {
            releaseEveryBotKey();
            BOT.currentKey =
                null;
            return;
        }

        releaseEveryBotKey(
            BOT.currentKey
        );
    }

    /*
        Drive exactly one path edge.

        The important bit is lane alignment:
        - before moving horizontally, y must be on the row center
        - before moving vertically, x must be on the column center

        That mirrors the game's own movement code, which checks closeness to
        the 16 mod 32 center before changing direction. It prevents corner
        clipping instead of trying to "fix" the path after a collision.
    */
    function perpendicularAligned(
        pose,
        axis,
        target,
        now
    ) {
        const current =
            axis === 'x'
                ? pose.x
                : pose.y;

        const error =
            target - current;

        if (
            Math.abs(error) <=
            BOT.centerTolerancePx
        ) {
            if (BOT.drive) {
                BOT.drive.alignAxis = null;
                BOT.drive.alignSign = 0;
            }
            return true;
        }

        if (BOT.drive) {
            if (
                BOT.drive.alignAxis !== axis ||
                !BOT.drive.alignSign
            ) {
                BOT.drive.alignAxis = axis;
                BOT.drive.alignSign =
                    Math.sign(error) || 1;
            }

            /*
                If one browser update carries us past the center, alignment is
                complete. Do NOT reverse for a tiny correction; that was the
                source of the side-to-side "snake" movement.
            */
            if (
                (
                    BOT.drive.alignSign > 0 &&
                    current >= target
                ) ||
                (
                    BOT.drive.alignSign < 0 &&
                    current <= target
                )
            ) {
                BOT.drive.alignAxis = null;
                BOT.drive.alignSign = 0;
                return true;
            }

            const sign =
                BOT.drive.alignSign;

            holdMoveKey(
                axis === 'x'
                    ? (
                        sign > 0
                            ? 'd'
                            : 'a'
                    )
                    : (
                        sign > 0
                            ? 's'
                            : 'w'
                    ),
                now
            );

            return false;
        }

        return false;
    }

    function reachedStepTarget(
        pose,
        step,
        targetX,
        targetY
    ) {
        const t =
            BOT.reachedTolerancePx;

        switch (step.key) {
        case 'd':
            return pose.x >= targetX - t;
        case 'a':
            return pose.x <= targetX + t;
        case 's':
            return pose.y >= targetY - t;
        case 'w':
            return pose.y <= targetY + t;
        default:
            return false;
        }
    }

    function driveTowardStep(
        pose,
        step,
        now
    ) {
        if (step.wait) {
            releaseMoveKey();

            return (
                now >=
                step.arrivalAt
            );
        }

        const targetX =
            tileCenter(step.x);

        const targetY =
            tileCenter(step.y);

        const horizontal =
            step.key === 'a' ||
            step.key === 'd';

        if (horizontal) {
            if (
                !perpendicularAligned(
                    pose,
                    'y',
                    targetY,
                    now
                )
            ) {
                return false;
            }
        }
        else {
            if (
                !perpendicularAligned(
                    pose,
                    'x',
                    targetX,
                    now
                )
            ) {
                return false;
            }
        }

        /*
            Crucial: completion is directional. If we pass a center between
            two 50 ms samples, the edge is complete; we never reverse back to
            the exact pixel center.
        */
        if (
            reachedStepTarget(
                pose,
                step,
                targetX,
                targetY
            )
        ) {
            return true;
        }

        // Follow the path direction itself, never the sign of a noisy error.
        holdMoveKey(
            step.key,
            now
        );

        return false;
    }


    function pushRecentBombEvent(
        type,
        bomb,
        now,
        extra =
            {}
    ) {
        BOT.recentBombEvents.push({
            type,
            at:
                now,
            id:
                bomb?.id ??
                null,
            x:
                bomb?.x ??
                null,
            y:
                bomb?.y ??
                null,
            ...extra
        });

        if (
            BOT.recentBombEvents.length >
            30
        ) {
            BOT.recentBombEvents.splice(
                0,
                BOT.recentBombEvents.length -
                30
            );
        }
    }


    function pendingBombAsRecord() {
        const pending =
            BOT.pendingOwnBomb;

        if (
            !pending ||
            pending.confirmed
        ) {
            return null;
        }

        return {
            id:
                pending.id,

            ours:
                true,

            pending:
                true,

            confirmed:
                false,

            uncertain:
                false,

            rangeKnown:
                true,

            x:
                pending.x,

            y:
                pending.y,

            canonicalX:
                pending.canonicalX ??
                canonicalTileCoord(
                    pending.x
                ),

            canonicalY:
                pending.canonicalY ??
                canonicalTileCoord(
                    pending.y
                ),

            /*
                IMPORTANT:
                This is the predicted ACTUAL strength, not the conservative
                pre-placement safety envelope.
            */
            radius:
                pending.radius,

            safetyRadius:
                pending.safetyRadius,

            firstSeenAt:
                pending.placedAt,

            lastVisualAt:
                pending.placedAt,

            placedAt:
                pending.placedAt,

            explodeAt:
                pending.explodeAt,

            earliestExplodeAt:
                pending.explodeAt,

            latestExplodeAt:
                pending.explodeAt
        };
    }

    // ============================================================
    // BOMB REGISTRY
    // ============================================================


    function findNearbyBombRecord(
        visual
    ) {
        /*
            Deliberately exact. The old one-tile fuzzy matcher merged adjacent
            bombs and was a major source of ghost/teleporting bomb records.
        */
        for (
            const bomb
            of BOT.bombs.values()
        ) {
            if (
                bomb.x === visual.x &&
                bomb.y === visual.y
            ) {
                return bomb;
            }
        }

        return null;
    }




    function removeBombRecord(
        bomb,
        reason =
            'removed',
        now =
            performance.now()
    ) {
        if (!bomb) {
            return;
        }

        BOT.bombs.delete(
            bomb.id
        );

        if (
            bomb.ours
        ) {
            BOT.recentOwnBombTiles.set(
                canonicalTileKey(
                    bomb.canonicalX ??
                        bomb.x,
                    bomb.canonicalY ??
                        bomb.y
                ),
                {
                    x:
                        bomb.x,
                    y:
                        bomb.y,
                    canonicalX:
                        bomb.canonicalX ??
                        canonicalTileCoord(
                            bomb.x
                        ),
                    canonicalY:
                        bomb.canonicalY ??
                        canonicalTileCoord(
                            bomb.y
                        ),
                    removedAt:
                        now,
                    reason,
                    explodeAt:
                        bomb.explodeAt ??
                        now
                }
            );
        }

        pushRecentBombEvent(
            reason,
            bomb,
            now
        );

        if (
            BOT.escapeBombId ===
                bomb.id
        ) {
            BOT.escapeBombId =
                null;
        }
    }




    function refreshBombRegistry(
        now
    ) {
        cleanupExactObservations(
            now
        );

        const reference =
            plannerReferenceTile();

        const visuals =
            activeBombVisuals(
                now
            );

        const visualByLogicalTile =
            new Map();

        for (
            const visual
            of visuals
        ) {
            const cx =
                visual.canonicalX ??
                canonicalTileCoord(
                    visual.x
                );

            const cy =
                visual.canonicalY ??
                canonicalTileCoord(
                    visual.y
                );

            visualByLogicalTile.set(
                tileKey(
                    cx,
                    cy
                ),
                visual
            );
        }

        for (
            const [key, old]
            of BOT.recentOwnBombTiles
        ) {
            if (
                now -
                    old.removedAt >
                1600
            ) {
                BOT.recentOwnBombTiles.delete(
                    key
                );
            }
        }

        for (
            const bomb
            of BOT.bombs.values()
        ) {
            bomb.seenNow =
                false;

            if (
                bomb.canonicalX == null
            ) {
                bomb.canonicalX =
                    canonicalTileCoord(
                        bomb.x
                    );
            }

            if (
                bomb.canonicalY == null
            ) {
                bomb.canonicalY =
                    canonicalTileCoord(
                        bomb.y
                    );
            }

            const localized =
                localizeCanonicalTile(
                    bomb.canonicalX,
                    bomb.canonicalY,
                    reference
                );

            if (!bomb.modelRef) {
                bomb.x =
                    localized.x;

                bomb.y =
                    localized.y;
            }
        }

        /*
            Confirm our own pending placement by LOGICAL tile, so a camera/map
            alias shift cannot make the same bomb disappear.
        */
        if (
            BOT.pendingOwnBomb &&
            !BOT.pendingOwnBomb.confirmed
        ) {
            const pending =
                BOT.pendingOwnBomb;

            pending.canonicalX ??=
                canonicalTileCoord(
                    pending.x
                );

            pending.canonicalY ??=
                canonicalTileCoord(
                    pending.y
                );

            const key =
                tileKey(
                    pending.canonicalX,
                    pending.canonicalY
                );

            const visual =
                visualByLogicalTile.get(
                    key
                );

            if (visual) {
                const localized =
                    localizeCanonicalTile(
                        pending.canonicalX,
                        pending.canonicalY,
                        reference
                    );

                const bomb = {
                    id:
                        pending.id,

                    ours:
                        true,

                    confirmed:
                        true,

                    uncertain:
                        false,

                    rangeKnown:
                        true,

                    canonicalX:
                        pending.canonicalX,

                    canonicalY:
                        pending.canonicalY,

                    x:
                        localized.x,

                    y:
                        localized.y,

                    radius:
                        confirmedOwnBombRadiusFromVisual(
                            visual,
                            pending.basePower
                        ),

                    safetyRadius:
                        pending.safetyRadius,

                    radiusIsUpperBound:
                        false,

                    specialFrames:
                        Array.isArray(
                            visual.frames
                        )
                            ? [
                                ...visual.frames
                            ]
                            : [
                                visual.frame
                            ],

                    firstSeenAt:
                        now,

                    lastVisualAt:
                        now,

                    placedAt:
                        pending.placedAt,

                    explodeAt:
                        pending.explodeAt,

                    earliestExplodeAt:
                        pending.explodeAt,

                    latestExplodeAt:
                        pending.explodeAt,

                    identityStickyUntil:
                        pending.explodeAt +
                        BOT.firePostMs +
                        450,

                    seenNow:
                        true
                };

                /*
                    Our own bomb can also be matched to the exact BombModel.
                    This removes the remaining bomb_super visual ambiguity.
                */
                const capture =
                    findCapturedBombStrengthForTile(
                        bomb.x,
                        bomb.y,
                        now
                    );

                applyCapturedBombStrength(
                    bomb,
                    capture,
                    now
                );

                BOT.bombs.set(
                    bomb.id,
                    bomb
                );

                pending.confirmed =
                    true;

                visualByLogicalTile.delete(
                    key
                );

                pushRecentBombEvent(
                    'confirmed-own',
                    bomb,
                    now
                );
            }
            else if (
                now >
                    pending.confirmBy
            ) {
                pushRecentBombEvent(
                    'placement-failed',
                    pending,
                    now
                );

                BOT.pendingOwnBomb =
                    null;

                if (
                    BOT.plan &&
                    BOT.plan.type ===
                        'escape-own-bomb'
                ) {
                    clearPlan(
                        'bomb-placement-not-confirmed'
                    );
                }
            }
        }

        for (
            const visual
            of visuals
        ) {
            const cx =
                visual.canonicalX ??
                canonicalTileCoord(
                    visual.x
                );

            const cy =
                visual.canonicalY ??
                canonicalTileCoord(
                    visual.y
                );

            const key =
                tileKey(
                    cx,
                    cy
                );

            let bomb =
                null;

            for (
                const candidate
                of BOT.bombs.values()
            ) {
                if (
                    candidate.canonicalX ===
                        cx &&
                    candidate.canonicalY ===
                        cy
                ) {
                    bomb =
                        candidate;

                    if (
                        candidate.ours
                    ) {
                        break;
                    }
                }
            }

            if (!bomb) {
                const recentOwn =
                    BOT.recentOwnBombTiles.get(
                        key
                    );

                if (
                    recentOwn &&
                    now -
                        recentOwn.removedAt <
                        1000
                ) {
                    continue;
                }

                const localized =
                    localizeCanonicalTile(
                        cx,
                        cy,
                        reference
                    );

                bomb = {
                    id:
                        BOT.nextBombId++,

                    ours:
                        false,

                    confirmed:
                        true,

                    uncertain:
                        true,

                    rangeKnown:
                        false,

                    canonicalX:
                        cx,

                    canonicalY:
                        cy,

                    x:
                        localized.x,

                    y:
                        localized.y,

                    radius:
                        Math.max(
                            1,
                            BOT.unknownBombRadius
                        ),

                    firstSeenAt:
                        now,

                    lastVisualAt:
                        now,

                    placedAt:
                        null,

                    earliestExplodeAt:
                        now,

                    latestExplodeAt:
                        now +
                        BOT.fuseMs,

                    explodeAt:
                        null,

                    fuseStage:
                        null,

                    fuseTimingSource:
                        'first-seen-fallback',

                    seenNow:
                        true
                };

                const capture =
                    findCapturedBombStrengthForTile(
                        bomb.x,
                        bomb.y,
                        now
                    );

                if (
                    !applyCapturedBombStrength(
                        bomb,
                        capture,
                        now
                    )
                ) {
                    refineForeignBombTimingFromVisual(
                        bomb,
                        visual,
                        now
                    );
                }

                BOT.bombs.set(
                    bomb.id,
                    bomb
                );

                pushRecentBombEvent(
                    'seen-foreign',
                    bomb,
                    now
                );
            }
            else {
                bomb.lastVisualAt =
                    now;

                bomb.seenNow =
                    true;

                bomb.canonicalX =
                    cx;

                bomb.canonicalY =
                    cy;

                const localized =
                    localizeCanonicalTile(
                        cx,
                        cy,
                        reference
                    );

                if (!bomb.modelRef) {
                    bomb.x =
                        localized.x;

                    bomb.y =
                        localized.y;
                }

                if (
                    !bomb.ours &&
                    !bomb.modelRef
                ) {
                    const capture =
                        findCapturedBombStrengthForTile(
                            bomb.x,
                            bomb.y,
                            now
                        );

                    applyCapturedBombStrength(
                        bomb,
                        capture,
                        now
                    );
                }

                if (
                    !bomb.ours &&
                    bomb.rangeKnown ===
                        false
                ) {
                    refineForeignBombTimingFromVisual(
                        bomb,
                        visual,
                        now
                    );
                }

                if (
                    bomb.ours
                ) {
                    const frames =
                        Array.isArray(
                            visual.frames
                        )
                            ? visual.frames
                            : [
                                visual.frame
                            ];

                    if (
                        frames.includes(
                            7
                        )
                    ) {
                        bomb.radius =
                            Math.max(
                                bomb.radius,
                                confirmedOwnBombRadiusFromVisual(
                                    visual,
                                    BOT.power
                                )
                            );
                    }
                }
            }
        }

        /*
            Refresh exact s/N/position after visual association. This also
            keeps kicked bombs attached to their model identity.
        */
        refreshBombsFromLiveModels(
            now
        );

        for (
            const bomb
            of [
                ...BOT.bombs.values()
            ]
        ) {
            if (
                bombHasObservedFire(
                    bomb,
                    now
                )
            ) {
                removeBombRecord(
                    bomb,
                    bomb.ours
                        ? 'own-fire-observed'
                        : 'foreign-fire-observed',
                    now
                );

                continue;
            }

            if (
                bomb.modelRef
            ) {
                const liveN =
                    Number(
                        bomb.modelRef.N
                    );

                /*
                    A live model is stronger evidence than a missed canvas
                    frame. Never remove it because the bomb happened to be
                    culled or redrawn under a different wrapped alias.
                */
                if (
                    Number.isFinite(
                        liveN
                    ) &&
                    liveN >
                        0
                ) {
                    continue;
                }

                if (
                    Number.isFinite(
                        bomb.latestExplodeAt
                    ) &&
                    now <=
                        bomb.latestExplodeAt +
                        BOT.firePostMs +
                        450
                ) {
                    continue;
                }

                removeBombRecord(
                    bomb,
                    'model-ended',
                    now
                );

                continue;
            }

            if (
                bomb.ours
            ) {
                if (
                    now <=
                    (
                        bomb.identityStickyUntil ??
                        (
                            bomb.explodeAt +
                            BOT.firePostMs +
                            450
                        )
                    )
                ) {
                    continue;
                }

                removeBombRecord(
                    bomb,
                    'own-fuse-window-ended',
                    now
                );

                continue;
            }

            /*
                Renderer-only foreign bombs are retained through their plausible
                fuse window. A 650ms canvas gap is no longer allowed to erase
                a bomb.
            */
            const keepUntil =
                Math.max(
                    Number(
                        bomb.latestExplodeAt
                    ) ||
                    0,
                    (
                        bomb.firstSeenAt ||
                        now
                    ) +
                    1000
                ) +
                BOT.firePostMs +
                350;

            if (
                now <=
                    keepUntil
            ) {
                continue;
            }

            removeBombRecord(
                bomb,
                'foreign-fuse-window-ended',
                now
            );
        }

        if (
            BOT.pendingOwnBomb &&
            BOT.pendingOwnBomb.confirmed
        ) {
            BOT.pendingOwnBomb =
                null;
        }
    }


    function placeOwnBomb(
        pose,
        now
    ) {
        if (
            BOT.pendingOwnBomb &&
            !BOT.pendingOwnBomb.confirmed
        ) {
            return null;
        }

        const info =
            readAuthoritativePlayerInfo(
                true
            );

        const x =
            pose.tileX;

        const y =
            pose.tileY;

        /*
            getPlayerInfo() includes the named `bomb` stat. For bombs the bot
            itself placed, we can avoid sending a placement request when all
            known capacity is already occupied.
        */
        const capacity =
            info &&
            Number.isFinite(
                Number(info.bomb)
            )
                ? Math.max(
                    1,
                    Math.floor(
                        Number(info.bomb)
                    )
                )
                : null;

        if (capacity != null) {
            let activeOwn =
                0;

            for (
                const bomb
                of BOT.bombs.values()
            ) {
                if (bomb.ours) {
                    activeOwn++;
                }
            }

            if (
                BOT.pendingOwnBomb &&
                !BOT.pendingOwnBomb.confirmed
            ) {
                activeOwn++;
            }

            if (
                activeOwn >= capacity
            ) {
                return null;
            }
        }

        for (
            const bomb
            of BOT.bombs.values()
        ) {
            if (
                bomb.x === x &&
                bomb.y === y
            ) {
                return null;
            }
        }

        /*
            Re-read player power immediately before placement. The compiled
            DefaultBombFactory assigns BombModel.s from this stat.
        */
        if (
            info &&
            Number.isFinite(
                Number(info.power)
            ) &&
            Number(info.power) > 0
        ) {
            BOT.power =
                Math.max(
                    1,
                    Math.floor(
                        Number(
                            info.power
                        )
                    )
                );

            BOT.bombSuperCount =
                Math.max(
                    0,
                    Math.floor(
                        Number(
                            info.bomb_super
                        ) ||
                        0
                    )
                );
        }

        const basePower =
            info &&
            Number(info.power) > 0
                ? Math.max(
                    1,
                    Math.floor(
                        Number(
                            info.power
                        )
                    )
                )
                : Math.max(
                    1,
                    BOT.power
                );

        const pending = {
            id:
                BOT.nextBombId++,
            x,
            y,

            /*
                `basePower` is the expected BombModel.s from the player's
                current power stat.

                `safetyRadius` is deliberately separate. It may be one larger
                while bomb_super eligibility is unresolved, but it is ONLY
                used before placement to prove that an escape route is safe.
                It is not shown as the bomb's predicted fire footprint.
            */
            basePower,

            radius:
                basePower,

            safetyRadius:
                predictedNextOwnBombRadius(
                    info
                ),

            radiusIsUpperBound:
                false,

            placedAt:
                now,

            explodeAt:
                now +
                BOT.fuseMs,

            confirmBy:
                now +
                BOT.bombConfirmMs,

            confirmed:
                false
        };

        BOT.pendingOwnBomb =
            pending;

        BOT.escapeBombId =
            pending.id;

        /*
            Prefer appInputBomb(), the same native input bridge exposed by
            app.nocache.js. Fall back to K only if that object is unavailable.
        */
        if (
            !sendGameBombInput()
        ) {
            dispatchBotKey(
                'k',
                'keydown'
            );

            setTimeout(
                () => {
                    dispatchBotKey(
                        'k',
                        'keyup'
                    );
                },
                35
            );
        }

        pushRecentBombEvent(
            'placement-requested',
            pending,
            now,
            {
                radius:
                    pending.radius
            }
        );

        return pending;
    }

    // ============================================================
    // EXPLOSION SIMULATION
    // ============================================================

    /*
        Source-backed blast rules used below.

        Standard bomb:
          - BombModel constructor starts N=40.
          - jQ() decrements N and explodes at 0.
          - DefaultBombFactory sets BombModel.s from player power.
          - XP() walks four cardinal rays one 32px tile at a time.
          - ordinary solid objects stop a ray before the object.
          - entity type 85 is a separate solid game entity (VaultUnit), not
            an ordinary brick tile; it terminates the ray when hit.
          - another bomb is notified/triggered when fire reaches its tile.
          - map-tile collision classes 3/5 shorten the remaining ray and >=7
            terminate it on that tile. Ordinary destructible bricks are map
            tiles, not type-85 entities. The renderer exposes the tile sprite
            name but not the source config's numeric `.s` collision class, so
            the bot uses the observed brick/tough tile names for the terminal
            destructible case and does not fabricate 3/5 mappings.
    */



    function refreshBombsFromLiveModels(
        now =
            performance.now()
    ) {
        const offset =
            activeModelRenderOffset();

        for (
            const bomb
            of BOT.bombs.values()
        ) {
            const model =
                bomb.modelRef;

            if (!model) {
                continue;
            }

            const strength =
                Math.floor(
                    Number(
                        model.s
                    )
                );

            if (
                Number.isFinite(
                    strength
                ) &&
                strength > 0
            ) {
                bomb.radius =
                    strength;
                bomb.modelStrength =
                    strength;
                bomb.rangeKnown =
                    true;
                bomb.uncertain =
                    false;
            }

            const rawX =
                Number(
                    model.fb?.e
                );

            const rawY =
                Number(
                    model.fb?.f
                );

            if (
                Number.isFinite(rawX) &&
                Number.isFinite(rawY) &&
                offset.valid
            ) {
                const modelX =
                    rawX >>
                    5;

                const modelY =
                    rawY >>
                    5;

                bomb.modelTileX =
                    modelX;
                bomb.modelTileY =
                    modelY;

                bomb.x =
                    modelX +
                    offset.x;

                bomb.y =
                    modelY +
                    offset.y;

                bomb.canonicalX =
                    bomb.x;

                bomb.canonicalY =
                    bomb.y;
            }

            const liveN =
                Number(
                    model.N
                );

            if (
                Number.isFinite(
                    liveN
                )
            ) {
                bomb.modelN =
                    liveN;

                if (
                    liveN > 0 &&
                    liveN <= 40
                ) {
                    const remaining =
                        liveN *
                        (
                            BOT.fuseMs /
                            40
                        );

                    bomb.earliestExplodeAt =
                        now +
                        remaining -
                        120;

                    bomb.latestExplodeAt =
                        now +
                        remaining +
                        160;

                    bomb.fuseTimingSource =
                        'live-BombModel.N';
                }
            }
        }
    }



    function allBombRecordsForSimulation() {
        const reference =
            plannerReferenceTile();

        const bombs =
            [
                ...BOT.bombs.values()
            ].map(
                bomb => {
                    const canonicalX =
                        bomb.canonicalX ??
                        canonicalTileCoord(
                            bomb.x
                        );

                    const canonicalY =
                        bomb.canonicalY ??
                        canonicalTileCoord(
                            bomb.y
                        );

                    const localized =
                        localizeCanonicalTile(
                            canonicalX,
                            canonicalY,
                            reference
                        );

                    return {
                        ...bomb,
                        canonicalX,
                        canonicalY,
                        x:
                            localized.x,
                        y:
                            localized.y
                    };
                }
            );

        const pending =
            pendingBombAsRecord();

        if (pending) {
            const localized =
                localizeCanonicalTile(
                    pending.canonicalX,
                    pending.canonicalY,
                    reference
                );

            bombs.push({
                ...pending,
                x:
                    localized.x,
                y:
                    localized.y
            });
        }

        return bombs;
    }

    function bombAtTileFromList(
        bombs,
        x,
        y,
        excludeId =
            null
    ) {
        for (
            const bomb
            of bombs
        ) {
            if (
                bomb.id !== excludeId &&
                bomb.x === x &&
                bomb.y === y
            ) {
                return bomb;
            }
        }

        return null;
    }


    function traceBlastCellsObservedWorld(
        bomb,
        bombs,
        tileStateFn,
        radius =
            bomb.radius
    ) {
        const temporary = {
            ...bomb,
            radius
        };

        const footprint =
            predictedDirectionalFootprint(
                temporary,
                bombs,
                tileStateFn
            );

        return {
            cells:
                footprint.cells,

            triggered:
                footprint.triggered,

            directions:
                footprint.directions,

            strength:
                footprint.strength
        };
    }


    function simulatedTileState(
        x,
        y,
        hpOverrides,
        destroyed
    ) {
        const key =
            tileKey(
                x,
                y
            );

        if (
            destroyed.has(
                key
            )
        ) {
            return {
                type:
                    'walkable',
                hp: 0,
                walkable:
                    true,
                name:
                    'destroyed'
            };
        }

        const info =
            tileInfo(
                x,
                y
            );

        if (
            info.type ===
            'destructible'
        ) {
            return {
                ...info,
                hp:
                    hpOverrides.has(
                        key
                    )
                        ? hpOverrides.get(
                            key
                        )
                        : info.hp
            };
        }

        return info;
    }

    function addHazardInterval(
        schedule,
        x,
        y,
        start,
        end,
        kind,
        bombId =
            null
    ) {
        const key =
            tileKey(
                x,
                y
            );

        let list =
            schedule.hazards.get(
                key
            );

        if (!list) {
            list = [];
            schedule.hazards.set(
                key,
                list
            );
        }

        list.push({
            start,
            end,
            kind,
            bombId
        });

        schedule.overlay.push({
            x,
            y,
            start,
            end,
            kind,
            bombId
        });
    }



    function predictedDirectionalFootprint(
        bomb,
        bombs =
            allBombRecordsForSimulation(),
        tileStateFn =
            (
                (x, y) =>
                    tileInfo(
                        x,
                        y
                    )
            )
    ) {
        const result = {
            center: {
                x:
                    bomb.x,
                y:
                    bomb.y
            },

            strength:
                Math.max(
                    0,
                    Number(
                        bomb.radius
                    ) ||
                    0
                ),

            directions:
                {},

            cells: [
                {
                    x:
                        bomb.x,
                    y:
                        bomb.y,
                    center:
                        true
                }
            ],

            triggered:
                []
        };

        /*
            `strength` is only the maximum number of steps XP() starts with.
            Actual reach is determined independently in each direction.

            app.nocache XP():
              n = B (BombModel.s)
              walk one 32px tile at a time
              blockers/bombs/terrain can terminate or shorten that ray

            We therefore never call `strength` the actual fire length.
        */
        for (
            const dir
            of CARDINALS
        ) {
            const ray = {
                key:
                    dir.key,
                maximum:
                    result.strength,
                actual:
                    0,
                stoppedBy:
                    'strength',
                cells: []
            };

            for (
                let step = 1;
                step <=
                    result.strength;
                step++
            ) {
                const x =
                    bomb.x +
                    dir.dx *
                    step;

                const y =
                    bomb.y +
                    dir.dy *
                    step;

                const state =
                    tileStateFn(
                        x,
                        y
                    );

                if (
                    !state ||
                    (
                        !state.exists &&
                        state.name !==
                            'destroyed'
                    )
                ) {
                    ray.stoppedBy =
                        'unknown-terrain';
                    break;
                }

                if (
                    state.type ===
                    'blocked'
                ) {
                    ray.stoppedBy =
                        'solid';
                    break;
                }

                const cell = {
                    x,
                    y,
                    step,
                    dir
                };

                ray.cells.push(
                    cell
                );

                result.cells.push(
                    cell
                );

                ray.actual =
                    step;

                const otherBomb =
                    bombAtTileFromList(
                        bombs,
                        x,
                        y,
                        bomb.id
                    );

                if (otherBomb) {
                    result.triggered.push(
                        otherBomb
                    );

                    ray.stoppedBy =
                        'bomb';

                    break;
                }

                if (
                    state.type ===
                    'destructible'
                ) {
                    ray.stoppedBy =
                        'destructible';

                    break;
                }

                if (
                    step ===
                    result.strength
                ) {
                    ray.stoppedBy =
                        'strength';
                }
            }

            result.directions[
                dir.key
            ] =
                ray;
        }

        return result;
    }

    function staticBlastCells(
        bomb,
        radius =
            bomb.radius
    ) {
        const bombs =
            allBombRecordsForSimulation();

        return traceBlastCellsObservedWorld(
            bomb,
            bombs,
            (x, y) =>
                tileInfo(
                    x,
                    y
                ),
            radius
        ).cells;
    }


    function simulateExplosionSchedule(
        now
    ) {
        const schedule = {
            now,
            hazards:
                new Map(),
            bombOccupancy:
                new Map(),
            overlay: [],
            exactEvents: [],
            uncertainBombs: [],
            latestExactEnd:
                now
        };

        const bombs =
            allBombRecordsForSimulation()
                .map(
                    bomb => ({
                        ...bomb
                    })
                );

        const byId =
            new Map();

        const byTile =
            new Map();

        for (
            const bomb
            of bombs
        ) {
            if (
                bomb.ours
            ) {
                const exact =
                    Number(
                        bomb.explodeAt
                    );

                bomb.earliest =
                    exact;

                bomb.latest =
                    exact;
            }
            else {
                bomb.earliest =
                    Math.max(
                        now,
                        Number(
                            bomb.earliestExplodeAt ??
                            now
                        )
                    );

                bomb.latest =
                    Math.max(
                        bomb.earliest,
                        Number(
                            bomb.latestExplodeAt ??
                            (
                                bomb.firstSeenAt +
                                BOT.fuseMs
                            )
                        )
                    );
            }

            byId.set(
                bomb.id,
                bomb
            );

            byTile.set(
                tileKey(
                    bomb.x,
                    bomb.y
                ),
                bomb
            );

            /*
                A bomb is a movement obstacle while its model may still exist.
                For uncertain foreign bombs, use the latest possible explosion
                bound rather than perpetually extending the occupancy.
            */
            schedule.bombOccupancy.set(
                tileKey(
                    bomb.x,
                    bomb.y
                ),
                bomb.latest +
                    BOT.firePostMs
            );
        }

        /*
            Fixed-point chain envelope.

            If A's possible blast can hit B, then B cannot survive later than
            A's latest possible detonation, and it may be triggered as early
            as A's earliest. This preserves uncertainty for foreign bombs
            instead of drawing a fabricated exact red timestamp.
        */
        let changed =
            true;

        let guard =
            0;

        while (
            changed &&
            guard++ <
                bombs.length *
                4 +
                8
        ) {
            changed =
                false;

            for (
                const source
                of bombs
            ) {
                const traced =
                    traceBlastCellsObservedWorld(
                        source,
                        bombs,
                        (x, y) =>
                            tileInfo(
                                x,
                                y
                            ),
                        source.radius
                    );

                for (
                    const target
                    of traced.triggered
                ) {
                    const oldEarliest =
                        target.earliest;

                    const oldLatest =
                        target.latest;

                    target.earliest =
                        Math.min(
                            target.earliest,
                            source.earliest
                        );

                    target.latest =
                        Math.min(
                            target.latest,
                            source.latest
                        );

                    if (
                        target.latest <
                        target.earliest
                    ) {
                        target.earliest =
                            target.latest;
                    }

                    if (
                        Math.abs(
                            target.earliest -
                            oldEarliest
                        ) > 0.1 ||
                        Math.abs(
                            target.latest -
                            oldLatest
                        ) > 0.1
                    ) {
                        changed =
                            true;
                    }
                }
            }
        }

        /*
            Exact-time bombs are simulated chronologically with destructible HP
            state, so an earlier blast may destroy a brick and change a later
            blast. This is used for our confirmed/pending bombs and any bomb
            whose chain interval collapses to one exact time.
        */
        const exactBombs =
            bombs
                .filter(
                    bomb =>
                        bomb.rangeKnown !==
                            false &&
                        Math.abs(
                            bomb.latest -
                            bomb.earliest
                        ) <= 1
                )
                .map(
                    bomb => ({
                        ...bomb,
                        simExplodeAt:
                            bomb.earliest,
                        processed:
                            false
                    })
                );

        const exactByTile =
            new Map();

        for (
            const bomb
            of exactBombs
        ) {
            exactByTile.set(
                tileKey(
                    bomb.x,
                    bomb.y
                ),
                bomb
            );
        }

        const hpOverrides =
            new Map();

        const destroyed =
            new Set();

        for (;;) {
            let next =
                null;

            for (
                const bomb
                of exactBombs
            ) {
                if (
                    bomb.processed
                ) {
                    continue;
                }

                if (
                    !next ||
                    bomb.simExplodeAt <
                        next.simExplodeAt
                ) {
                    next =
                        bomb;
                }
            }

            if (!next) {
                break;
            }

            next.processed =
                true;

            /*
                Bomb occupancy is time-dependent. Once an exact bomb has
                exploded, it is removed from the world and must NOT block a
                later blast ray. Keep:
                  - the current source,
                  - exact bombs not processed yet,
                  - uncertain bombs whose latest possible explosion is still
                    at/after this event.
            */
            const bombsPresentAtEvent =
                bombs.filter(
                    candidate => {
                        if (
                            candidate.id ===
                            next.id
                        ) {
                            return true;
                        }

                        const exactCandidate =
                            exactByTile.get(
                                tileKey(
                                    candidate.x,
                                    candidate.y
                                )
                            );

                        if (
                            exactCandidate
                        ) {
                            return (
                                !exactCandidate.processed
                            );
                        }

                        return (
                            candidate.latest >=
                            next.simExplodeAt
                        );
                    }
                );

            const traced =
                traceBlastCellsObservedWorld(
                    next,
                    bombsPresentAtEvent,
                    (
                        x,
                        y
                    ) =>
                        simulatedTileState(
                            x,
                            y,
                            hpOverrides,
                            destroyed
                        ),
                    next.radius
                );

            const eventCells =
                [];

            for (
                const cell
                of traced.cells
            ) {
                eventCells.push({
                    x:
                        cell.x,
                    y:
                        cell.y
                });

                if (
                    cell.center
                ) {
                    continue;
                }

                const state =
                    simulatedTileState(
                        cell.x,
                        cell.y,
                        hpOverrides,
                        destroyed
                    );

                if (
                    state.type ===
                    'destructible'
                ) {
                    const key =
                        tileKey(
                            cell.x,
                            cell.y
                        );

                    const remaining =
                        Math.max(
                            0,
                            state.hp -
                            1
                        );

                    if (
                        remaining <= 0
                    ) {
                        destroyed.add(
                            key
                        );

                        hpOverrides.delete(
                            key
                        );
                    }
                    else {
                        hpOverrides.set(
                            key,
                            remaining
                        );
                    }
                }
            }

            /*
                Collapse exact own-to-own chain reactions. If a target is still
                in the exact set and is reached earlier, its event moves to the
                source explosion time.
            */
            for (
                const target
                of traced.triggered
            ) {
                const exactTarget =
                    exactByTile.get(
                        tileKey(
                            target.x,
                            target.y
                        )
                    );

                if (
                    exactTarget &&
                    !exactTarget.processed &&
                    exactTarget.simExplodeAt >
                        next.simExplodeAt
                ) {
                    exactTarget.simExplodeAt =
                        next.simExplodeAt;
                }
            }

            const start =
                next.simExplodeAt -
                BOT.firePreMs;

            const end =
                next.simExplodeAt +
                BOT.firePostMs;

            schedule.latestExactEnd =
                Math.max(
                    schedule.latestExactEnd,
                    end
                );

            schedule.exactEvents.push({
                bombId:
                    next.id,
                explodeAt:
                    next.simExplodeAt,
                radius:
                    next.radius,
                cells:
                    eventCells
            });

            for (
                const cell
                of eventCells
            ) {
                addHazardInterval(
                    schedule,
                    cell.x,
                    cell.y,
                    start,
                    end,
                    next.pending
                        ? 'predicted'
                        : 'exact',
                    next.id
                );
            }
        }

        /*
            FOREIGN BOMBS

            Exact BombModel.s makes the RANGE known immediately. Even if the
            precise explosion TIME is still a small interval, keep the real
            predicted footprint visible for the bomb's entire active fuse.

            Only bombs whose s capture was genuinely missed stay yellow.
        */
        for (
            const bomb
            of bombs
        ) {
            const exactTime =
                Math.abs(
                    bomb.latest -
                    bomb.earliest
                ) <= 1;

            if (
                bomb.rangeKnown !==
                    false &&
                exactTime
            ) {
                continue;
            }

            const actualFireStarted =
                bombHasObservedFire(
                    bomb,
                    now
                );

            const traced =
                traceBlastCellsObservedWorld(
                    bomb,
                    bombs,
                    (
                        x,
                        y
                    ) =>
                        tileInfo(
                            x,
                            y
                        ),
                    bomb.radius
                );

            if (
                bomb.rangeKnown !==
                    false
            ) {
                /*
                    Real strength, full-fuse display.
                */
                for (
                    const cell
                    of traced.cells
                ) {
                    addHazardInterval(
                        schedule,
                        cell.x,
                        cell.y,
                        bomb.earliest -
                            BOT.firePreMs,
                        bomb.latest +
                            BOT.firePostMs,
                        'predicted',
                        bomb.id
                    );
                }

                schedule.uncertainBombs.push({
                    id:
                        bomb.id,
                    x:
                        bomb.x,
                    y:
                        bomb.y,
                    earliest:
                        bomb.earliest,
                    latest:
                        bomb.latest,
                    radius:
                        bomb.radius,
                    rangeKnown:
                        true,
                    actualFireStarted
                });

                continue;
            }

            schedule.uncertainBombs.push({
                id:
                    bomb.id,
                x:
                    bomb.x,
                y:
                    bomb.y,
                earliest:
                    bomb.earliest,
                latest:
                    bomb.latest,
                radius:
                    bomb.radius,
                rangeKnown:
                    false,
                actualFireStarted
            });

            if (
                !actualFireStarted
            ) {
                /*
                    Unknown range fallback: conservative planner hazard only.
                    Never draw this guessed maximum as fire.
                */
                for (
                    const cell
                    of traced.cells
                ) {
                    const key =
                        tileKey(
                            cell.x,
                            cell.y
                        );

                    let list =
                        schedule.hazards.get(
                            key
                        );

                    if (!list) {
                        list = [];
                        schedule.hazards.set(
                            key,
                            list
                        );
                    }

                    list.push({
                        start:
                            bomb.earliest -
                            BOT.firePreMs,
                        end:
                            bomb.latest +
                            BOT.firePostMs,
                        kind:
                            'uncertain',
                        bombId:
                            bomb.id
                    });
                }

                schedule.overlay.push({
                    x:
                        bomb.x,
                    y:
                        bomb.y,
                    start:
                        bomb.earliest,
                    end:
                        bomb.latest +
                        BOT.firePostMs,
                    kind:
                        'uncertain',
                    bombId:
                        bomb.id
                });
            }
        }

        /*
            Actual blast sprites override predictions. BmExplosionView in the
            source uses a short-lived effect; mark observed cells immediately
            unsafe even if a bomb registry record disappeared the same frame.
        */
        for (
            const blast
            of exactWorldObservations.blasts.values()
        ) {
            const localizedBlast =
                localizeCanonicalTile(
                    blast.canonicalX ??
                        blast.x,
                    blast.canonicalY ??
                        blast.y
                );

            addHazardInterval(
                schedule,
                localizedBlast.x,
                localizedBlast.y,
                Math.min(
                    blast.firstSeenAt ??
                        now,
                    now
                ) -
                    80,
                Math.max(
                    blast.lastSeenAt ??
                        now,
                    now
                ) +
                    620,
                'actual',
                null
            );
        }

        return schedule;
    }

    function hazardIntervalsAt(
        schedule,
        x,
        y
    ) {
        return (
            schedule.hazards.get(
                tileKey(
                    x,
                    y
                )
            ) ||
            []
        );
    }

    function safeDuring(
        schedule,
        x,
        y,
        start,
        end
    ) {
        for (
            const interval
            of hazardIntervalsAt(
                schedule,
                x,
                y
            )
        ) {
            if (
                end >
                    interval.start &&
                start <
                    interval.end
            ) {
                return false;
            }
        }

        return true;
    }

    function bombOccupiesAt(
        schedule,
        x,
        y,
        atTime
    ) {
        const until =
            schedule.bombOccupancy.get(
                tileKey(
                    x,
                    y
                )
            );

        return (
            until != null &&
            atTime <
                until
        );
    }

    function liveBombOccupiesTile(
        schedule,
        x,
        y,
        atTime
    ) {
        /*
            A bomb tile is never an enterable destination while that bomb
            exists. This includes our own bomb after we step off it.

            The game may allow the player who just placed a bomb to EXIT the
            bomb's tile because they were already overlapping it. That is not
            the same thing as being allowed to enter the tile again.
        */
        return bombOccupiesAt(
            schedule,
            x,
            y,
            atTime
        );
    }

    // ============================================================
    // SPACE-TIME A*
    // ============================================================

    class MinHeap {
        constructor(compare) {
            this.a = [];
            this.compare =
                compare;
        }

        get size() {
            return this.a.length;
        }

        push(value) {
            const a =
                this.a;

            a.push(value);

            let i =
                a.length - 1;

            while (i > 0) {
                const p =
                    (
                        i - 1
                    ) >> 1;

                if (
                    this.compare(
                        a[p],
                        a[i]
                    ) <= 0
                ) {
                    break;
                }

                [
                    a[p],
                    a[i]
                ] = [
                    a[i],
                    a[p]
                ];

                i =
                    p;
            }
        }

        pop() {
            const a =
                this.a;

            if (
                a.length === 0
            ) {
                return null;
            }

            const top =
                a[0];

            const last =
                a.pop();

            if (
                a.length &&
                last
            ) {
                a[0] =
                    last;

                let i =
                    0;

                for (;;) {
                    const left =
                        i * 2 +
                        1;

                    const right =
                        left +
                        1;

                    let smallest =
                        i;

                    if (
                        left <
                            a.length &&
                        this.compare(
                            a[left],
                            a[smallest]
                        ) < 0
                    ) {
                        smallest =
                            left;
                    }

                    if (
                        right <
                            a.length &&
                        this.compare(
                            a[right],
                            a[smallest]
                        ) < 0
                    ) {
                        smallest =
                            right;
                    }

                    if (
                        smallest ===
                        i
                    ) {
                        break;
                    }

                    [
                        a[i],
                        a[smallest]
                    ] = [
                        a[smallest],
                        a[i]
                    ];

                    i =
                        smallest;
                }
            }

            return top;
        }
    }

    function pathStateKey(
        x,
        y,
        step,
        heading = ''
    ) {
        return (
            x +
            ',' +
            y +
            '@' +
            step +
            ':' +
            (heading || '-')
        );
    }

    function oppositeDirection(a, b) {
        return (
            (a === 'a' && b === 'd') ||
            (a === 'd' && b === 'a') ||
            (a === 'w' && b === 's') ||
            (a === 's' && b === 'w')
        );
    }

    function pathTurnCost(previousHeading, nextHeading, wait) {
        if (wait) {
            return 0.22;
        }

        if (
            !previousHeading ||
            !nextHeading ||
            previousHeading === nextHeading
        ) {
            return 0;
        }

        // Strongly discourage 180-degree reversals; mildly discourage turns.
        return oppositeDirection(
            previousHeading,
            nextHeading
        )
            ? 1.35
            : 0.38;
    }

    function reconstructSpaceTimePath(
        parents,
        endStateKey
    ) {
        const path = [];

        let key =
            endStateKey;

        while (
            parents.has(
                key
            )
        ) {
            const edge =
                parents.get(
                    key
                );

            path.push({
                x:
                    edge.x,
                y:
                    edge.y,
                key:
                    edge.key,
                wait:
                    edge.wait,
                departAt:
                    edge.departAt,
                arrivalAt:
                    edge.arrivalAt
            });

            key =
                edge.parent;
        }

        path.reverse();

        return path;
    }

    function pathTilePassable(
        x,
        y
    ) {
        const info =
            tileInfo(
                x,
                y
            );

        return (
            info.exists &&
            info.walkable
        );
    }

    function moveSafe(
        schedule,
        fromX,
        fromY,
        toX,
        toY,
        departAt,
        arrivalAt,
        startX,
        startY
    ) {
        if (
            !pathTilePassable(
                toX,
                toY
            )
        ) {
            return false;
        }

        /*
            Never ENTER a live bomb tile.

            There is intentionally no start-tile exception here: `toX/toY`
            is the destination of this edge. If the player begins on top of a
            newly placed bomb, the first edge leaves that bomb tile and is
            legal; a later edge attempting to return to it is rejected.
        */
        if (
            liveBombOccupiesTile(
                schedule,
                toX,
                toY,
                arrivalAt
            )
        ) {
            return false;
        }

        const duration =
            arrivalAt -
            departAt;

        /*
            During a tile crossing the character occupies parts of both cells.
            Checking overlapping halves is conservative and avoids paths that
            technically "arrive after" a blast while crossing through it.
        */
        if (
            !safeDuring(
                schedule,
                fromX,
                fromY,
                departAt,
                departAt +
                    duration *
                    0.58
            )
        ) {
            return false;
        }

        if (
            !safeDuring(
                schedule,
                toX,
                toY,
                departAt +
                    duration *
                    0.35,
                arrivalAt +
                    60
            )
        ) {
            return false;
        }

        return true;
    }

    function waitSafe(
        schedule,
        x,
        y,
        departAt,
        arrivalAt
    ) {
        if (
            liveBombOccupiesTile(
                schedule,
                x,
                y,
                departAt
            )
        ) {
            return false;
        }

        return safeDuring(
            schedule,
            x,
            y,
            departAt,
            arrivalAt +
                40
        );
    }

    function findPathToTarget(
        start,
        target,
        schedule,
        now,
        options =
            {}
    ) {
        if (
            start.x === target.x &&
            start.y === target.y
        ) {
            return [];
        }

        const maxSteps =
            options.maxSteps ?? 42;

        const maxNodes =
            options.maxNodes ?? 2600;

        const stepMs =
            tileTravelMs();

        const initialHeading =
            BOT.currentKey &&
            'wasd'.includes(BOT.currentKey)
                ? BOT.currentKey
                : null;

        const open =
            new MinHeap(
                (a, b) => {
                    const f = a.f - b.f;
                    if (Math.abs(f) > 1e-9) return f;

                    // Tie-break toward fewer turns, then closer target.
                    const t = a.turns - b.turns;
                    if (t) return t;

                    return a.h - b.h;
                }
            );

        const startKey =
            pathStateKey(
                start.x,
                start.y,
                0,
                initialHeading
            );

        const startH =
            manhattan(
                start.x,
                start.y,
                target.x,
                target.y
            );

        open.push({
            x: start.x,
            y: start.y,
            step: 0,
            g: 0,
            h: startH,
            f: startH,
            heading: initialHeading,
            turns: 0,
            stateKey: startKey
        });

        const best =
            new Map();

        best.set(
            startKey,
            0
        );

        const parents =
            new Map();

        let nodes = 0;

        while (
            open.size &&
            nodes++ < maxNodes
        ) {
            const current =
                open.pop();

            if (
                current.x === target.x &&
                current.y === target.y
            ) {
                return reconstructSpaceTimePath(
                    parents,
                    current.stateKey
                );
            }

            if (
                current.step >= maxSteps
            ) {
                continue;
            }

            const departAt =
                now +
                current.step *
                stepMs;

            const arrivalAt =
                departAt +
                stepMs;

            /*
                Prefer continuing straight when it is equally good, then moves
                that reduce Manhattan distance. This affects only tie order;
                safety/path correctness is still decided below.
            */
            const actions = [
                ...CARDINALS,
                {
                    dx: 0,
                    dy: 0,
                    key: null,
                    wait: true
                }
            ].sort((a, b) => {
                const aStraight =
                    !a.wait &&
                    a.key === current.heading
                        ? -2
                        : 0;

                const bStraight =
                    !b.wait &&
                    b.key === current.heading
                        ? -2
                        : 0;

                const ah =
                    manhattan(
                        current.x + a.dx,
                        current.y + a.dy,
                        target.x,
                        target.y
                    );

                const bh =
                    manhattan(
                        current.x + b.dx,
                        current.y + b.dy,
                        target.x,
                        target.y
                    );

                return (
                    aStraight + ah -
                    (bStraight + bh)
                );
            });

            for (const action of actions) {
                const nx =
                    current.x + action.dx;

                const ny =
                    current.y + action.dy;

                const nextStep =
                    current.step + 1;

                if (action.wait) {
                    if (
                        !waitSafe(
                            schedule,
                            current.x,
                            current.y,
                            departAt,
                            arrivalAt
                        )
                    ) {
                        continue;
                    }
                }
                else if (
                    !moveSafe(
                        schedule,
                        current.x,
                        current.y,
                        nx,
                        ny,
                        departAt,
                        arrivalAt,
                        start.x,
                        start.y
                    )
                ) {
                    continue;
                }

                const nextHeading =
                    action.wait
                        ? current.heading
                        : action.key;

                const turnCost =
                    pathTurnCost(
                        current.heading,
                        action.key,
                        !!action.wait
                    );

                const g =
                    current.g +
                    1 +
                    turnCost;

                const turns =
                    current.turns +
                    (
                        !action.wait &&
                        current.heading &&
                        action.key !== current.heading
                            ? 1
                            : 0
                    );

                const stateKey =
                    pathStateKey(
                        nx,
                        ny,
                        nextStep,
                        nextHeading
                    );

                const old =
                    best.get(
                        stateKey
                    );

                if (
                    old != null &&
                    old <= g
                ) {
                    continue;
                }

                best.set(
                    stateKey,
                    g
                );

                parents.set(
                    stateKey,
                    {
                        parent: current.stateKey,
                        x: nx,
                        y: ny,
                        key: action.key,
                        wait: !!action.wait,
                        departAt,
                        arrivalAt
                    }
                );

                const h =
                    manhattan(
                        nx,
                        ny,
                        target.x,
                        target.y
                    );

                open.push({
                    x: nx,
                    y: ny,
                    step: nextStep,
                    g,
                    h,
                    f: g + h,
                    heading: nextHeading,
                    turns,
                    stateKey
                });
            }
        }

        return null;
    }

    function findEscapePath(
        start,
        schedule,
        now,
        escapeBomb =
            null
    ) {
        const maxSteps = 30;
        const maxNodes = 2600;
        const stepMs = tileTravelMs();

        const initialHeading =
            BOT.currentKey &&
            'wasd'.includes(BOT.currentKey)
                ? BOT.currentKey
                : null;

        const open =
            new MinHeap(
                (a, b) => {
                    const g = a.g - b.g;
                    if (Math.abs(g) > 1e-9) return g;
                    return a.turns - b.turns;
                }
            );

        const startKey =
            pathStateKey(
                start.x,
                start.y,
                0,
                initialHeading
            );

        open.push({
            x: start.x,
            y: start.y,
            step: 0,
            g: 0,
            turns: 0,
            heading: initialHeading,
            stateKey: startKey
        });

        const best =
            new Map([
                [startKey, 0]
            ]);

        const parents =
            new Map();

        let nodes = 0;

        while (
            open.size &&
            nodes++ < maxNodes
        ) {
            const current =
                open.pop();

            const arrivalAt =
                now +
                current.step *
                stepMs;

            if (current.step > 0) {
                let safeUntil =
                    arrivalAt +
                    1000;

                if (escapeBomb) {
                    safeUntil =
                        Math.max(
                            safeUntil,
                            escapeBomb.explodeAt +
                                BOT.firePostMs +
                                120
                        );
                }

                const safelyParked =
                    safeDuring(
                        schedule,
                        current.x,
                        current.y,
                        arrivalAt,
                        safeUntil
                    );

                const leftBombTile =
                    !escapeBomb ||
                    current.x !== escapeBomb.x ||
                    current.y !== escapeBomb.y;

                if (
                    safelyParked &&
                    leftBombTile
                ) {
                    return reconstructSpaceTimePath(
                        parents,
                        current.stateKey
                    );
                }
            }

            if (
                current.step >= maxSteps
            ) {
                continue;
            }

            const departAt = arrivalAt;
            const nextArrival =
                departAt + stepMs;

            const actions = [
                ...CARDINALS,
                {
                    dx: 0,
                    dy: 0,
                    key: null,
                    wait: true
                }
            ].sort((a, b) => {
                const as =
                    !a.wait && a.key === current.heading
                        ? -1
                        : 0;
                const bs =
                    !b.wait && b.key === current.heading
                        ? -1
                        : 0;
                return as - bs;
            });

            for (const action of actions) {
                const nx = current.x + action.dx;
                const ny = current.y + action.dy;
                const nextStep = current.step + 1;

                if (action.wait) {
                    if (
                        !waitSafe(
                            schedule,
                            current.x,
                            current.y,
                            departAt,
                            nextArrival
                        )
                    ) {
                        continue;
                    }
                }
                else if (
                    !moveSafe(
                        schedule,
                        current.x,
                        current.y,
                        nx,
                        ny,
                        departAt,
                        nextArrival,
                        start.x,
                        start.y
                    )
                ) {
                    continue;
                }

                const nextHeading =
                    action.wait
                        ? current.heading
                        : action.key;

                const turnCost =
                    pathTurnCost(
                        current.heading,
                        action.key,
                        !!action.wait
                    );

                const g =
                    current.g +
                    1 +
                    turnCost;

                const turns =
                    current.turns +
                    (
                        !action.wait &&
                        current.heading &&
                        action.key !== current.heading
                            ? 1
                            : 0
                    );

                const stateKey =
                    pathStateKey(
                        nx,
                        ny,
                        nextStep,
                        nextHeading
                    );

                const old =
                    best.get(stateKey);

                if (
                    old != null &&
                    old <= g
                ) {
                    continue;
                }

                best.set(
                    stateKey,
                    g
                );

                parents.set(
                    stateKey,
                    {
                        parent: current.stateKey,
                        x: nx,
                        y: ny,
                        key: action.key,
                        wait: !!action.wait,
                        departAt,
                        arrivalAt: nextArrival
                    }
                );

                open.push({
                    x: nx,
                    y: ny,
                    step: nextStep,
                    g,
                    turns,
                    heading: nextHeading,
                    stateKey
                });
            }
        }

        return null;
    }


    function pathEndTime(
        path,
        now
    ) {
        if (
            !path ||
            path.length === 0
        ) {
            return now;
        }

        const last =
            path[
                path.length - 1
            ];

        return Number.isFinite(
            Number(
                last.arrivalAt
            )
        )
            ? Number(
                last.arrivalAt
            )
            : now +
                path.length *
                tileTravelMs();
    }

    function countUsefulOpenNeighbors(
        x,
        y
    ) {
        let count =
            0;

        for (
            const dir
            of CARDINALS
        ) {
            const info =
                tileInfo(
                    x + dir.dx,
                    y + dir.dy
                );

            if (
                info.exists &&
                info.walkable
            ) {
                count++;
            }
        }

        return count;
    }

    function simulateBombAt(
        stand,
        radius,
        placedAt,
        now
    ) {
        const fakeId =
            -900000000 -
            (
                BOT.planSerial %
                100000
            );

        const fake = {
            id:
                fakeId,
            ours:
                true,
            pending:
                false,
            confirmed:
                true,
            uncertain:
                false,
            rangeKnown:
                true,
            x:
                stand.x,
            y:
                stand.y,
            radius,
            firstSeenAt:
                placedAt,
            lastVisualAt:
                placedAt,
            placedAt,
            explodeAt:
                placedAt +
                BOT.fuseMs,
            earliestExplodeAt:
                placedAt +
                BOT.fuseMs,
            latestExplodeAt:
                placedAt +
                BOT.fuseMs
        };

        BOT.bombs.set(
            fakeId,
            fake
        );

        const oldEscape =
            BOT.escapeBombId;

        BOT.escapeBombId =
            fakeId;

        const schedule =
            simulateExplosionSchedule(
                now
            );

        const escape =
            findEscapePath(
                {
                    x:
                        stand.x,
                    y:
                        stand.y
                },
                schedule,
                placedAt,
                fake
            );

        BOT.bombs.delete(
            fakeId
        );

        BOT.escapeBombId =
            oldEscape;

        return {
            fake,
            schedule,
            escape
        };
    }

    function findApproachForFocusBlock(
        start,
        block,
        schedule,
        now
    ) {
        if (!block) {
            return null;
        }

        const info =
            tileInfo(
                block.x,
                block.y
            );

        if (
            info.type !==
            'destructible'
        ) {
            return null;
        }

        let best =
            null;

        for (
            const dir
            of CARDINALS
        ) {
            const stand = {
                x:
                    block.x -
                    dir.dx,
                y:
                    block.y -
                    dir.dy
            };

            const standInfo =
                tileInfo(
                    stand.x,
                    stand.y
                );

            if (
                !standInfo.exists ||
                !standInfo.walkable
            ) {
                continue;
            }

            const path =
                findPathToTarget(
                    start,
                    stand,
                    schedule,
                    now,
                    {
                        maxSteps:
                            38,
                        maxNodes:
                            2000
                    }
                );

            if (!path) {
                continue;
            }

            const arriveAt =
                pathEndTime(
                    path,
                    now
                );

            const viability =
                simulateBombAt(
                    stand,
                    predictedNextOwnBombRadius(
                        BOT.playerInfo
                    ),
                    arriveAt,
                    now
                );

            if (
                !viability.escape ||
                viability.escape.length === 0
            ) {
                continue;
            }

            const score =
                path.length *
                    10 +
                viability.escape.length *
                    4 +
                Math.max(
                    0,
                    info.hp -
                    1
                ) *
                    34;

            if (
                !best ||
                score <
                    best.score
            ) {
                best = {
                    block: {
                        x:
                            block.x,
                        y:
                            block.y,
                        hp:
                            info.hp,
                        name:
                            info.name
                    },
                    stand,
                    path,
                    arriveAt,
                    escapePreview:
                        viability.escape,
                    score
                };
            }
        }

        return best;
    }

    // ============================================================
    // GOAL SELECTION
    // ============================================================

    function pickupStillActive(
        plan,
        now
    ) {
        return activePickups(
            now
        ).some(
            pickup =>
                pickup.key ===
                plan.pickupKey
        );
    }

    function choosePickupPlan(
        start,
        schedule,
        now
    ) {
        const pickups =
            activePickups(
                now
            );

        let best =
            null;

        /*
            Usually only a handful of ItemUnits are visible. Run exact
            space-time A* for each and score utility against route length.
        */
        for (
            const pickup
            of pickups
        ) {
            const priority =
                PICKUP_PRIORITY[
                    pickup.name
                ] ??
                0;

            if (
                priority <= 0
            ) {
                continue;
            }

            const path =
                findPathToTarget(
                    start,
                    pickup,
                    schedule,
                    now,
                    {
                        maxSteps:
                            34,
                        maxNodes:
                            1800
                    }
                );

            if (!path) {
                continue;
            }

            const score =
                priority *
                10 -
                path.length *
                12;

            if (
                !best ||
                score >
                    best.score
            ) {
                best = {
                    pickup,
                    path,
                    score
                };
            }
        }

        return best;
    }

    function collectBreakCandidates(
        start,
        radius =
            13
    ) {
        const candidates = [];

        for (
            let y =
                start.y -
                radius;
            y <=
                start.y +
                radius;
            y++
        ) {
            for (
                let x =
                    start.x -
                    radius;
                x <=
                    start.x +
                    radius;
                x++
            ) {
                const info =
                    tileInfo(
                        x,
                        y
                    );

                if (
                    info.type !==
                    'destructible'
                ) {
                    continue;
                }

                candidates.push({
                    x,
                    y,
                    hp:
                        info.hp,
                    name:
                        info.name,
                    rough:
                        manhattan(
                            start.x,
                            start.y,
                            x,
                            y
                        ) +
                        Math.max(
                            0,
                            info.hp -
                            1
                        ) *
                        2.5
                });
            }
        }

        candidates.sort(
            (
                a,
                b
            ) =>
                a.rough -
                b.rough
        );

        return candidates.slice(
            0,
            12
        );
    }


    function chooseBreakPlan(
        start,
        schedule,
        now
    ) {
        /*
            If we already committed to a block and it still exists, keep
            working on it. This is especially important for tough bricks.
        */
        if (
            BOT.focusBlock
        ) {
            const focused =
                findApproachForFocusBlock(
                    start,
                    BOT.focusBlock,
                    schedule,
                    now
                );

            if (focused) {
                focused.score -=
                    80;

                return focused;
            }

            const focusedInfo =
                tileInfo(
                    BOT.focusBlock.x,
                    BOT.focusBlock.y
                );

            if (
                focusedInfo.type !==
                'destructible'
            ) {
                BOT.revealTile = {
                    x:
                        BOT.focusBlock.x,
                    y:
                        BOT.focusBlock.y
                };

                BOT.revealWaitUntil =
                    now +
                    450;

                BOT.focusBlock =
                    null;
            }
        }

        const blocks =
            collectBreakCandidates(
                start,
                14
            );

        /*
            Stage 1: cheap route search. Do not run a full escape simulation for
            every possible block/stand pair.
        */
        const approachCandidates =
            [];

        for (
            const block
            of blocks.slice(
                0,
                10
            )
        ) {
            for (
                const dir
                of CARDINALS
            ) {
                const stand = {
                    x:
                        block.x -
                        dir.dx,
                    y:
                        block.y -
                        dir.dy
                };

                const info =
                    tileInfo(
                        stand.x,
                        stand.y
                    );

                if (
                    !info.exists ||
                    !info.walkable
                ) {
                    continue;
                }

                const path =
                    findPathToTarget(
                        start,
                        stand,
                        schedule,
                        now,
                        {
                            maxSteps:
                                38,
                            maxNodes:
                                1800
                        }
                    );

                if (!path) {
                    continue;
                }

                const hpPenalty =
                    Math.max(
                        0,
                        block.hp -
                        1
                    ) *
                    32;

                const openingBonus =
                    countUsefulOpenNeighbors(
                        block.x,
                        block.y
                    ) *
                    5;

                const regularBrickBonus =
                    block.hp === 1
                        ? 35
                        : 0;

                approachCandidates.push({
                    block,
                    stand,
                    path,
                    roughScore:
                        path.length *
                            10 +
                        hpPenalty -
                        openingBonus -
                        regularBrickBonus
                });
            }
        }

        approachCandidates.sort(
            (
                a,
                b
            ) =>
                a.roughScore -
                b.roughScore
        );

        /*
            Stage 2: only the best-looking approaches are accepted if placing a
            bomb there has a complete time-aware escape route. This prevents
            walking into dead ends merely because a brick is nearby.
        */
        let best =
            null;

        for (
            const candidate
            of approachCandidates.slice(
                0,
                7
            )
        ) {
            const arriveAt =
                pathEndTime(
                    candidate.path,
                    now
                );

            const viability =
                simulateBombAt(
                    candidate.stand,
                    BOT.power,
                    arriveAt,
                    now
                );

            if (
                !viability.escape ||
                viability.escape.length === 0
            ) {
                continue;
            }

            const score =
                candidate.roughScore +
                viability.escape.length *
                    5;

            if (
                !best ||
                score <
                    best.score
            ) {
                best = {
                    block:
                        candidate.block,
                    stand:
                        candidate.stand,
                    path:
                        candidate.path,
                    escapePreview:
                        viability.escape,
                    arriveAt,
                    score
                };
            }
        }

        if (best) {
            BOT.focusBlock = {
                x:
                    best.block.x,
                y:
                    best.block.y,
                hp:
                    best.block.hp,
                name:
                    best.block.name
            };
        }

        return best;
    }

    function adjacentDestructible(
        pose
    ) {
        let best =
            null;

        for (
            const dir
            of CARDINALS
        ) {
            const x =
                pose.tileX +
                dir.dx;

            const y =
                pose.tileY +
                dir.dy;

            const info =
                tileInfo(
                    x,
                    y
                );

            if (
                info.type !==
                'destructible'
            ) {
                continue;
            }

            if (
                !best ||
                info.hp <
                    best.hp
            ) {
                best = {
                    x,
                    y,
                    hp:
                        info.hp,
                    name:
                        info.name
                };
            }
        }

        return best;
    }

    // ============================================================
    // PLAN STATE
    // ============================================================

    function setPlan(
        type,
        path,
        target,
        meta =
            {}
    ) {
        BOT.plan = {
            id:
                ++BOT.planSerial,
            type,
            path:
                path ||
                [],
            index: 0,
            target:
                target
                    ? {
                        x:
                            target.x,
                        y:
                            target.y
                    }
                    : null,
            createdAt:
                performance.now(),
            ...meta
        };

        BOT.drive =
            null;

        BOT.state =
            type;

        return BOT.plan;
    }

    function clearPlan(
        reason =
            ''
    ) {
        releaseMoveKey();

        BOT.plan =
            null;

        BOT.drive =
            null;

        BOT.state =
            'idle';

        BOT.replanReason =
            reason;
    }

    function currentPlanStep() {
        if (
            !BOT.plan
        ) {
            return null;
        }

        return (
            BOT.plan.path[
                BOT.plan.index
            ] ||
            null
        );
    }

    function planComplete() {
        return (
            !BOT.plan ||
            BOT.plan.index >=
                BOT.plan.path.length
        );
    }

    function stepStillSafe(
        pose,
        step,
        schedule,
        now
    ) {
        if (!step) {
            return true;
        }

        if (
            step.wait
        ) {
            return safeDuring(
                schedule,
                pose.tileX,
                pose.tileY,
                now,
                Math.max(
                    now,
                    step.arrivalAt
                ) +
                    50
            );
        }

        if (
            !pathTilePassable(
                step.x,
                step.y
            )
        ) {
            return false;
        }

        if (
            liveBombOccupiesTile(
                schedule,
                step.x,
                step.y,
                now +
                    tileTravelMs()
            )
        ) {
            return false;
        }

        return moveSafe(
            schedule,
            pose.tileX,
            pose.tileY,
            step.x,
            step.y,
            now,
            now +
                tileTravelMs(),
            pose.tileX,
            pose.tileY
        );
    }

    function followCurrentPlan(
        pose,
        schedule,
        now
    ) {
        if (
            !BOT.plan
        ) {
            return false;
        }

        let step =
            currentPlanStep();

        if (!step) {
            releaseMoveKey();
            return false;
        }

        if (
            !stepStillSafe(
                pose,
                step,
                schedule,
                now
            )
        ) {
            clearPlan(
                'next-step-became-unsafe'
            );

            return false;
        }

        if (
            !BOT.drive ||
            BOT.drive.planId !==
                BOT.plan.id ||
            BOT.drive.index !==
                BOT.plan.index
        ) {
            BOT.drive = {
                planId:
                    BOT.plan.id,
                index:
                    BOT.plan.index,
                startedAt:
                    now,
                lastProgressAt:
                    now,
                bestDistance:
                    Infinity,
                alignAxis:
                    null,
                alignSign:
                    0
            };
        }

        const targetX =
            step.wait
                ? pose.x
                : tileCenter(
                    step.x
                );

        const targetY =
            step.wait
                ? pose.y
                : tileCenter(
                    step.y
                );

        const distance =
            step.wait
                ? 0
                : Math.hypot(
                    pose.x -
                        targetX,
                    pose.y -
                        targetY
                );

        if (
            distance +
                0.5 <
            BOT.drive.bestDistance
        ) {
            BOT.drive.bestDistance =
                distance;

            BOT.drive.lastProgressAt =
                now;
        }

        const completed =
            driveTowardStep(
                pose,
                step,
                now
            );

        if (completed) {
            const completedKey =
                step.key;

            BOT.plan.index++;

            const nextStep =
                currentPlanStep();

            BOT.drive =
                null;

            if (
                !nextStep ||
                nextStep.wait ||
                nextStep.key !== completedKey
            ) {
                releaseMoveKey();
            }
            /*
                Otherwise keep the same WASD key held continuously through the
                next straight cell. This removes a key-up/key-down pulse at
                every tile boundary and makes long corridors genuinely straight.
            */

            return (
                !planComplete()
            );
        }

        const timeout =
            Math.max(
                700,
                tileTravelMs() *
                2.4
            );

        if (
            now -
                BOT.drive.lastProgressAt >
            timeout
        ) {
            clearPlan(
                'movement-stuck'
            );

            return false;
        }

        return true;
    }

    // ============================================================
    // PICKUP COLLECTION
    // ============================================================

    function beginPickupCollectionWatch(
        pickup
    ) {
        BOT.pendingPickupCollection = {
            key:
                pickup.key,
            name:
                pickup.name,
            x:
                pickup.x,
            y:
                pickup.y,
            startedAt:
                performance.now()
        };
    }


    function applyCollectedPickup(
        name
    ) {
        /*
            getPlayerInfo() exposes exact named inventory/stat values. Force a
            refresh after a pickup instead of guessing speed/range increments.
        */
        const before =
            BOT.playerInfo;

        setTimeout(
            () => {
                const after =
                    readAuthoritativePlayerInfo(
                        true
                    );

                if (
                    after &&
                    Number(after.power) > 0
                ) {
                    BOT.power =
                        Math.max(
                            1,
                            Math.floor(
                                Number(
                                    after.power
                                )
                            )
                        );

                    BOT.bombSuperCount =
                        Math.max(
                            0,
                            Math.floor(
                                Number(
                                    after.bomb_super
                                ) ||
                                0
                            )
                        );

                    BOT.powerSource =
                        'getPlayerInfo after pickup';
                }
                else if (
                    name ===
                    'power'
                ) {
                    // Fallback only when the API object is unavailable.
                    BOT.power++;
                    BOT.powerSource =
                        'pickup fallback';
                }

                if (
                    name ===
                    'bomb'
                ) {
                    BOT.bombCapacityPickups++;
                }
                else if (
                    name ===
                    'scate'
                ) {
                    BOT.speedPickups++;
                }
            },
            80
        );

        if (BOT.debug) {
            console.log(
                '[GOB Bot v4] collected',
                name,
                {
                    before
                }
            );
        }
    }

    function updatePickupCollectionWatch(
        pose,
        now
    ) {
        const pending =
            BOT.pendingPickupCollection;

        if (!pending) {
            return;
        }

        const stillThere =
            exactWorldObservations.pickups.has(
                pending.key
            );

        if (
            stillThere
        ) {
            if (
                now -
                    pending.startedAt >
                    2500
            ) {
                BOT.pendingPickupCollection =
                    null;
            }

            return;
        }

        const distance =
            Math.hypot(
                pose.x -
                    tileCenter(
                        pending.x
                    ),
                pose.y -
                    tileCenter(
                        pending.y
                    )
            );

        if (
            distance <= 20
        ) {
            applyCollectedPickup(
                pending.name
            );
        }

        BOT.pendingPickupCollection =
            null;
    }

    // ============================================================
    // AUTO RESPAWN
    // ============================================================

    function isVisibleElement(el) {
        if (!el) {
            return false;
        }

        const style =
            getComputedStyle(
                el
            );

        if (
            style.display ===
                'none' ||
            style.visibility ===
                'hidden' ||
            Number(
                style.opacity
            ) === 0
        ) {
            return false;
        }

        const rect =
            el.getBoundingClientRect();

        return (
            rect.width > 0 &&
            rect.height > 0
        );
    }

    function tryAutoRespawn(
        now
    ) {
        if (
            !BOT.enabled ||
            !BOT.autoRespawn ||
            now -
                BOT.lastRespawnCheck <
                BOT.respawnCheckEveryMs
        ) {
            return false;
        }

        BOT.lastRespawnCheck =
            now;

        const exact =
            document.querySelector(
                'button.button_style.buy_border.ng-binding[ng-click="commandSpace()"]'
            );

        let button =
            null;

        if (
            exact &&
            /^respawn\b/i.test(
                String(
                    exact.innerText ||
                    exact.textContent ||
                    ''
                ).trim()
            ) &&
            isVisibleElement(
                exact
            )
        ) {
            button =
                exact;
        }

        if (!button) {
            button =
                [
                    ...document.querySelectorAll(
                        'button[ng-click="commandSpace()"]'
                    )
                ].find(
                    el =>
                        /^respawn\b/i.test(
                            String(
                                el.innerText ||
                                el.textContent ||
                                ''
                            ).trim()
                        ) &&
                        isVisibleElement(
                            el
                        )
                ) ||
                null;
        }

        if (!button) {
            return false;
        }

        releaseEveryBotKey();
        clearPlan(
            'respawn'
        );

        try {
            button.click();
        } catch {
            button.dispatchEvent(
                new MouseEvent(
                    'click',
                    {
                        bubbles:
                            true,
                        cancelable:
                            true
                    }
                )
            );
        }

        BOT.bombs.clear();
        BOT.pendingOwnBomb =
            null;
        BOT.escapeBombId =
            null;
        BOT.pendingPickupCollection =
            null;
        BOT.lastPlayerWorld =
            null;
        BOT.lastMotionSample =
            null;

        BOT.focusBlock =
            null;

        BOT.revealWaitUntil =
            0;

        BOT.revealTile =
            null;

        BOT.playerInfo =
            null;

        BOT.playerInfoAt =
            0;

        /*
            Do not guess the respawned power if the native API is available;
            it is refreshed on the next think. Keep 2 only as fallback.
        */
        BOT.power = 2;
        BOT.powerSource =
            'respawn fallback';

        return true;
    }

    // ============================================================
    // DECISION MAKING
    // ============================================================

    function makeStartNode(
        pose
    ) {
        return {
            x:
                pose.tileX,
            y:
                pose.tileY
        };
    }


    function currentTileThreatenedSoon(
        pose,
        schedule,
        now
    ) {
        /*
            700ms was effectively "wait until the last moment". Give the bot
            enough time to clear several tiles before the earliest plausible
            blast, while still using the schedule's actual timing intervals.
        */
        const lookahead =
            Math.max(
                1800,
                tileTravelMs() *
                    7
            );

        return (
            !safeDuring(
                schedule,
                pose.tileX,
                pose.tileY,
                now,
                now +
                    lookahead
            )
        );
    }

    function planEscape(
        pose,
        schedule,
        now,
        type =
            'escape'
    ) {
        const start =
            makeStartNode(
                pose
            );

        const escapeBomb =
            BOT.escapeBombId != null
                ? BOT.bombs.get(
                    BOT.escapeBombId
                ) ||
                    null
                : null;

        const path =
            findEscapePath(
                start,
                schedule,
                now,
                escapeBomb
            );

        if (!path) {
            return false;
        }

        const last =
            path[
                path.length -
                1
            ] ||
            start;

        setPlan(
            type,
            path,
            last,
            {
                bombId:
                    escapeBomb?.id ??
                    null
            }
        );

        return true;
    }


    function simulateOwnBombPlacement(
        pose,
        now
    ) {
        const info =
            readAuthoritativePlayerInfo();

        /*
            This is intentionally the conservative SAFETY radius.
            It can be one tile longer than the expected visible footprint
            while bomb_super eligibility is unresolved. This simulation exists
            only to answer "can I escape if the worst case happens?"
        */
        const radius =
            predictedNextOwnBombRadius(
                info
            );

        const fake = {
            id:
                -999999,
            ours:
                true,
            confirmed:
                true,
            uncertain:
                false,
            rangeKnown:
                true,
            x:
                pose.tileX,
            y:
                pose.tileY,
            radius,
            firstSeenAt:
                now,
            lastVisualAt:
                now,
            placedAt:
                now,
            explodeAt:
                now +
                BOT.fuseMs,
            seenNow:
                true
        };

        BOT.bombs.set(
            fake.id,
            fake
        );

        const oldEscape =
            BOT.escapeBombId;

        BOT.escapeBombId =
            fake.id;

        const schedule =
            simulateExplosionSchedule(
                now
            );

        const path =
            findEscapePath(
                makeStartNode(
                    pose
                ),
                schedule,
                now,
                fake
            );

        BOT.bombs.delete(
            fake.id
        );

        BOT.escapeBombId =
            oldEscape;

        return {
            fake,
            schedule,
            path
        };
    }


    function maybeCreateNewGoal(
        pose,
        schedule,
        now
    ) {
        const start =
            makeStartNode(
                pose
            );

        /*
            1. A real ItemUnit is always productive. It may have just been
            revealed by the block we destroyed.
        */
        const pickupGoal =
            choosePickupPlan(
                start,
                schedule,
                now
            );

        if (
            pickupGoal
        ) {
            setPlan(
                'pickup',
                pickupGoal.path,
                pickupGoal.pickup,
                {
                    pickupKey:
                        pickupGoal.pickup.key,
                    pickupName:
                        pickupGoal.pickup.name
                }
            );

            return true;
        }

        /*
            Hidden items can appear a few frames after terrain changes. When a
            focused block disappears, briefly wait at a safe location for the
            real items.png draw before selecting another brick.
        */
        if (
            BOT.revealWaitUntil >
                now
        ) {
            BOT.state =
                'waiting-for-reveal';

            releaseMoveKey();

            return false;
        }

        if (
            BOT.revealWaitUntil &&
            BOT.revealWaitUntil <=
                now
        ) {
            BOT.revealWaitUntil =
                0;

            BOT.revealTile =
                null;
        }

        /*
            2. If centered beside the focused destructible block, bomb it only
            after simulating the bomb and proving a complete escape route.
        */
        let adjacent =
            null;

        if (
            BOT.focusBlock &&
            manhattan(
                pose.tileX,
                pose.tileY,
                BOT.focusBlock.x,
                BOT.focusBlock.y
            ) === 1 &&
            tileInfo(
                BOT.focusBlock.x,
                BOT.focusBlock.y
            ).type ===
                'destructible'
        ) {
            adjacent = {
                ...BOT.focusBlock
            };
        }
        else {
            adjacent =
                adjacentDestructible(
                    pose
                );
        }

        if (
            adjacent &&
            playerNearTileCenter(
                pose
            )
        ) {
            const simulation =
                simulateOwnBombPlacement(
                    pose,
                    now
                );

            if (
                simulation.path &&
                simulation.path.length
            ) {
                const requested =
                    placeOwnBomb(
                        pose,
                        now
                    );

                if (
                    requested
                ) {
                    /*
                        The pending bomb is intentionally not a confirmed bomb
                        record yet. Keep the already-proven escape path and let
                        refreshBombRegistry() confirm the visual on the exact
                        tile. If confirmation fails, the escape goal is cleared.
                    */
                    setPlan(
                        'escape-own-bomb',
                        simulation.path,
                        simulation.path[
                            simulation.path.length -
                            1
                        ] ||
                        start,
                        {
                            bombId:
                                requested.id,
                            attackedBlock:
                                adjacent
                        }
                    );

                    /*
                        Do NOT assign simulation.schedule to BOT.lastSchedule.

                        simulation.schedule uses the conservative safety radius
                        by design. Showing/replanning against that temporary
                        envelope caused the yellow maximum-range flicker and
                        plan restarts. The next normal bot tick builds the live
                        schedule from pending.radius (expected strength), and
                        confirmation adjusts it only if the real bomb proves a
                        special +1 modifier.
                    */
                    return true;
                }
            }
        }

        /*
            3. Productive block destruction. Candidate approaches are accepted
            only if a bomb placed at the eventual stand tile has a safe escape.
        */
        const breakGoal =
            chooseBreakPlan(
                start,
                schedule,
                now
            );

        if (
            breakGoal
        ) {
            BOT.focusBlock = {
                x:
                    breakGoal.block.x,
                y:
                    breakGoal.block.y,
                hp:
                    breakGoal.block.hp,
                name:
                    breakGoal.block.name
            };

            setPlan(
                'attack',
                breakGoal.path,
                breakGoal.stand,
                {
                    block:
                        breakGoal.block,
                    escapePreview:
                        breakGoal.escapePreview
                }
            );

            return true;
        }

        /*
            No productive goal. Normally stop. If physical movement has been
            absent for ~4.2 seconds, perform one SAFE purposeful reposition.
            This keeps the bot from sitting inert indefinitely without turning
            it into random wandering.
        */
        const idleFor =
            now -
            (
                BOT.lastPhysicalMoveAt ||
                now
            );

        if (
            idleFor >=
                BOT.idleMoveAfterMs
        ) {
            const reposition =
                chooseSafeRepositionPlan(
                    start,
                    schedule,
                    now
                );

            if (reposition) {
                setPlan(
                    'reposition',
                    reposition.path,
                    reposition.target,
                    {
                        antiIdle:
                            true
                    }
                );

                return true;
            }
        }

        BOT.state =
            'idle-no-productive-goal';

        releaseMoveKey();

        return false;
    }


    function validateCurrentGoal(
        pose,
        schedule,
        now
    ) {
        if (
            !BOT.plan
        ) {
            return false;
        }

        if (
            BOT.plan.type ===
            'pickup'
        ) {
            if (
                pickupStillActive(
                    BOT.plan,
                    now
                )
            ) {
                return true;
            }

            const atTarget =
                BOT.plan.target &&
                Math.hypot(
                    pose.x -
                        tileCenter(
                            BOT.plan.target.x
                        ),
                    pose.y -
                        tileCenter(
                            BOT.plan.target.y
                        )
                ) <=
                    20;

            if (
                atTarget
            ) {
                beginPickupCollectionWatch(
                    {
                        key:
                            BOT.plan.pickupKey,
                        name:
                            BOT.plan.pickupName,
                        x:
                            BOT.plan.target.x,
                        y:
                            BOT.plan.target.y
                    }
                );
            }

            clearPlan(
                'pickup-disappeared'
            );

            return false;
        }

        if (
            BOT.plan.type ===
            'attack'
        ) {
            const block =
                BOT.plan.block;

            if (!block) {
                clearPlan(
                    'attack-no-block'
                );

                return false;
            }

            const info =
                tileInfo(
                    block.x,
                    block.y
                );

            if (
                info.type !==
                'destructible'
            ) {
                BOT.focusBlock =
                    null;

                BOT.revealTile = {
                    x:
                        block.x,
                    y:
                        block.y
                };

                BOT.revealWaitUntil =
                    now +
                    450;

                clearPlan(
                    'block-opened-wait-for-item'
                );

                return false;
            }

            BOT.focusBlock = {
                x:
                    block.x,
                y:
                    block.y,
                hp:
                    info.hp,
                name:
                    info.name
            };

            return true;
        }

        if (
            BOT.plan.type ===
                'escape-own-bomb' ||
            BOT.plan.type ===
                'escape'
        ) {
            return true;
        }

        return true;
    }


    function finishCompletedPlan(
        pose,
        schedule,
        now
    ) {
        if (
            !BOT.plan ||
            !planComplete()
        ) {
            return false;
        }

        const plan =
            BOT.plan;

        releaseMoveKey();

        if (
            plan.type ===
            'pickup'
        ) {
            beginPickupCollectionWatch(
                {
                    key:
                        plan.pickupKey,
                    name:
                        plan.pickupName,
                    x:
                        plan.target.x,
                    y:
                        plan.target.y
                }
            );

            clearPlan(
                'pickup-target-reached'
            );

            return true;
        }

        if (
            plan.type ===
            'attack'
        ) {
            if (
                plan.block
            ) {
                BOT.focusBlock = {
                    ...plan.block
                };
            }

            clearPlan(
                'attack-position-reached'
            );

            return true;
        }

        if (
            plan.type ===
                'escape-own-bomb' ||
            plan.type ===
                'escape'
        ) {
            const safeNow =
                safeDuring(
                    schedule,
                    pose.tileX,
                    pose.tileY,
                    now,
                    now +
                        850
                );

            if (
                safeNow
            ) {
                BOT.escapeBombId =
                    null;

                const attacked =
                    plan.attackedBlock ||
                    null;

                clearPlan(
                    'escape-complete'
                );

                if (
                    attacked
                ) {
                    const info =
                        tileInfo(
                            attacked.x,
                            attacked.y
                        );

                    if (
                        info.type ===
                        'destructible'
                    ) {
                        /*
                            Tough block survived. Keep it as the productive
                            objective for the next bomb cycle.
                        */
                        BOT.focusBlock = {
                            x:
                                attacked.x,
                            y:
                                attacked.y,
                            hp:
                                info.hp,
                            name:
                                info.name
                        };
                    }
                    else {
                        BOT.focusBlock =
                            null;

                        BOT.revealTile = {
                            x:
                                attacked.x,
                            y:
                                attacked.y
                        };

                        BOT.revealWaitUntil =
                            now +
                            450;
                    }
                }

                return true;
            }

            clearPlan(
                'escape-ended-but-still-dangerous'
            );

            return true;
        }

        clearPlan(
            'plan-complete'
        );

        return true;
    }



    function rebasePointByAliasShift(
        point,
        dx,
        dy
    ) {
        if (
            !point ||
            typeof point !==
                'object'
        ) {
            return;
        }

        if (
            Number.isFinite(
                Number(
                    point.x
                )
            )
        ) {
            point.x +=
                dx;
        }

        if (
            Number.isFinite(
                Number(
                    point.y
                )
            )
        ) {
            point.y +=
                dy;
        }
    }

    function rebasePlannerForAliasShift(
        dx,
        dy
    ) {
        if (
            dx === 0 &&
            dy === 0
        ) {
            return;
        }

        if (
            Math.abs(dx) %
                WORLD_WRAP_TILES !==
                0 ||
            Math.abs(dy) %
                WORLD_WRAP_TILES !==
                0
        ) {
            return;
        }

        const plan =
            BOT.plan;

        if (plan) {
            for (
                const step
                of plan.path ||
                []
            ) {
                rebasePointByAliasShift(
                    step,
                    dx,
                    dy
                );
            }

            rebasePointByAliasShift(
                plan.target,
                dx,
                dy
            );

            rebasePointByAliasShift(
                plan.block,
                dx,
                dy
            );

            rebasePointByAliasShift(
                plan.attackedBlock,
                dx,
                dy
            );
        }

        rebasePointByAliasShift(
            BOT.focusBlock,
            dx,
            dy
        );

        rebasePointByAliasShift(
            BOT.revealTile,
            dx,
            dy
        );

        rebasePointByAliasShift(
            BOT.pendingPickupCollection,
            dx,
            dy
        );

        /*
            Bombs have canonical identity and are reprojected separately.
        */
        BOT.drive =
            null;

        BOT.replanReason =
            'renderer-alias-rebased';
    }

    function detectAndHandleAliasShift(
        pose
    ) {
        const previous =
            BOT.aliasPose ||
            null;

        BOT.aliasPose = {
            tileX:
                pose.tileX,
            tileY:
                pose.tileY
        };

        if (!previous) {
            return;
        }

        const rawDX =
            pose.tileX -
            previous.tileX;

        const rawDY =
            pose.tileY -
            previous.tileY;

        const aliasDX =
            Math.abs(rawDX) >=
                WORLD_WRAP_TILES -
                    1
                ? Math.round(
                    rawDX /
                    WORLD_WRAP_TILES
                ) *
                    WORLD_WRAP_TILES
                : 0;

        const aliasDY =
            Math.abs(rawDY) >=
                WORLD_WRAP_TILES -
                    1
                ? Math.round(
                    rawDY /
                    WORLD_WRAP_TILES
                ) *
                    WORLD_WRAP_TILES
                : 0;

        if (
            aliasDX ||
            aliasDY
        ) {
            rebasePlannerForAliasShift(
                aliasDX,
                aliasDY
            );

            /*
                Reset speed/activity samples so the coordinate-frame jump is
                never misread as physical movement.
            */
            BOT.lastMotionSample =
                null;

            BOT.lastPhysicalPose =
                null;
        }
    }

    function botThink() {
        if (
            window.__GOB_ACTIVE_BOT_VERSION__ !==
                '5.2.0'
        ) {
            releaseMoveKey();
            return;
        }

        if (
            !BOT.enabled
        ) {
            releaseMoveKey();
            return;
        }

        const now =
            performance.now();

        if (
            now -
                BOT.lastThinkAt <
            BOT.thinkEveryMs
        ) {
            return;
        }

        BOT.lastThinkAt =
            now;

        readAuthoritativePlayerInfo();

        if (
            tryAutoRespawn(
                now
            )
        ) {
            return;
        }

        const pose =
            getPlayerPose();

        if (!pose) {
            releaseMoveKey();
            return;
        }

        detectAndHandleAliasShift(
            pose
        );

        updateActivityWatchdog(
            pose,
            now
        );

        /*
            v3.1's renderer-derived pose is the movement authority.
            getPlayerInfo() remains useful for stats/alive state, but it is
            deliberately NOT allowed to replace x/y/tileX/tileY used by the
            movement controller.
        */
        const authoritativeInfo =
            BOT.playerInfo;

        if (
            authoritativeInfo &&
            authoritativeInfo.alive === false
        ) {
            releaseMoveKey();
            return;
        }

        updateMeasuredSpeed(
            pose,
            now
        );

        periodicStuckKeyCheck(
            now
        );

        cleanupExactObservations(
            now
        );

        refreshBombRegistry(
            now
        );

        updatePickupCollectionWatch(
            pose,
            now
        );

        const schedule =
            simulateExplosionSchedule(
                now
            );

        BOT.lastSchedule =
            schedule;

        /*
            Safety has absolute priority. A route is interrupted only when the
            current tile/crossing becomes genuinely unsafe in the schedule.
        */
        const threatened =
            currentTileThreatenedSoon(
                pose,
                schedule,
                now
            );

        if (
            threatened &&
            (
                !BOT.plan ||
                !(
                    BOT.plan.type ===
                        'escape' ||
                    BOT.plan.type ===
                        'escape-own-bomb'
                )
            )
        ) {
            clearPlan(
                'emergency-danger'
            );

            if (
                planEscape(
                    pose,
                    schedule,
                    now,
                    'escape'
                )
            ) {
                followCurrentPlan(
                    pose,
                    schedule,
                    now
                );

                return;
            }

            /*
                If no safe route exists, do not jitter between equally bad
                neighbors. Stop and let the next authoritative state update
                decide whether a route opens.
            */
            releaseMoveKey();

            BOT.state =
                'trapped-no-safe-route';

            return;
        }

        if (
            BOT.plan
        ) {
            validateCurrentGoal(
                pose,
                schedule,
                now
            );
        }

        if (
            BOT.plan
        ) {
            const running =
                followCurrentPlan(
                    pose,
                    schedule,
                    now
                );

            if (running) {
                return;
            }

            if (
                BOT.plan &&
                planComplete()
            ) {
                finishCompletedPlan(
                    pose,
                    schedule,
                    now
                );
            }

            if (
                BOT.plan
            ) {
                return;
            }
        }

        maybeCreateNewGoal(
            pose,
            schedule,
            now
        );

        if (
            BOT.plan
        ) {
            followCurrentPlan(
                pose,
                BOT.lastSchedule ||
                    schedule,
                now
            );
        }
    }

    function botLoop() {
        requestAnimationFrame(
            botLoop
        );

        try {
            botThink();
        } catch (error) {
            console.error(
                '[GOB Bot v5.2]',
                error
            );

            releaseMoveKey();
        }
    }

    // ============================================================
    // DANGER OVERLAY
    // ============================================================

    function ensureDangerOverlay() {
        if (
            BOT.dangerOverlay &&
            BOT.dangerOverlay.isConnected
        ) {
            return true;
        }

        if (
            !findLayers()
        ) {
            return false;
        }

        const canvas =
            document.createElement(
                'canvas'
            );

        canvas.id =
            'gob-bomb-danger-overlay';

        canvas.style.cssText = `
            position: fixed;
            pointer-events: none;
            z-index: 2147483640;
            background: transparent;
        `;

        document.body.appendChild(
            canvas
        );

        BOT.dangerOverlay =
            canvas;

        BOT.dangerCtx =
            canvas.getContext(
                '2d'
            );

        return true;
    }

    function syncDangerOverlay() {
        if (
            !BOT.dangerOverlay ||
            !layer1
        ) {
            return false;
        }

        const rect =
            layer1.getBoundingClientRect();

        const canvas =
            BOT.dangerOverlay;

        canvas.style.left =
            rect.left +
            'px';

        canvas.style.top =
            rect.top +
            'px';

        canvas.style.width =
            rect.width +
            'px';

        canvas.style.height =
            rect.height +
            'px';

        if (
            canvas.width !==
            layer1.width
        ) {
            canvas.width =
                layer1.width;
        }

        if (
            canvas.height !==
            layer1.height
        ) {
            canvas.height =
                layer1.height;
        }

        return true;
    }


    function drawWorldCell(
        ctx,
        x,
        y,
        fillStyle,
        alpha
    ) {
        if (
            !worldMatrix
        ) {
            return;
        }

        /*
            A wrong box is worse than a missing box. The terrain renderer
            refreshes continuously while the game is visible; if we somehow
            have not seen a terrain matrix recently, do not render using a
            stale transform.
        */
        if (
            worldMatrixSeenAt &&
            performance.now() -
                worldMatrixSeenAt >
                750
        ) {
            return;
        }

        const reference =
            plannerReferenceTile();

        /*
            Schedule coordinates are normally already localized. Reproject
            only when they are an obvious 32-tile alias away from the current
            player. This prevents a logical cell from being drawn on the wrong
            wrapped copy of the map.
        */
        let drawX =
            Number(x);

        let drawY =
            Number(y);

        if (
            Number.isFinite(drawX) &&
            Math.abs(
                drawX -
                reference.x
            ) >
                WORLD_WRAP_TILES /
                    2
        ) {
            drawX =
                nearestWrappedAlias(
                    drawX,
                    reference.x
                );
        }

        if (
            Number.isFinite(drawY) &&
            Math.abs(
                drawY -
                reference.y
            ) >
                WORLD_WRAP_TILES /
                    2
        ) {
            drawY =
                nearestWrappedAlias(
                    drawY,
                    reference.y
                );
        }

        const p1 =
            transformPoint(
                worldMatrix,
                drawX *
                    TILE_SIZE,
                drawY *
                    TILE_SIZE
            );

        const p2 =
            transformPoint(
                worldMatrix,
                (
                    drawX +
                    1
                ) *
                    TILE_SIZE,
                (
                    drawY +
                    1
                ) *
                    TILE_SIZE
            );

        const left =
            Math.min(
                p1.x,
                p2.x
            );

        const top =
            Math.min(
                p1.y,
                p2.y
            );

        const width =
            Math.abs(
                p2.x -
                p1.x
            );

        const height =
            Math.abs(
                p2.y -
                p1.y
            );

        /*
            Sanity guard against accidentally using a local object transform.
            A gameplay tile should stay roughly tile-sized on screen, not
            hundreds/thousands of pixels wide.
        */
        if (
            !Number.isFinite(left) ||
            !Number.isFinite(top) ||
            !Number.isFinite(width) ||
            !Number.isFinite(height) ||
            width < 4 ||
            height < 4 ||
            width > 128 ||
            height > 128
        ) {
            return;
        }

        ctx.globalAlpha =
            alpha;

        ctx.fillStyle =
            fillStyle;

        ctx.fillRect(
            left,
            top,
            width,
            height
        );

        ctx.globalAlpha =
            1;

        ctx.strokeStyle =
            'rgba(255,255,255,0.42)';

        ctx.lineWidth =
            1;

        ctx.strokeRect(
            left +
                0.5,
            top +
                0.5,
            Math.max(
                0,
                width -
                    1
            ),
            Math.max(
                0,
                height -
                    1
            )
        );
    }

    function renderDangerOverlay() {
        requestAnimationFrame(
            renderDangerOverlay
        );

        const now =
            performance.now();

        if (
            window.__GOB_ACTIVE_BOT_VERSION__ !==
                '5.2.0'
        ) {
            return;
        }

        if (
            now -
                BOT.lastOverlayDrawAt <
            33
        ) {
            return;
        }

        BOT.lastOverlayDrawAt =
            now;

        if (
            !BOT.enabled ||
            !BOT.lastSchedule ||
            !worldMatrix
        ) {
            if (
                BOT.dangerCtx &&
                BOT.dangerOverlay
            ) {
                BOT.dangerCtx.clearRect(
                    0,
                    0,
                    BOT.dangerOverlay.width,
                    BOT.dangerOverlay.height
                );
            }

            return;
        }

        if (
            !ensureDangerOverlay() ||
            !syncDangerOverlay()
        ) {
            return;
        }

        const ctx =
            BOT.dangerCtx;

        ctx.setTransform(
            1,
            0,
            0,
            1,
            0,
            0
        );

        ctx.clearRect(
            0,
            0,
            BOT.dangerOverlay.width,
            BOT.dangerOverlay.height
        );

        /*
            Collapse duplicate cells, preferring actual > exact > uncertain.
        */
        const cells =
            new Map();

        const rank = {
            uncertain: 1,
            predicted: 2,
            exact: 3,
            actual: 4
        };

        for (
            const entry
            of BOT.lastSchedule.overlay
        ) {
            if (
                entry.end <
                now
            ) {
                continue;
            }

            const key =
                canonicalTileKey(
                    entry.x,
                    entry.y
                );

            const old =
                cells.get(
                    key
                );

            if (
                !old ||
                rank[
                    entry.kind
                ] >
                rank[
                    old.kind
                ]
            ) {
                cells.set(
                    key,
                    entry
                );
            }
        }

        for (
            const cell
            of cells.values()
        ) {
            if (
                cell.kind ===
                'uncertain'
            ) {
                drawWorldCell(
                    ctx,
                    cell.x,
                    cell.y,
                    'rgb(255,190,40)',
                    0.34
                );
            }
            else if (
                cell.kind ===
                'predicted'
            ) {
                const until =
                    Math.max(
                        0,
                        cell.start -
                            now
                    );

                const urgency =
                    1 -
                    Math.min(
                        1,
                        until /
                            BOT.fuseMs
                    );

                drawWorldCell(
                    ctx,
                    cell.x,
                    cell.y,
                    'rgb(255,75,75)',
                    0.24 +
                        urgency *
                        0.20
                );
            }
            else if (
                cell.kind ===
                'actual'
            ) {
                drawWorldCell(
                    ctx,
                    cell.x,
                    cell.y,
                    'rgb(255,35,35)',
                    0.62
                );
            }
            else {
                const until =
                    Math.max(
                        0,
                        cell.start -
                            now
                    );

                const urgency =
                    1 -
                    Math.min(
                        1,
                        until /
                            BOT.fuseMs
                    );

                drawWorldCell(
                    ctx,
                    cell.x,
                    cell.y,
                    'rgb(255,55,55)',
                    0.28 +
                        urgency *
                        0.24
                );
            }
        }
    }

    // ============================================================
    // PUBLIC COMMANDS
    // ============================================================

    window.gobBotStart =
        function() {
            window.__GOB_ACTIVE_BOT_VERSION__ =
                '5.2.0';

            BOT.enabled =
                true;

            BOT.lastPhysicalMoveAt =
                performance.now();

            BOT.state =
                'idle';

            releaseEveryBotKey();

            console.log(
                '[GOB Bot v5.2] ON'
            );

            return true;
        };

    window.gobBotStop =
        function() {
            BOT.enabled =
                false;

            clearPlan(
                'stopped'
            );

            releaseEveryBotKey();

            console.log(
                '[GOB Bot v5.2] OFF'
            );

            return false;
        };

    window.gobBotToggle =
        function() {
            return BOT.enabled
                ? window.gobBotStop()
                : window.gobBotStart();
        };

    window.gobSetBlastRadius =
        function(radius) {
            const value =
                Math.max(
                    1,
                    Math.floor(
                        Number(
                            radius
                        ) || 1
                    )
                );

            BOT.power =
                value;

            console.log(
                '[GOB Bot v5.2] manual fallback bomb power =',
                value
            );

            return value;
        };

    window.gobPowerups =
        function() {
            const list =
                activePickups(
                    performance.now()
                );

            console.table(
                list
            );

            return list;
        };

    window.gobWorldTraceDebug =
        function() {
            const offset =
                activeModelRenderOffset();

            const rows =
                [
                    'layer0',
                    'layer1'
                ].map(
                    id => {
                        const canvas =
                            document.getElementById(
                                id
                            );

                        return {
                            id,
                            visible:
                                isCanvasActivelyVisible(
                                    canvas
                                ),
                            connected:
                                !!canvas?.isConnected,
                            width:
                                canvas?.width ??
                                null,
                            height:
                                canvas?.height ??
                                null
                        };
                    }
                );

            console.table(
                rows
            );

            const result = {
                modelToRenderOffset:
                    offset,
                visibleBombObservations:
                    exactWorldObservations.bombs.size,
                visibleBlastCells:
                    exactWorldObservations.blasts.size,
                trackedBombs:
                    BOT.bombs.size
            };

            console.log(
                '[GOB active-world trace]',
                result
            );

            return result;
        };

    window.gobDisplayDebug =
        function() {
            const now =
                performance.now();

            const matrix =
                worldMatrix
                    ? {
                        a:
                            worldMatrix.a,
                        b:
                            worldMatrix.b,
                        c:
                            worldMatrix.c,
                        d:
                            worldMatrix.d,
                        e:
                            worldMatrix.e,
                        f:
                            worldMatrix.f
                    }
                    : null;

            const result = {
                matrixSource:
                    worldMatrixSource,

                matrixLayer:
                    worldMatrixLayer,

                matrixAgeMs:
                    worldMatrixSeenAt
                        ? Math.round(
                            now -
                            worldMatrixSeenAt
                        )
                        : null,

                matrix,

                overlayCells:
                    BOT.lastSchedule?.overlay?.length ??
                    0,

                bombs:
                    BOT.bombs.size,

                capturedModels:
                    bombModelHookState()?.records?.length ??
                    0
            };

            console.log(
                '[GOB display]',
                result
            );

            return result;
        };

    window.gobActivityDebug =
        function() {
            const now =
                performance.now();

            const result = {
                state:
                    BOT.state,
                currentKey:
                    BOT.currentKey,
                idleForMs:
                    Math.round(
                        now -
                        (
                            BOT.lastPhysicalMoveAt ||
                            now
                        )
                    ),
                idleThresholdMs:
                    BOT.idleMoveAfterMs,
                planType:
                    BOT.plan?.type ??
                    null,
                planIndex:
                    BOT.plan?.index ??
                    null,
                planLength:
                    BOT.plan?.path?.length ??
                    null,
                lastReplanReason:
                    BOT.replanReason,
                matrixAgeMs:
                    worldMatrixSeenAt
                        ? Math.round(
                            now -
                            worldMatrixSeenAt
                        )
                        : null,
                matrixLayer:
                    worldMatrixLayer
            };

            console.table(
                [result]
            );

            return result;
        };

    window.gobBombs =
        function() {
            const list =
                [
                    ...BOT.bombs.values()
                ].map(
                    bomb => ({
                        id:
                            bomb.id,
                        x:
                            bomb.x,
                        y:
                            bomb.y,
                        ours:
                            bomb.ours,
                        confirmed:
                            bomb.confirmed,
                        radius:
                            bomb.radius,
                        rangeKnown:
                            bomb.rangeKnown !==
                                false,
                        canonicalX:
                            bomb.canonicalX ??
                            canonicalTileCoord(
                                bomb.x
                            ),
                        canonicalY:
                            bomb.canonicalY ??
                            canonicalTileCoord(
                                bomb.y
                            ),
                        modelStrength:
                            Number.isFinite(
                                Number(
                                    bomb.modelRef?.s
                                )
                            )
                                ? Number(
                                    bomb.modelRef.s
                                )
                                : null,
                        modelN:
                            Number.isFinite(
                                Number(
                                    bomb.modelRef?.N
                                )
                            )
                                ? Number(
                                    bomb.modelRef.N
                                )
                                : null,
                        msToExplosion:
                            bomb.ours
                                ? Math.round(
                                    bomb.explodeAt -
                                    performance.now()
                                )
                                : null,

                        earliestMs:
                            bomb.ours
                                ? Math.round(
                                    bomb.explodeAt -
                                    performance.now()
                                )
                                : Math.round(
                                    (
                                        bomb.earliestExplodeAt ??
                                        performance.now()
                                    ) -
                                    performance.now()
                                ),

                        latestMs:
                            bomb.ours
                                ? Math.round(
                                    bomb.explodeAt -
                                    performance.now()
                                )
                                : Math.round(
                                    (
                                        bomb.latestExplodeAt ??
                                        (
                                            bomb.firstSeenAt +
                                            BOT.fuseMs
                                        )
                                    ) -
                                    performance.now()
                                )
                    })
                );

            console.table(
                list
            );

            return list;
        };

    window.gobTileInfo =
        function(x, y) {
            return tileInfo(
                x,
                y
            );
        };


    window.gobMapAround =
        function(radius = 4) {
            const pose =
                getPlayerPose();

            if (!pose) {
                console.log(
                    '[GOB Bot v5.2] player position unavailable'
                );
                return [];
            }

            radius =
                Math.max(
                    1,
                    Math.min(
                        12,
                        Math.floor(
                            Number(radius) || 4
                        )
                    )
                );

            const rows = [];

            for (
                let y = pose.tileY - radius;
                y <= pose.tileY + radius;
                y++
            ) {
                const row = {
                    y
                };

                for (
                    let x = pose.tileX - radius;
                    x <= pose.tileX + radius;
                    x++
                ) {
                    const info =
                        tileInfo(x, y);

                    row['x' + x] =
                        info.exists
                            ? info.name
                            : '?';
                }

                rows.push(row);
            }

            console.table(rows);
            return rows;
        };



    window.gobBombTapStatus =
        function() {
            const state =
                bombModelHookState();

            const result = {
                present:
                    !!state,
                prototypeCaptured:
                    !!state?.prototypeCaptured,
                capturedAt:
                    state?.capturedAt ??
                    null,
                recordCount:
                    Array.isArray(
                        state?.records
                    )
                        ? state.records.length
                        : 0,
                lastError:
                    state?.lastError ??
                    null
            };

            console.table(
                [result]
            );

            return result;
        };

    window.gobBombModelHookStatus =
        window.gobBombTapStatus;

    window.gobCapturedBombModels =
        function() {
            const state =
                bombModelHookState();

            const now =
                performance.now();

            const rows =
                (
                    Array.isArray(
                        state?.records
                    )
                        ? state.records
                        : []
                )
                    .slice(-40)
                    .map(
                        record => {
                            const model =
                                record.model;

                            const tile =
                                capturedBombTile(
                                    record
                                );

                            return {
                                strength:
                                    Number.isFinite(
                                        Number(
                                            model?.s
                                        )
                                    )
                                        ? Number(
                                            model.s
                                        )
                                        : record.strength,

                                N:
                                    Number.isFinite(
                                        Number(
                                            model?.N
                                        )
                                    )
                                        ? Number(
                                            model.N
                                        )
                                        : null,

                                q:
                                    Number.isFinite(
                                        Number(
                                            model?.q
                                        )
                                    )
                                        ? Number(
                                            model.q
                                        )
                                        : null,

                                t:
                                    Number.isFinite(
                                        Number(
                                            model?.t
                                        )
                                    )
                                        ? Number(
                                            model.t
                                        )
                                        : null,

                                tileX:
                                    tile?.x ??
                                    null,

                                tileY:
                                    tile?.y ??
                                    null,

                                canonicalX:
                                    tile?.canonicalX ??
                                    null,

                                canonicalY:
                                    tile?.canonicalY ??
                                    null,

                                rawX:
                                    tile?.rawX ??
                                    null,

                                rawY:
                                    tile?.rawY ??
                                    null,

                                ageMs:
                                    Math.round(
                                        now -
                                        Number(
                                            record.firstCapturedAt ??
                                            record.lastCapturedAt ??
                                            now
                                        )
                                    )
                            };
                        }
                    );

            console.table(
                rows
            );

            return rows;
        };

    window.gobBombTimingDebug =
        function() {
            const now =
                performance.now();

            refreshBombRegistry(
                now
            );

            const visuals =
                activeBombVisuals(
                    now
                );

            const visualByKey =
                new Map(
                    visuals.map(
                        v => [
                            canonicalTileKey(
                                v.canonicalX ??
                                    v.x,
                                v.canonicalY ??
                                    v.y
                            ),
                            v
                        ]
                    )
                );

            const rows =
                [
                    ...BOT.bombs.values()
                ].map(
                    bomb => {
                        const visual =
                            visualByKey.get(
                                canonicalTileKey(
                                    bomb.canonicalX ??
                                        bomb.x,
                                    bomb.canonicalY ??
                                        bomb.y
                                )
                            );

                        return {
                            id:
                                bomb.id,

                            ours:
                                !!bomb.ours,

                            x:
                                bomb.x,

                            y:
                                bomb.y,

                            fuseStage:
                                visual?.fuseStage ??
                                bomb.fuseStage ??
                                null,

                            timingSource:
                                bomb.fuseTimingSource ??
                                (
                                    bomb.ours
                                        ? 'own-placement'
                                        : 'unknown'
                                ),

                            earliestMs:
                                bomb.ours
                                    ? Math.round(
                                        bomb.explodeAt -
                                        now
                                    )
                                    : Math.round(
                                        (
                                            bomb.earliestExplodeAt ??
                                            now
                                        ) -
                                        now
                                    ),

                            latestMs:
                                bomb.ours
                                    ? Math.round(
                                        bomb.explodeAt -
                                        now
                                    )
                                    : Math.round(
                                        (
                                            bomb.latestExplodeAt ??
                                            now
                                        ) -
                                        now
                                    ),

                            visualAgeMs:
                                Math.round(
                                    now -
                                    bomb.lastVisualAt
                                ),

                            wickError:
                                visual?.fuseStageError != null
                                    ? Number(
                                        visual.fuseStageError.toFixed(
                                            2
                                        )
                                    )
                                    : null
                        };
                    }
                );

            console.table(
                rows
            );

            return rows;
        };

    window.gobActualFire =
        function() {
            const now =
                performance.now();

            cleanupExactObservations(
                now
            );

            const rows =
                [
                    ...exactWorldObservations.blasts.values()
                ]
                    .filter(
                        cell =>
                            now -
                            cell.lastSeenAt <=
                            750
                    )
                    .map(
                        cell => ({
                            x:
                                cell.x,
                            y:
                                cell.y,
                            source:
                                cell.source ||
                                'legacy-blast',
                            ageMs:
                                Math.round(
                                    now -
                                    cell.lastSeenAt
                                )
                        })
                    );

            console.table(
                rows
            );

            return rows;
        };

    window.gobBombIdentityDebug =
        function() {
            const now =
                performance.now();

            refreshBombRegistry(
                now
            );

            const rows =
                [
                    ...BOT.bombs.values()
                ].map(
                    bomb => ({
                        id:
                            bomb.id,

                        ours:
                            !!bomb.ours,

                        uncertain:
                            !!bomb.uncertain,

                        x:
                            bomb.x,

                        y:
                            bomb.y,

                        radius:
                            bomb.radius,

                        rangeKnown:
                            bomb.rangeKnown !==
                                false,

                        modelStrength:
                            bomb.modelStrength ??
                            null,

                        modelN:
                            bomb.modelN ??
                            null,

                        modelEntityId:
                            bomb.modelEntityId ??
                            null,

                        liveModelS:
                            Number.isFinite(
                                Number(
                                    bomb.modelRef?.s
                                )
                            )
                                ? Number(
                                    bomb.modelRef.s
                                )
                                : null,

                        liveModelN:
                            Number.isFinite(
                                Number(
                                    bomb.modelRef?.N
                                )
                            )
                                ? Number(
                                    bomb.modelRef.N
                                )
                                : null,

                        modelTileX:
                            bomb.modelTileX ??
                            null,

                        modelTileY:
                            bomb.modelTileY ??
                            null,

                        seenNow:
                            !!bomb.seenNow,

                        visualAgeMs:
                            Math.round(
                                now -
                                bomb.lastVisualAt
                            ),

                        explodeInMs:
                            bomb.explodeAt != null
                                ? Math.round(
                                    bomb.explodeAt -
                                    now
                                )
                                : null,

                        stickyForMs:
                            bomb.ours
                                ? Math.round(
                                    (
                                        bomb.identityStickyUntil ??
                                        0
                                    ) -
                                    now
                                )
                                : null
                    })
                );

            console.table(
                rows
            );

            return rows;
        };

    window.gobExplosionDebug =
        function() {
            const now =
                performance.now();

            refreshBombRegistry(
                now
            );

            const bombs =
                allBombRecordsForSimulation();

            const rows =
                bombs.map(
                    bomb => {
                        const fp =
                            predictedDirectionalFootprint(
                                bomb,
                                bombs
                            );

                        return {
                            id:
                                bomb.id,

                            ours:
                                !!bomb.ours,

                            pending:
                                !!bomb.pending,

                            x:
                                bomb.x,

                            y:
                                bomb.y,

                            strength:
                                fp.strength,

                            right:
                                fp.directions.d?.actual ??
                                0,

                            left:
                                fp.directions.a?.actual ??
                                0,

                            down:
                                fp.directions.s?.actual ??
                                0,

                            up:
                                fp.directions.w?.actual ??
                                0,

                            rightStop:
                                fp.directions.d?.stoppedBy,

                            leftStop:
                                fp.directions.a?.stoppedBy,

                            downStop:
                                fp.directions.s?.stoppedBy,

                            upStop:
                                fp.directions.w?.stoppedBy,

                            safetyRadius:
                                bomb.safetyRadius ??
                                null
                        };
                    }
                );

            console.table(
                rows
            );

            return rows;
        };

    window.gobMovementDebug =
        function() {
            const pose =
                getPlayerPose();

            const info =
                readAuthoritativePlayerInfo(
                    true
                );

            const step =
                currentPlanStep();

            const result = {
                enabled:
                    BOT.enabled,

                pose,

                authoritativeInfo:
                    info,

                currentKey:
                    BOT.currentKey,

                plan:
                    BOT.plan
                        ? {
                            type:
                                BOT.plan.type,
                            index:
                                BOT.plan.index,
                            length:
                                BOT.plan.path.length,
                            target:
                                BOT.plan.target
                        }
                        : null,

                nextStep:
                    step ||
                    null,

                state:
                    BOT.state,

                lastPlanReason:
                    BOT.lastPlanReason
            };

            console.log(
                '[GOB Bot v5.2] movement debug',
                result
            );

            return result;
        };

    window.gobGameApi =
        function() {
            const api =
                discoverGameApi(
                    true
                );

            const info =
                readAuthoritativePlayerInfo(
                    true
                );

            const result = {
                found:
                    !!api,
                methods:
                    api
                        ? {
                            getPlayerInfo:
                                typeof api.getPlayerInfo,
                            appInputKeys:
                                typeof api.appInputKeys,
                            appInputBomb:
                                typeof api.appInputBomb
                        }
                        : null,
                playerInfo:
                    info ||
                    null
            };

            console.log(
                '[GOB Bot v4] game API',
                result
            );

            return result;
        };

    window.gobBombEvents =
        function() {
            console.table(
                BOT.recentBombEvents
            );

            return [
                ...BOT.recentBombEvents
            ];
        };

    window.gobBotStatus =
        function() {
            const pose =
                getPlayerPose();

            const plan =
                BOT.plan;

            const status = {
                enabled:
                    BOT.enabled,
                state:
                    BOT.state,

                player:
                    pose,

                playerSprite:
                    BOT.lastPlayerCandidateName,

                power:
                    BOT.power,

                powerSource:
                    BOT.powerSource,

                bombSuperCount:
                    BOT.bombSuperCount,

                currentBaseBombStrength:
                    BOT.power,

                nextBombSafetyRadius:
                    predictedNextOwnBombRadius(
                        BOT.playerInfo
                    ),

                explosionModel:
                    'strength is max potential; actual footprint is traced separately per direction',

                authoritativePlayerInfo:
                    BOT.playerInfo,

                gameApiFound:
                    !!BOT.gameApi,

                focusBlock:
                    BOT.focusBlock,

                revealWaitMs:
                    Math.max(
                        0,
                        Math.round(
                            BOT.revealWaitUntil -
                            performance.now()
                        )
                    ),

                terrainCoordinates:
                    'absolute-world',

                movementSpeedPxPerSec:
                    Math.round(
                        BOT.movementSpeedPxPerSec
                    ),

                tileTravelMs:
                    Math.round(
                        tileTravelMs()
                    ),

                heldKey:
                    BOT.currentKey,

                bombs:
                    [
                        ...BOT.bombs.values()
                    ].map(
                        b => ({
                            id:
                                b.id,
                            x:
                                b.x,
                            y:
                                b.y,
                            ours:
                                b.ours,
                            confirmed:
                                b.confirmed,
                            radius:
                                b.radius
                        })
                    ),

                powerups:
                    activePickups(
                        performance.now()
                    ),

                plan:
                    plan
                        ? {
                            id:
                                plan.id,
                            type:
                                plan.type,
                            target:
                                plan.target,
                            index:
                                plan.index,
                            steps:
                                plan.path.length,
                            next:
                                currentPlanStep(),
                            pickupName:
                                plan.pickupName ||
                                null,
                            block:
                                plan.block ||
                                null
                        }
                        : null,

                lastReplanReason:
                    BOT.replanReason
            };

            console.log(
                '[GOB Bot v5.2]',
                status
            );

            return status;
        };

    window.addEventListener(
        'keydown',
        event => {
            if (
                event.code ===
                    'F8' &&
                !event.repeat
            ) {
                event.preventDefault();

                window.gobBotToggle();
            }
        },
        true
    );

    requestAnimationFrame(
        botLoop
    );

    requestAnimationFrame(
        renderDangerOverlay
    );

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
        let readinessTries = 0;

        const readinessInterval =
            setInterval(() => {
                readinessTries++;

                try {
                    rebuildSpriteIndex();
                } catch {}

                if (
                    (
                        realTileFrames.length > 20 &&
                        runtimeSprites.length > 15
                    ) ||
                    readinessTries >= 12
                ) {
                    clearInterval(
                        readinessInterval
                    );
                }
            }, 700);

        setInterval(() => {
            findLayers();
        }, 1000);

        findLayers();

        // No overlay canvas and no labels.

        console.log(
            '[GOB Bot v5.2] exact bomb/item detection + space-time A* + lane-safe movement'
        );

        console.log(
            'Commands:'
        );

        console.log(
            '[GOB Bot v5.2] commands: gobBotStart(), gobBotStop(), gobBotStatus(), gobMovementDebug(), gobWorldTraceDebug(), gobDisplayDebug(), gobActivityDebug(), gobBombTapStatus(), gobCapturedBombModels(), gobBombTimingDebug(), gobActualFire(), gobBombIdentityDebug(), gobExplosionDebug(), gobGameApi(), gobBombs(), gobBombEvents(), gobPowerups(), gobMapAround(r), gobSetBlastRadius(n)'
        );

        console.log(
            '[GOB Bot v5.2] F8 toggles the bot'
        );

        setTimeout(
            () => {
                const tap =
                    bombModelHookState();

                if (
                    !tap?.prototypeCaptured
                ) {
                    console.warn(
                        '[GOB Bot v5.2] BombModel tap not captured. Fully reload with only v5.2 enabled and check gobBombTapStatus().'
                    );
                }
            },
            3000
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
