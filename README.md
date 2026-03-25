<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>COSMOS — Solar System</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { background: #000; overflow: hidden; font-family: 'Segoe UI', sans-serif; }
    canvas { display: block; }

    .planet-label {
      color: #aaccff; font-size: 10px; letter-spacing: 2px; text-transform: uppercase;
      padding: 3px 8px; background: rgba(0,8,25,0.7);
      border: 1px solid rgba(100,150,255,0.3); border-radius: 3px;
      pointer-events: none; white-space: nowrap; transition: opacity 0.3s;
    }

    #info-panel {
      position: fixed; top: 50%; right: 28px; transform: translateY(-50%);
      width: 270px; background: rgba(0,4,18,0.88);
      border: 1px solid rgba(80,130,255,0.25); border-radius: 10px;
      padding: 26px; color: #fff; backdrop-filter: blur(14px);
      display: none; z-index: 20;
    }
    #info-panel.visible { display: block; animation: slideIn 0.3s ease; }
    #info-panel h2 { font-size: 1.5rem; font-weight: 200; letter-spacing: 5px; color: #99bbff; margin-bottom: 4px; }
    #info-panel .sub { font-size: 0.6rem; color: #ffffff33; letter-spacing: 3px; text-transform: uppercase; margin-bottom: 14px; }
    #info-panel p { font-size: 0.78rem; line-height: 1.8; color: #9999bb; }
    #info-panel .stats { margin-top: 16px; border-top: 1px solid #ffffff0d; padding-top: 12px; }
    #info-panel .stat { display: flex; justify-content: space-between; padding: 5px 0; font-size: 0.7rem; border-bottom: 1px solid #ffffff08; }
    #info-panel .stat span:first-child { color: #ffffff44; }
    #info-panel .stat span:last-child { color: #7799ff; }
    #info-close { position: absolute; top: 14px; right: 14px; background: none; border: none; color: #ffffff33; cursor: pointer; font-size: 0.95rem; }
    #info-close:hover { color: #fff; }

    #hud { position: fixed; bottom: 22px; left: 22px; display: flex; flex-direction: column; gap: 8px; z-index: 20; }
    .hud-btn {
      padding: 7px 18px; background: rgba(255,255,255,0.05);
      border: 1px solid rgba(255,255,255,0.13); color: #ffffffaa;
      border-radius: 5px; cursor: pointer; font-size: 0.68rem;
      letter-spacing: 2px; text-transform: uppercase;
      backdrop-filter: blur(8px); transition: all 0.2s;
    }
    .hud-btn:hover, .hud-btn.active { background: rgba(60,110,255,0.22); border-color: rgba(90,140,255,0.5); color: #aaccff; }

    #title { position: fixed; top: 26px; left: 50%; transform: translateX(-50%); text-align: center; z-index: 10; pointer-events: none; }
    #title h1 { font-size: 1.9rem; font-weight: 100; letter-spacing: 14px; color: #fff; text-shadow: 0 0 50px #3355ff99; }
    #title p { font-size: 0.58rem; letter-spacing: 5px; color: #ffffff2a; margin-top: 5px; text-transform: uppercase; }

    #audio-btn {
      position: fixed; top: 26px; left: 22px;
      background: rgba(255,255,255,0.05); border: 1px solid rgba(255,255,255,0.13);
      color: #ffffff77; border-radius: 5px; cursor: pointer;
      font-size: 0.68rem; letter-spacing: 2px; padding: 7px 14px;
      backdrop-filter: blur(8px); z-index: 20; text-transform: uppercase; transition: all 0.2s;
    }
    #audio-btn:hover { background: rgba(60,110,255,0.22); color: #aaccff; }

    #tour-tag {
      position: fixed; top: 26px; right: 22px;
      font-size: 0.6rem; letter-spacing: 3px; color: #aaccff77;
      z-index: 20; display: none; text-transform: uppercase;
    }
    #tour-tag.on { display: block; }

    #wasd-hint {
      position: fixed; bottom: 22px; right: 22px;
      font-size: 0.6rem; color: #ffffff33; letter-spacing: 1px;
      z-index: 20; display: none; line-height: 2; text-align: right;
    }
    #wasd-hint.on { display: block; }

    #label-renderer { position: fixed; top: 0; left: 0; pointer-events: none; z-index: 5; }

    @keyframes slideIn {
      from { opacity: 0; transform: translateY(-50%) translateX(14px); }
      to   { opacity: 1; transform: translateY(-50%) translateX(0); }
    }
  </style>
</head>
<body>

<div id="title"><h1>COSMOS</h1><p>Our Solar System · Interactive</p></div>
<button id="audio-btn">♪ 사운드 켜기</button>

<div id="info-panel">
  <button id="info-close">✕</button>
  <h2 id="info-name"></h2>
  <div class="sub" id="info-eng"></div>
  <p id="info-desc"></p>
  <div class="stats" id="info-stats"></div>
</div>

<div id="hud">
  <button class="hud-btn" id="btn-tour">⟳ 자동 투어</button>
  <button class="hud-btn" id="btn-ship">✦ 우주선 모드</button>
  <button class="hud-btn" id="btn-reset">⌂ 초기화</button>
</div>

<div id="tour-tag">● AUTO TOUR</div>
<div id="wasd-hint">W/S · 전진 / 후진<br>A/D · 좌 / 우<br>Q/E · 상 / 하<br>ESC · 종료</div>

<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.165.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.165.0/examples/jsm/"
  }
}
</script>

<script type="module">
import * as THREE from 'three';
import { OrbitControls }                      from 'three/addons/controls/OrbitControls.js';
import { CSS2DRenderer, CSS2DObject }          from 'three/addons/renderers/CSS2DRenderer.js';
import { EffectComposer }                      from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass }                          from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass }                     from 'three/addons/postprocessing/UnrealBloomPass.js';

// ═══════════════════════════════════════════════════════
//  PLANET DATA
// ═══════════════════════════════════════════════════════
const PLANETS = [
  { name:'수성', eng:'Mercury', dist:16,  r:0.38, color:0xaaaaaa, spd:4.74, tilt:0.03,
    desc:'태양에 가장 가까운 행성. 대기가 거의 없어 낮에는 430°C, 밤에는 -180°C로 극단적인 온도 차이를 보입니다.',
    stats:[['지름','4,879 km'],['공전주기','88일'],['위성','0개'],['온도','-180 ~ 430°C']] },
  { name:'금성', eng:'Venus',   dist:24,  r:0.95, color:0xddaa55, spd:1.76, tilt:177,
    desc:'태양계에서 가장 뜨거운 행성. 두꺼운 CO₂ 대기의 온실효과로 평균 온도가 약 462°C에 달합니다.',
    stats:[['지름','12,104 km'],['공전주기','225일'],['위성','0개'],['온도','~462°C']] },
  { name:'지구', eng:'Earth',   dist:34,  r:1.0,  color:0x2255bb, spd:1.0,  tilt:23.4, hasMoon:true,
    desc:'우리가 사는 푸른 행성. 액체 상태의 물과 적절한 대기를 가진, 생명체가 사는 유일하게 알려진 행성입니다.',
    stats:[['지름','12,742 km'],['공전주기','365일'],['위성','1개 (달)'],['온도','-88 ~ 58°C']] },
  { name:'화성', eng:'Mars',    dist:46,  r:0.53, color:0xcc4422, spd:0.53, tilt:25.2,
    desc:'붉은 행성. 산화철로 덮인 표면이 붉게 보이며, 태양계 최고봉 올림푸스 산(25km)이 있습니다.',
    stats:[['지름','6,779 km'],['공전주기','687일'],['위성','2개'],['온도','-125 ~ 20°C']] },
  { name:'목성', eng:'Jupiter', dist:72,  r:4.2,  color:0xddaa77, spd:0.084,tilt:3.1,  isJupiter:true,
    desc:'태양계 최대 행성. 300년 이상 지속되는 대적점 폭풍과 강렬한 줄무늬가 특징입니다.',
    stats:[['지름','139,820 km'],['공전주기','12년'],['위성','95개'],['온도','-110°C']] },
  { name:'토성', eng:'Saturn',  dist:100, r:3.5,  color:0xddcc88, spd:0.034,tilt:26.7, hasRings:true,
    desc:'아름다운 고리를 가진 행성. 고리는 주로 얼음과 암석으로 이루어져 있으며 두께는 수십 미터에 불과합니다.',
    stats:[['지름','116,460 km'],['공전주기','29년'],['위성','146개'],['밀도','물보다 낮음']] },
  { name:'천왕성', eng:'Uranus', dist:126, r:2.0, color:0x88ccee, spd:0.012, tilt:97.8,
    desc:'옆으로 누워 공전하는 행성. 자전축이 98도 기울어져 있어 계절 변화가 극단적입니다.',
    stats:[['지름','50,724 km'],['공전주기','84년'],['위성','28개'],['온도','-195°C']] },
  { name:'해왕성', eng:'Neptune', dist:148, r:1.9, color:0x3344cc, spd:0.006, tilt:28.3,
    desc:'태양계 가장 바깥 행성. 초속 600m에 달하는 강풍이 부는 얼음 거대 행성입니다.',
    stats:[['지름','49,244 km'],['공전주기','165년'],['위성','16개'],['온도','-200°C']] },
];

// ═══════════════════════════════════════════════════════
//  RENDERER
// ═══════════════════════════════════════════════════════
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 0.85;
document.body.appendChild(renderer.domElement);

const labelRenderer = new CSS2DRenderer();
labelRenderer.setSize(innerWidth, innerHeight);
labelRenderer.domElement.id = 'label-renderer';
document.body.appendChild(labelRenderer.domElement);

const scene  = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(55, innerWidth / innerHeight, 0.1, 3000);
camera.position.set(0, 80, 220);

const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.05;
controls.minDistance = 4;
controls.maxDistance = 700;

const composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));
const bloom = new UnrealBloomPass(new THREE.Vector2(innerWidth, innerHeight), 1.15, 0.5, 0.08);
composer.addPass(bloom);

// ═══════════════════════════════════════════════════════
//  MILKY WAY + STARS
// ═══════════════════════════════════════════════════════
(function buildMilkyWay() {
  const N = 14000;
  const pos = new Float32Array(N * 3), col = new Float32Array(N * 3);
  for (let i = 0; i < N; i++) {
    const theta = Math.random() * Math.PI * 2;
    const phi   = (Math.random() - 0.5) * Math.PI * (0.15 + Math.random() * 0.85);
    const r     = 900 + Math.random() * 300;
    pos[i*3]   = r * Math.cos(phi) * Math.cos(theta);
    pos[i*3+1] = r * Math.sin(phi);
    pos[i*3+2] = r * Math.cos(phi) * Math.sin(theta);
    const t = Math.random();
    if (t < 0.55)      { col[i*3]=1;   col[i*3+1]=1;   col[i*3+2]=1;   }
    else if (t < 0.72) { col[i*3]=0.7; col[i*3+1]=0.8; col[i*3+2]=1;   }
    else if (t < 0.87) { col[i*3]=1;   col[i*3+1]=0.9; col[i*3+2]=0.7; }
    else               { col[i*3]=1;   col[i*3+1]=0.6; col[i*3+2]=0.5; }
  }
  const geo = new THREE.BufferGeometry();
  geo.setAttribute('position', new THREE.BufferAttribute(pos, 3));
  geo.setAttribute('color',    new THREE.BufferAttribute(col, 3));
  scene.add(new THREE.Points(geo, new THREE.PointsMaterial({ size: 0.9, vertexColors: true, transparent: true, opacity: 0.85 })));
})();

(function buildForegroundStars() {
  const pos = new Float32Array(4000 * 3);
  for (let i = 0; i < pos.length; i++) pos[i] = (Math.random() - 0.5) * 700;
  const geo = new THREE.BufferGeometry();
  geo.setAttribute('position', new THREE.BufferAttribute(pos, 3));
  scene.add(new THREE.Points(geo, new THREE.PointsMaterial({ color: 0xffffff, size: 0.28, transparent: true, opacity: 0.65 })));
})();

// ═══════════════════════════════════════════════════════
//  NEBULA SPRITES
// ═══════════════════════════════════════════════════════
function makeNebula(x, y, z, hex, op, scale) {
  const cv = document.createElement('canvas'); cv.width = cv.height = 256;
  const ctx = cv.getContext('2d');
  const g = ctx.createRadialGradient(128,128,0,128,128,128);
  g.addColorStop(0,   hex+'cc');
  g.addColorStop(0.4, hex+'44');
  g.addColorStop(1,   hex+'00');
  ctx.fillStyle = g; ctx.fillRect(0,0,256,256);
  const sp = new THREE.Sprite(new THREE.SpriteMaterial({
    map: new THREE.CanvasTexture(cv), transparent: true, opacity: op,
    blending: THREE.AdditiveBlending, depthWrite: false
  }));
  sp.position.set(x,y,z); sp.scale.setScalar(scale); scene.add(sp);
}
makeNebula(-200,  60, -450, '#2244cc', 0.22, 550);
makeNebula( 320, -30, -550, '#aa2288', 0.18, 600);
makeNebula( 80,   90, -380, '#116688', 0.20, 450);
makeNebula(-130, -70, -320, '#441166', 0.24, 480);
makeNebula( 160,  40,  -90, '#884422', 0.16, 350);

// ═══════════════════════════════════════════════════════
//  SUN
// ═══════════════════════════════════════════════════════
const sunGroup = new THREE.Group(); scene.add(sunGroup);
const sunMesh  = new THREE.Mesh(new THREE.SphereGeometry(6,64,64), new THREE.MeshBasicMaterial({ color:0xffcc33 }));
sunGroup.add(sunMesh);

for (let i = 0; i < 4; i++) {
  const cv = document.createElement('canvas'); cv.width = cv.height = 256;
  const ctx = cv.getContext('2d');
  const g = ctx.createRadialGradient(128,128,0,128,128,128);
  g.addColorStop(0,    '#ffee88ff');
  g.addColorStop(0.15, '#ffaa2299');
  g.addColorStop(0.5,  '#ff440022');
  g.addColorStop(1,    '#ff000000');
  ctx.fillStyle = g; ctx.fillRect(0,0,256,256);
  const sp = new THREE.Sprite(new THREE.SpriteMaterial({
    map: new THREE.CanvasTexture(cv), transparent: true,
    opacity: 0.55 - i * 0.1, blending: THREE.AdditiveBlending, depthWrite: false
  }));
  sp.scale.setScalar(24 + i * 16); sunGroup.add(sp);
}
const sunLabel = (() => {
  const d = document.createElement('div'); d.className = 'planet-label'; d.textContent = 'SUN';
  const obj = new CSS2DObject(d); obj.position.set(0,8,0); sunGroup.add(obj); return obj;
})();

const sunLight = new THREE.PointLight(0xffddaa, 2200, 1400);
scene.add(sunLight);
scene.add(new THREE.AmbientLight(0x111133, 1.0));

// ═══════════════════════════════════════════════════════
//  TEXTURE HELPERS
// ═══════════════════════════════════════════════════════
function makeBaseTex(baseColor, patches = [], cloudAlpha = 0) {
  const sz = 512;
  const cv = document.createElement('canvas'); cv.width = cv.height = sz;
  const ctx = cv.getContext('2d');
  const bc = new THREE.Color(baseColor);
  ctx.fillStyle = `rgb(${(bc.r*255)|0},${(bc.g*255)|0},${(bc.b*255)|0})`;
  ctx.fillRect(0,0,sz,sz);
  patches.forEach(([col, count, minR, maxR]) => {
    for (let i = 0; i < count; i++) {
      const x = Math.random()*sz, y = Math.random()*sz;
      const r = minR + Math.random()*(maxR-minR);
      const g2 = ctx.createRadialGradient(x,y,0,x,y,r);
      g2.addColorStop(0, col+'cc'); g2.addColorStop(1, col+'00');
      ctx.fillStyle = g2; ctx.beginPath(); ctx.arc(x,y,r,0,Math.PI*2); ctx.fill();
    }
  });
  if (cloudAlpha > 0) {
    for (let i = 0; i < 18; i++) {
      const x=Math.random()*sz, y=Math.random()*sz, r=10+Math.random()*55;
      const g2 = ctx.createRadialGradient(x,y,0,x,y,r);
      g2.addColorStop(0, `rgba(255,255,255,${cloudAlpha})`); g2.addColorStop(1, 'rgba(255,255,255,0)');
      ctx.fillStyle = g2; ctx.beginPath(); ctx.ellipse(x,y,r*2.2,r*0.55,Math.random()*Math.PI,0,Math.PI*2); ctx.fill();
    }
  }
  return new THREE.CanvasTexture(cv);
}

function makeEarthTex() {
  const sz = 512;
  const cv = document.createElement('canvas'); cv.width = cv.height = sz;
  const ctx = cv.getContext('2d');
  ctx.fillStyle = '#091830'; ctx.fillRect(0,0,sz,sz);
  const continents = ['#1a5c2a','#2a7a3a','#3a8c4a','#2d6e30','#1e5528','#4a9050','#235c25'];
  for (let i = 0; i < 14; i++) {
    const x=Math.random()*sz, y=Math.random()*sz, r=18+Math.random()*80;
    const c = continents[i%continents.length];
    const g2 = ctx.createRadialGradient(x,y,0,x,y,r);
    g2.addColorStop(0,c+'ff'); g2.addColorStop(0.6,c+'88'); g2.addColorStop(1,c+'00');
    ctx.fillStyle=g2; ctx.beginPath(); ctx.arc(x,y,r,0,Math.PI*2); ctx.fill();
  }
  for (let i = 0; i < 20; i++) {
    const x=Math.random()*sz, y=Math.random()*sz, r=8+Math.random()*50;
    const g2=ctx.createRadialGradient(x,y,0,x,y,r);
    g2.addColorStop(0,'rgba(255,255,255,0.35)'); g2.addColorStop(1,'rgba(255,255,255,0)');
    ctx.fillStyle=g2; ctx.beginPath(); ctx.ellipse(x,y,r*2.5,r*0.6,Math.random()*Math.PI,0,Math.PI*2); ctx.fill();
  }
  return new THREE.CanvasTexture(cv);
}

function makeJupiterTex() {
  const sz=512; const cv=document.createElement('canvas'); cv.width=cv.height=sz;
  const ctx=cv.getContext('2d');
  const bands=[0xddaa77,0xcc8855,0xeebbaa,0xcc9966,0xddaa88,0xbb8844,0xeeccaa,0xcc7755,0xddaa99,0xcc8866];
  bands.forEach((col,i)=>{
    const c=new THREE.Color(col);
    ctx.fillStyle=`rgb(${(c.r*255)|0},${(c.g*255)|0},${(c.b*255)|0})`;
    ctx.fillRect(0, i*(sz/bands.length), sz, sz/bands.length+2);
  });
  const gx=sz*0.65,gy=sz*0.55,gr=30;
  const gg=ctx.createRadialGradient(gx,gy,0,gx,gy,gr);
  gg.addColorStop(0,'#cc3311ff'); gg.addColorStop(0.7,'#aa221188'); gg.addColorStop(1,'#aa221100');
  ctx.fillStyle=gg; ctx.beginPath(); ctx.ellipse(gx,gy,gr,gr*0.55,0,0,Math.PI*2); ctx.fill();
  return new THREE.CanvasTexture(cv);
}

// ═══════════════════════════════════════════════════════
//  TERMINATOR SHADER (day / night)
// ═══════════════════════════════════════════════════════
const sunDirU = new THREE.Vector3(1,0.25,0).normalize();

function makeTerminatorMat(map) {
  return new THREE.ShaderMaterial({
    uniforms: { map: { value: map }, sunDir: { value: sunDirU } },
    vertexShader:`
      varying vec2 vUv; varying vec3 vWorldNormal;
      void main(){
        vUv=uv;
        vWorldNormal=normalize((modelMatrix*vec4(normal,0.0)).xyz);
        gl_Position=projectionMatrix*modelViewMatrix*vec4(position,1.0);
      }`,
    fragmentShader:`
      uniform sampler2D map; uniform vec3 sunDir;
      varying vec2 vUv; varying vec3 vWorldNormal;
      void main(){
        vec4 tex=texture2D(map,vUv);
        float d=dot(vWorldNormal,normalize(sunDir));
        float term=smoothstep(-0.12,0.22,d);
        vec3 night=tex.rgb*0.04;
        vec3 day=tex.rgb*(0.25+max(d,0.0)*0.85);
        gl_FragColor=vec4(mix(night,day,term),1.0);
      }`,
  });
}

// ═══════════════════════════════════════════════════════
//  PLANETS
// ═══════════════════════════════════════════════════════
const planetMeshes = [];
const pAngles = PLANETS.map(() => Math.random() * Math.PI * 2);

PLANETS.forEach((data, i) => {
  // Orbit line
  const orbitPts = Array.from({length:129},(_,k)=>{
    const a=(k/128)*Math.PI*2;
    return new THREE.Vector3(Math.cos(a)*data.dist, 0, Math.sin(a)*data.dist);
  });
  scene.add(new THREE.Line(
    new THREE.BufferGeometry().setFromPoints(orbitPts),
    new THREE.LineBasicMaterial({ color:0x334466, transparent:true, opacity:0.28 })
  ));

  // Mesh
  const geo = new THREE.SphereGeometry(data.r, 64, 64);
  let mat;
  if (data.name==='지구')   mat = makeTerminatorMat(makeEarthTex());
  else if (data.isJupiter)  mat = new THREE.MeshStandardMaterial({ map:makeJupiterTex(), roughness:0.8 });
  else if (data.name==='토성') mat = new THREE.MeshStandardMaterial({ map:makeBaseTex(data.color,[['#ffffcc',8,20,60],['#ccaa66',6,10,40]]), roughness:0.75 });
  else if (data.name==='금성') mat = new THREE.MeshStandardMaterial({ map:makeBaseTex(data.color,[['#ffcc88',12,15,60]],0.35), roughness:0.9 });
  else if (data.name==='화성') mat = new THREE.MeshStandardMaterial({ map:makeBaseTex(data.color,[['#884422',10,10,45],['#ddaa88',8,5,25]]), roughness:0.95 });
  else                       mat = new THREE.MeshStandardMaterial({ map:makeBaseTex(data.color,[['#ffffff',8,8,30]]), roughness:0.85 });

  const mesh = new THREE.Mesh(geo, mat);
  mesh.rotation.z = THREE.MathUtils.degToRad(data.tilt||0);
  mesh.userData = { ...data, idx:i };
  scene.add(mesh);
  planetMeshes.push(mesh);

  // Atmosphere
  if (['지구','금성','목성','해왕성','천왕성'].includes(data.name)) {
    const atmCol = data.name==='지구'?0x2266ff : data.name==='금성'?0xffaa44 : data.color;
    const atm = new THREE.Mesh(
      new THREE.SphereGeometry(data.r*1.07,32,32),
      new THREE.MeshBasicMaterial({ color:atmCol, transparent:true, opacity:0.09, side:THREE.BackSide, blending:THREE.AdditiveBlending, depthWrite:false })
    );
    scene.add(atm); mesh.userData.atm = atm;
  }

  // Saturn rings
  if (data.hasRings) {
    const rg = new THREE.Group();
    [[data.r*1.35,data.r*2.0,0xccbbaa,0.38],[data.r*2.1,data.r*2.65,0xaaaacc,0.24],[data.r*2.7,data.r*3.05,0x9999bb,0.17]].forEach(([ir,or,col,op])=>{
      const geo2=new THREE.RingGeometry(ir,or,180);
      const p=geo2.attributes.position,u=geo2.attributes.uv;
      for(let j=0;j<p.count;j++){const x=p.getX(j),y=p.getY(j),rv=Math.sqrt(x*x+y*y);u.setXY(j,(rv-ir)/(or-ir),0);}
      rg.add(new THREE.Mesh(geo2,new THREE.MeshBasicMaterial({ color:col,transparent:true,opacity:op,side:THREE.DoubleSide,blending:THREE.AdditiveBlending,depthWrite:false })));
    });
    rg.rotation.x=Math.PI/2.25; scene.add(rg); mesh.userData.rings=rg;
  }

  // Earth moon
  if (data.hasMoon) {
    const moon = new THREE.Mesh(
      new THREE.SphereGeometry(0.27,32,32),
      new THREE.MeshStandardMaterial({ map:makeBaseTex(0x888899,[['#666677',8,5,20]]), roughness:0.95 })
    );
    scene.add(moon); mesh.userData.moon=moon;
  }

  // CSS2D label
  const div = document.createElement('div'); div.className='planet-label'; div.textContent=data.eng.toUpperCase();
  const lbl = new CSS2DObject(div); lbl.position.set(0, data.r+0.9, 0); mesh.add(lbl); mesh.userData.lbl=lbl;
});

// ═══════════════════════════════════════════════════════
//  ASTEROID BELT
// ═══════════════════════════════════════════════════════
const asteroids = [];
for (let i=0;i<450;i++){
  const a=Math.random()*Math.PI*2, r=58+Math.random()*11, sz=0.03+Math.random()*0.13;
  const m=new THREE.Mesh(new THREE.DodecahedronGeometry(sz,0),new THREE.MeshStandardMaterial({color:0x556677,roughness:0.9}));
  m.position.set(Math.cos(a)*r,(Math.random()-0.5)*2,Math.sin(a)*r);
  m.rotation.set(Math.random()*Math.PI,Math.random()*Math.PI,0);
  m.userData={a,r,spd:0.0003+Math.random()*0.0006}; scene.add(m); asteroids.push(m);
}

// ═══════════════════════════════════════════════════════
//  BLACK HOLE
// ═══════════════════════════════════════════════════════
const bhGroup = new THREE.Group(); bhGroup.position.set(-260,-25,-200); scene.add(bhGroup);

bhGroup.add(new THREE.Mesh(new THREE.SphereGeometry(9,64,64),new THREE.MeshBasicMaterial({color:0x000000})));

const accMat = new THREE.ShaderMaterial({
  uniforms:{ time:{value:0} },
  vertexShader:`varying vec2 vUv; void main(){vUv=uv;gl_Position=projectionMatrix*modelViewMatrix*vec4(position,1.0);}`,
  fragmentShader:`
    uniform float time; varying vec2 vUv;
    void main(){
      vec2 uv=vUv-0.5; float r=length(uv);
      float a=atan(uv.y,uv.x);
      float band=sin(a*7.0-time*2.5+r*22.0)*0.5+0.5;
      float fade=smoothstep(0.0,0.06,r)*smoothstep(0.5,0.08,r);
      vec3 col=mix(vec3(1.0,0.85,0.2),vec3(1.0,0.15,0.02),r*2.0)*band;
      gl_FragColor=vec4(col,fade*0.9);
    }`,
  transparent:true, side:THREE.DoubleSide, blending:THREE.AdditiveBlending, depthWrite:false,
});
const accDisk = new THREE.Mesh(new THREE.PlaneGeometry(45,45), accMat);
accDisk.rotation.x=Math.PI/2.4; bhGroup.add(accDisk);

// Lensing glow sprite
{
  const cv=document.createElement('canvas'); cv.width=cv.height=256;
  const ctx=cv.getContext('2d');
  const g=ctx.createRadialGradient(128,128,18,128,128,128);
  g.addColorStop(0,'#00000000'); g.addColorStop(0.25,'#7733ffaa'); g.addColorStop(0.55,'#2200aa33'); g.addColorStop(1,'#00000000');
  ctx.fillStyle=g; ctx.fillRect(0,0,256,256);
  const sp=new THREE.Sprite(new THREE.SpriteMaterial({map:new THREE.CanvasTexture(cv),transparent:true,opacity:0.85,blending:THREE.AdditiveBlending,depthWrite:false}));
  sp.scale.setScalar(55); bhGroup.add(sp);
}

const bhDiv=document.createElement('div'); bhDiv.className='planet-label'; bhDiv.textContent='BLACK HOLE';
const bhLbl=new CSS2DObject(bhDiv); bhLbl.position.set(0,13,0); bhGroup.add(bhLbl);

// ═══════════════════════════════════════════════════════
//  WORMHOLE
// ═══════════════════════════════════════════════════════
const whGroup = new THREE.Group(); whGroup.position.set(240,35,-140); scene.add(whGroup);

const whMat = new THREE.ShaderMaterial({
  uniforms:{ time:{value:0} },
  vertexShader:`varying vec2 vUv; void main(){vUv=uv;gl_Position=projectionMatrix*modelViewMatrix*vec4(position,1.0);}`,
  fragmentShader:`
    uniform float time; varying vec2 vUv;
    void main(){
      vec2 uv=vUv-0.5; float r=length(uv);
      float a=atan(uv.y,uv.x);
      float spiral=sin(a*6.0-r*16.0+time*3.5)*0.5+0.5;
      float hole=smoothstep(0.48,0.0,r)*smoothstep(0.0,0.07,r);
      float glow=exp(-r*4.5)*0.9;
      vec3 c1=vec3(0.2,0.5,1.0), c2=vec3(0.85,0.15,1.0);
      vec3 col=mix(c1,c2,spiral)*(hole*2.2+glow)+vec3(1.0)*glow*0.4;
      float alpha=hole*spiral*1.6+glow;
      gl_FragColor=vec4(col,clamp(alpha,0.0,1.0));
    }`,
  transparent:true, side:THREE.DoubleSide, blending:THREE.AdditiveBlending, depthWrite:false,
});
const whPlane = new THREE.Mesh(new THREE.PlaneGeometry(25,25), whMat); whGroup.add(whPlane);
const whRing  = new THREE.Mesh(new THREE.TorusGeometry(12.5,0.65,16,100),new THREE.MeshBasicMaterial({color:0x8844ff,transparent:true,opacity:0.65})); whGroup.add(whRing);

const whDiv=document.createElement('div'); whDiv.className='planet-label'; whDiv.textContent='WORMHOLE';
const whLbl=new CSS2DObject(whDiv); whLbl.position.set(0,15,0); whGroup.add(whLbl);

// ═══════════════════════════════════════════════════════
//  SHOOTING STARS (유성우)
// ═══════════════════════════════════════════════════════
const shooters=[];
let ssTimer=0;

function spawnShooter(){
  const start=new THREE.Vector3((Math.random()-0.5)*500, 120+Math.random()*80, (Math.random()-0.5)*500);
  const vel=new THREE.Vector3((Math.random()-0.5)*2-1.5, -1.8-Math.random()*1.5, (Math.random()-0.5)*1.5).normalize().multiplyScalar(4+Math.random()*5);
  const TRAIL=24;
  const pos=new Float32Array(TRAIL*3).fill(0).map((_,j)=>j%3===0?start.x:j%3===1?start.y:start.z);
  const geo=new THREE.BufferGeometry(); geo.setAttribute('position',new THREE.BufferAttribute(pos,3));
  const line=new THREE.Line(geo,new THREE.LineBasicMaterial({color:0xffffff,transparent:true,opacity:0.9}));
  scene.add(line);
  shooters.push({line,geo,pos:start.clone(),vel,life:0,max:55+Math.random()*45,trail:Array.from({length:TRAIL},()=>start.clone())});
}

function tickShooters(){
  ssTimer++;
  if(ssTimer>150+Math.random()*120){ssTimer=0;spawnShooter();if(Math.random()<0.4)spawnShooter();}
  for(let i=shooters.length-1;i>=0;i--){
    const s=shooters[i]; s.life++;
    s.pos.add(s.vel);
    s.trail.push(s.pos.clone());
    if(s.trail.length>24) s.trail.shift();
    const attr=s.geo.attributes.position;
    s.trail.forEach((p,j)=>attr.setXYZ(j,p.x,p.y,p.z));
    attr.needsUpdate=true;
    s.line.material.opacity=0.9*(1-s.life/s.max);
    if(s.life>=s.max){ scene.remove(s.line); s.geo.dispose(); shooters.splice(i,1); }
  }
}

// ═══════════════════════════════════════════════════════
//  WEB AUDIO (ambient space music)
// ═══════════════════════════════════════════════════════
let audioCtx=null, masterGain=null, audioOn=false;

function startAudio(){
  if(!audioCtx){
    audioCtx=new (window.AudioContext||window.webkitAudioContext)();
    masterGain=audioCtx.createGain(); masterGain.gain.value=0.12; masterGain.connect(audioCtx.destination);

    // Drone chord (Am-ish)
    [55,82.4,110,130.8,164.8,220].forEach(freq=>{
      const osc=audioCtx.createOscillator(), g=audioCtx.createGain();
      osc.type='sine'; osc.frequency.value=freq; g.gain.value=0.07+Math.random()*0.04;
      osc.connect(g); g.connect(masterGain); osc.start();
      const lfo=audioCtx.createOscillator(), lg=audioCtx.createGain();
      lfo.frequency.value=0.04+Math.random()*0.09; lg.gain.value=freq*0.003;
      lfo.connect(lg); lg.connect(osc.frequency); lfo.start();
    });

    // Space wind noise
    const buf=audioCtx.createBuffer(1,audioCtx.sampleRate*4,audioCtx.sampleRate);
    const d=buf.getChannelData(0); for(let i=0;i<d.length;i++) d[i]=(Math.random()*2-1)*0.012;
    const src=audioCtx.createBufferSource(); src.buffer=buf; src.loop=true;
    const lp=audioCtx.createBiquadFilter(); lp.type='lowpass'; lp.frequency.value=350;
    src.connect(lp); lp.connect(masterGain); src.start();
  } else {
    masterGain.gain.setTargetAtTime(0.12,audioCtx.currentTime,0.5);
  }
  audioOn=true; document.getElementById('audio-btn').textContent='♪ 사운드 끄기';
}
function stopAudio(){
  if(masterGain) masterGain.gain.setTargetAtTime(0,audioCtx.currentTime,0.5);
  audioOn=false; document.getElementById('audio-btn').textContent='♪ 사운드 켜기';
}
document.getElementById('audio-btn').addEventListener('click',()=>{ audioOn?stopAudio():startAudio(); });

// ═══════════════════════════════════════════════════════
//  RAYCASTER / CLICK → INFO PANEL + CAMERA FLY
// ═══════════════════════════════════════════════════════
const ray=new THREE.Raycaster(), mouse2=new THREE.Vector2();
let camAnim=null;

function flyTo(targetPos,dist,onDone){
  if(tourMode) return;
  const dir=camera.position.clone().sub(targetPos).normalize();
  const end=targetPos.clone().add(dir.multiplyScalar(dist));
  camAnim={s:camera.position.clone(),e:end,tgt:targetPos.clone(),t:0,dur:100,cb:onDone};
}

function showInfo(mesh){
  const d=mesh.userData;
  document.getElementById('info-name').textContent=d.name;
  document.getElementById('info-eng').textContent=d.eng;
  document.getElementById('info-desc').textContent=d.desc;
  document.getElementById('info-stats').innerHTML=d.stats.map(([k,v])=>`<div class="stat"><span>${k}</span><span>${v}</span></div>`).join('');
  document.getElementById('info-panel').classList.add('visible');
}

renderer.domElement.addEventListener('click',e=>{
  if(shipMode) return;
  mouse2.x=(e.clientX/innerWidth)*2-1;
  mouse2.y=-(e.clientY/innerHeight)*2+1;
  ray.setFromCamera(mouse2,camera);
  const hits=ray.intersectObjects(planetMeshes);
  if(hits.length){
    const m=hits[0].object;
    showInfo(m);
    flyTo(m.position.clone(), m.userData.r*6.5);
  }
});
document.getElementById('info-close').addEventListener('click',()=>document.getElementById('info-panel').classList.remove('visible'));

// ═══════════════════════════════════════════════════════
//  SPACESHIP MODE (WASD)
// ═══════════════════════════════════════════════════════
let shipMode=false;
const keys={};
document.addEventListener('keydown',e=>{ keys[e.key.toLowerCase()]=true; if(e.key==='Escape') exitShip(); });
document.addEventListener('keyup',  e=>{ keys[e.key.toLowerCase()]=false; });

function enterShip(){ shipMode=true; controls.enabled=false; document.getElementById('btn-ship').classList.add('active'); document.getElementById('wasd-hint').classList.add('on'); }
function exitShip(){  shipMode=false; controls.enabled=true;  document.getElementById('btn-ship').classList.remove('active'); document.getElementById('wasd-hint').classList.remove('on'); }
document.getElementById('btn-ship').addEventListener('click',()=>shipMode?exitShip():enterShip());

let mDown=false, lmx=0, lmy=0;
renderer.domElement.addEventListener('mousedown',e=>{ mDown=true; lmx=e.clientX; lmy=e.clientY; });
renderer.domElement.addEventListener('mouseup',  ()=>mDown=false);
renderer.domElement.addEventListener('mousemove',e=>{
  if(!shipMode||!mDown) return;
  camera.rotation.order='YXZ';
  camera.rotation.y-=(e.clientX-lmx)*0.0022;
  camera.rotation.x=Math.max(-1.3,Math.min(1.3,camera.rotation.x-(e.clientY-lmy)*0.0022));
  lmx=e.clientX; lmy=e.clientY;
});

function tickShip(){
  if(!shipMode) return;
  const spd=1.4;
  const fwd=new THREE.Vector3(); camera.getWorldDirection(fwd);
  const right=new THREE.Vector3().crossVectors(fwd,camera.up).normalize();
  if(keys['w']) camera.position.addScaledVector(fwd,spd);
  if(keys['s']) camera.position.addScaledVector(fwd,-spd);
  if(keys['a']) camera.position.addScaledVector(right,-spd);
  if(keys['d']) camera.position.addScaledVector(right,spd);
  if(keys['q']) camera.position.y+=spd;
  if(keys['e']) camera.position.y-=spd;
}

// ═══════════════════════════════════════════════════════
//  AUTO TOUR
// ═══════════════════════════════════════════════════════
let tourMode=false, tourIdx=0, tourWait=0;
const TOUR_PAUSE=220;

function nextTour(){
  if(!tourMode) return;
  const m=planetMeshes[tourIdx%planetMeshes.length];
  const d=m.userData;
  const dir=new THREE.Vector3(0.8,0.5,0.9).normalize();
  const end=m.position.clone().add(dir.multiplyScalar(d.r*7));
  camAnim={s:camera.position.clone(),e:end,tgt:m.position.clone(),t:0,dur:130,cb:()=>showInfo(m)};
}

document.getElementById('btn-tour').addEventListener('click',()=>{
  tourMode=!tourMode;
  document.getElementById('btn-tour').classList.toggle('active',tourMode);
  document.getElementById('tour-tag').classList.toggle('on',tourMode);
  if(tourMode){ tourIdx=0; tourWait=0; nextTour(); }
  else { document.getElementById('info-panel').classList.remove('visible'); }
});

function tickTour(){
  if(!tourMode||camAnim) return;
  tourWait++;
  if(tourWait>=TOUR_PAUSE){ tourWait=0; tourIdx=(tourIdx+1)%planetMeshes.length; nextTour(); }
}

// ═══════════════════════════════════════════════════════
//  RESET
// ═══════════════════════════════════════════════════════
document.getElementById('btn-reset').addEventListener('click',()=>{
  tourMode=false;
  document.getElementById('btn-tour').classList.remove('active');
  document.getElementById('tour-tag').classList.remove('on');
  document.getElementById('info-panel').classList.remove('visible');
  camAnim={s:camera.position.clone(),e:new THREE.Vector3(0,80,220),tgt:new THREE.Vector3(0,0,0),t:0,dur:110};
});

// ═══════════════════════════════════════════════════════
//  RESIZE
// ═══════════════════════════════════════════════════════
window.addEventListener('resize',()=>{
  camera.aspect=innerWidth/innerHeight; camera.updateProjectionMatrix();
  renderer.setSize(innerWidth,innerHeight); labelRenderer.setSize(innerWidth,innerHeight); composer.setSize(innerWidth,innerHeight);
});

// ═══════════════════════════════════════════════════════
//  ANIMATE
// ═══════════════════════════════════════════════════════
const clock=new THREE.Clock();

function animate(){
  requestAnimationFrame(animate);
  const t=clock.getElapsedTime();

  // Sun
  sunMesh.scale.setScalar(1+Math.sin(t*1.4)*0.018);

  // Day/night cycle — sun direction rotates slowly
  const sAngle=t*0.018;
  sunDirU.set(Math.cos(sAngle),0.28,Math.sin(sAngle)).normalize();
  sunLight.position.set(Math.cos(sAngle)*6,1.8,Math.sin(sAngle)*6);

  // Planets
  PLANETS.forEach((data,i)=>{
    pAngles[i]+=data.spd*0.0009;
    const px=Math.cos(pAngles[i])*data.dist;
    const pz=Math.sin(pAngles[i])*data.dist;
    planetMeshes[i].position.set(px,0,pz);
    planetMeshes[i].rotation.y=t*(0.25+i*0.04);

    if(planetMeshes[i].userData.atm)   planetMeshes[i].userData.atm.position.copy(planetMeshes[i].position);
    if(planetMeshes[i].userData.rings) planetMeshes[i].userData.rings.position.copy(planetMeshes[i].position);

    if(planetMeshes[i].userData.moon){
      const mo=planetMeshes[i].userData.moon;
      mo.position.set(px+Math.cos(t*1.3)*data.r*3.2, Math.sin(t*0.5)*0.4, pz+Math.sin(t*1.3)*data.r*3.2);
    }

    // Terminator shader update
    if(planetMeshes[i].material.uniforms?.sunDir){
      // Convert world sun dir to local planet space
      const inv=new THREE.Matrix4().copy(planetMeshes[i].matrixWorld).invert();
      const local=sunDirU.clone().transformDirection(inv);
      planetMeshes[i].material.uniforms.sunDir.value.copy(local);
    }

    // Label opacity fades when zoomed out too far
    const dist=camera.position.distanceTo(planetMeshes[i].position);
    if(planetMeshes[i].userData.lbl)
      planetMeshes[i].userData.lbl.element.style.opacity = dist < data.dist*2.5 ? '1' : '0.3';
  });

  // Asteroids
  asteroids.forEach(a=>{ a.userData.a+=a.userData.spd; a.position.x=Math.cos(a.userData.a)*a.userData.r; a.position.z=Math.sin(a.userData.a)*a.userData.r; a.rotation.y+=0.008; });

  // Black hole
  accMat.uniforms.time.value=t;
  bhGroup.rotation.y=t*0.04;
  accDisk.lookAt(camera.position); // keep disk vaguely facing camera for the lensing illusion

  // Wormhole
  whMat.uniforms.time.value=t;
  whPlane.lookAt(camera.position);
  whRing.rotation.z=t*0.9;
  whGroup.rotation.y=t*0.025;

  // Shooting stars
  tickShooters();

  // Ship
  tickShip();

  // Camera animation
  if(camAnim){
    camAnim.t++;
    const p=Math.min(camAnim.t/camAnim.dur,1);
    const e=1-Math.pow(1-p,3);
    camera.position.lerpVectors(camAnim.s,camAnim.e,e);
    controls.target.lerpVectors(controls.target,camAnim.tgt,e);
    if(p>=1){ if(camAnim.cb) camAnim.cb(); camAnim=null; }
  }

  // Tour
  tickTour();

  controls.update();
  composer.render();
  labelRenderer.render(scene,camera);
}

animate();
</script>
</body>
</html>
