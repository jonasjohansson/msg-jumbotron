# MSG Jumbotron Forced Perspective Basketball — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a real-time Three.js forced perspective basketball animation that creates a 3D illusion on the MSG jumbotron when viewed from courtside.

**Architecture:** Single HTML file with inline Three.js. Anamorphic camera positioned below the scene looking up ~65° to match courtside viewing angle. Virtual box (3 walls + floor) creates depth cues. Basketball bounces left↔right with gravity simulation and dynamic shadow.

**Tech Stack:** Three.js (CDN), vanilla HTML/CSS/JS, single file

---

### Task 1: Scaffold HTML + Three.js Boilerplate

**Files:**
- Create: `index.html`

**Step 1: Create the base HTML file with Three.js setup**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MSG Jumbotron — Forced Perspective</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { background: #000; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        canvas { display: block; }
    </style>
</head>
<body>
    <script type="importmap">
    {
        "imports": {
            "three": "https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js"
        }
    }
    </script>
    <script type="module">
        import * as THREE from 'three';

        // --- Config ---
        const ASPECT = 16 / 9;
        const COLORS = {
            knickOrange: 0xF58426,
            knickBlue: 0x006BB6,
            knickSilver: 0xBEC0C2,
            black: 0x000000,
            white: 0xFFFFFF,
            hardwood: 0xC8956C,
        };

        // --- Renderer ---
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        renderer.setClearColor(COLORS.black);
        document.body.appendChild(renderer.domElement);

        function resize() {
            let w = window.innerWidth;
            let h = window.innerHeight;
            if (w / h > ASPECT) {
                w = h * ASPECT;
            } else {
                h = w / ASPECT;
            }
            renderer.setSize(w, h);
        }
        resize();
        window.addEventListener('resize', resize);

        // --- Scene & Camera ---
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(50, ASPECT, 0.1, 100);

        // --- Animation Loop ---
        function animate() {
            requestAnimationFrame(animate);
            renderer.render(scene, camera);
        }
        animate();
    </script>
</body>
</html>
```

**Step 2: Open in browser and verify**

Run: `open index.html` (or local server)
Expected: Black 16:9 canvas centered on screen

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: scaffold HTML with Three.js boilerplate and 16:9 canvas"
```

---

### Task 2: Build the Virtual Box (Room)

**Files:**
- Modify: `index.html`

**Step 1: Add the box geometry — floor + 3 walls**

Add after the camera setup, before the animation loop:

```javascript
// --- Room / Virtual Box ---
const BOX = { width: 8, height: 5, depth: 6 };

// Floor — hardwood court
const floorGeo = new THREE.PlaneGeometry(BOX.width, BOX.depth);
const floorMat = new THREE.MeshStandardMaterial({ color: COLORS.hardwood });
const floor = new THREE.Mesh(floorGeo, floorMat);
floor.rotation.x = -Math.PI / 2;
floor.position.y = 0;
floor.receiveShadow = true;
scene.add(floor);

// Court lines (center circle + mid line)
const linesMat = new THREE.MeshBasicMaterial({ color: COLORS.knickOrange });

// Center circle
const circleGeo = new THREE.RingGeometry(0.9, 1.0, 64);
const circle = new THREE.Mesh(circleGeo, linesMat);
circle.rotation.x = -Math.PI / 2;
circle.position.y = 0.005;
scene.add(circle);

// Center line
const lineGeo = new THREE.PlaneGeometry(0.05, BOX.depth);
const centerLine = new THREE.Mesh(lineGeo, linesMat);
centerLine.rotation.x = -Math.PI / 2;
centerLine.position.y = 0.005;
scene.add(centerLine);

// Back wall
const backWallGeo = new THREE.PlaneGeometry(BOX.width, BOX.height);
const wallMat = new THREE.MeshStandardMaterial({ color: 0x0a1a3a });
const backWall = new THREE.Mesh(backWallGeo, wallMat);
backWall.position.set(0, BOX.height / 2, -BOX.depth / 2);
scene.add(backWall);

// Left wall
const sideWallGeo = new THREE.PlaneGeometry(BOX.depth, BOX.height);
const leftWall = new THREE.Mesh(sideWallGeo, wallMat.clone());
leftWall.position.set(-BOX.width / 2, BOX.height / 2, 0);
leftWall.rotation.y = Math.PI / 2;
scene.add(leftWall);

// Right wall
const rightWall = new THREE.Mesh(sideWallGeo, wallMat.clone());
rightWall.position.set(BOX.width / 2, BOX.height / 2, 0);
rightWall.rotation.y = -Math.PI / 2;
scene.add(rightWall);
```

**Step 2: Position the anamorphic camera**

```javascript
// Camera: below and in front, looking up — courtside perspective
camera.position.set(0, -3.5, 7);
camera.lookAt(0, BOX.height * 0.4, 0);
```

**Step 3: Add lighting**

```javascript
// --- Lighting ---
const ambientLight = new THREE.AmbientLight(0xffffff, 0.4);
scene.add(ambientLight);

const spotLight = new THREE.SpotLight(0xffffff, 1.5, 30, Math.PI / 4, 0.5);
spotLight.position.set(0, BOX.height + 2, 0);
spotLight.castShadow = true;
spotLight.shadow.mapSize.width = 1024;
spotLight.shadow.mapSize.height = 1024;
scene.add(spotLight);

const rimLight = new THREE.DirectionalLight(COLORS.knickBlue, 0.3);
rimLight.position.set(0, 2, 5);
scene.add(rimLight);
```

**Step 4: Verify in browser**

Expected: Looking up into a room with hardwood floor, dark blue walls, court lines, lit from above

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add virtual box room with floor, walls, court lines, and anamorphic camera"
```

---

### Task 3: Add the Basketball

**Files:**
- Modify: `index.html`

**Step 1: Create the basketball mesh**

Add after room geometry:

```javascript
// --- Basketball ---
const ballRadius = 0.35;
const ballGeo = new THREE.SphereGeometry(ballRadius, 32, 32);
const ballMat = new THREE.MeshStandardMaterial({
    color: COLORS.knickOrange,
    roughness: 0.8,
    metalness: 0.1,
});
const ball = new THREE.Mesh(ballGeo, ballMat);
ball.castShadow = true;
ball.position.set(-BOX.width / 2 + 1, 2, 0);
scene.add(ball);

// Seam lines on ball (using thin torus geometries)
const seamMat = new THREE.MeshBasicMaterial({ color: 0x222222 });
const seamRadius = ballRadius + 0.005;
const seamTube = 0.01;

const seam1 = new THREE.Mesh(
    new THREE.TorusGeometry(seamRadius, seamTube, 8, 64),
    seamMat
);
ball.add(seam1);

const seam2 = new THREE.Mesh(
    new THREE.TorusGeometry(seamRadius, seamTube, 8, 64),
    seamMat
);
seam2.rotation.y = Math.PI / 2;
ball.add(seam2);

const seam3 = new THREE.Mesh(
    new THREE.TorusGeometry(seamRadius, seamTube, 8, 64),
    seamMat
);
seam3.rotation.x = Math.PI / 2;
ball.add(seam3);
```

**Step 2: Add the ball shadow on the floor**

```javascript
// --- Ball Shadow (projected on floor) ---
const shadowGeo = new THREE.CircleGeometry(0.4, 32);
const shadowMat = new THREE.MeshBasicMaterial({
    color: 0x000000,
    transparent: true,
    opacity: 0.4,
});
const ballShadow = new THREE.Mesh(shadowGeo, shadowMat);
ballShadow.rotation.x = -Math.PI / 2;
ballShadow.position.y = 0.01;
scene.add(ballShadow);
```

**Step 3: Verify in browser**

Expected: Orange basketball with seam lines floating in the room, dark shadow circle on the floor

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add basketball mesh with seam lines and floor shadow"
```

---

### Task 4: Animate the Bounce

**Files:**
- Modify: `index.html`

**Step 1: Add bounce physics and animation**

Add before the `animate()` function:

```javascript
// --- Bounce Animation ---
const BOUNCE = {
    xMin: -BOX.width / 2 + 1.2,
    xMax: BOX.width / 2 - 1.2,
    gravity: 9.8,
    bounceHeight: 3.5,
    speed: 2.5,
    dampening: 0.92,
    minBounceHeight: 0.8,
};

let ballState = {
    x: BOUNCE.xMin,
    vy: 0,
    y: BOUNCE.bounceHeight,
    direction: 1, // 1 = right, -1 = left
    currentBounceHeight: BOUNCE.bounceHeight,
    spinAngle: 0,
};

const clock = new THREE.Clock();

function updateBall(dt) {
    // Horizontal movement
    ballState.x += BOUNCE.speed * ballState.direction * dt;

    // Reverse direction at walls
    if (ballState.x >= BOUNCE.xMax) {
        ballState.x = BOUNCE.xMax;
        ballState.direction = -1;
        ballState.currentBounceHeight = BOUNCE.bounceHeight; // reset energy
    } else if (ballState.x <= BOUNCE.xMin) {
        ballState.x = BOUNCE.xMin;
        ballState.direction = 1;
        ballState.currentBounceHeight = BOUNCE.bounceHeight;
    }

    // Vertical bounce (gravity)
    ballState.vy -= BOUNCE.gravity * dt;
    ballState.y += ballState.vy * dt;

    // Floor collision
    if (ballState.y <= ballRadius) {
        ballState.y = ballRadius;
        ballState.currentBounceHeight *= BOUNCE.dampening;
        if (ballState.currentBounceHeight < BOUNCE.minBounceHeight) {
            ballState.currentBounceHeight = BOUNCE.minBounceHeight;
        }
        ballState.vy = Math.sqrt(2 * BOUNCE.gravity * ballState.currentBounceHeight);
    }

    // Ball spin
    ballState.spinAngle += ballState.direction * dt * 5;

    // Apply to mesh
    ball.position.x = ballState.x;
    ball.position.y = ballState.y;
    ball.rotation.z = ballState.spinAngle;

    // Shadow follows ball X, scales with height
    ballShadow.position.x = ballState.x;
    ballShadow.position.z = 0;
    const heightRatio = 1 - Math.min(ballState.y / (BOUNCE.bounceHeight + 1), 1);
    const shadowScale = 0.5 + heightRatio * 0.8;
    ballShadow.scale.set(shadowScale, shadowScale, 1);
    shadowMat.opacity = 0.15 + heightRatio * 0.35;
}
```

**Step 2: Update the animate loop**

Replace the existing `animate()` with:

```javascript
function animate() {
    requestAnimationFrame(animate);
    const dt = Math.min(clock.getDelta(), 0.05); // cap delta to avoid huge jumps
    updateBall(dt);
    renderer.render(scene, camera);
}
animate();
```

**Step 3: Verify in browser**

Expected: Basketball bounces left to right and back, losing energy per bounce, shadow tracks underneath and scales dynamically

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add basketball bounce animation with gravity, dampening, and dynamic shadow"
```

---

### Task 5: Add Knicks Branding & Polish

**Files:**
- Modify: `index.html`

**Step 1: Add Knicks text/logo on the court**

Add after court lines:

```javascript
// --- Knicks Branding ---
// "KNICKS" text on the court using canvas texture
const brandCanvas = document.createElement('canvas');
brandCanvas.width = 512;
brandCanvas.height = 256;
const ctx = brandCanvas.getContext('2d');
ctx.fillStyle = '#C8956C'; // match hardwood
ctx.fillRect(0, 0, 512, 256);
ctx.fillStyle = '#006BB6';
ctx.font = 'bold 80px Arial, sans-serif';
ctx.textAlign = 'center';
ctx.textBaseline = 'middle';
ctx.fillText('KNICKS', 256, 100);
ctx.font = 'bold 36px Arial, sans-serif';
ctx.fillStyle = '#F58426';
ctx.fillText('NEW YORK', 256, 170);

const brandTexture = new THREE.CanvasTexture(brandCanvas);
const brandGeo = new THREE.PlaneGeometry(3, 1.5);
const brandMat = new THREE.MeshStandardMaterial({
    map: brandTexture,
    transparent: false,
});
const brandMesh = new THREE.Mesh(brandGeo, brandMat);
brandMesh.rotation.x = -Math.PI / 2;
brandMesh.position.set(0, 0.006, 0);
scene.add(brandMesh);
```

**Step 2: Add edge glow to walls for depth**

```javascript
// Subtle edge lighting on walls
const edgeLight1 = new THREE.PointLight(COLORS.knickBlue, 0.5, 8);
edgeLight1.position.set(-BOX.width / 2, 0.5, 0);
scene.add(edgeLight1);

const edgeLight2 = new THREE.PointLight(COLORS.knickBlue, 0.5, 8);
edgeLight2.position.set(BOX.width / 2, 0.5, 0);
scene.add(edgeLight2);
```

**Step 3: Add "MSG" text on back wall**

```javascript
// MSG text on back wall
const msgCanvas = document.createElement('canvas');
msgCanvas.width = 512;
msgCanvas.height = 256;
const msgCtx = msgCanvas.getContext('2d');
msgCtx.fillStyle = '#0a1a3a';
msgCtx.fillRect(0, 0, 512, 256);
msgCtx.fillStyle = '#BEC0C2';
msgCtx.font = 'bold 48px Arial, sans-serif';
msgCtx.textAlign = 'center';
msgCtx.textBaseline = 'middle';
msgCtx.fillText('MADISON SQUARE GARDEN', 256, 128);

const msgTexture = new THREE.CanvasTexture(msgCanvas);
const msgGeo = new THREE.PlaneGeometry(6, 1.5);
const msgMat = new THREE.MeshStandardMaterial({ map: msgTexture });
const msgMesh = new THREE.Mesh(msgGeo, msgMat);
msgMesh.position.set(0, BOX.height - 1.5, -BOX.depth / 2 + 0.01);
scene.add(msgMesh);
```

**Step 4: Verify in browser**

Expected: "KNICKS / NEW YORK" on the court floor, "MADISON SQUARE GARDEN" on the back wall, blue edge glow on side walls

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add Knicks branding on court, MSG text on back wall, and edge lighting"
```

---

### Task 6: Final Tuning & Fullscreen

**Files:**
- Modify: `index.html`

**Step 1: Add fullscreen toggle on double-click**

```javascript
// --- Fullscreen ---
renderer.domElement.addEventListener('dblclick', () => {
    if (!document.fullscreenElement) {
        renderer.domElement.requestFullscreen();
    } else {
        document.exitFullscreen();
    }
});
```

**Step 2: Fine-tune camera angle for best illusion**

Test and adjust `camera.position` and `camera.lookAt` values. Start with:
- `camera.position.set(0, -3.5, 7)` and `camera.lookAt(0, 2, 0)`
- Adjust until the walls create strong converging lines and the ball appears to break out of the screen plane

**Step 3: Verify the full experience in browser**

Expected: Complete forced perspective illusion — courtside viewer looking up sees a basketball bouncing inside a Knicks-branded virtual room. Double-click for fullscreen.

**Step 4: Final commit**

```bash
git add index.html
git commit -m "feat: add fullscreen toggle and fine-tune camera for forced perspective"
```
