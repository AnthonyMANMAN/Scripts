(() => {
  // ============================================================
  // Shared state
  // ============================================================
  let layer0Ctx = null;
  let layer1Ctx = null;

  let showWorldGrid = false;
  let showBombBoxes = false; // toggles ONLY the bomb square outline
  const blockPositions = {};
  const bombPositions = {};
  const shadowPositions = {};

  const PROTO_PATCH_FLAG = '__combinedOverlayPatched';

  // EDIT 1: frame counter + unique bomb ids
  let frameCount = 0;
  const tickFrame = () => { frameCount++; requestAnimationFrame(tickFrame); };
  requestAnimationFrame(tickFrame);
  let nextBombId = 1;

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
  // Bomb tracking (layer1) — original predictive matching
  // ============================================================

  const MATCH_RADIUS = 60;

  // EDIT 2: camera compensation, sampled ONCE per frame (not per draw).
  let lastCamE = null, lastCamF = null, camSampledFrame = -1, camMove = 0;
  const compensateForCamera = (ctx) => {
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
  };

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

      const bombCenterX = bomb.x + bomb.w / 2;
      const bombCenterY = bomb.y + bomb.h / 2;

      // EDIT 5: box outline is the only thing gated by showBombBoxes
      const boxSize = 32;
      if (showBombBoxes) {
        layer1Ctx.strokeRect(bombCenterX - boxSize / 2, bombCenterY - boxSize / 2, boxSize, boxSize);
      }

      const timerX = bombCenterX + boxSize / 2 - 10;
      const timerY = bombCenterY - boxSize / 2 + 10;
      layer1Ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
      layer1Ctx.beginPath();
      layer1Ctx.arc(timerX, timerY, 10, 0, Math.PI * 2);
      layer1Ctx.fill();

      layer1Ctx.fillStyle = remaining > 1 ? '#0f0' : '#f00';
      layer1Ctx.fillText(remaining.toFixed(1), timerX, timerY);

      // At 1 second or less remaining (until it explodes), draw a red
      // dashed line 96px out from the bomb in each cardinal direction.
      // Always drawn regardless of showBombBoxes.
      if (remaining <= 1) {
        if (!bomb.blastLogged) {
          bomb.blastLogged = true;
          console.log('blast lines: drawing for', key, 'at', bombCenterX, bombCenterY);
        }
        layer1Ctx.save();
        layer1Ctx.globalAlpha = 1;
        layer1Ctx.strokeStyle = 'red';
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
  };

  let lastFillWasGrey = false;
  let lastGlobalAlpha = 1.0;

  const handleLayer1DrawImage = (ctx, image, args) => {
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

    // Match existing bomb by proximity to its PREDICTED position
    // (last center + last observed frame-to-frame displacement).
    // EDIT 3: a bomb already matched to a draw THIS frame can't be matched
    // again (stops bombs stealing each other's entries); a draw sitting
    // right on top of an already-matched bomb is a duplicate draw, ignored.
    // EDIT 4: radius is widened by how far the camera moved this frame.
    const radius = MATCH_RADIUS + camMove;
    let existingKey = null;
    let minDist = Infinity;
    let duplicate = false;
    for (const key in bombPositions) {
      const bomb = bombPositions[key];
      const bombCenterX = bomb.x + bomb.w / 2;
      const bombCenterY = bomb.y + bomb.h / 2;

      if (bomb.lastFrame === frameCount) {
        if (Math.abs(screenCenterX - bombCenterX) < 6 && Math.abs(screenCenterY - bombCenterY) < 6) {
          duplicate = true;
          break;
        }
        continue;
      }

      const predictedX = bombCenterX + (bomb.lastDX || 0);
      const predictedY = bombCenterY + (bomb.lastDY || 0);
      const dist = Math.sqrt(
        Math.pow(screenCenterX - predictedX, 2) + Math.pow(screenCenterY - predictedY, 2)
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

      bomb.x = x; bomb.y = y; bomb.w = w; bomb.h = h;
      bomb.lastSeen = currentTime;
      bomb.lastFrame = frameCount;
    } else {
      bombPositions['bomb_' + (nextBombId++)] = {
        x, y, w, h,
        placedTime: currentTime,
        lastSeen: currentTime,
        lastFrame: frameCount,
        lastDX: 0,
        lastDY: 0
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
        if (this.canvas.id === 'layer0') {
          compensateForCamera(this);
          handleLayer0DrawImage(this, image, args);
        }
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
        console.log('✓ Bomb timer overlay hooked (predictive matching)');
      }
    }
  };

  patchPrototypeOnce();
  hookAllCanvases();

  const observer = new MutationObserver(hookAllCanvases);
  observer.observe(document.body, { childList: true, subtree: true });

  // Fallback redraw in case no bomb drawImage happens for a tick.
  setInterval(drawBombBoxes, 50);

  window.toggleWorldGrid = () => {
    showWorldGrid = !showWorldGrid;
    console.log('World grid:', showWorldGrid ? 'ON' : 'OFF');
  };
  window.toggleBombBoxes = () => {
    showBombBoxes = !showBombBoxes;
    console.log('Bomb boxes:', showBombBoxes ? 'ON' : 'OFF', '(timer & blast lines unaffected)');
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

  console.log('Combined overlay active: world grid + bomb timers (predictive matching, no throw/velocity tracking).');
})();
