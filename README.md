# sabin_portfolio
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Sabin — 3D Portfolio</title>
  <meta name="description" content="Interactive 3D portfolio for Sabin" />
  <link rel="icon" href="data:;base64,iVBORw0KGgo=" />
  <style>
    /* Minimal reset + layout */
    :root{--bg:#0b0f14;--card:#0f1720;--accent:#00d1ff;--glass:rgba(255,255,255,0.04)}
    *{box-sizing:border-box;margin:0;padding:0}
    html,body,#app{height:100%}
    body{font-family:Inter,system-ui,Arial;color:#e6eef6;background:linear-gradient(180deg,#071019 0%, #071827 100%);}
    header{position:fixed;left:0;right:0;top:0;height:64px;display:flex;align-items:center;justify-content:space-between;padding:0 24px;backdrop-filter:blur(6px);background:linear-gradient(180deg,rgba(255,255,255,0.02),transparent);z-index:20}
    .brand{display:flex;gap:12px;align-items:center}
    .brand h1{font-size:18px;letter-spacing:0.6px}
    nav a{color:#cfeffd;margin-left:14px;text-decoration:none}
    main{padding-top:92px;padding-left:24px;padding-right:24px}
    .hero{display:flex;gap:32px;align-items:center}
    .intro{max-width:540px}
    .intro h2{font-size:34px;margin-bottom:8px}
    .intro p{opacity:0.85;line-height:1.45}
    /* canvas area */
    #scene-wrap{width:100%;height:520px;border-radius:16px;background:linear-gradient(180deg,rgba(255,255,255,0.02),transparent);box-shadow:0 6px 30px rgba(2,6,23,0.6);overflow:hidden;margin-top:22px}
    footer{color:#9fb7c6;opacity:0.9;padding:24px 24px}

    /* Project overlay (page-wise) */
    .project-panel{position:fixed;inset:64px 36px 36px;border-radius:12px;background:linear-gradient(180deg, rgba(10,14,18,0.96), rgba(9,13,16,0.98));box-shadow:0 20px 60px rgba(1,4,7,0.75);padding:20px;z-index:30;display:none}
    .project-panel.show{display:flex}
    .project-grid{display:grid;grid-template-columns:1fr 420px;gap:20px;height:calc(100% - 40px)}
    .project-content{overflow:auto;padding:14px;border-radius:10px;background:linear-gradient(180deg, rgba(255,255,255,0.012), transparent)}
    pre{background:var(--card);padding:12px;border-radius:8px;overflow:auto;font-family:monospace;font-size:13px}
    .close-btn{cursor:pointer;border:0;background:var(--accent);color:#012;padding:8px 12px;border-radius:8px}

    /* small responsive */
    @media (max-width:920px){.project-grid{grid-template-columns:1fr}#scene-wrap{height:420px}}
  </style>
</head>
<body>
  <div id="app">
    <header>
      <div class="brand">
        <svg width="38" height="38" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><circle cx="12" cy="12" r="10" stroke="#00d1ff" stroke-width="1.2"/></svg>
        <h1>Sabin — Interactive Portfolio</h1>
      </div>
      <nav>
        <a href="#about">About</a>
        <a href="#projects">Projects</a>
        <a href="#contact">Contact</a>
      </nav>
    </header>

    <main>
      <section class="hero">
        <div class="intro">
          <h2>Hello — I'm Sabin</h2>
          <p>Electronics & Communication engineering student building smart industrial systems, embedded projects, and interactive web experiences. This portfolio shows projects as interactive 3D modules — click a project card to open a page-like panel with details, code and downloads.</p>
        </div>
      </section>

      <section id="scene-wrap">
        <!-- Three.js canvas will mount here -->
        <canvas id="three-canvas"></canvas>
      </section>

      <section id="projects" style="margin-top:18px">
        <h3 style="color:#dff9ff;margin-bottom:8px">Projects</h3>
        <p style="opacity:0.85">Below are your projects represented as 3D cards. Clicking opens the full page for that project (code, description, downloads).</p>
      </section>
    </main>

    <div class="project-panel" id="project-panel" role="dialog" aria-hidden="true">
      <div class="project-grid">
        <div class="project-content" id="project-content">
          <!-- Project details injected here -->
        </div>
        <aside style="padding:10px">
          <button class="close-btn" id="close-panel">Close</button>
          <div style="height:12px"></div>
          <div style="font-size:13px;opacity:0.9">Quick actions</div>
          <div style="margin-top:10px;display:flex;flex-direction:column;gap:8px">
            <a id="download-zip" href="#" download="project.zip" style="text-decoration:none"><button class="close-btn">Download ZIP</button></a>
            <a id="open-github" target="_blank" rel="noopener noreferrer" href="#" style="text-decoration:none"><button class="close-btn">Open on GitHub</button></a>
          </div>
        </aside>
      </div>
    </div>

    <footer>
      <small>Built with HTML, Three.js. Hosted on GitHub Pages.</small>
    </footer>
  </div>

  <!-- Three.js CDN -->
  <script type="module">
    import * as THREE from 'https://unpkg.com/three@0.160.0/build/three.module.js';
    // Minimal interactive 3D card scene
    const canvas = document.getElementById('three-canvas');
    const renderer = new THREE.WebGLRenderer({canvas, antialias:true, alpha:true});
    renderer.setPixelRatio(window.devicePixelRatio);
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(35,1,0.1,1000);
    camera.position.set(0,0,6);

    // lights
    const amb = new THREE.AmbientLight(0xffffff,0.6);
    scene.add(amb);
    const dir = new THREE.DirectionalLight(0xffffff,0.6);
    dir.position.set(5,10,7);
    scene.add(dir);

    // Create several "cards" (thin boxes) representing projects
    const cards = [];
    const loader = new THREE.TextureLoader();

    const projects = [
      {id:'smart-industrial', title:'Smart Industrial Monitoring', summary:'Arduino + sensors, I2C LCD & Blynk integration', thumbnail:''},
      {id:'hybrid-solar', title:'Hybrid Solar Tracker', summary:'LDR-controlled servos + energy harvesting demo', thumbnail:''},
      {id:'portfolio', title:'This Portfolio', summary:'Interactive 3D web experience', thumbnail:''}
    ];

    const cardGeo = new THREE.BoxGeometry(1.6,1,0.08);
    const matBase = new THREE.MeshStandardMaterial({color:0x072935, metalness:0.2, roughness:0.4});

    projects.forEach((p,i)=>{
      const m = matBase.clone();
      const mesh = new THREE.Mesh(cardGeo,m);
      mesh.position.set((i-1)*1.9,0,0);
      mesh.userData = p;
      scene.add(mesh);
      cards.push(mesh);

      // add simple label as canvas texture
      const labelCanvas = document.createElement('canvas'); labelCanvas.width=512; labelCanvas.height=320;
      const ctx = labelCanvas.getContext('2d');
      ctx.fillStyle = '#061017'; ctx.fillRect(0,0,labelCanvas.width,labelCanvas.height);
      ctx.fillStyle = '#bff6ff'; ctx.font = '42px sans-serif'; ctx.fillText(p.title,28,80);
      ctx.font='20px sans-serif'; ctx.fillStyle='#a7e8f6'; ctx.fillText(p.summary,28,120);
      const tex = new THREE.CanvasTexture(labelCanvas);
      const faceMat = new THREE.MeshBasicMaterial({map:tex});
      const facePlane = new THREE.Mesh(new THREE.PlaneGeometry(1.44,0.86), faceMat);
      facePlane.position.set(0,0,0.05);
      mesh.add(facePlane);
    });

    // raycaster for clicks
    const ray = new THREE.Raycaster();
    const mouse = new THREE.Vector2();
    function onPointer(e){
      const rect = renderer.domElement.getBoundingClientRect();
      mouse.x = ((e.clientX - rect.left)/rect.width)*2 -1;
      mouse.y = -((e.clientY - rect.top)/rect.height)*2 +1;
      ray.setFromCamera(mouse,camera);
      const hits = ray.intersectObjects(cards,true);
      if(hits.length){
        const p = hits[0].object.parent.userData || hits[0].object.userData;
        openProjectPanel(p.id);
      }
    }
    renderer.domElement.addEventListener('pointerdown', onPointer);

    function resize(){
      const w = canvas.clientWidth || canvas.parentElement.clientWidth;
      const h = canvas.clientHeight || 520;
      renderer.setSize(w,h,true);
      camera.aspect = w/h; camera.updateProjectionMatrix();
    }
    window.addEventListener('resize', resize);

    // animate cards
    const clock = new THREE.Clock();
    function animate(){
      const t = clock.getElapsedTime();
      cards.forEach((c,i)=>{
        c.rotation.y = Math.sin(t*0.7 + i*0.5)*0.12;
        c.rotation.x = Math.sin(t*0.2 + i*0.8)*0.06;
        c.position.y = Math.sin(t*0.4 + i*0.3)*0.12;
      });
      renderer.render(scene,camera);
      requestAnimationFrame(animate);
    }

    // initial resize + animate
    resize(); animate();

    // ----- Project panel logic -----
    const panel = document.getElementById('project-panel');
    const content = document.getElementById('project-content');
    const closeBtn = document.getElementById('close-panel');
    const downloadBtn = document.getElementById('download-zip');
    const openGithub = document.getElementById('open-github');

    closeBtn.addEventListener('click', ()=>{panel.classList.remove('show'); panel.setAttribute('aria-hidden','true');});

    const projectDetails = {
      'smart-industrial': {
        title:'Smart Industrial Monitoring System',
        description:`This project integrates multiple sensors (DHT11, BMP280, current sensor, vibration, IR for RPM) with an Arduino UNO. It shows readings on an I2C LCD and can optionally push data to mobile via Blynk (NodeMCU version). Below is the consolidated Arduino sketch for the UNO setup. Use the ZIP action to download project files (circuit diagram, Fritzing / PCB images).`,
        code: `// SMART INDUSTRIAL - Arduino UNO (I2C LCD + sensors)\n#include <Wire.h>\n#include <LiquidCrystal_I2C.h>\n#include <DHT.h>\n#include <Adafruit_BMP280.h>\n\n#define DHTPIN 2\n#define DHTTYPE DHT11\n#define IR_PIN 4 // RPM sensor (digital)\n#define BUZZER 6\n#define RELAY 5\n#define CURRENT_SENSOR A1\n#define VIBRATION_SENSOR A0\n\nLiquidCrystal_I2C lcd(0x27,16,2);\nDHT dht(DHTPIN, DHTTYPE);\nAdafruit_BMP280 bmp;\n\nvolatile unsigned long irCount = 0;\nunsigned long lastRPMMillis = 0;\nfloat rpm = 0;\n\nvoid IRAM_ATTR countIR(){ irCount++; }\n\nvoid setup(){\n  Serial.begin(9600);\n  lcd.init(); lcd.backlight();\n  dht.begin();\n  if(!bmp.begin(0x76)){ Serial.println("BMP280 not found"); }\n  pinMode(IR_PIN, INPUT_PULLUP);\n  attachInterrupt(digitalPinToInterrupt(IR_PIN), countIR, FALLING);\n  pinMode(BUZZER, OUTPUT); pinMode(RELAY, OUTPUT);\n}\n\nvoid loop(){\n  // read sensors\n  float h = dht.readHumidity();\n  float t = dht.readTemperature();\n  float pressure = bmp.readPressure()/100.0F;\n  int currentRaw = analogRead(CURRENT_SENSOR);\n  int vib = analogRead(VIBRATION_SENSOR);\n\n  // compute RPM every 2 seconds\n  if(millis() - lastRPMMillis >= 2000){\n    noInterrupts();\n    unsigned long c = irCount; irCount = 0;\n    interrupts();\n    rpm = (c / 2.0) * 60.0; // if one pulse per revolution; adjust as needed\n    lastRPMMillis = millis();\n  }\n\n  // display on lcd\n  lcd.clear();\n  lcd.setCursor(0,0);\n  lcd.print("T:"); lcd.print(t,1); lcd.print("C H:"); lcd.print(h,0);\n  lcd.setCursor(0,1);\n  lcd.print("P:"); lcd.print(pressure,0); lcd.print(" R:"); lcd.print((int)rpm);\n\n  // simple alarm rule\n  if(currentRaw > 600 || vib > 600){ digitalWrite(BUZZER, HIGH); } else { digitalWrite(BUZZER, LOW); }\n\n  delay(1000);\n}\n`}
      },
      'hybrid-solar':{
        title:'Hybrid Solar Tracker',
        description:'LDR-based 2-axis solar tracker with servos and optional piezo sensor integration. (Prototype details, wiring and Arduino sketch included in ZIP.)',
        code:'// Short description -- full code included in downloadable ZIP.'
      },
      'portfolio':{
        title:'Interactive 3D Portfolio',
        description:'This website source (single-file) that you are viewing. Hosted on GitHub Pages. Replace thumbnails and GLTF models in the assets folder to show unique 3D models per project.',
        code:'// This page is built using Three.js. Use your own assets/GLTF models in production.'
      }
    };

    function openProjectPanel(id){
      const p = projectDetails[id];
      content.innerHTML = `<h2 style=\"color:#cfeffd\">${p.title}</h2><p style=\"opacity:0.9\">${p.description}</p><h3 style=\"margin-top:12px;color:#aeefff\">Code (preview)</h3><pre>${p.code}</pre>`;
      // wire quick actions
      downloadBtn.href = '#'; // replace with generated zip link if you create one
      openGithub.href = 'https://github.com/sabinsvg/sabin-portfolio';
      panel.classList.add('show'); panel.setAttribute('aria-hidden','false');
    }

  </script>
</body>
</html>
