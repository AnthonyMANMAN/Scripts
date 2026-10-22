(function() {
  if (document.getElementById('_skinEditor')) return;

  const panel = document.createElement('div');
  panel.id = '_skinEditor';
  panel.style.cssText = `
    position: fixed; top: 20px; right: 20px; width: 340px;
    background: #1a1a1a; color: #eee; font-family: monospace;
    font-size: 12px; border-radius: 8px; z-index: 999999;
    border: 1px solid #333; box-shadow: 0 4px 24px rgba(0,0,0,0.5);
    max-height: 90vh; overflow-y: auto;
  `;

  panel.innerHTML = `
    <div style="padding:10px 14px; background:#111; border-radius:8px 8px 0 0; display:flex; justify-content:space-between; align-items:center; cursor:move;" id="_skinDrag">
      <span style="font-weight:bold; color:#fff;">skin editor</span>
      <span id="_skinClose" style="cursor:pointer; color:#888; font-size:16px;">✕</span>
    </div>
    <div style="padding:12px 14px;">

      <div style="margin-bottom:8px;">
        <div style="color:#888; font-size:11px; margin-bottom:4px;">skin name</div>
        <input id="_sName" value="Omega02" style="width:100%; background:#2a2a2a; border:1px solid #333; color:#eee; padding:4px 8px; border-radius:4px; font-family:monospace; font-size:12px;">
      </div>

      <div style="margin-bottom:8px;">
        <div style="color:#888; font-size:11px; margin-bottom:4px;">image url</div>
        <input id="_sUrl" value="https://raw.githubusercontent.com/Kix34/Scripts/refs/heads/main/Omega021.png" style="width:100%; background:#2a2a2a; border:1px solid #333; color:#eee; padding:4px 8px; border-radius:4px; font-family:monospace; font-size:12px;">
      </div>

      <div style="color:#888; font-size:11px; margin:10px 0 6px; border-top:1px solid #333; padding-top:8px;">dimensions</div>
      ${['frameWidth:55:16:256','frameHeight:55:16:256','renderWidth:80:8:256','renderHeight:80:8:256','soulWidth:55:8:256','renderShiftY:-4:-64:64'].map(s => {
        const [id,val,min,max] = s.split(':');
        return `<div style="display:flex;align-items:center;gap:8px;margin-bottom:6px;">
          <span style="width:130px;color:#aaa;">${id}</span>
          <input type="range" id="_s_${id}" min="${min}" max="${max}" step="1" value="${val}" style="flex:1;">
          <span id="_sv_${id}" style="width:36px;text-align:right;color:#fff;">${val}</span>
        </div>`;
      }).join('')}

      <div style="color:#888; font-size:11px; margin:10px 0 6px; border-top:1px solid #333; padding-top:8px;">animation</div>
      ${['animSpeed:2400:100:8000','animCount:4:1:16','animCountSide:4:1:16','idleAnimSpeed:4000:100:10000','idleAnimCount:4:1:16','idleAnimCountSide:4:1:16'].map(s => {
        const [id,val,min,max] = s.split(':');
        return `<div style="display:flex;align-items:center;gap:8px;margin-bottom:6px;">
          <span style="width:130px;color:#aaa;">${id}</span>
          <input type="range" id="_s_${id}" min="${min}" max="${max}" step="${id==='animSpeed'||id==='idleAnimSpeed'?'100':'1'}" value="${val}" style="flex:1;">
          <span id="_sv_${id}" style="width:36px;text-align:right;color:#fff;">${val}</span>
        </div>`;
      }).join('')}

      <div style="margin-bottom:6px;">
        <span style="color:#aaa; width:130px; display:inline-block;">publicFrame</span>
        <input id="_s_publicFrame" value="2,0" style="width:140px; background:#2a2a2a; border:1px solid #333; color:#eee; padding:2px 6px; border-radius:4px; font-family:monospace; font-size:12px;">
      </div>
      <div style="margin-bottom:6px;">
        <span style="color:#aaa; width:130px; display:inline-block;">idleCol</span>
        <input id="_s_idleCol" value="0,1,2,3" style="width:140px; background:#2a2a2a; border:1px solid #333; color:#eee; padding:2px 6px; border-radius:4px; font-family:monospace; font-size:12px;">
      </div>
      <div style="margin-bottom:6px;">
        <span style="color:#aaa; width:130px; display:inline-block;">lookLut</span>
        <input id="_s_lookLut" value="0,1,2,3,4,5,6,7" style="width:140px; background:#2a2a2a; border:1px solid #333; color:#eee; padding:2px 6px; border-radius:4px; font-family:monospace; font-size:12px;">
      </div>

      <div style="color:#888; font-size:11px; margin:10px 0 6px; border-top:1px solid #333; padding-top:8px;">flags</div>
      ${[['hasSoul',true],['idleAnim',true],['animMirrorLeft',false],['alwaysRun',false],['alwaysOnTop',false],['animStand',false]].map(([id,def]) =>
        `<label style="display:flex;align-items:center;gap:8px;margin-bottom:6px;cursor:pointer;">
          <input type="checkbox" id="_s_${id}" ${def?'checked':''}>
          <span style="color:#aaa;">${id}</span>
        </label>`
      ).join('')}

      <div style="border-top:1px solid #333; margin-top:10px; padding-top:10px; display:flex; gap:8px;">
        <button id="_skinApply" style="flex:1; padding:6px; background:#2563eb; color:#fff; border:none; border-radius:4px; cursor:pointer; font-family:monospace; font-weight:bold;">apply</button>
        <button id="_skinCopy" style="flex:1; padding:6px; background:#2a2a2a; color:#eee; border:1px solid #444; border-radius:4px; cursor:pointer; font-family:monospace;">copy code</button>
      </div>
      <div id="_skinStatus" style="margin-top:8px; font-size:11px; color:#4ade80; min-height:16px;"></div>
    </div>
  `;

  document.body.appendChild(panel);

  panel.querySelectorAll('input[type=range]').forEach(el => {
    el.addEventListener('input', () => {
      document.getElementById('_sv_' + el.id.replace('_s_', '')).textContent = el.value;
    });
  });

  function parseArr(str) {
    return str.split(',').map(s => parseInt(s.trim())).filter(n => !isNaN(n));
  }

  function buildSkin() {
    const g = id => document.getElementById('_s_' + id);
    const gv = id => parseInt(g(id).value);
    const gc = id => g(id).checked;
    const skin = {
      frameWidth:        gv('frameWidth'),
      frameHeight:       gv('frameHeight'),
      renderWidth:       gv('renderWidth'),
      renderHeight:      gv('renderHeight'),
      soulWidth:         gv('soulWidth'),
      animSpeed:         gv('animSpeed'),
      animCount:         gv('animCount'),
      animCountSide:     gv('animCountSide'),
      idleAnimSpeed:     gv('idleAnimSpeed'),
      idleAnimCount:     gv('idleAnimCount'),
      idleAnimCountSide: gv('idleAnimCountSide'),
      publicFrame:       parseArr(g('publicFrame').value),
      idleCol:           parseArr(g('idleCol').value),
      lookLut:           parseArr(g('lookLut').value),
      hasSoul:           gc('hasSoul'),
      idleAnim:          gc('idleAnim'),
      animMirrorLeft:    gc('animMirrorLeft'),
    };
    if (gc('alwaysRun'))   skin.alwaysRun   = true;
    if (gc('alwaysOnTop')) skin.alwaysOnTop  = true;
    if (gc('animStand'))   skin.animStand    = true;
    return skin;
  }

  document.getElementById('_skinApply').addEventListener('click', () => {
    const name   = document.getElementById('_sName').value.trim();
    const imgUrl = document.getElementById('_sUrl').value.trim();
    const spr    = sprites._dynlist.find(s => s.name === name);
    const status = document.getElementById('_skinStatus');
    if (!spr) { status.style.color='#f87171'; status.textContent = name + ' not found'; return; }

    const skin = buildSkin();
    Object.assign(spr.skin, skin);
    spr.renderWidth  = skin.renderWidth;
    spr.renderHeight = skin.renderHeight;
    spr.renderShiftY = parseInt(document.getElementById('_s_renderShiftY').value);

    if (imgUrl) {
      spr.src = imgUrl;
      if (spr.texture) spr.texture.src = imgUrl;
      const img = new Image();
      img.crossOrigin = 'anonymous';
      img.onload = () => {
        spr.texture.width  = img.width;
        spr.texture.height = img.height;
        try { spr.texture.element = img; } catch(e) {}
        spr.loaded = true;
        status.style.color='#4ade80';
        status.textContent = 'applied! ' + img.width + 'x' + img.height;
      };
      img.onerror = () => { status.style.color='#f87171'; status.textContent = 'image failed to load'; };
      img.src = imgUrl;
    } else {
      status.style.color='#4ade80';
      status.textContent = 'applied!';
    }
  });

  document.getElementById('_skinCopy').addEventListener('click', () => {
    const name   = document.getElementById('_sName').value.trim();
    const imgUrl = document.getElementById('_sUrl').value.trim();
    const rsy    = parseInt(document.getElementById('_s_renderShiftY').value);
    const skin   = buildSkin();
    const lines  = Object.entries(skin).map(([k,v]) => `  spr.skin.${k} = ${JSON.stringify(v)};`).join('\n');
    const imgBlock = imgUrl ? `\n  spr.src = '${imgUrl}';\n  if (spr.texture) spr.texture.src = '${imgUrl}';\n  const img = new Image();\n  img.crossOrigin = 'anonymous';\n  img.onload = () => { spr.texture.width=img.width; spr.texture.height=img.height; try{spr.texture.element=img;}catch(e){} spr.loaded=true; };\n  img.src = '${imgUrl}';` : '';
    const code = `const spr = sprites._dynlist.find(s => s.name === '${name}');\nif (spr) {\n${lines}\n  spr.renderWidth = ${skin.renderWidth};\n  spr.renderHeight = ${skin.renderHeight};\n  spr.renderShiftY = ${rsy};${imgBlock}\n}`;
    navigator.clipboard.writeText(code);
    document.getElementById('_skinStatus').style.color='#4ade80';
    document.getElementById('_skinStatus').textContent = 'code copied!';
  });

  document.getElementById('_skinClose').addEventListener('click', () => panel.remove());

  const drag = document.getElementById('_skinDrag');
  let ox=0, oy=0, mx=0, my=0;
  drag.addEventListener('mousedown', e => {
    ox = panel.offsetLeft - e.clientX;
    oy = panel.offsetTop  - e.clientY;
    document.addEventListener('mousemove', onMove);
    document.addEventListener('mouseup', () => document.removeEventListener('mousemove', onMove), {once:true});
  });
  function onMove(e) {
    panel.style.left = (e.clientX + ox) + 'px';
    panel.style.top  = (e.clientY + oy) + 'px';
    panel.style.right = 'auto';
  }
})();
