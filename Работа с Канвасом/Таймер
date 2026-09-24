(() => {
  // ============================================================
  // Shared state
  // ============================================================
  let layer0Ctx = null;
  let layer1Ctx = null;

  let showWorldGrid = true;
  const blockPositions = {};
  const bombPositions = {};
  const shadowPositions = {};

  const PROTO_PATCH_FLAG = '__combinedOverlayPatched';

  // ============================================================
  // World grid (layer0) — unchanged from original
  // ============================================================
  const drawWorldGrid = () => {
    if (!layer0Ctx || !showWorldGrid) return;

    const transform = layer0Ctx.getTransform();
    if (Math.abs(transform.e) < 10 && Math.abs(transform.f) < 10) return;

    layer0Ctx.strokeStyle = 'rgba(255, 0, 0, 0.6)';
    layer0Ctx.lineWidth = 1;

    const tileSize = 32;
    const scale = transform.a;

    const worldLeft = -transform.e / scale;
    const worldTop = -transform.f / scale;
    const worldRight = worldLeft + layer0Ctx.canvas.width / scale;
    const worldBottom = worldTop + layer0Ctx.canvas.height / scale;

    const startX = Math.floor(worldLeft / tileSize) * tileSize;
    const startY = Math.floor(worldTop / tileSize) * tileSize;

    for (let x = startX; x <= worldRight + tileSize; x += tileSize) {
      layer0Ctx.beginPath();
      layer0Ctx.moveTo(x, worldTop - tileSize);
      layer0Ctx.lineTo(x, worldBottom + tileSize);
      layer0Ctx.stroke();
    }
    for (let y = startY; y <= worldBottom + tileSize; y += tileSize) {
      layer0Ctx.beginPath();
      layer0Ctx.moveTo(worldLeft - tileSize, y);
      layer0Ctx.lineTo(worldRight + tileSize, y);
      layer0Ctx.stroke();
    }

    layer0Ctx.fillStyle = 'rgba(255, 255, 0, 0.9)';
    layer0Ctx.font = 'bold 10px monospace';
    for (let x = startX; x <= worldRight; x += tileSize) {
      for (let y = startY; y <= worldBottom; y += tileSize) {
        const tileX = Math.floor(x / tileSize);
        const tileY = Math.floor(y / tileSize);
        const wrappedX = ((tileX % 32) + 32) % 32;
        const wrappedY = ((tileY % 32) + 32) % 32;
        layer0Ctx.fillText(`${wrappedX},${wrappedY}`, x + 2, y + 12);
      }
    }
  };

  const handleLayer0DrawImage = (ctx, image, args) => {
    let worldX, worldY, width, height;
    if (args.length === 2) {
      worldX = args[0]; worldY = args[1]; width = image.width; height = image.height;
    } else if (args.length === 4) {
      worldX = args[0]; worldY = args[1]; width = args[2]; height = args[3];
    } else if (args.length === 8) {
      worldX = args[4]; worldY = args[5]; width = args[6]; height = args[7];
    } else {
      return;
    }

    if (width >= 16 && width <= 64 && height >= 16 && height <= 64) {
      const tileX = Math.floor(worldX / 32);
      const tileY = Math.floor(worldY / 32);
      const wrappedX = ((tileX % 32) + 32) % 32;
      const wrappedY = ((tileY % 32) + 32) % 32;
      const key = `${wrappedX},${wrappedY}`;
      blockPositions[key] = {
        x: worldX, y: worldY, width, height,
        tileX: wrappedX, tileY: wrappedY,
        image: image.src || 'unknown'
      };
    }
  };

  // ============================================================
  // Bomb tracking (layer1)
  //
  // This game bakes camera scroll directly into the x/y it passes to
  // drawImage() — it does not use ctx.setTransform for this canvas,
  // so there is no transform to invert and no reliable way to recover
  // a "true" world position. Per instruction, throw/flight detection
  // (which depended on measuring velocity, and therefore broke when
  // panning looked like motion) has been removed entirely. The timer
  // is now a simple, non-pausing countdown from when the bomb was
  // first seen — nothing here can be fooled by camera movement,
  // because nothing here measures movement anymore.
  //
  // Matching an existing tracked bomb to a new drawImage call still
  // uses on-screen proximity, but with a generous radius so a bomb
  // that merely shifted on screen because the camera panned doesn't
  // get dropped and re-created (which was the "gets lost" symptom).
  // ============================================================

  const MATCH_RADIUS = 60;

  const drawBombBoxes = () => {
    if (!layer1Ctx) return;

    const now = Date.now();
    layer1Ctx.save();
    layer1Ctx.strokeStyle = 'red';
    layer1Ctx.lineWidth = 3;
    layer1Ctx.font = 'bold 12px Arial';
    layer1Ctx.textAlign = 'center';
    layer1Ctx.textBaseline = 'middle';

    for (const key in bombPositions) {
      const bomb = bombPositions[key];

      // Simple non-pausing countdown from when the bomb was first seen.
      const elapsed = (now - bomb.placedTime) / 1000;
      const remaining = Math.max(0, 4.3 - elapsed);

      if (remaining === 0) { delete bombPositions[key]; continue; }
      if (now - bomb.lastSeen > 200) { delete bombPositions[key]; continue; }

      // Raw screen coords from the most recent drawImage() call for
      // this bomb — always current, always correct for this frame.
      const bombCenterX = bomb.x + bomb.w / 2;
      const bombCenterY = bomb.y + bomb.h / 2;

      const boxSize = 32;
      layer1Ctx.strokeRect(bombCenterX - boxSize / 2, bombCenterY - boxSize / 2, boxSize, boxSize);

      const timerX = bombCenterX + boxSize / 2 - 10;
      const timerY = bombCenterY - boxSize / 2 + 10;
      layer1Ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
      layer1Ctx.beginPath();
      layer1Ctx.arc(timerX, timerY, 10, 0, Math.PI * 2);
      layer1Ctx.fill();

      layer1Ctx.fillStyle = remaining > 1 ? '#0f0' : '#f00';
      layer1Ctx.fillText(remaining.toFixed(1), timerX, timerY);
    }

    layer1Ctx.restore();
  };

  let lastFillWasGrey = false;
  let lastGlobalAlpha = 1.0;

  const handleLayer1DrawImage = (ctx, image, args) => {
    // drawImage(img, 0, 0, 64, 64, x, y, w, h) — image is separate,
    // so args here is [0, 0, 64, 64, x, y, w, h] (length 8).
    if (args.length !== 8 || args[0] !== 0 || args[1] !== 0 || args[2] !== 64 || args[3] !== 64) {
      return;
    }

    const x = args[4];
    const y = args[5];
    const w = args[6];
    const h = args[7];
    const currentTime = Date.now();

    const screenCenterX = x + w / 2;
    const screenCenterY = y + h / 2;

    const isShadow = lastFillWasGrey && w >= 50 && w <= 70 && h >= 50 && h <= 70;
    const isGreyOutline = (lastFillWasGrey && !isShadow) || lastGlobalAlpha < 0.8;

    if (lastFillWasGrey) lastFillWasGrey = false;
    lastGlobalAlpha = 1.0;

    if (isShadow) {
      const shadowKey = `shadow_${Math.round(screenCenterX / 8)}_${Math.round(screenCenterY / 8)}`;
      shadowPositions[shadowKey] = { x: screenCenterX, y: screenCenterY, lastSeen: currentTime };
      for (const key in shadowPositions) {
        if (currentTime - shadowPositions[key].lastSeen > 200) delete shadowPositions[key];
      }
      return;
    }
    if (isGreyOutline) return;

    // Match existing bomb by on-screen proximity. A generous radius
    // means a bomb that only shifted because the camera panned still
    // matches, instead of being dropped and re-created (which reset
    // its timer).
    let existingKey = null;
    let minDist = Infinity;
    for (const key in bombPositions) {
      const bomb = bombPositions[key];
      const bombCenterX = bomb.x + bomb.w / 2;
      const bombCenterY = bomb.y + bomb.h / 2;
      const dist = Math.sqrt(
        Math.pow(screenCenterX - bombCenterX, 2) + Math.pow(screenCenterY - bombCenterY, 2)
      );
      if (dist < MATCH_RADIUS && dist < minDist) {
        minDist = dist;
        existingKey = key;
      }
    }

    if (existingKey) {
      const bomb = bombPositions[existingKey];
      bomb.x = x; bomb.y = y; bomb.w = w; bomb.h = h;
      bomb.lastSeen = currentTime;

      const newKey = `bomb_${Math.round(screenCenterX / 8)}_${Math.round(screenCenterY / 8)}`;
      if (newKey !== existingKey) {
        bombPositions[newKey] = bomb;
        delete bombPositions[existingKey];
      }
    } else {
      const posKey = `bomb_${Math.round(screenCenterX / 8)}_${Math.round(screenCenterY / 8)}`;
      bombPositions[posKey] = {
        x, y, w, h,
        placedTime: currentTime,
        lastSeen: currentTime
      };
    }

    drawBombBoxes();
  };

  // ============================================================
  // Single shared prototype patch
  // ============================================================
  const patchPrototypeOnce = () => {
    const proto = CanvasRenderingContext2D.prototype;
    if (proto[PROTO_PATCH_FLAG]) return;
    proto[PROTO_PATCH_FLAG] = true;

    const originalDrawImage = proto.drawImage;
    const originalFillRect = proto.fillRect;
    const originalSave = proto.save;
    const originalRestore = proto.restore;
    const alphaDesc = Object.getOwnPropertyDescriptor(proto, 'globalAlpha');
    const originalAlphaSetter = alphaDesc?.set || function () {};
    const originalAlphaGetter = alphaDesc?.get || function () { return 1.0; };

    const saveCounts = new WeakMap();

    Object.defineProperty(proto, 'globalAlpha', {
      get() { return originalAlphaGetter.call(this); },
      set(value) {
        if (this.canvas && this.canvas.id === 'layer1') lastGlobalAlpha = value;
        originalAlphaSetter.call(this, value);
      },
      configurable: true
    });

    proto.fillRect = function (...args) {
      if (this.canvas && this.canvas.id === 'layer1') {
        const fillStyle = this.fillStyle;
        if (typeof fillStyle === 'string') {
          const lower = fillStyle.toLowerCase();
          lastFillWasGrey =
            lower.includes('grey') || lower.includes('gray') ||
            (lower.match(/#[0-9a-f]{6}/) &&
              parseInt(lower.slice(1, 3), 16) === parseInt(lower.slice(3, 5), 16) &&
              parseInt(lower.slice(3, 5), 16) === parseInt(lower.slice(5, 7), 16));
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
        if (count > 0) drawWorldGrid();
        saveCounts.set(this, count - 1);
      }
      return originalRestore.apply(this, args);
    };

    proto.drawImage = function (image, ...args) {
      const result = originalDrawImage.apply(this, [image, ...args]);
      if (this.canvas) {
        if (this.canvas.id === 'layer0') handleLayer0DrawImage(this, image, args);
        else if (this.canvas.id === 'layer1') handleLayer1DrawImage(this, image, args);
      }
      return result;
    };
  };

  // ============================================================
  // Canvas discovery
  // ============================================================
  const hookAllCanvases = () => {
    const canvases = document.querySelectorAll('canvas');
    for (const canvas of canvases) {
      const ctx = canvas.getContext('2d');
      if (!ctx) continue;
      if (canvas.id === 'layer0' && !layer0Ctx) {
        layer0Ctx = ctx;
        console.log('✓ World grid hooked - 32px tiles, 32x32 wrapped world');
      }
      if (canvas.id === 'layer1' && !layer1Ctx) {
        layer1Ctx = ctx;
        console.log('✓ Bomb timer overlay hooked (world-space velocity tracking)');
      }
    }
  };

  patchPrototypeOnce();
  hookAllCanvases();

  const observer = new MutationObserver(hookAllCanvases);
  observer.observe(document.body, { childList: true, subtree: true });

  // Fallback redraw in case no bomb drawImage happens for a tick
  // (e.g. game briefly stops rendering bombs). Uses last-known raw
  // screen coords, same as synchronous draws — no transform assumptions.
  setInterval(drawBombBoxes, 50);

  window.toggleWorldGrid = () => {
    showWorldGrid = !showWorldGrid;
    console.log('World grid:', showWorldGrid ? 'ON' : 'OFF');
  };
  window.getBlockAt = (tileX, tileY) => {
    const wrappedX = ((tileX % 32) + 32) % 32;
    const wrappedY = ((tileY % 32) + 32) % 32;
    return blockPositions[`${wrappedX},${wrappedY}`] || null;
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

  console.log('Combined overlay active: world grid + bomb timers (world-space velocity).');
})();
