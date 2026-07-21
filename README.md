# mygame2026
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Voxel Craft</title>
<style>
  * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
  html, body {
    margin: 0; padding: 0; width: 100%; height: 100%; overflow: hidden;
    background: #87CEEB; font-family: 'Courier New', monospace;
    user-select: none;
  }
  canvas { display: block; }
  #crosshair {
    position: fixed; top: 50%; left: 50%; width: 20px; height: 20px;
    transform: translate(-50%, -50%); pointer-events: none; z-index: 20;
  }
  #crosshair::before, #crosshair::after {
    content: ""; position: absolute; background: rgba(255,255,255,0.85);
    box-shadow: 0 0 2px rgba(0,0,0,0.8);
  }
  #crosshair::before { top: 9px; left: 0; width: 20px; height: 2px; }
  #crosshair::after { left: 9px; top: 0; width: 2px; height: 20px; }
  #hud {
    position: fixed; bottom: 0; left: 0; width: 100%;
    display: flex; flex-direction: column; align-items: center;
    z-index: 15; pointer-events: none;
  }
  .slot {
    width: 52px; height: 52px; margin: 3px;
    background: rgba(20,20,20,0.55);
    border: 3px solid #555;
    box-shadow: inset 0 0 0 2px #222;
    position: relative;
    display: flex; align-items: center; justify-content: center;
    pointer-events: auto;
  }
  .slot.selected { border-color: #fff; box-shadow: inset 0 0 0 2px #fff; }
  .slot .swatch {
    width: 34px; height: 34px; border-radius: 3px;
    box-shadow: inset -4px -4px 0 rgba(0,0,0,0.25), inset 3px 3px 0 rgba(255,255,255,0.15);
  }
  .slot .count {
    position: absolute; bottom: 2px; right: 4px; color: #fff;
    font-size: 13px; text-shadow: 1px 1px 0 #000, -1px -1px 0 #000;
  }
  #hotbar { display: flex; margin-bottom: 10px; }
  #clockWrap {
    position: fixed; top: 14px; right: 14px; z-index: 12;
    display: flex; align-items: center; gap: 8px;
    background: rgba(0,0,0,0.35); padding: 6px 12px; border-radius: 20px;
    color: #fff; font-size: 12px; text-shadow: 1px 1px 0 #000;
  }
  #clockIcon {
    width: 18px; height: 18px; border-radius: 50%; background: #ffd34d;
    box-shadow: 0 0 6px #ffd34d;
  }
  #panelOverlay {
    position: fixed; inset: 0; background: rgba(0,0,0,0.55);
    display: none; align-items: center; justify-content: center; z-index: 30;
  }
  .panel {
    background: #c6c6c6; border: 6px solid #8b8b8b;
    box-shadow: inset 0 0 0 4px #dcdcdc, 0 0 0 4px #3d3d3d;
    padding: 16px; color: #3d3d3d;
    max-height: 92vh; overflow-y: auto;
  }
  #personalCraftGrid.grid9 { grid-template-columns: repeat(9, 40px); }
  #personalCraftGrid .slot { width: 40px; height: 40px; margin: 2px; }
  #personalCraftGrid .slot .swatch { width: 26px; height: 26px; }
  #personalCraftGrid .slot .count { font-size: 11px; }
  .panel h2 { margin: 0 0 10px 0; font-size: 15px; }
  .grid { display: grid; gap: 4px; }
  .grid3 { grid-template-columns: repeat(3, 52px); }
  .grid9 { grid-template-columns: repeat(9, 52px); }
  .grid7 { grid-template-columns: repeat(7, 52px); }
  .rowgap { margin-top: 14px; }
  .craftRow { display: flex; align-items: center; gap: 14px; }
  .arrow { font-size: 26px; color: #3d3d3d; }
  #closeHint { text-align: center; font-size: 11px; margin-top: 10px; color: #555; }
  #cursorItem {
    position: fixed; width: 40px; height: 40px; pointer-events: none; z-index: 40;
    display: none; align-items: center; justify-content: center;
  }
  #cursorItem .swatch { width: 34px; height: 34px; border-radius: 3px; }
  #startScreen {
    position: fixed; inset: 0; background: linear-gradient(#3a6ea5, #7fb3e0);
    z-index: 100; display: flex; flex-direction: column; align-items: center; justify-content: center;
    color: #fff; text-align: center;
  }
  #startScreen h1 { font-size: 42px; text-shadow: 3px 3px 0 #2b2b2b; letter-spacing: 2px; margin-bottom: 6px; }
  #startScreen p { opacity: 0.9; margin: 4px 0; font-size: 14px; }
  #startBtn {
    margin-top: 22px; padding: 14px 34px; font-size: 16px; cursor: pointer;
    background: #6b8e3d; color: #fff; border: 4px solid #4a6329;
  }
  #startBtn:hover { background: #79a047; }
  #hint {
    position: fixed; top: 14px; left: 14px; color: #fff; font-size: 12px;
    text-shadow: 1px 1px 0 #000; z-index: 12; line-height: 1.6;
    background: rgba(0,0,0,0.35); padding: 8px 12px; max-width: 230px;
  }
  #recipeMsg { font-size: 11px; margin-top: 8px; min-height: 14px; color: #444; }
  #nightOverlay {
    position: fixed; inset: 0; pointer-events: none; z-index: 5;
    background: rgba(10,10,40,0); transition: background 0.5s;
  }
</style>
</head>
<body>

<div id="startScreen">
  <h1>⛏ VOXEL CRAFT</h1>
  <p>나만의 복셀 세계를 탐험하고, 캐고, 만들어보세요.</p>
  <p>WASD 이동 · 스페이스 점프 · 좌클릭 채굴 · 우클릭 설치/상호작용</p>
  <button id="startBtn">클릭해서 시작하기</button>
</div>

<div id="hint">
  E : 인벤토리 · B : 대형 보관함(7x7) · 1-9 : 아이템 선택 · F5 : 시점 전환<br>
  좌클릭 : 블록 캐기 · 우클릭 : 블록 설치 / 상호작용<br>
  제작대(밝은 갈색 블록)를 설치하고 우클릭하면 제작창이 열려요<br>
  깊이 파고들면 석탄·철·다이아몬드 광석이 나와요 (기반암은 파괴 불가)<br>
  낮과 밤이 순환합니다 🌙
</div>

<div id="clockWrap"><div id="clockIcon"></div><span id="clockText">낮</span></div>
<div id="nightOverlay"></div>
<div id="crosshair"></div>

<div id="hud"><div id="hotbar"></div></div>

<div id="panelOverlay">
  <div class="panel" id="invPanel">
    <h2>인벤토리</h2>
    <div class="grid grid9" id="invGrid"></div>
    <div class="rowgap craftRow">
      <div class="grid grid9" id="personalCraftGrid"></div>
      <div class="arrow">➜</div>
      <div class="slot" id="personalCraftOutput"></div>
    </div>
    <div class="rowgap grid grid9" id="hotbarMirror"></div>
    <div id="closeHint">E 또는 ESC 를 눌러 닫기</div>
  </div>

  <div class="panel" id="storagePanel" style="display:none;">
    <h2>대형 보관함 (7x7)</h2>
    <div class="grid grid7" id="storageGrid"></div>
    <div class="rowgap grid grid9" id="hotbarMirror3"></div>
    <div id="closeHint">B 또는 ESC 를 눌러 닫기</div>
  </div>

  <div class="panel" id="tablePanel" style="display:none;">
    <h2>제작대</h2>
    <div class="craftRow">
      <div class="grid grid3" id="tableCraftGrid"></div>
      <div class="arrow">➜</div>
      <div class="slot" id="tableCraftOutput"></div>
    </div>
    <div id="recipeMsg"></div>
    <div class="rowgap grid grid9" id="hotbarMirror2"></div>
    <div id="closeHint">E 또는 ESC 를 눌러 닫기</div>
  </div>
</div>

<div id="cursorItem"><div class="swatch"></div><div class="count" style="position:absolute;bottom:0;right:2px;color:#fff;font-size:12px;"></div></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
// ---------- Block/Item definitions ----------
const BLOCK = { AIR:0, GRASS:1, DIRT:2, STONE:3, LOG:4, LEAVES:5, PLANKS:6, TABLE:7,
                BEDROCK:8, COAL_ORE:9, IRON_ORE:10, DIAMOND_ORE:11 };
const BLOCK_COLOR = {
  [BLOCK.GRASS]:0x5b9c3f, [BLOCK.DIRT]:0x7a5230, [BLOCK.STONE]:0x8a8a8a,
  [BLOCK.LOG]:0x5c4326, [BLOCK.LEAVES]:0x3f7a2e, [BLOCK.PLANKS]:0xb98a4f,
  [BLOCK.TABLE]:0x9c5a2e, [BLOCK.BEDROCK]:0x2b2b2b, [BLOCK.COAL_ORE]:0x4a4a4a,
  [BLOCK.IRON_ORE]:0xcaa472, [BLOCK.DIAMOND_ORE]:0x59d8d3,
};
const BLOCK_NAME = {
  [BLOCK.GRASS]:'잔디 블록', [BLOCK.DIRT]:'흙', [BLOCK.STONE]:'돌',
  [BLOCK.LOG]:'통나무', [BLOCK.LEAVES]:'나뭇잎', [BLOCK.PLANKS]:'판자', [BLOCK.TABLE]:'제작대',
  [BLOCK.BEDROCK]:'기반암', [BLOCK.COAL_ORE]:'석탄 광석', [BLOCK.IRON_ORE]:'철 광석', [BLOCK.DIAMOND_ORE]:'다이아몬드 광석',
};
const ITEM = {
  STICK:100, WOOD_PICKAXE:101, STONE_PICKAXE:102, COAL:103, IRON:104, DIAMOND:105, IRON_PICKAXE:106, DIAMOND_PICKAXE:107,
  WOOD_AXE:108, STONE_AXE:109, IRON_AXE:110, DIAMOND_AXE:111,
  WOOD_SHOVEL:112, STONE_SHOVEL:113, IRON_SHOVEL:114, DIAMOND_SHOVEL:115,
  WOOD_SWORD:116, STONE_SWORD:117, IRON_SWORD:118, DIAMOND_SWORD:119,
};
const TIER_COLOR = { wood:0xd2a86a, stone:0xb0b0b0, iron:0xe3e3e3, diamond:0x8ff5f0 };
const ITEM_COLOR = {
  [ITEM.STICK]:0xa9784a,
  [ITEM.WOOD_PICKAXE]:TIER_COLOR.wood, [ITEM.WOOD_AXE]:TIER_COLOR.wood, [ITEM.WOOD_SHOVEL]:TIER_COLOR.wood, [ITEM.WOOD_SWORD]:TIER_COLOR.wood,
  [ITEM.STONE_PICKAXE]:TIER_COLOR.stone, [ITEM.STONE_AXE]:TIER_COLOR.stone, [ITEM.STONE_SHOVEL]:TIER_COLOR.stone, [ITEM.STONE_SWORD]:TIER_COLOR.stone,
  [ITEM.IRON_PICKAXE]:TIER_COLOR.iron, [ITEM.IRON_AXE]:TIER_COLOR.iron, [ITEM.IRON_SHOVEL]:TIER_COLOR.iron, [ITEM.IRON_SWORD]:TIER_COLOR.iron,
  [ITEM.DIAMOND_PICKAXE]:TIER_COLOR.diamond, [ITEM.DIAMOND_AXE]:TIER_COLOR.diamond, [ITEM.DIAMOND_SHOVEL]:TIER_COLOR.diamond, [ITEM.DIAMOND_SWORD]:TIER_COLOR.diamond,
  [ITEM.COAL]:0x2a2a2a, [ITEM.IRON]:0xe0c39a, [ITEM.DIAMOND]:0x7fe8e3,
};
const ITEM_NAME = {
  [ITEM.STICK]:'막대기',
  [ITEM.WOOD_PICKAXE]:'나무 곡괭이', [ITEM.WOOD_AXE]:'나무 도끼', [ITEM.WOOD_SHOVEL]:'나무 삽', [ITEM.WOOD_SWORD]:'나무 검',
  [ITEM.STONE_PICKAXE]:'돌 곡괭이', [ITEM.STONE_AXE]:'돌 도끼', [ITEM.STONE_SHOVEL]:'돌 삽', [ITEM.STONE_SWORD]:'돌 검',
  [ITEM.IRON_PICKAXE]:'철 곡괭이', [ITEM.IRON_AXE]:'철 도끼', [ITEM.IRON_SHOVEL]:'철 삽', [ITEM.IRON_SWORD]:'철 검',
  [ITEM.DIAMOND_PICKAXE]:'다이아몬드 곡괭이', [ITEM.DIAMOND_AXE]:'다이아몬드 도끼', [ITEM.DIAMOND_SHOVEL]:'다이아몬드 삽', [ITEM.DIAMOND_SWORD]:'다이아몬드 검',
  [ITEM.COAL]:'석탄', [ITEM.IRON]:'철', [ITEM.DIAMOND]:'다이아몬드',
};
function colorOf(id){ return BLOCK_COLOR[id] !== undefined ? BLOCK_COLOR[id] : ITEM_COLOR[id]; }
function nameOf(id){ return BLOCK_NAME[id] !== undefined ? BLOCK_NAME[id] : ITEM_NAME[id]; }
const PICKAXE_TIER = { [ITEM.WOOD_PICKAXE]:1, [ITEM.STONE_PICKAXE]:2, [ITEM.IRON_PICKAXE]:3, [ITEM.DIAMOND_PICKAXE]:4 };
const BLOCK_MIN_TIER = { [BLOCK.STONE]:1, [BLOCK.COAL_ORE]:1, [BLOCK.IRON_ORE]:2, [BLOCK.DIAMOND_ORE]:3 };
function canMine(blockId, heldItemId){
  const minTier = BLOCK_MIN_TIER[blockId];
  if (minTier===undefined) return true;
  return (PICKAXE_TIER[heldItemId]||0) >= minTier;
}
function dropForBlock(blockId){
  if (blockId===BLOCK.COAL_ORE) return ITEM.COAL;
  if (blockId===BLOCK.IRON_ORE) return ITEM.IRON;
  if (blockId===BLOCK.DIAMOND_ORE) return ITEM.DIAMOND;
  return blockId;
}

// ---------- World storage ----------
const CHUNK_SIZE = 12;
const WORLD_HEIGHT = 120;
const SURFACE_BASE = 100;
const RENDER_DIST = 3;

const world = new Map();
const chunkData = new Map();
const chunkMeshes = new Map();

function key(x,y,z){ return x+','+y+','+z; }
function getBlock(x,y,z){
  if (y<0 || y>=WORLD_HEIGHT) return BLOCK.AIR;
  const v = world.get(key(x,y,z));
  return v===undefined ? BLOCK.AIR : v;
}
function setBlock(x,y,z,id){
  if (id===BLOCK.AIR) world.delete(key(x,y,z));
  else world.set(key(x,y,z), id);
}

function hash2(x,y){
  let h = (x*374761393 + y*668265263) | 0;
  h = (h ^ (h >> 13)) * 1274126177;
  h = h ^ (h >> 16);
  return ((h >>> 0) % 100000) / 100000;
}
function smooth(t){ return t*t*(3-2*t); }
function lerp(a,b,t){ return a + (b-a)*t; }
function valueNoise(x,y){
  const x0=Math.floor(x), y0=Math.floor(y), x1=x0+1, y1=y0+1;
  const sx=smooth(x-x0), sy=smooth(y-y0);
  const n00=hash2(x0,y0), n10=hash2(x1,y0), n01=hash2(x0,y1), n11=hash2(x1,y1);
  return lerp(lerp(n00,n10,sx), lerp(n01,n11,sx), sy);
}
function terrainHeight(x,z){
  let h = valueNoise(x*0.045, z*0.045)*7;
  h += valueNoise(x*0.09+50, z*0.09+50)*3;
  return Math.floor(h) + SURFACE_BASE;
}
function treeChance(x,z){ return hash2(x*3.1+7,z*7.3+3); }
function oreForStone(x,y,z){
  const r = hash2(x*13.7 + y*0.91, z*17.3 + y*1.37);
  if (y < 16  && r > 0.985) return BLOCK.DIAMOND_ORE;
  if (y < 45  && r > 0.965) return BLOCK.IRON_ORE;
  if (y < 95  && r > 0.95)  return BLOCK.COAL_ORE;
  return BLOCK.STONE;
}

function generateChunkData(cx, cz){
  const ck = cx+','+cz;
  if (chunkData.has(ck)) return;
  chunkData.set(ck, true);
  const startX = cx*CHUNK_SIZE, startZ = cz*CHUNK_SIZE;
  for (let lx=0; lx<CHUNK_SIZE; lx++){
    for (let lz=0; lz<CHUNK_SIZE; lz++){
      const x = startX+lx, z = startZ+lz;
      const h = terrainHeight(x,z);
      for (let y=0; y<=h; y++){
        let id;
        if (y===0) id = BLOCK.BEDROCK;
        else if (y===h) id = BLOCK.GRASS;
        else if (y>=h-3) id = BLOCK.DIRT;
        else id = oreForStone(x,y,z);
        setBlock(x,y,z,id);
      }
      if (lx>1 && lx<CHUNK_SIZE-2 && lz>1 && lz<CHUNK_SIZE-2 && treeChance(x,z) > 0.975){
        const trunkH = 3 + Math.floor(treeChance(x+1,z+1)*2);
        for (let ty=1; ty<=trunkH; ty++) setBlock(x,h+ty,z,BLOCK.LOG);
        for (let dx=-2; dx<=2; dx++){
          for (let dz=-2; dz<=2; dz++){
            for (let dy=trunkH-1; dy<=trunkH+1; dy++){
              if (Math.abs(dx)===2 && Math.abs(dz)===2) continue;
              if (dx===0 && dz===0 && dy<trunkH+1) continue;
              if (getBlock(x+dx,h+dy,z+dz)===BLOCK.AIR) setBlock(x+dx,h+dy,z+dz,BLOCK.LEAVES);
            }
          }
        }
        setBlock(x,h+trunkH+2,z,BLOCK.LEAVES);
      }
    }
  }
}

// ---------- Three.js setup ----------
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87CEEB);
scene.fog = new THREE.Fog(0x87CEEB, 40, 95);

const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 300);
const renderer = new THREE.WebGLRenderer({ antialias:true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio,2));
document.body.appendChild(renderer.domElement);

const hemi = new THREE.HemisphereLight(0xffffff, 0x6b6b6b, 0.9);
scene.add(hemi);
const sun = new THREE.DirectionalLight(0xffffff, 1.0);
scene.add(sun);
const moonLight = new THREE.DirectionalLight(0x8fa8ff, 0.0);
scene.add(moonLight);

const sunMesh = new THREE.Mesh(new THREE.CircleGeometry(6,16), new THREE.MeshBasicMaterial({color:0xfff2b0, fog:false}));
scene.add(sunMesh);
const moonMesh = new THREE.Mesh(new THREE.CircleGeometry(4.2,16), new THREE.MeshBasicMaterial({color:0xdfe6ff, fog:false}));
scene.add(moonMesh);

window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

// ---------- Player ----------
const player = {
  pos: new THREE.Vector3(0, 20, 0),
  vel: new THREE.Vector3(0,0,0),
  yaw: 0, pitch: 0,
  onGround: false,
  width: 0.6, height: 1.8,
};

const bodyGroup = new THREE.Group();
scene.add(bodyGroup);
function bodyPart(w,h,d,color,y){
  const m = new THREE.Mesh(new THREE.BoxGeometry(w,h,d), new THREE.MeshLambertMaterial({color}));
  m.position.y = y;
  return m;
}
const skin = 0xd9a066, shirt = 0x3d6fa8, pants = 0x35405c;
bodyGroup.add(bodyPart(0.5,0.7,0.3, shirt, 0.55));
bodyGroup.add(bodyPart(0.45,0.45,0.45, skin, 1.05));
const armR = bodyPart(0.18,0.6,0.18, skin, 0.55); armR.position.x=0.34; bodyGroup.add(armR);
const armL = bodyPart(0.18,0.6,0.18, skin, 0.55); armL.position.x=-0.34; bodyGroup.add(armL);
const legR = bodyPart(0.2,0.6,0.2, pants, -0.35); legR.position.x=0.14; bodyGroup.add(legR);
const legL = bodyPart(0.2,0.6,0.2, pants, -0.35); legL.position.x=-0.14; bodyGroup.add(legL);

const handGroup = new THREE.Group();
handGroup.add(bodyPart(0.22,0.5,0.22, skin, 0));
handGroup.position.set(0.35,-0.4,-0.6);
camera.add(handGroup);
scene.add(camera);

let thirdPerson = false;

// ---------- Chunk mesh building ----------
function buildChunkMesh(cx, cz){
  generateChunkData(cx,cz);
  generateChunkData(cx+1,cz); generateChunkData(cx-1,cz);
  generateChunkData(cx,cz+1); generateChunkData(cx,cz-1);

  const ck = cx+','+cz;
  const old = chunkMeshes.get(ck);
  if (old){ scene.remove(old); old.geometry.dispose(); old.material.dispose(); }

  const positions=[], normals=[], colors=[];
  const faceDirs = [
    {n:[1,0,0], corners:[[1,0,0],[1,1,0],[1,1,1],[1,0,1]]},
    {n:[-1,0,0], corners:[[0,0,1],[0,1,1],[0,1,0],[0,0,0]]},
    {n:[0,1,0], corners:[[0,1,0],[0,1,1],[1,1,1],[1,1,0]]},
    {n:[0,-1,0], corners:[[0,0,1],[0,0,0],[1,0,0],[1,0,1]]},
    {n:[0,0,1], corners:[[1,0,1],[1,1,1],[0,1,1],[0,0,1]]},
    {n:[0,0,-1], corners:[[0,0,0],[0,1,0],[1,1,0],[1,0,0]]},
  ];
  const startX = cx*CHUNK_SIZE, startZ = cz*CHUNK_SIZE;
  for (let lx=0; lx<CHUNK_SIZE; lx++){
    for (let lz=0; lz<CHUNK_SIZE; lz++){
      const x = startX+lx, z = startZ+lz;
      for (let y=0; y<WORLD_HEIGHT; y++){
        const id = getBlock(x,y,z);
        if (id===BLOCK.AIR) continue;
        const c = new THREE.Color(BLOCK_COLOR[id]);
        for (const f of faceDirs){
          const nx=x+f.n[0], ny=y+f.n[1], nz=z+f.n[2];
          if (getBlock(nx,ny,nz)!==BLOCK.AIR) continue;
          const shade = f.n[1]===1?1:(f.n[1]===-1?0.6:0.8);
          for (const co of f.corners){
            positions.push(x+co[0], y+co[1], z+co[2]);
            normals.push(f.n[0],f.n[1],f.n[2]);
            colors.push(c.r*shade, c.g*shade, c.b*shade);
          }
        }
      }
    }
  }
  const geometry = new THREE.BufferGeometry();
  const idx = [];
  for (let i=0;i<positions.length/3;i+=4){ idx.push(i,i+1,i+2, i,i+2,i+3); }
  geometry.setIndex(idx);
  geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions,3));
  geometry.setAttribute('normal', new THREE.Float32BufferAttribute(normals,3));
  geometry.setAttribute('color', new THREE.Float32BufferAttribute(colors,3));
  const material = new THREE.MeshLambertMaterial({ vertexColors:true });
  const mesh = new THREE.Mesh(geometry, material);
  scene.add(mesh);
  chunkMeshes.set(ck, mesh);
}

function chunkCoord(x,z){ return [Math.floor(x/CHUNK_SIZE), Math.floor(z/CHUNK_SIZE)]; }

function ensureDataAroundPlayer(){
  const [pcx,pcz] = chunkCoord(player.pos.x, player.pos.z);
  const R = RENDER_DIST + 1;
  for (let dx=-R; dx<=R; dx++){
    for (let dz=-R; dz<=R; dz++) generateChunkData(pcx+dx, pcz+dz);
  }
}

let chunkQueue = [];
const queuedSet = new Set();
function refreshChunkQueue(){
  ensureDataAroundPlayer();
  const [pcx,pcz] = chunkCoord(player.pos.x, player.pos.z);
  const needed = [];
  for (let dx=-RENDER_DIST; dx<=RENDER_DIST; dx++){
    for (let dz=-RENDER_DIST; dz<=RENDER_DIST; dz++){
      const cx=pcx+dx, cz=pcz+dz, ck=cx+','+cz;
      if (!chunkMeshes.has(ck) && !queuedSet.has(ck)){
        needed.push({cx,cz,dist:dx*dx+dz*dz});
        queuedSet.add(ck);
      }
    }
  }
  needed.sort((a,b)=>a.dist-b.dist);
  chunkQueue.push(...needed);
}
function processChunkQueue(){
  let budget = 1;
  while (budget>0 && chunkQueue.length){
    const {cx,cz} = chunkQueue.shift();
    queuedSet.delete(cx+','+cz);
    buildChunkMesh(cx,cz);
    budget--;
  }
}
function rebuildChunkAndNeighborsAt(x,z){
  const [cx,cz] = chunkCoord(x,z);
  buildChunkMesh(cx,cz);
  const lx = ((x%CHUNK_SIZE)+CHUNK_SIZE)%CHUNK_SIZE;
  const lz = ((z%CHUNK_SIZE)+CHUNK_SIZE)%CHUNK_SIZE;
  if (lx===0) buildChunkMesh(cx-1,cz);
  if (lx===CHUNK_SIZE-1) buildChunkMesh(cx+1,cz);
  if (lz===0) buildChunkMesh(cx,cz-1);
  if (lz===CHUNK_SIZE-1) buildChunkMesh(cx,cz+1);
}

(function spawnPlayer(){
  generateChunkData(0,0);
  const h = terrainHeight(0,0);
  player.pos.set(0.5, h+3, 0.5);
})();

// ---------- Inventory ----------
const HOTBAR_SIZE = 9, INV_SIZE = 18, BIG_STORAGE_SIZE = 49;
const hotbar = new Array(HOTBAR_SIZE).fill(null);
const invStorage = new Array(INV_SIZE).fill(null);
const bigStorage = new Array(BIG_STORAGE_SIZE).fill(null);
let selectedSlot = 0;
let cursorStack = null;

function addItem(id, count){
  count = count||1;
  const all = [...hotbar, ...invStorage];
  for (let i=0;i<all.length;i++){
    if (all[i] && all[i].id===id){ all[i].count += count; renderAll(); return; }
  }
  for (let i=0;i<hotbar.length;i++){
    if (!hotbar[i]){ hotbar[i]={id,count}; renderAll(); return; }
  }
  for (let i=0;i<invStorage.length;i++){
    if (!invStorage[i]){ invStorage[i]={id,count}; renderAll(); return; }
  }
}
function removeFromStack(arr,i,amount){
  if (!arr[i]) return;
  arr[i].count -= amount;
  if (arr[i].count<=0) arr[i]=null;
}

// ---------- UI ----------
const hotbarEl = document.getElementById('hotbar');
const invGridEl = document.getElementById('invGrid');
const hotbarMirrorEl = document.getElementById('hotbarMirror');
const hotbarMirror2El = document.getElementById('hotbarMirror2');
const hotbarMirror3El = document.getElementById('hotbarMirror3');
const storageGridEl = document.getElementById('storageGrid');
const personalCraftGridEl = document.getElementById('personalCraftGrid');
const personalCraftOutputEl = document.getElementById('personalCraftOutput');
const tableCraftGridEl = document.getElementById('tableCraftGrid');
const tableCraftOutputEl = document.getElementById('tableCraftOutput');
const cursorItemEl = document.getElementById('cursorItem');
const recipeMsgEl = document.getElementById('recipeMsg');

const personalCraft = new Array(81).fill(null);
const tableCraft = new Array(9).fill(null);

function makeSlotEl(stack, onClick){
  const el = document.createElement('div');
  el.className = 'slot';
  if (stack){
    const sw = document.createElement('div');
    sw.className = 'swatch';
    sw.style.background = '#'+colorOf(stack.id).toString(16).padStart(6,'0');
    el.appendChild(sw);
    const cnt = document.createElement('div');
    cnt.className = 'count';
    cnt.textContent = stack.count>1? stack.count : '';
    el.appendChild(cnt);
    el.title = nameOf(stack.id);
  }
  el.addEventListener('click', onClick);
  return el;
}
function slotClickHandler(arr, i){
  return () => {
    if (cursorStack){
      if (arr[i] && arr[i].id===cursorStack.id){
        arr[i].count += cursorStack.count; cursorStack=null;
      } else {
        const tmp = arr[i]; arr[i]=cursorStack; cursorStack=tmp;
      }
    } else if (arr[i]){
      cursorStack = arr[i]; arr[i]=null;
    }
    renderAll();
  };
}
function renderHotbarStrip(container, mirrorSelectable){
  container.innerHTML='';
  hotbar.forEach((stack,i)=>{
    const el = makeSlotEl(stack, slotClickHandler(hotbar,i));
    if (mirrorSelectable && i===selectedSlot) el.classList.add('selected');
    container.appendChild(el);
  });
}
function renderAll(){
  renderHotbarStrip(hotbarEl, true);
  renderHotbarStrip(hotbarMirrorEl, false);
  renderHotbarStrip(hotbarMirror2El, false);
  renderHotbarStrip(hotbarMirror3El, false);

  invGridEl.innerHTML='';
  invStorage.forEach((stack,i)=> invGridEl.appendChild(makeSlotEl(stack, slotClickHandler(invStorage,i))));

  storageGridEl.innerHTML='';
  bigStorage.forEach((stack,i)=> storageGridEl.appendChild(makeSlotEl(stack, slotClickHandler(bigStorage,i))));

  personalCraftGridEl.innerHTML='';
  personalCraft.forEach((stack,i)=> personalCraftGridEl.appendChild(makeSlotEl(stack, ()=>{
    slotClickHandler(personalCraft,i)(); updatePersonalCraftOutput();
  })));
  updatePersonalCraftOutput();

  tableCraftGridEl.innerHTML='';
  tableCraft.forEach((stack,i)=> tableCraftGridEl.appendChild(makeSlotEl(stack, ()=>{
    slotClickHandler(tableCraft,i)(); updateTableCraftOutput();
  })));
  updateTableCraftOutput();

  if (cursorStack){
    cursorItemEl.style.display='flex';
    cursorItemEl.querySelector('.swatch').style.background = '#'+colorOf(cursorStack.id).toString(16).padStart(6,'0');
    cursorItemEl.querySelector('.count').textContent = cursorStack.count>1?cursorStack.count:'';
  } else cursorItemEl.style.display='none';
}
document.addEventListener('mousemove', e=>{
  cursorItemEl.style.left = (e.clientX-20)+'px';
  cursorItemEl.style.top = (e.clientY-20)+'px';
});

// ---------- Recipes ----------
function toolSet(materialId, tag){
  return [
    { input:{[materialId]:3, [ITEM.STICK]:2}, output:{id:ITEM[tag+'_PICKAXE'], count:1} },
    { input:{[materialId]:3, [ITEM.STICK]:1}, output:{id:ITEM[tag+'_AXE'], count:1} },
    { input:{[materialId]:1, [ITEM.STICK]:2}, output:{id:ITEM[tag+'_SHOVEL'], count:1} },
    { input:{[materialId]:2, [ITEM.STICK]:1}, output:{id:ITEM[tag+'_SWORD'], count:1} },
  ];
}
const RECIPES = [
  { input:{[BLOCK.LOG]:1}, output:{id:BLOCK.PLANKS, count:4} },
  { input:{[BLOCK.PLANKS]:2}, output:{id:ITEM.STICK, count:4} },
  { input:{[BLOCK.PLANKS]:4}, output:{id:BLOCK.TABLE, count:1} },
  ...toolSet(BLOCK.PLANKS, 'WOOD'),
  ...toolSet(BLOCK.STONE, 'STONE'),
  ...toolSet(ITEM.IRON, 'IRON'),
  ...toolSet(ITEM.DIAMOND, 'DIAMOND'),
];
const TABLE_RECIPES = RECIPES;

function tally(arr){
  const t = {};
  arr.forEach(s=>{ if (s) t[s.id]=(t[s.id]||0)+s.count; });
  return t;
}
function matchRecipe(arr, recipeList){
  const t = tally(arr);
  const keys = Object.keys(t);
  for (const r of recipeList){
    const rk = Object.keys(r.input);
    if (rk.length !== keys.length) continue;
    let ok = true;
    for (const k of rk){ if (t[k] !== r.input[k]) { ok=false; break; } }
    if (ok) return r;
  }
  return null;
}
let personalMatched = null, tableMatched = null;
function updatePersonalCraftOutput(){
  personalMatched = matchRecipe(personalCraft, RECIPES);
  personalCraftOutputEl.innerHTML='';
  if (personalMatched){
    personalCraftOutputEl.appendChild(makeSlotEl({id:personalMatched.output.id, count:personalMatched.output.count}, ()=>{
      if (!personalMatched || cursorStack) return;
      cursorStack = {id:personalMatched.output.id, count:personalMatched.output.count};
      for (let i=0;i<personalCraft.length;i++) personalCraft[i]=null;
      renderAll();
    }));
  }
}
function updateTableCraftOutput(){
  tableMatched = matchRecipe(tableCraft, TABLE_RECIPES);
  tableCraftOutputEl.innerHTML='';
  recipeMsgEl.textContent = tableMatched ? ('제작 가능: '+nameOf(tableMatched.output.id)) : '';
  if (tableMatched){
    tableCraftOutputEl.appendChild(makeSlotEl({id:tableMatched.output.id, count:tableMatched.output.count}, ()=>{
      if (!tableMatched || cursorStack) return;
      cursorStack = {id:tableMatched.output.id, count:tableMatched.output.count};
      for (let i=0;i<tableCraft.length;i++) tableCraft[i]=null;
      renderAll();
    }));
  }
}

// ---------- Panels ----------
const panelOverlay = document.getElementById('panelOverlay');
const invPanel = document.getElementById('invPanel');
const tablePanel = document.getElementById('tablePanel');
const storagePanel = document.getElementById('storagePanel');
let panelOpen = null;

function openPanel(which){
  panelOpen = which;
  panelOverlay.style.display='flex';
  invPanel.style.display = which==='inv' ? 'block':'none';
  tablePanel.style.display = which==='table' ? 'block':'none';
  storagePanel.style.display = which==='storage' ? 'block':'none';
  document.exitPointerLock();
  renderAll();
}
function closePanel(){
  function returnAll(arr){ arr.forEach((s,i)=>{ if (s){ addItem(s.id,s.count); arr[i]=null; } }); }
  returnAll(personalCraft);
  returnAll(tableCraft);
  if (cursorStack){ addItem(cursorStack.id, cursorStack.count); cursorStack=null; }
  panelOpen = null;
  panelOverlay.style.display='none';
  renderer.domElement.requestPointerLock();
}

// ---------- Controls ----------
const keys = {};
let started = false;
document.getElementById('startBtn').addEventListener('click', ()=>{
  document.getElementById('startScreen').style.display='none';
  renderer.domElement.requestPointerLock();
  started = true;
});
document.addEventListener('keydown', e=>{
  keys[e.code]=true;
  if (e.code==='KeyE'){ if (panelOpen) closePanel(); else if (started) openPanel('inv'); }
  if (e.code==='KeyB'){ if (panelOpen) closePanel(); else if (started) openPanel('storage'); }
  if (e.code==='Escape' && panelOpen) closePanel();
  if (e.code==='F5'){ thirdPerson = !thirdPerson; e.preventDefault(); }
  const num = parseInt(e.key);
  if (!panelOpen && num>=1 && num<=9){ selectedSlot = num-1; renderAll(); }
});
document.addEventListener('keyup', e=> keys[e.code]=false);
document.addEventListener('mousemove', e=>{
  if (document.pointerLockElement !== renderer.domElement) return;
  player.yaw -= e.movementX * 0.0022;
  player.pitch -= e.movementY * 0.0022;
  player.pitch = Math.max(-Math.PI/2+0.05, Math.min(Math.PI/2-0.05, player.pitch));
});
renderer.domElement.addEventListener('click', ()=>{
  if (started && !panelOpen) renderer.domElement.requestPointerLock();
});

// ---------- Raycast ----------
function getLookDir(){
  return new THREE.Vector3(
    -Math.sin(player.yaw)*Math.cos(player.pitch),
    Math.sin(player.pitch),
    -Math.cos(player.yaw)*Math.cos(player.pitch)
  ).normalize();
}
function raycastBlocks(maxDist){
  const origin = new THREE.Vector3(player.pos.x, player.pos.y+player.height*0.85, player.pos.z);
  const dir = getLookDir();
  const step = 0.05;
  let prev = null;
  for (let t=0; t<maxDist; t+=step){
    const p = origin.clone().addScaledVector(dir, t);
    const bx=Math.floor(p.x), by=Math.floor(p.y), bz=Math.floor(p.z);
    if (getBlock(bx,by,bz) !== BLOCK.AIR){
      return { block:[bx,by,bz], place: prev };
    }
    prev = [bx,by,bz];
  }
  return null;
}

// ---------- Dropped items ----------
const droppedItems = [];
function spawnDroppedItem(id, x, y, z){
  const geo = new THREE.BoxGeometry(0.32,0.32,0.32);
  const mat = new THREE.MeshLambertMaterial({ color: colorOf(id) });
  const mesh = new THREE.Mesh(geo, mat);
  mesh.position.set(x + (Math.random()-0.5)*0.3, y, z + (Math.random()-0.5)*0.3);
  scene.add(mesh);
  droppedItems.push({
    mesh, id, count:1, landed:false, restY:0,
    vel: new THREE.Vector3((Math.random()-0.5)*1.2, 3.2, (Math.random()-0.5)*1.2),
  });
}
function updateDroppedItems(dt){
  const now = performance.now();
  for (let i=droppedItems.length-1; i>=0; i--){
    const it = droppedItems[i];
    if (!it.landed){
      it.vel.y -= 18*dt;
      const next = it.mesh.position.clone().addScaledVector(it.vel, dt);
      if (getBlock(Math.floor(next.x), Math.floor(next.y-0.16), Math.floor(next.z)) !== BLOCK.AIR){
        next.y = Math.floor(next.y) + 1.16;
        it.vel.set(0,0,0);
        it.landed = true;
        it.restY = next.y;
      }
      it.mesh.position.copy(next);
    
