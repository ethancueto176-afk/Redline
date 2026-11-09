 REDLINE
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>REDLINE ARENA</title>
<style>
  body {
    margin: 0;
    overflow: hidden;
    background: #080808;
    color: white;
    font-family: Arial, sans-serif;
  }
  #enter {
    position: fixed;
    z-index: 5;
    top: 18px;
    left: 50%;
    transform: translateX(-50%);
    background: #e51b35;
    color: white;
    border: 0;
    border-radius: 12px;
    padding: 15px 25px;
    font-size: 18px;
    font-weight: bold;
  }
  #status {
    position: fixed;
    z-index: 5;
    bottom: 15px;
    left: 0;
    width: 100%;
    text-align: center;
    color: white;
    font-size: 14px;
    pointer-events: none;
  }
</style>
</head>
<body>

<button id="enter">ENTER VR</button>
<div id="status">REDLINE ARENA — WAITING FOR YOU</div>

<script type="module">
import * as THREE from
'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x080808);
scene.fog = new THREE.Fog(0x080808, 8, 35);

const camera = new THREE.PerspectiveCamera(
  70, window.innerWidth / window.innerHeight, 0.1, 100
);
camera.position.set(0, 1.6, 3);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.xr.enabled = true;
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff, 0x333333, 2));

const redLight = new THREE.PointLight(0xff1735, 30, 18);
redLight.position.set(0, 4, -3);
scene.add(redLight);

// ARENA FLOOR
const floor = new THREE.Mesh(
  new THREE.PlaneGeometry(40, 40),
  new THREE.MeshStandardMaterial({
    color: 0x171717,
    roughness: 0.9
  })
);
floor.rotation.x = -Math.PI / 2;
floor.position.y = 0;
scene.add(floor);

const grid = new THREE.GridHelper(40, 40, 0xff1735, 0x444444);
grid.position.y = 0.02;
scene.add(grid);

// ARENA WALLS
for (let i = 0; i < 4; i++) {
  const wall = new THREE.Mesh(
    new THREE.BoxGeometry(12, 4, 0.3),
    new THREE.MeshStandardMaterial({
      color: 0x292929,
      emissive: 0x330008
    })
  );
  wall.position.set(0, 2, -9);
  scene.add(wall);
  break;
}

// LIGHT PILLARS
for (let x of [-5, 5]) {
  const pillar = new THREE.Mesh(
    new THREE.BoxGeometry(0.25, 4, 0.25),
    new THREE.MeshStandardMaterial({
      color: 0xff1735,
      emissive: 0xaa0018
    })
  );
  pillar.position.set(x, 2, -5);
  scene.add(pillar);
}

// TEXTURE MAKER
function makeTextTexture(text, color = "#ffffff") {
  const canvas = document.createElement("canvas");
  canvas.width = 1024;
  canvas.height = 256;
  const ctx = canvas.getContext("2d");

  ctx.fillStyle = "#191919";
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  ctx.fillStyle = color;
  ctx.font = "bold 76px Arial";
  ctx.textAlign = "center";
  ctx.textBaseline = "middle";
  ctx.fillText(text, 512, 128);

  const texture = new THREE.CanvasTexture(canvas);
  texture.colorSpace = THREE.SRGBColorSpace;
  return texture;
}

// 3D LOBBY PANEL
const lobby = new THREE.Group();
lobby.position.set(0, 1.55, -2.8);
scene.add(lobby);

const panel = new THREE.Mesh(
  new THREE.BoxGeometry(2.8, 3.4, 0.12),
  new THREE.MeshStandardMaterial({
    color: 0x171717,
    emissive: 0x260008,
    metalness: 0.4,
    roughness: 0.5
  })
);
lobby.add(panel);

// TITLE
const title = new THREE.Mesh(
  new THREE.PlaneGeometry(2.5, 0.45),
  new THREE.MeshBasicMaterial({
    map: makeTextTexture("REDLINE ARENA", "#ff263f")
  })
);
title.position.set(0, 1.25, 0.08);
lobby.add(title);

// STATUS DISPLAY
const statusCanvas = document.createElement("canvas");
statusCanvas.width = 1024;
statusCanvas.height = 180;
const statusContext = statusCanvas.getContext("2d");

function updateStatus(message) {
  statusContext.fillStyle = "#191919";
  statusContext.fillRect(0, 0, 1024, 180);
  statusContext.fillStyle = "#ff4055";
  statusContext.font = "bold 48px Arial";
  statusContext.textAlign = "center";
  statusContext.textBaseline = "middle";
  statusContext.fillText(message, 512, 90);

  statusTexture.needsUpdate = true;
}

const statusTexture = new THREE.CanvasTexture(statusCanvas);
statusTexture.colorSpace = THREE.SRGBColorSpace;

const statusDisplay = new THREE.Mesh(
  new THREE.PlaneGeometry(2.5, 0.4),
  new THREE.MeshBasicMaterial({ map: statusTexture })
);
statusDisplay.position.set(0, -1.45, 0.09);
lobby.add(statusDisplay);
updateStatus("CHOOSE YOUR MODE");

// BUTTONS
const buttons = [];

function makeButton(label, y, action) {
  const texture = makeTextTexture(label);

  const button = new THREE.Mesh(
    new THREE.BoxGeometry(2.25, 0.34, 0.12),
    [
      new THREE.MeshStandardMaterial({ color: 0x9d1025 }),
      new THREE.MeshStandardMaterial({ color: 0x9d1025 }),
      new THREE.MeshStandardMaterial({ color: 0xff263f }),
      new THREE.MeshStandardMaterial({ color: 0x55000d }),
      new THREE.MeshBasicMaterial({ map: texture }),
      new THREE.MeshStandardMaterial({ color: 0x55000d })
    ]
  );

  button.position.set(0, y, 0.12);
  button.userData.action = action;
  button.userData.baseColor = 0xff263f;
  lobby.add(button);
  buttons.push(button);
}

makeButton("BOSS BATTLE", 0.72, () => {
  updateStatus("BOSS BATTLE SELECTED!");
  setPage("BOSS BATTLE");
});

makeButton("PLAYER VS PLAYER", 0.27, () => {
  updateStatus("PLAYER VS PLAYER SELECTED!");
  setPage("PLAYER VS PLAYER");
});

makeButton("CHAOS MODE", -0.18, () => {
  updateStatus("CHAOS MODE SELECTED!");
  setPage("CHAOS MODE");
});

makeButton("PUBLIC SERVERS", -0.72, () => {
  updateStatus("ONLINE SERVER SETUP COMING NEXT");
});

makeButton("PRIVATE ROOM", -1.17, () => {
  updateStatus("PRIVATE ROOM SETUP COMING NEXT");
});

function setPage(mode) {
  console.log("Selected mode:", mode);
}

// CONTROLLER POINTERS
const raycaster = new THREE.Raycaster();
const controllers = [];

for (let i = 0; i < 2; i++) {
  const controller = renderer.xr.getController(i);
  scene.add(controller);

  const ray = new THREE.Line(
    new THREE.BufferGeometry().setFromPoints([
      new THREE.Vector3(0, 0, 0),
      new THREE.Vector3(0, 0, -5)
    ]),
    new THREE.LineBasicMaterial({ color: 0xff263f })
  );
  controller.add(ray);

  controller.addEventListener("select", () => {
    raycaster.setFromXRController(controller);
    const hits = raycaster.intersectObjects(buttons);

    if (hits.length > 0) {
      hits[0].object.userData.action();
    }
  });

  controllers.push(controller);
}

// DESKTOP MOUSE TESTING
renderer.domElement.addEventListener("click", (event) => {
  const pointer = new THREE.Vector2(
    (event.clientX / window.innerWidth) * 2 - 1,
    -(event.clientY / window.innerHeight) * 2 + 1
  );

  raycaster.setFromCamera(pointer, camera);
  const hits = raycaster.intersectObjects(buttons);

  if (hits.length > 0) {
    hits[0].object.userData.action();
  }
});

// ENTER VR
document.getElementById("enter").addEventListener("click", async () => {
  try {
    if (!navigator.xr) {
      updateStatus("WEBXR NOT AVAILABLE");
      return;
    }

    const supported =
      await navigator.xr.isSessionSupported("immersive-vr");

    if (!supported) {
      updateStatus("VR NOT SUPPORTED HERE");
      return;
    }

    const session = await navigator.xr.requestSession(
      "immersive-vr",
      {
        optionalFeatures: ["local-floor", "hand-tracking"]
      }
    );

    renderer.xr.setReferenceSpaceType("local-floor");
    await renderer.xr.setSession(session);

    document.getElementById("enter").style.display = "none";
    document.getElementById("status").textContent =
      "VR CONNECTED — REDLINE ARENA";

    updateStatus("CHOOSE YOUR MODE");

    session.addEventListener("end", () => {
      document.getElementById("enter").style.display = "block";
    });

  } catch (error) {
    updateStatus("VR ERROR: " + error.message);
  }
});

// RESIZE
window.addEventListener("resize", () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

// GAME LOOP
renderer.setAnimationLoop(() => {
  renderer.render(scene, camera);
});
</script>
</body>
</html>