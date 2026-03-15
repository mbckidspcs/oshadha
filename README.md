<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Neon Racer 3D - Cyber Glow</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        body { margin: 0; overflow: hidden; background: #000; font-family: 'Segoe UI', sans-serif; touch-action: none; }
        #ui {
            position: absolute;
            top: 10px;
            left: 10px;
            color: #fff;
            pointer-events: none;
            text-shadow: 0 0 10px #00f2ff;
            z-index: 10;
        }
        #speedometer { font-size: 32px; font-weight: bold; color: #00f2ff; letter-spacing: 3px; font-family: monospace; }
        #location-display { 
            font-size: 14px; color: #fff; font-weight: bold; 
            padding: 6px 15px; background: rgba(0,0,0,0.9); 
            border-radius: 4px; display: inline-block; 
            border: 2px solid #00f2ff; margin-bottom: 8px; 
            box-shadow: 0 0 15px rgba(0, 242, 255, 0.4);
            letter-spacing: 2px;
            transition: all 0.5s ease;
        }
        #crash-counter { font-size: 16px; color: #ff0055; font-weight: bold; margin-top: 5px; text-transform: uppercase; }

        /* Mobile Controls */
        #mobile-controls {
            position: absolute;
            bottom: 30px;
            left: 0;
            width: 100%;
            height: 180px;
            display: flex;
            justify-content: space-between;
            align-items: flex-end;
            padding: 0 30px;
            box-sizing: border-box;
            z-index: 50;
            pointer-events: none;
        }

        .touch-btn {
            width: 80px;
            height: 80px;
            background: rgba(0, 0, 0, 0.6);
            border: 2px solid #00f2ff;
            box-shadow: 0 0 10px rgba(0, 242, 255, 0.3);
            border-radius: 12px;
            display: flex;
            justify-content: center;
            align-items: center;
            color: #00f2ff;
            font-weight: bold;
            font-size: 12px;
            pointer-events: auto;
            user-select: none;
            -webkit-user-select: none;
            touch-action: none;
            text-transform: uppercase;
        }
        .touch-btn:active { background: #00f2ff; color: #000; box-shadow: 0 0 30px #00f2ff; }

        #joystick-container {
            width: 140px;
            height: 140px;
            background: rgba(0, 0, 0, 0.5);
            border-radius: 50%;
            position: relative;
            pointer-events: auto;
            border: 2px solid #00f2ff;
            box-shadow: 0 0 15px rgba(0, 242, 255, 0.2);
        }
        #joystick-knob {
            width: 60px;
            height: 60px;
            background: #00f2ff;
            border-radius: 50%;
            position: absolute;
            top: 40px;
            left: 40px;
            box-shadow: 0 0 25px #00f2ff;
        }

        .right-controls { display: flex; flex-direction: column; gap: 15px; align-items: flex-end; }
        .pedal-group { display: flex; gap: 15px; }

        #start-screen {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: #000;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: #fff;
            z-index: 500;
        }

        .btn-glow {
            padding: 20px 60px; font-size: 24px; cursor: pointer;
            background: transparent; border: 3px solid #00f2ff; color: #00f2ff; font-weight: bold;
            border-radius: 4px; box-shadow: 0 0 20px rgba(0, 242, 255, 0.5);
            text-transform: uppercase; letter-spacing: 5px;
            transition: 0.3s;
        }
        .btn-glow:hover { background: #00f2ff; color: #000; box-shadow: 0 0 50px #00f2ff; }

        #game-over {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.95);
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: #fff;
            z-index: 100;
        }
    </style>
</head>
<body>
    <div id="start-screen">
        <h1 style="font-size: 64px; color: #00f2ff; margin-bottom: 20px; text-shadow: 0 0 40px #00f2ff; letter-spacing: 10px; font-family: monospace;">NEON_RACER</h1>
        <button class="btn-glow" onclick="startGame()">INITIATE</button>
    </div>

    <div id="ui">
        <div id="location-display">SYSTEM_BOOT</div>
        <div id="speedometer">000</div>
        <div id="crash-counter">INTEGRITY: 100%</div>
    </div>

    <div id="mobile-controls">
        <div id="joystick-container">
            <div id="joystick-knob"></div>
        </div>
        <div class="right-controls">
            <div class="pedal-group">
                <div class="touch-btn" id="btn-flight" style="width: 60px; height: 60px; border-color: #ff00ff; color: #ff00ff;">FLT</div>
                <div class="touch-btn" id="btn-drift" style="border-color: #ffff00; color: #ffff00;">DRIFT</div>
            </div>
            <div class="pedal-group">
                <div class="touch-btn" id="btn-brake" style="border-color: #ff0055; color: #ff0055;">STOP</div>
                <div class="touch-btn" id="btn-gas" style="border-color: #00ff88; color: #00ff88; width: 100px; height: 100px;">CORE</div>
            </div>
        </div>
    </div>

    <div id="game-over">
        <h1 style="font-size: 60px; color: #ff0055; text-shadow: 0 0 20px #ff0055; font-family: monospace;">LINK_LOST</h1>
        <div id="final-score" style="font-size: 24px; margin-bottom: 30px; color: #00f2ff;">0.00 LY</div>
        <button class="btn-glow" onclick="restartGame()">REBOOT</button>
    </div>

    <script>
        let scene, camera, renderer, car, clock;
        let speed = 0, health = 100, score = 0;
        let isGameOver = false, isGameStarted = false, isFlightMode = false;
        
        const keys = { w: false, s: false, a: false, d: false, space: false };
        const traffic = [], worldItems = [];
        const collisionBox = new THREE.Box3(), tempBox = new THREE.Box3();
        
        const ROAD_WIDTH = 26, WORLD_END = 12000;
        const BIOMES = [
            { name: "CYBER_COAST", start: 0, color: 0x00f2ff, fog: 0x020205 },
            { name: "DATA_VILLAGE", start: 1500, color: 0x00ff88, fog: 0x020502 },
            { name: "SULFUR_VOID", start: 3000, color: 0xffff00, fog: 0x050500 },
            { name: "CRYPTO_GLACIER", start: 5000, color: 0x0088ff, fog: 0x000205 },
            { name: "MAGMA_KERNEL", start: 8500, color: 0xff0055, fog: 0x050002 } // Fixed syntax error here
        ];

        function getRoadX(z) {
            if (z > 5000 && z < 8500) return Math.sin(z * 0.004) * 40;
            if (z >= 8500) return (Math.cos(z * 0.007) * 50);
            return 0;
        }

        function createBeautifulCar(colorValue, isPlayer = false) {
            const carGroup = new THREE.Group();
            const bodyMat = new THREE.MeshStandardMaterial({ 
                color: colorValue, 
                emissive: colorValue, 
                emissiveIntensity: isPlayer ? 1 : 0.5, 
                wireframe: true 
            });
            const coreMat = new THREE.MeshStandardMaterial({ color: 0x000000, metalness: 1, roughness: 0 });
            
            const shell = new THREE.Mesh(new THREE.BoxGeometry(1.8, 0.5, 4), bodyMat);
            shell.position.y = 0.5;
            carGroup.add(shell);

            const core = new THREE.Mesh(new THREE.BoxGeometry(1.6, 0.4, 3.8), coreMat);
            core.position.y = 0.5;
            carGroup.add(core);

            const glass = new THREE.Mesh(new THREE.BoxGeometry(1.2, 0.4, 1.2), bodyMat);
            glass.position.set(0, 0.8, 0.2);
            carGroup.add(glass);

            if (isPlayer) {
                const glow = new THREE.PointLight(colorValue, 4, 15);
                glow.position.set(0, 0.5, 0);
                carGroup.add(glow);
            }

            return carGroup;
        }

        function buildBiomes() {
            const SEGMENT_SIZE = 50;
            for (let z = 0; z < WORLD_END; z += SEGMENT_SIZE) {
                const biome = [...BIOMES].reverse().find(b => z >= b.start);
                const x1 = getRoadX(z), x2 = getRoadX(z + SEGMENT_SIZE);
                
                const roadGeo = new THREE.PlaneGeometry(ROAD_WIDTH, SEGMENT_SIZE + 2, 4, 4);
                const roadMat = new THREE.MeshStandardMaterial({ 
                    color: 0x000000, 
                    emissive: biome.color, 
                    emissiveIntensity: 0.3,
                    wireframe: true 
                });
                const road = new THREE.Mesh(roadGeo, roadMat);
                road.rotation.x = -Math.PI / 2;
                road.rotation.z = Math.atan2(x2 - x1, SEGMENT_SIZE);
                road.position.set((x1 + x2) / 2, 0.05, z + SEGMENT_SIZE / 2);
                scene.add(road);

                // Railings
                const railMat = new THREE.MeshBasicMaterial({ color: biome.color });
                const lRail = new THREE.Mesh(new THREE.BoxGeometry(0.2, 0.8, SEGMENT_SIZE), railMat);
                lRail.position.set(x1 - ROAD_WIDTH/2, 0.4, z + SEGMENT_SIZE/2);
                scene.add(lRail);
                const rRail = lRail.clone();
                rRail.position.x = x1 + ROAD_WIDTH/2;
                scene.add(rRail);

                if (z % 300 === 0) {
                    const h = 40 + Math.random() * 80;
                    const mono = new THREE.Mesh(
                        new THREE.BoxGeometry(8, h, 8),
                        new THREE.MeshBasicMaterial({ color: biome.color, wireframe: true })
                    );
                    const side = Math.random() > 0.5 ? 1 : -1;
                    mono.position.set(x1 + (50 + Math.random() * 40) * side, h/2, z);
                    scene.add(mono);
                }
            }
        }

        function init() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x020205);
            scene.fog = new THREE.Fog(0x020205, 50, 600);
            
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 2000);
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            document.body.appendChild(renderer.domElement);

            scene.add(new THREE.AmbientLight(0x404040, 1.5));

            buildBiomes();
            car = createBeautifulCar(0x00f2ff, true);
            scene.add(car);

            for (let i = 0; i < 60; i++) {
                const tCar = createBeautifulCar(0xff0055);
                const z = Math.random() * WORLD_END;
                tCar.position.set(getRoadX(z) + (Math.random() > 0.5 ? 6 : -6), 0, z);
                tCar.userData.speed = 0.3 + Math.random() * 0.4;
                scene.add(tCar); traffic.push(tCar);
            }

            initControls();
            animate();
        }

        function initControls() {
            const joystick = document.getElementById('joystick-container');
            const knob = document.getElementById('joystick-knob');
            let isTouching = false;

            const handleJoystick = (e) => {
                const touch = (e.touches && e.touches[0]) || e;
                const rect = joystick.getBoundingClientRect();
                const centerX = rect.left + rect.width / 2;
                const centerY = rect.top + rect.height / 2;
                let dx = touch.clientX - centerX;
                let dy = touch.clientY - centerY;
                const dist = Math.sqrt(dx*dx + dy*dy);
                const maxDist = 45;
                if (dist > maxDist) { dx = (dx / dist) * maxDist; dy = (dy / dist) * maxDist; }
                knob.style.transform = `translate(${dx}px, ${dy}px)`;
                keys.a = dx < -15; keys.d = dx > 15;
            };

            joystick.addEventListener('touchstart', (e) => { e.preventDefault(); isTouching = true; handleJoystick(e); });
            window.addEventListener('touchmove', (e) => { if (isTouching) handleJoystick(e); }, { passive: false });
            window.addEventListener('touchend', () => { 
                isTouching = false; knob.style.transform = 'translate(0, 0)';
                keys.a = false; keys.d = false;
            });

            const setupBtn = (id, key) => {
                const el = document.getElementById(id);
                el.addEventListener('touchstart', (e) => { e.preventDefault(); keys[key] = true; });
                el.addEventListener('touchend', (e) => { e.preventDefault(); keys[key] = false; });
            };
            setupBtn('btn-gas', 'w'); setupBtn('btn-brake', 's'); setupBtn('btn-drift', 'space');
            document.getElementById('btn-flight').addEventListener('touchstart', (e) => {
                e.preventDefault(); isFlightMode = !isFlightMode;
                document.getElementById('btn-flight').style.background = isFlightMode ? '#ff00ff' : '';
                document.getElementById('btn-flight').style.color = isFlightMode ? '#000' : '';
            });
            
            // Keyboard fallbacks
            window.addEventListener('keydown', e => { 
                if (e.key.toLowerCase() === 'w') keys.w = true;
                if (e.key.toLowerCase() === 's') keys.s = true;
                if (e.key.toLowerCase() === 'a') keys.a = true;
                if (e.key.toLowerCase() === 'd') keys.d = true;
                if (e.key === ' ') keys.space = true;
            });
            window.addEventListener('keyup', e => {
                if (e.key.toLowerCase() === 'w') keys.w = false;
                if (e.key.toLowerCase() === 's') keys.s = false;
                if (e.key.toLowerCase() === 'a') keys.a = false;
                if (e.key.toLowerCase() === 'd') keys.d = false;
                if (e.key === ' ') keys.space = false;
            });
        }

        function startGame() {
            document.getElementById('start-screen').style.display = 'none';
            isGameStarted = true;
        }

        function restartGame() {
            isGameOver = false; health = 100; speed = 0; score = 0;
            car.position.set(0, 0, 0); car.rotation.set(0, 0, 0);
            document.getElementById('game-over').style.display = 'none';
        }

        function animate() {
            requestAnimationFrame(animate);
            if (!isGameStarted || isGameOver) return;

            if (keys.w) speed += 0.03;
            if (keys.s) speed -= 0.04;
            speed *= 0.985;
            speed = Math.max(-0.4, Math.min(2.0, speed));

            if (isFlightMode) {
                car.position.y = THREE.MathUtils.lerp(car.position.y, 15, 0.05);
            } else {
                car.position.y = THREE.MathUtils.lerp(car.position.y, 0, 0.1);
            }

            if (Math.abs(speed) > 0.01) {
                const turnFactor = keys.space ? 1.8 : 1;
                if (keys.a) car.rotation.y += 0.04 * turnFactor;
                if (keys.d) car.rotation.y -= 0.04 * turnFactor;
            }

            car.translateZ(speed);
            if (car.position.z > WORLD_END) car.position.z = 0;
            if (car.position.z < 0) car.position.z = WORLD_END;

            // Update Biome UI and World FX
            const currentBiome = [...BIOMES].reverse().find(b => car.position.z >= b.start);
            if (currentBiome) {
                document.getElementById('location-display').innerText = currentBiome.name;
                document.getElementById('location-display').style.borderColor = '#' + currentBiome.color.toString(16).padStart(6, '0');
                scene.fog.color.set(currentBiome.fog);
            }

            // Traffic & Collision
            collisionBox.setFromObject(car);
            traffic.forEach(t => {
                t.position.z += t.userData.speed;
                if (t.position.z > WORLD_END) t.position.z = 0;
                
                if (Math.abs(car.position.z - t.position.z) < 10 && car.position.y < 3) {
                    tempBox.setFromObject(t);
                    if (collisionBox.intersectsBox(tempBox)) {
                        health -= 1.5;
                        speed *= 0.4;
                        if (health <= 0) {
                            isGameOver = true;
                            document.getElementById('game-over').style.display = 'flex';
                            document.getElementById('final-score').innerText = `${(score/1000).toFixed(2)} LY`;
                        }
                    }
                }
            });

            score += Math.abs(speed);
            document.getElementById('speedometer').innerText = Math.floor(Math.abs(speed) * 300).toString().padStart(3, '0');
            document.getElementById('crash-counter').innerText = `INTEGRITY: ${Math.max(0, Math.ceil(health))}%`;

            const camDist = 12 + (speed * 4);
            const camOffset = new THREE.Vector3(0, 4, -camDist).applyMatrix4(car.matrixWorld);
            camera.position.lerp(camOffset, 0.1);
            camera.lookAt(car.position.x, car.position.y + 1, car.position.z + 5);

            renderer.render(scene, camera);
        }

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        init();
    </script>
</body>
</html>
