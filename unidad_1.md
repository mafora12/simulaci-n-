# Unidad 1 · Actividad 07 — Reto de diseño: Navegar la incertidumbre

> **Encargo:** diseñar en p5.js una experiencia generativa en tiempo real donde el visitante pueda influir sobre un sistema sin controlarlo completamente, interpretando los cinco momentos: *posibilidad, tendencia, normalidad, excepción* e *influencia*.

---

## Intención conceptual #1 — Laboratorio molecular

El objetivo de esta primera aproximación fue desarrollar una simulación inspirada en el comportamiento de un laboratorio molecular, donde un conjunto de partículas cambia su comportamiento según las condiciones del sistema. La intención fue representar visualmente cómo la combinación de reglas simples puede generar comportamientos complejos.

**Conceptos utilizados:**

| Concepto | Uso en la simulación |
|---|---|
| Ruido Perlin | Movimientos suaves y continuos |
| Caminata aleatoria | Variaciones impredecibles en la trayectoria |
| Distribución normal | La mayoría de las moléculas tiende a formar pequeños grupos por atracción temporal |
| Lévy Flight | Desplazamientos largos y poco frecuentes que modifican la distribución del sistema |

**Interacción:** el usuario modifica el comportamiento del sistema sin detener la simulación. El movimiento vertical del mouse controla la *temperatura*, y cada clic cambia el nivel de aleatoriedad del movimiento de las moléculas.

---

## Experimentos y versiones intermedias

### Versión 1 — Mapeo de los cinco momentos

**Posibilidad** — representada por la caminata aleatoria: cada molécula puede moverse en cualquier dirección.

```javascript
let randomWalk = p5.Vector.random2D();
randomWalk.mult(randomWalkStrength * temperature);
this.vel.add(randomWalk);
```
> Cada actualización genera una dirección completamente aleatoria, por lo que cualquier trayectoria es posible.

**Tendencia** — corresponde al ruido Perlin.

```javascript
let angle = map(noise(this.tx, this.ty), 0, 1, 0, TWO_PI);
let perlin = p5.Vector.fromAngle(angle);
perlin.mult((0.35 - randomWalkStrength * 0.4) * temperature);
this.tx += 0.003;
this.ty += 0.003;
this.vel.add(perlin);
```
> Aunque el movimiento cambia constantemente, el ruido Perlin genera una pequeña tendencia a continuar en una dirección similar, produciendo trayectorias suaves.

**Normalidad** — la mayoría de las moléculas intenta permanecer cerca de otra.

```javascript
if (random() < 0.85) {
  this.groupTarget = molecules[int(random(molecules.length))];
} else {
  this.groupTarget = null;
}

if (this.groupTarget != null) {
  let attraction = p5.Vector.sub(this.groupTarget.pos, this.pos);
  attraction.normalize();
  attraction.mult(map(attraction.mag(), 0, 200, 0, 0.12));
  this.vel.add(attraction);
}
```
> Existe un 85 % de probabilidad de que una molécula permanezca cerca de otra, por lo que el comportamiento más frecuente es mantenerse agrupada.

**Excepción** — corresponde al Lévy Flight.

```javascript
let levyProbability = levyProbabilityBase;
levyProbability *= temperature;

if (random() < levyProbability) {
  let jump = p5.Vector.random2D();
  jump.mult(random(80, 180));
  this.pos.add(jump);
}
```

**Influencia** — dos mecanismos distintos:

```javascript
temperature = map(mouseY, height, 0, 0.5, 2.2);
```

```javascript
function mousePressed() {
  randomLevel = (randomLevel + 1) % RANDOMNESS.length;
  randomWalkStrength = RANDOMNESS[randomLevel];
}
```

### Control de versiones

<details>
<summary><strong>Código — primer intento (con error)</strong></summary>

```javascript
let molecules = [];
const NUM_MOLECULES = 250;

let temperature = 1;
let globalTime = 0;
const LINK_DISTANCE = 55;

this.groupTarget = null;
this.changeTimer = int(random(60, 180));
this.energy = 0;

function setup() {
  createCanvas(windowWidth, windowHeight);
  colorMode(HSB, 360, 100, 100, 100);
  noStroke();
  for (let i = 0; i < NUM_MOLECULES; i++) {
    molecules.push(new Molecule());
  }
}

function draw() {
  background(220, 30, 6, 18);
  globalTime += 0.003;
  temperature = map(mouseY, height, 0, 0.5, 2.2);
  drawTemperatureIndicator();

  for (let molecule of molecules) molecule.update();
  drawConnections();
  for (let molecule of molecules) molecule.display();
}

function drawConnections() {
  for (let i = 0; i < molecules.length; i++) {
    for (let j = i + 1; j < molecules.length; j++) {
      let d = dist(molecules[i].pos.x, molecules[i].pos.y, molecules[j].pos.x, molecules[j].pos.y);
      if (d < LINK_DISTANCE) {
        stroke(180, 20, 100, map(d, 0, LINK_DISTANCE, 80, 0));
        strokeWeight(map(d, 0, LINK_DISTANCE, 2, 0.3));
        line(molecules[i].pos.x, molecules[i].pos.y, molecules[j].pos.x, molecules[j].pos.y);
      }
    }
  }
}

function drawTemperatureIndicator() {
  noStroke();
  fill(15, 80, 100);
  rect(25, 25, 25, 160);
  let h = map(temperature, 0.5, 2.2, 0, 160);
  fill(0, 80, 100);
  rect(25, 185 - h, 25, h);
  fill(0, 0, 100);
  textSize(14);
  textAlign(LEFT);
  text("Temperatura", 20, 210);
  text(nf(temperature, 1, 2), 20, 230);
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}

class Molecule {
  constructor() {
    this.pos = createVector(random(width), random(height));
    this.vel = createVector();
    this.tx = random(1000);
    this.ty = random(5000);
    this.size = random(5, 9);
    this.hue = random(170, 220);
    this.angle = random(TWO_PI);
  }

  update() {
    let angle = map(noise(this.tx, this.ty), 0, 1, 0, TWO_PI);
    let perlin = p5.Vector.fromAngle(angle);
    perlin.mult(0.25 * temperature);
    this.tx += 0.003;
    this.ty += 0.003;

    let randomWalk = p5.Vector.random2D();
    randomWalk.mult(0.08 * temperature);

    this.changeTimer--;
    if (this.changeTimer <= 0) {
      if (random() < 0.85) {
        this.groupTarget = molecules[int(random(molecules.length))];
      } else {
        this.groupTarget = null;
      }
      this.changeTimer = int(random(80, 200));
    }

    if (this.groupTarget != null) {
      let attraction = p5.Vector.sub(this.groupTarget.pos, this.pos);
      let d = attraction.mag();
      attraction.normalize();
      attraction.mult(map(d, 0, 200, 0, 0.12));
      this.vel.add(attraction);
    }

    let levyProbability = 0.0006;
    levyProbability *= temperature;
    if (random() < levyProbability) {
      let jump = p5.Vector.random2D();
      jump.mult(random(80, 180));
      this.pos.add(jump);
    }

    this.vel.add(perlin);
    this.vel.add(randomWalk);
    this.vel.limit(2.5 * temperature);
    this.pos.add(this.vel);
    this.vel.mult(0.96);

    if (this.pos.x < 0) this.pos.x = width;
    if (this.pos.x > width) this.pos.x = 0;
    if (this.pos.y < 0) this.pos.y = height;
    if (this.pos.y > height) this.pos.y = 0;
  }
}
```
</details>

Este código no se dejaba ejecutar por el siguiente error:

![Error en consola](https://github.com/user-attachments/assets/72b01c62-6d8c-48ce-84c6-77ef1404ec0e)

**Diagnóstico:** había líneas sueltas antes de `setup()` que usaban `this` fuera de cualquier función o clase — algo inválido en JavaScript:

```javascript
this.groupTarget = null;
this.changeTimer = int(random(60, 180));
this.energy = 0;
```

Esas asignaciones pertenecían al constructor de la clase `Molecule` y no estaban ubicadas ahí; además faltaba una llave de cierre `}` al final del archivo. Se reorganizó y quedó así:

<details>
<summary><strong>Código — corregido y funcional</strong></summary>

```javascript
let molecules = [];
const NUM_MOLECULES = 250;

let temperature = 1;
let globalTime = 0;
const LINK_DISTANCE = 55;

const RANDOMNESS = [0.05, 0.10, 0.18, 0.30, 0.50];
let randomLevel = 0;
let randomWalkStrength = RANDOMNESS[randomLevel];

const LEVY_PROBABILITIES = [0.0002, 0.0006, 0.003, 0.01, 0.03];
let levyProbIndex = 0;
let levyProbabilityBase = LEVY_PROBABILITIES[levyProbIndex];

function setup() {
  createCanvas(windowWidth, windowHeight);
  colorMode(HSB, 360, 100, 100, 100);
  noStroke();
  for (let i = 0; i < NUM_MOLECULES; i++) {
    molecules.push(new Molecule());
  }
}

function draw() {
  background(220, 30, 6, 18);
  globalTime += 0.003;
  temperature = map(mouseY, height, 0, 0.5, 2.2);
  drawTemperatureIndicator();

  for (let molecule of molecules) molecule.update();
  drawConnections();
  for (let molecule of molecules) molecule.display();
}

function drawConnections() {
  for (let i = 0; i < molecules.length; i++) {
    for (let j = i + 1; j < molecules.length; j++) {
      let d = dist(molecules[i].pos.x, molecules[i].pos.y, molecules[j].pos.x, molecules[j].pos.y);
      if (d < LINK_DISTANCE) {
        stroke(180, 20, 100, map(d, 0, LINK_DISTANCE, 80, 0));
        strokeWeight(map(d, 0, LINK_DISTANCE, 2, 0.3));
        line(molecules[i].pos.x, molecules[i].pos.y, molecules[j].pos.x, molecules[j].pos.y);
      }
    }
  }
}

function drawTemperatureIndicator() {
  noStroke();
  fill(15, 80, 100);
  rect(25, 25, 25, 160);
  let h = map(temperature, 0.5, 2.2, 0, 160);
  fill(0, 80, 100);
  rect(25, 185 - h, 25, h);
  fill(0, 0, 100);
  textSize(14);
  textAlign(LEFT);
  text("Temperatura", 20, 210);
  text(nf(temperature, 1, 2), 20, 230);
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}

function mousePressed() {
  randomLevel = (randomLevel + 1) % RANDOMNESS.length;
  randomWalkStrength = RANDOMNESS[randomLevel];
}

class Molecule {
  constructor() {
    this.pos = createVector(random(width), random(height));
    this.vel = createVector();
    this.tx = random(1000);
    this.ty = random(5000);
    this.size = random(5, 9);
    this.hue = random(170, 220);
    this.angle = random(TWO_PI);
    this.groupTarget = null;
    this.changeTimer = int(random(60, 180));
    this.energy = 0;
  }

  update() {
    let angle = map(noise(this.tx, this.ty), 0, 1, 0, TWO_PI);
    let perlin = p5.Vector.fromAngle(angle);
    perlin.mult((0.35 - randomWalkStrength * 0.4) * temperature);
    this.tx += 0.003;
    this.ty += 0.003;

    let randomWalk = p5.Vector.random2D();
    randomWalk.mult(randomWalkStrength * temperature);

    this.changeTimer--;
    if (this.changeTimer <= 0) {
      if (random() < 0.85) {
        this.groupTarget = molecules[int(random(molecules.length))];
      } else {
        this.groupTarget = null;
      }
      this.changeTimer = int(random(80, 200));
    }

    if (this.groupTarget != null) {
      let attraction = p5.Vector.sub(this.groupTarget.pos, this.pos);
      let d = attraction.mag();
      attraction.normalize();
      attraction.mult(map(d, 0, 200, 0, 0.12));
      this.vel.add(attraction);
    }

    let levyProbability = levyProbabilityBase;
    levyProbability *= temperature;
    if (random() < levyProbability) {
      let jump = p5.Vector.random2D();
      jump.mult(random(80, 180));
      this.pos.add(jump);
    }

    this.vel.add(perlin);
    this.vel.add(randomWalk);
    this.vel.limit(2.5 * temperature);
    this.pos.add(this.vel);
    this.vel.mult(0.96);

    if (this.pos.x < 0) this.pos.x = width;
    if (this.pos.x > width) this.pos.x = 0;
    if (this.pos.y < 0) this.pos.y = height;
    if (this.pos.y > height) this.pos.y = 0;
  }

  display() {
    push();
    translate(this.pos.x, this.pos.y);
    fill(this.hue, 70, 100, 90);
    ellipse(0, 0, this.size, this.size);
    pop();
  }
}
```
</details>

**Resultados:**

![Resultado 1](https://github.com/user-attachments/assets/c595bbe2-dad0-4c6f-b947-abea34d2b19b)
![Resultado 2](https://github.com/user-attachments/assets/3ced8439-8eb7-498d-9f4a-fab219cf3bdd)

> **Reflexión:** observando el resultado, me pareció algo aburrido para una feria de ciencias e innovación, así que decidí pensar en otra idea más acorde. Solo mostrar moléculas moviéndose y uniéndose aleatoriamente no me pareció tan conceptual, sino más experimental, ya que la forma de demostrarlo mediante la interacción con la temperatura no era la más ideal ni la más divertida. Sentí que podía dar mucho más.

---

## Intención conceptual #2 — Estación de sonar / localizador

Pensé bastante en la idea de que la pieza fuera parte de una experiencia más grande. Después de repasar varias ferias a las que he ido, algo que me llamó la atención fue que normalmente no tienen buenos ejemplificadores de cómo podría ser en la vida real un lugar como un submarino, un laboratorio, o demás espacios que uno solo se imagina. A partir de esa idea decidí crear una especie de localizador que combinara lo visto en clase con un diseño que tuviera estilo y utilidad, apoyándome también en un trabajo anterior de diseño gráfico que me inspiró en lo visual.

### Proceso — Paso 1: sistema base

Empecé con algo sencillo pero completo: puntos brillantes moviéndose en un fondo oscuro, con un solo campo de flujo de ruido Perlin que evoluciona en el tiempo, más reglas de paso que combinan random walk, distribución normal y Lévy flight, uniendo los estados en un mismo parámetro global ("coherencia") que sube y baja lentamente.

**¿Cómo se interpreta cada momento?**

- **Posibilidad** → al comenzar la simulación, todas las moléculas se mueven en direcciones completamente aleatorias. Ninguna dirección tiene más peso que otra.
- **Tendencia** → a medida que avanza la simulación, el ruido Perlin hace que las moléculas sigan trayectorias más suaves y parecidas entre sí.
- **Normalidad** → la mayor parte del tiempo las moléculas mantienen un comportamiento similar: permanecen cerca de otras y siguen el flujo general del sistema.
- **Excepción** → de vez en cuando ocurre un evento poco probable en el que una molécula realiza un salto mucho más largo gracias al Lévy Flight, explorando nuevas zonas.
- **Influencia** → la posición del visitante (mouse/touch) no mueve partículas directamente: cambia las probabilidades de dirección, no controla trayectorias.

**¿Por qué cumple las condiciones?**

- Combina 4 conceptos de la unidad: random walk, ruido Perlin (flow field), distribución normal (sigma del paso) y Lévy flight (excepción) — supera el mínimo de 3.
- Una sola pieza continua: todo vive en el mismo sistema de partículas y noise field, nunca cambia de "sketch".
- Sigue funcionando sin visitante: el noise field evoluciona solo con el tiempo (t como tercera dimensión del Perlin).
- Variación entre ejecuciones: semilla aleatoria distinta cada vez, pero la identidad visual se mantiene.

<img width="637" alt="Primer resultado del sistema base" src="https://github.com/user-attachments/assets/ac1bfe3f-2ac4-4a29-a96e-347f66d4f86d">

<details>
<summary><strong>Código — Paso 1</strong></summary>

```javascript
let particles = [];
const N = 260;

const NOISE_SCALE = 0.0032;
const FIELD_TURNS = 2.2;     
let zoff = 0;

const HIGH_SIGMA = 2.6;      
const LOW_SIGMA  = 0.10;     

const LEVY_PROB   = 0.0011;  
const LEVY_ALPHA  = 1.25;     
const LEVY_SCALE  = 7;

let echoes = [];        

let pointerActive = false;
let lastMoveTime = -9999;
const POINTER_RADIUS = 230;

let seed;

function setup() {
  const cnv = createCanvasFit();
  cnv.parent(document.body);
  pixelDensity(1);
  seed = floor(random(1000000));
  noiseSeed(seed);
  randomSeed(seed);
  colorMode(HSB, 360, 100, 100, 100);
  background(228, 60, 4);

  for (let i = 0; i < N; i++) {
    particles.push(makeParticle());
  }
}

function makeParticle() {
  return {
    x: random(width),
    y: random(height),
    angle: random(TWO_PI),
    speed: random(0.6, 1.4),
    hueShift: random(-14, 14)
  };
}

function windowResized() {
  createCanvasFit();
  background(228, 60, 4);
}

function createCanvasFit() {

  const targetRatio = 9 / 16;
  let w, h;
  if (windowWidth / windowHeight > targetRatio) {
    h = windowHeight;
    w = h * targetRatio;
  } else {
    w = windowWidth;
    h = w / targetRatio;
  }
  return createCanvas(floor(w), floor(h));
}

function coherenceNow() {
 
  const slow = 0.5 + 0.42 * sin(frameCount * 0.0021);
  const drift = (noise(4000, frameCount * 0.0007) - 0.5) * 0.28;
  return constrain(slow + drift, 0.04, 0.97);
}

function updatePointerState() {
  const moving = (abs(mouseX - pmouseX) > 0.5 || abs(mouseY - pmouseY) > 0.5);
  if (moving) lastMoveTime = millis();
  const inside = mouseX >= 0 && mouseX <= width && mouseY >= 0 && mouseY <= height;
  pointerActive = inside && (millis() - lastMoveTime < 900);
}

function fieldAngleAt(x, y) {
  let n = noise(x * NOISE_SCALE, y * NOISE_SCALE, zoff);
  let ang = n * TWO_PI * FIELD_TURNS;
  for (let e of echoes) {
    const dx = e.x - x, dy = e.y - y;
    const d = sqrt(dx * dx + dy * dy) + 0.001;
    if (d < e.reach) {
      const pull = (1 - d / e.reach) * e.strength;
      const toEcho = atan2(dy, dx);
      ang = lerpAngle(ang, toEcho, pull * 0.5);
    }
  }

  if (pointerActive) {
    const dx = x - mouseX, dy = y - mouseY;
    const d = sqrt(dx * dx + dy * dy);
    if (d < POINTER_RADIUS) {
      const w = 1 - d / POINTER_RADIUS;
      const swirl = atan2(dy, dx) + HALF_PI; 
      ang = lerpAngle(ang, swirl, w * 0.6);
    }
  }

  return ang;
}

function lerpAngle(a, b, t) {
  let diff = ((b - a + PI) % TWO_PI + TWO_PI) % TWO_PI - PI;
  return a + diff * t;
}

function localSigma(x, y, baseSigma) {
  if (!pointerActive) return baseSigma;
  const d = dist(x, y, mouseX, mouseY);
  if (d > POINTER_RADIUS) return baseSigma;
  const w = 1 - d / POINTER_RADIUS;
  return lerp(baseSigma, HIGH_SIGMA, w * 0.85);
}

function draw() {
  updatePointerState();
  const coherence = coherenceNow();
  const sigmaAngle = lerp(HIGH_SIGMA, LOW_SIGMA, coherence);

  blendMode(BLEND);
  noStroke();
  fill(228, 55, 4, 16);
  rect(0, 0, width, height);

  blendMode(ADD);

  for (let p of particles) {
    const base = fieldAngleAt(p.x, p.y);
    const sigma = localSigma(p.x, p.y, sigmaAngle);
    let angle = base + randomGaussian() * sigma;

    let stepLen;
    const isException = random() < LEVY_PROB;

    if (isException) {
      const u = random(0.015, 1);
      stepLen = min(pow(u, -1 / LEVY_ALPHA) * LEVY_SCALE, max(width, height) * 0.35);
      angle = random(TWO_PI);
    } else {
      stepLen = p.speed * abs(randomGaussian(1, 0.22));
    }

    p.x += cos(angle) * stepLen;
    p.y += sin(angle) * stepLen;

    if (p.x < 0) p.x += width;
    if (p.x > width) p.x -= width;
    if (p.y < 0) p.y += height;
    if (p.y > height) p.y -= height;

    if (isException) {
      echoes.push({ x: p.x, y: p.y, life: 260, maxLife: 260, reach: 160, strength: 0.9 });
    }

    const hue = lerp(200, 40, coherence) + p.hueShift;
    const sat = 55;
    const bri = 78;
    fill(hue, sat, bri, 55);
    circle(p.x, p.y, isException ? 4.5 : 2.4);
  }

  blendMode(ADD);
  for (let i = echoes.length - 1; i >= 0; i--) {
    const e = echoes[i];
    const t = e.life / e.maxLife;
    noFill();
    stroke(46, 70, 90, 26 * t);
    strokeWeight(1);
    circle(e.x, e.y, e.reach * 0.5 * (1.4 - t));
    e.life -= 1;
    e.strength = 0.9 * (e.life / e.maxLife);
    if (e.life <= 0) echoes.splice(i, 1);
  }

  if (pointerActive) {
    blendMode(BLEND);
    noFill();
    stroke(190, 30, 100, 8);
    strokeWeight(1);
    circle(mouseX, mouseY, POINTER_RADIUS * 1.6);
  }

  zoff += 0.0016;
}

function touchMoved() {
  lastMoveTime = millis();
  return false;
}
```
</details>

### Proceso — Paso 2: piel de instrumento científico

Al ver el resultado del paso 1 sentí que le faltaba terminar de amarrarse visualmente, así que le pedí ayuda a la IA para darle un aspecto más ordenado y "bello". Le describí cómo lo imaginaba:

- Verde fosforescente monocromático.
- Grid tipo radar: círculos concéntricos y líneas radiales desde el centro, más una cuadrícula fina tipo radar de submarino.
- Barrido de radar rotando lento (como si buscara), con estela que refuerce la idea de "búsqueda" en vez de decoración.
- Ecos: anillos que se expanden y se apagan, como si el instrumento detectara un evento u objeto raro.
- Textura tipo pantalla, para que se sintiera un monitor y no un lienzo de arte.
- Señalización con datos en vivo arriba a la izquierda: seed, estado (POSIBILIDAD/TENDENCIA/NORMALIDAD según coherencia), contador de excepciones, y si el sensor (visitante) está activo.
- El puntero, actuando como un sensor que indica si "detecta" o no la presencia de los visitantes.

<img width="642" alt="Piel de instrumento científico" src="https://github.com/user-attachments/assets/3ea9acf9-8f79-4ea7-8302-2c8719c8348b">

<details>
<summary><strong>Código — Paso 2</strong></summary>

```javascript
let particles = [];
const N = 240;

const NOISE_SCALE = 0.0032;
const FIELD_TURNS = 2.2;
let zoff = 0;

const HIGH_SIGMA = 2.6;
const LOW_SIGMA  = 0.10;

const LEVY_PROB  = 0.0011;
const LEVY_ALPHA = 1.25;
const LEVY_SCALE = 7;

let echoes = [];
let eventCount = 0;

let pointerActive = false;
let lastMoveTime = -9999;
const POINTER_RADIUS = 230;

let seed;
let sweepAngle = 0;

function setup() {
  createCanvasFit();
  pixelDensity(1);
  seed = floor(random(1000000));
  noiseSeed(seed);
  randomSeed(seed);
  colorMode(HSB, 360, 100, 100, 100);
  textFont('Courier New');
  background(0, 0, 1);

  for (let i = 0; i < N; i++) particles.push(makeParticle());
}

function makeParticle() {
  return { x: random(width), y: random(height), speed: random(0.6, 1.4) };
}

function windowResized() {
  createCanvasFit();
  background(0, 0, 1);
}

function createCanvasFit() {
  const targetRatio = 9 / 16;
  let w, h;
  if (windowWidth / windowHeight > targetRatio) {
    h = windowHeight; w = h * targetRatio;
  } else {
    w = windowWidth; h = w / targetRatio;
  }
  return createCanvas(floor(w), floor(h));
}

function coherenceNow() {
  const slow = 0.5 + 0.42 * sin(frameCount * 0.0021);
  const drift = (noise(4000, frameCount * 0.0007) - 0.5) * 0.28;
  return constrain(slow + drift, 0.04, 0.97);
}

function updatePointerState() {
  const moving = (abs(mouseX - pmouseX) > 0.5 || abs(mouseY - pmouseY) > 0.5);
  if (moving) lastMoveTime = millis();
  const inside = mouseX >= 0 && mouseX <= width && mouseY >= 0 && mouseY <= height;
  pointerActive = inside && (millis() - lastMoveTime < 900);
}

function lerpAngle(a, b, t) {
  let diff = ((b - a + PI) % TWO_PI + TWO_PI) % TWO_PI - PI;
  return a + diff * t;
}

function fieldAngleAt(x, y) {
  let n = noise(x * NOISE_SCALE, y * NOISE_SCALE, zoff);
  let ang = n * TWO_PI * FIELD_TURNS;

  for (let e of echoes) {
    const dx = e.x - x, dy = e.y - y;
    const d = sqrt(dx * dx + dy * dy) + 0.001;
    if (d < e.reach) {
      const pull = (1 - d / e.reach) * e.strength;
      ang = lerpAngle(ang, atan2(dy, dx), pull * 0.5);
    }
  }

  if (pointerActive) {
    const dx = x - mouseX, dy = y - mouseY;
    const d = sqrt(dx * dx + dy * dy);
    if (d < POINTER_RADIUS) {
      const w = 1 - d / POINTER_RADIUS;
      const swirl = atan2(dy, dx) + HALF_PI;
      ang = lerpAngle(ang, swirl, w * 0.6);
    }
  }
  return ang;
}

function localSigma(x, y, baseSigma) {
  if (!pointerActive) return baseSigma;
  const d = dist(x, y, mouseX, mouseY);
  if (d > POINTER_RADIUS) return baseSigma;
  const w = 1 - d / POINTER_RADIUS;
  return lerp(baseSigma, HIGH_SIGMA, w * 0.85);
}


function drawRadarGrid() {
  push();
  translate(width / 2, height / 2);
  noFill();
  stroke(130, 45, 30, 22);
  strokeWeight(1);
  const maxR = dist(0, 0, width / 2, height / 2);
  for (let r = maxR / 5; r <= maxR; r += maxR / 5) circle(0, 0, r * 2);
  for (let a = 0; a < TWO_PI; a += PI / 6) line(0, 0, cos(a) * maxR, sin(a) * maxR);
  pop();

  stroke(130, 40, 22, 10);
  strokeWeight(1);
  for (let x = 0; x < width; x += 34) line(x, 0, x, height);
  for (let y = 0; y < height; y += 34) line(0, y, width, y);
}

function drawSweep() {
  push();
  translate(width / 2, height / 2);
  const maxR = dist(0, 0, width / 2, height / 2);
  noStroke();
  for (let i = 0; i < 18; i++) {
    const a = sweepAngle - i * 0.02;
    fill(130, 60, 55, 3.2 * (1 - i / 18));
    beginShape();
    vertex(0, 0);
    vertex(cos(a) * maxR, sin(a) * maxR);
    vertex(cos(a - 0.02) * maxR, sin(a - 0.02) * maxR);
    endShape(CLOSE);
  }
  stroke(130, 25, 95, 45);
  strokeWeight(1.4);
  line(0, 0, cos(sweepAngle) * maxR, sin(sweepAngle) * maxR);
  pop();
}

function drawScanlines() {
  noStroke();
  fill(0, 0, 0, 7);
  for (let y = 0; y < height; y += 3) rect(0, y, width, 1);
  noFill();
  for (let i = 0; i < 24; i++) {
    stroke(0, 0, 0, 3);
    strokeWeight(i);
    rect(i / 2, i / 2, width - i, height - i);
  }
}

function drawHUD(coherence) {
  push();
  noStroke();
  fill(130, 20, 95, 85);
  textSize(11);
  textLeading(15);
  const estado = coherence > 0.62 ? 'NORMALIDAD' : (coherence > 0.32 ? 'TENDENCIA' : 'POSIBILIDAD');
  const txt =
`SISTEMA · NAVEGAR LA INCERTIDUMBRE
SEED       ${seed}
ESTADO     ${estado}
COHERENCIA ${nf(coherence, 1, 2)}
EXCEPC.    ${eventCount}
SENSOR     ${pointerActive ? 'ACTIVO' : 'EN ESPERA'}`;
  text(txt, 16, 22);
  pop();
}

function draw() {
  updatePointerState();
  const coherence = coherenceNow();
  const sigmaAngle = lerp(HIGH_SIGMA, LOW_SIGMA, coherence);

  blendMode(BLEND);
  noStroke();
  fill(0, 0, 1, 14);
  rect(0, 0, width, height);

  drawRadarGrid();

  blendMode(ADD);
  drawSweep();

  for (let p of particles) {
    const base = fieldAngleAt(p.x, p.y);
    const sigma = localSigma(p.x, p.y, sigmaAngle);
    let angle = base + randomGaussian() * sigma;

    let stepLen;
    const isException = random() < LEVY_PROB;

    if (isException) {
      const u = random(0.015, 1);
      stepLen = min(pow(u, -1 / LEVY_ALPHA) * LEVY_SCALE, max(width, height) * 0.35);
      angle = random(TWO_PI);
    } else {
      stepLen = p.speed * abs(randomGaussian(1, 0.22));
    }

    p.x += cos(angle) * stepLen;
    p.y += sin(angle) * stepLen;

    if (p.x < 0) p.x += width;
    if (p.x > width) p.x -= width;
    if (p.y < 0) p.y += height;
    if (p.y > height) p.y -= height;

    if (isException) {
      echoes.push({ x: p.x, y: p.y, life: 260, maxLife: 260, reach: 160, strength: 0.9 });
      eventCount++;
    }

    const bri = lerp(55, 95, coherence * 0.4 + 0.3);
    fill(130, 55, bri, 60);
    circle(p.x, p.y, isException ? 4.5 : 2.2);
  }

  for (let i = echoes.length - 1; i >= 0; i--) {
    const e = echoes[i];
    const t = e.life / e.maxLife;
    noFill();
    stroke(130, 30, 95, 30 * t);
    strokeWeight(1);
    circle(e.x, e.y, e.reach * 0.55 * (1.4 - t));
    e.life -= 1;
    e.strength = 0.9 * (e.life / e.maxLife);
    if (e.life <= 0) echoes.splice(i, 1);
  }

  if (pointerActive) {
    blendMode(BLEND);
    noFill();
    stroke(130, 25, 95, 55);
    strokeWeight(1);
    circle(mouseX, mouseY, 26);
    line(mouseX - 16, mouseY, mouseX - 8, mouseY);
    line(mouseX + 8, mouseY, mouseX + 16, mouseY);
    line(mouseX, mouseY - 16, mouseX, mouseY - 8);
    line(mouseX, mouseY + 8, mouseX, mouseY + 16);
    drawingContext.setLineDash([3, 4]);
    stroke(130, 25, 95, 20);
    circle(mouseX, mouseY, POINTER_RADIUS * 1.5);
    drawingContext.setLineDash([]);
  }

  blendMode(BLEND);
  drawScanlines();
  drawHUD(coherence);

  sweepAngle += 0.012;
  zoff += 0.0016;
}

function touchMoved() {
  lastMoveTime = millis();
  return false;
}
```
</details>

### Arreglo 1 — Interacción más notoria

Veía la interacción un poco aburrida y plana, nada interesante, así que —con ayuda de la IA, porque no tenía muy claro cómo hacerlo— decidí modificar el puntero para que fuera más expresivo: si el sensor se acerca a las partículas, estas se iluminan y se mueven más rápido, simulando un posible "ataque"; si el sensor se queda quieto, la amenaza desaparece junto con él. También se hicieron ajustes independientes:

- **Radio de influencia más grande** (230 → 300 px) y caída del efecto más lenta (`pow(..., 0.6)`), así se siente en más área y no solo pegado al cursor.
- **Remolino mucho más fuerte:** el peso subió de 0.6 a 0.95, así cerca del puntero las partículas casi obedecen por completo al giro.
- **Pulsos de sensor:** cada ~260 ms el puntero emite un anillo que se expande y se apaga, como un radar detectando presencia activa.
- **Retícula con núcleo brillante** (antes eran solo líneas finas casi invisibles).

<img width="646" alt="Interacción mejorada" src="https://github.com/user-attachments/assets/e75f5237-6422-472d-82ac-6df99f6b8343">

<details>
<summary><strong>Código — Arreglo 1</strong></summary>

```javascript
let particles = [];
const N = 240;

const NOISE_SCALE = 0.0032;
const FIELD_TURNS = 2.2;
let zoff = 0;

const HIGH_SIGMA = 2.6;
const LOW_SIGMA  = 0.10;

const LEVY_PROB  = 0.0011;
const LEVY_ALPHA = 1.25;
const LEVY_SCALE = 7;

let echoes = [];
let eventCount = 0;

let pointerActive = false;
let lastMoveTime = -9999;
const POINTER_RADIUS = 300;
let pings = []; 
let lastPing = 0;

let seed;
let sweepAngle = 0;
let mono; 

function preload() {
}

function setup() {
  createCanvasFit();
  pixelDensity(1);
  seed = floor(random(1000000));
  noiseSeed(seed);
  randomSeed(seed);
  colorMode(HSB, 360, 100, 100, 100);
  textFont('Courier New');
  background(0, 0, 1);

  for (let i = 0; i < N; i++) particles.push(makeParticle());
}

function makeParticle() {
  return {
    x: random(width),
    y: random(height),
    speed: random(0.6, 1.4)
  };
}

function windowResized() {
  createCanvasFit();
  background(0, 0, 1);
}

function createCanvasFit() {
  const targetRatio = 9 / 16;
  let w, h;
  if (windowWidth / windowHeight > targetRatio) {
    h = windowHeight; w = h * targetRatio;
  } else {
    w = windowWidth; h = w / targetRatio;
  }
  return createCanvas(floor(w), floor(h));
}

function coherenceNow() {
  const slow = 0.5 + 0.42 * sin(frameCount * 0.0021);
  const drift = (noise(4000, frameCount * 0.0007) - 0.5) * 0.28;
  return constrain(slow + drift, 0.04, 0.97);
}

function updatePointerState() {
  const moving = (abs(mouseX - pmouseX) > 0.5 || abs(mouseY - pmouseY) > 0.5);
  if (moving) lastMoveTime = millis();
  const inside = mouseX >= 0 && mouseX <= width && mouseY >= 0 && mouseY <= height;
  pointerActive = inside && (millis() - lastMoveTime < 900);
}

function lerpAngle(a, b, t) {
  let diff = ((b - a + PI) % TWO_PI + TWO_PI) % TWO_PI - PI;
  return a + diff * t;
}

function fieldAngleAt(x, y) {
  let n = noise(x * NOISE_SCALE, y * NOISE_SCALE, zoff);
  let ang = n * TWO_PI * FIELD_TURNS;

  for (let e of echoes) {
    const dx = e.x - x, dy = e.y - y;
    const d = sqrt(dx * dx + dy * dy) + 0.001;
    if (d < e.reach) {
      const pull = (1 - d / e.reach) * e.strength;
      ang = lerpAngle(ang, atan2(dy, dx), pull * 0.5);
    }
  }

  if (pointerActive) {
    const dx = x - mouseX, dy = y - mouseY;
    const d = sqrt(dx * dx + dy * dy);
    if (d < POINTER_RADIUS) {
      const w = pow(1 - d / POINTER_RADIUS, 0.6); 
      const swirl = atan2(dy, dx) + HALF_PI;
      ang = lerpAngle(ang, swirl, w * 0.95);
    }
  }
  return ang;
}

function localSigma(x, y, baseSigma) {
  if (!pointerActive) return baseSigma;
  const d = dist(x, y, mouseX, mouseY);
  if (d > POINTER_RADIUS) return baseSigma;
  const w = pow(1 - d / POINTER_RADIUS, 0.6);
  return lerp(baseSigma, HIGH_SIGMA * 1.3, w);
}

function pointerSpeedBoost(x, y) {
  if (!pointerActive) return 1;
  const d = dist(x, y, mouseX, mouseY);
  if (d > POINTER_RADIUS) return 1;
  const w = pow(1 - d / POINTER_RADIUS, 0.6);
  return lerp(1, 2.2, w); 
}

function drawRadarGrid() {
  push();
  translate(width / 2, height / 2);
  noFill();
  stroke(130, 45, 30, 22);
  strokeWeight(1);
  const maxR = dist(0, 0, width / 2, height / 2);
  for (let r = maxR / 5; r <= maxR; r += maxR / 5) circle(0, 0, r * 2);
  for (let a = 0; a < TWO_PI; a += PI / 6) {
    line(0, 0, cos(a) * maxR, sin(a) * maxR);
  }
  pop();
  stroke(130, 40, 22, 10);
  strokeWeight(1);
  for (let x = 0; x < width; x += 34) line(x, 0, x, height);
  for (let y = 0; y < height; y += 34) line(0, y, width, y);
}

function drawSweep() {
  push();
  translate(width / 2, height / 2);
  const maxR = dist(0, 0, width / 2, height / 2);
  noStroke();
  for (let i = 0; i < 18; i++) {
    const a = sweepAngle - i * 0.02;
    fill(130, 60, 55, 3.2 * (1 - i / 18));
    beginShape();
    vertex(0, 0);
    vertex(cos(a) * maxR, sin(a) * maxR);
    vertex(cos(a - 0.02) * maxR, sin(a - 0.02) * maxR);
    endShape(CLOSE);
  }
  stroke(130, 25, 95, 45);
  strokeWeight(1.4);
  line(0, 0, cos(sweepAngle) * maxR, sin(sweepAngle) * maxR);
  pop();
}

function drawScanlines() {
  noStroke();
  fill(0, 0, 0, 7);
  for (let y = 0; y < height; y += 3) rect(0, y, width, 1);
  noFill();
  for (let i = 0; i < 24; i++) {
    stroke(0, 0, 0, 3);
    strokeWeight(i);
    rect(i / 2, i / 2, width - i, height - i);
  }
}

function drawHUD(coherence) {
  push();
  noStroke();
  fill(130, 20, 95, 85);
  textSize(11);
  textLeading(15);
  const estado = coherence > 0.62 ? 'NORMALIDAD' : (coherence > 0.32 ? 'TENDENCIA' : 'POSIBILIDAD');
  const txt =
`SISTEMA · NAVEGAR LA INCERTIDUMBRE
SEED       ${seed}
ESTADO     ${estado}
COHERENCIA ${nf(coherence, 1, 2)}
EXCEPC.    ${eventCount}
SENSOR     ${pointerActive ? 'ACTIVO' : 'EN ESPERA'}`;
  text(txt, 16, 22);
  pop();
}

function draw() {
  updatePointerState();
  const coherence = coherenceNow();
  const sigmaAngle = lerp(HIGH_SIGMA, LOW_SIGMA, coherence);

  blendMode(BLEND);
  noStroke();
  fill(0, 0, 1, 14);
  rect(0, 0, width, height);

  drawRadarGrid();

  blendMode(ADD);
  drawSweep();

  for (let p of particles) {
    const base = fieldAngleAt(p.x, p.y);
    const sigma = localSigma(p.x, p.y, sigmaAngle);
    let angle = base + randomGaussian() * sigma;
    const boost = pointerSpeedBoost(p.x, p.y);

    let stepLen;
    const isException = random() < LEVY_PROB;

    if (isException) {
      const u = random(0.015, 1);
      stepLen = min(pow(u, -1 / LEVY_ALPHA) * LEVY_SCALE, max(width, height) * 0.35);
      angle = random(TWO_PI);
    } else {
      stepLen = p.speed * abs(randomGaussian(1, 0.22)) * boost;
    }

    p.x += cos(angle) * stepLen;
    p.y += sin(angle) * stepLen;

    if (p.x < 0) p.x += width;
    if (p.x > width) p.x -= width;
    if (p.y < 0) p.y += height;
    if (p.y > height) p.y -= height;

    if (isException) {
      echoes.push({ x: p.x, y: p.y, life: 260, maxLife: 260, reach: 160, strength: 0.9 });
      eventCount++;
    }
    const near = pointerActive ? constrain(1 - dist(p.x, p.y, mouseX, mouseY) / POINTER_RADIUS, 0, 1) : 0;
    const bri = lerp(lerp(50, 92, coherence * 0.4 + 0.3), 100, near);
    const sat = lerp(60, 12, near); // hacia blanco cerca del visitante
    const size = lerp(isException ? 4.5 : 2.2, isException ? 7 : 5, near);
    fill(130, sat, bri, near > 0.05 ? 85 : 60);
    circle(p.x, p.y, size);
  }

  if (pointerActive && millis() - lastPing > 260) {
    pings.push({ x: mouseX, y: mouseY, life: 55, maxLife: 55 });
    lastPing = millis();
  }
  for (let i = pings.length - 1; i >= 0; i--) {
    const pg = pings[i];
    const t = pg.life / pg.maxLife;
    noFill();
    stroke(130, 20, 100, 60 * t);
    strokeWeight(1.6);
    circle(pg.x, pg.y, (1 - t) * POINTER_RADIUS * 1.4);
    pg.life -= 1;
    if (pg.life <= 0) pings.splice(i, 1);
  }

  for (let i = echoes.length - 1; i >= 0; i--) {
    const e = echoes[i];
    const t = e.life / e.maxLife;
    noFill();
    stroke(130, 30, 95, 30 * t);
    strokeWeight(1);
    circle(e.x, e.y, e.reach * 0.55 * (1.4 - t));
    e.life -= 1;
    e.strength = 0.9 * (e.life / e.maxLife);
    if (e.life <= 0) echoes.splice(i, 1);
  }

  if (pointerActive) {
    blendMode(ADD);
    noStroke();
    fill(130, 15, 100, 30);
    circle(mouseX, mouseY, 46);
    blendMode(BLEND);
    noFill();
    stroke(130, 15, 100, 85);
    strokeWeight(1.4);
    circle(mouseX, mouseY, 26);
    line(mouseX - 20, mouseY, mouseX - 8, mouseY);
    line(mouseX + 8, mouseY, mouseX + 20, mouseY);
    line(mouseX, mouseY - 20, mouseX, mouseY - 8);
    line(mouseX, mouseY + 8, mouseX, mouseY + 20);
    drawingContext.setLineDash([3, 4]);
    stroke(130, 25, 95, 35);
    circle(mouseX, mouseY, POINTER_RADIUS * 1.5);
    drawingContext.setLineDash([]);
  }

  blendMode(BLEND);
  drawScanlines();
  drawHUD(coherence);

  sweepAngle += 0.012;
  zoff += 0.0016;
}

function touchMoved() {
  lastMoveTime = millis();
  return false;
}
```
</details>

### Arreglo 2 — Mockup visual (submarino)

Por último quise hacer un poco de diseño y ajusté la parte visual, tipo mockup, para dar una idea de cómo se vería instalado. En esto me apoyé en la IA, ya que no manejaba muy bien el lenguaje de color y forma para construirlo de manera óptima:

- Formato horizontal 16:9 en vez de vertical.
- Panel metálico con remaches, tuberías laterales decorativas, y una placa con el nombre "ESTACIÓN DE SONAR — DECK 2".
- Pantalla empotrada con bisel oscuro y un brillo diagonal tipo vidrio curvo, para que se sienta como un monitor físico y no un canvas.
- Diales y switches decorativos debajo de la pantalla (profundidad, presión, oxígeno) — puramente estéticos, no funcionales, solo para vender la idea de "consola real".

<details>
<summary><strong>Código — Arreglo 2 (sketch.js)</strong></summary>

```javascript
let particles = [];
const N = 240;

const NOISE_SCALE = 0.0032;
const FIELD_TURNS = 2.2;
let zoff = 0;

const HIGH_SIGMA = 2.6;
const LOW_SIGMA  = 0.10;

const LEVY_PROB  = 0.0011;
const LEVY_ALPHA = 1.25;
const LEVY_SCALE = 7;

let echoes = [];
let eventCount = 0;

let pointerActive = false;
let lastMoveTime = -9999;
let POINTER_RADIUS = 260;
let pings = [];
let lastPing = 0;

let seed;
let sweepAngle = 0;
let screenEl;

function setup() {
  screenEl = document.getElementById('screen');
  const cnv = createCanvasFit();
  cnv.parent('screen');
  pixelDensity(1);
  seed = floor(random(1000000));
  noiseSeed(seed);
  randomSeed(seed);
  colorMode(HSB, 360, 100, 100, 100);
  textFont('Courier New');
  background(0, 0, 1);

  for (let i = 0; i < N; i++) particles.push(makeParticle());
}

function makeParticle() {
  return { x: random(width), y: random(height), speed: random(0.6, 1.4) };
}

function windowResized() {
  createCanvasFit();
  background(0, 0, 1);
}

function createCanvasFit() {
  const w = screenEl.clientWidth;
  const h = screenEl.clientHeight;
  POINTER_RADIUS = min(w, h) * 0.45;
  return resizeOrCreate(w, h);
}

let _created = false;
function resizeOrCreate(w, h) {
  if (!_created) {
    _created = true;
    return createCanvas(floor(w), floor(h));
  } else {
    resizeCanvas(floor(w), floor(h));
    return null;
  }
}

function coherenceNow() {
  const slow = 0.5 + 0.42 * sin(frameCount * 0.0021);
  const drift = (noise(4000, frameCount * 0.0007) - 0.5) * 0.28;
  return constrain(slow + drift, 0.04, 0.97);
}

function updatePointerState() {
  const moving = (abs(mouseX - pmouseX) > 0.5 || abs(mouseY - pmouseY) > 0.5);
  if (moving) lastMoveTime = millis();
  const inside = mouseX >= 0 && mouseX <= width && mouseY >= 0 && mouseY <= height;
  pointerActive = inside && (millis() - lastMoveTime < 900);
}

function lerpAngle(a, b, t) {
  let diff = ((b - a + PI) % TWO_PI + TWO_PI) % TWO_PI - PI;
  return a + diff * t;
}

function fieldAngleAt(x, y) {
  let n = noise(x * NOISE_SCALE, y * NOISE_SCALE, zoff);
  let ang = n * TWO_PI * FIELD_TURNS;

  for (let e of echoes) {
    const dx = e.x - x, dy = e.y - y;
    const d = sqrt(dx * dx + dy * dy) + 0.001;
    if (d < e.reach) {
      const pull = (1 - d / e.reach) * e.strength;
      ang = lerpAngle(ang, atan2(dy, dx), pull * 0.5);
    }
  }

  if (pointerActive) {
    const dx = x - mouseX, dy = y - mouseY;
    const d = sqrt(dx * dx + dy * dy);
    if (d < POINTER_RADIUS) {
      const w = pow(1 - d / POINTER_RADIUS, 0.6);
      const swirl = atan2(dy, dx) + HALF_PI;
      ang = lerpAngle(ang, swirl, w * 0.95);
    }
  }
  return ang;
}

function localSigma(x, y, baseSigma) {
  if (!pointerActive) return baseSigma;
  const d = dist(x, y, mouseX, mouseY);
  if (d > POINTER_RADIUS) return baseSigma;
  const w = pow(1 - d / POINTER_RADIUS, 0.6);
  return lerp(baseSigma, HIGH_SIGMA * 1.3, w);
}

function pointerSpeedBoost(x, y) {
  if (!pointerActive) return 1;
  const d = dist(x, y, mouseX, mouseY);
  if (d > POINTER_RADIUS) return 1;
  const w = pow(1 - d / POINTER_RADIUS, 0.6);
  return lerp(1, 2.2, w);
}

function drawRadarGrid() {
  push();
  translate(width / 2, height / 2);
  noFill();
  stroke(130, 45, 30, 22);
  strokeWeight(1);
  const maxR = dist(0, 0, width / 2, height / 2);
  for (let r = maxR / 5; r <= maxR; r += maxR / 5) circle(0, 0, r * 2);
  for (let a = 0; a < TWO_PI; a += PI / 6) line(0, 0, cos(a) * maxR, sin(a) * maxR);
  pop();

  stroke(130, 40, 22, 10);
  strokeWeight(1);
  for (let x = 0; x < width; x += 34) line(x, 0, x, height);
  for (let y = 0; y < height; y += 34) line(0, y, width, y);
}

function drawSweep() {
  push();
  translate(width / 2, height / 2);
  const maxR = dist(0, 0, width / 2, height / 2);
  noStroke();
  for (let i = 0; i < 18; i++) {
    const a = sweepAngle - i * 0.02;
    fill(130, 60, 55, 3.2 * (1 - i / 18));
    beginShape();
    vertex(0, 0);
    vertex(cos(a) * maxR, sin(a) * maxR);
    vertex(cos(a - 0.02) * maxR, sin(a - 0.02) * maxR);
    endShape(CLOSE);
  }
  stroke(130, 25, 95, 45);
  strokeWeight(1.4);
  line(0, 0, cos(sweepAngle) * maxR, sin(sweepAngle) * maxR);
  pop();
}

function drawScanlines() {
  noStroke();
  fill(0, 0, 0, 7);
  for (let y = 0; y < height; y += 3) rect(0, y, width, 1);
  noFill();
  for (let i = 0; i < 24; i++) {
    stroke(0, 0, 0, 3);
    strokeWeight(i);
    rect(i / 2, i / 2, width - i, height - i);
  }
}

function drawHUD(coherence) {
  push();
  noStroke();
  fill(130, 20, 95, 85);
  textSize(11);
  textLeading(15);
  const estado = coherence > 0.62 ? 'NORMALIDAD' : (coherence > 0.32 ? 'TENDENCIA' : 'POSIBILIDAD');
  const txt =
`SISTEMA · NAVEGAR LA INCERTIDUMBRE
SEED       ${seed}
ESTADO     ${estado}
COHERENCIA ${nf(coherence, 1, 2)}
EXCEPC.    ${eventCount}
SENSOR     ${pointerActive ? 'ACTIVO' : 'EN ESPERA'}`;
  text(txt, 14, 20);
  pop();
}

function draw() {
  updatePointerState();
  const coherence = coherenceNow();
  const sigmaAngle = lerp(HIGH_SIGMA, LOW_SIGMA, coherence);

  blendMode(BLEND);
  noStroke();
  fill(0, 0, 1, 14);
  rect(0, 0, width, height);

  drawRadarGrid();

  blendMode(ADD);
  drawSweep();

  for (let p of particles) {
    const base = fieldAngleAt(p.x, p.y);
    const sigma = localSigma(p.x, p.y, sigmaAngle);
    let angle = base + randomGaussian() * sigma;
    const boost = pointerSpeedBoost(p.x, p.y);

    let stepLen;
    const isException = random() < LEVY_PROB;

    if (isException) {
      const u = random(0.015, 1);
      stepLen = min(pow(u, -1 / LEVY_ALPHA) * LEVY_SCALE, max(width, height) * 0.35);
      angle = random(TWO_PI);
    } else {
      stepLen = p.speed * abs(randomGaussian(1, 0.22)) * boost;
    }

    p.x += cos(angle) * stepLen;
    p.y += sin(angle) * stepLen;

    if (p.x < 0) p.x += width;
    if (p.x > width) p.x -= width;
    if (p.y < 0) p.y += height;
    if (p.y > height) p.y -= height;

    if (isException) {
      echoes.push({ x: p.x, y: p.y, life: 260, maxLife: 260, reach: 160, strength: 0.9 });
      eventCount++;
    }

    const near = pointerActive ? constrain(1 - dist(p.x, p.y, mouseX, mouseY) / POINTER_RADIUS, 0, 1) : 0;
    const bri = lerp(lerp(50, 92, coherence * 0.4 + 0.3), 100, near);
    const sat = lerp(60, 12, near);
    const size = lerp(isException ? 4.5 : 2.2, isException ? 7 : 5, near);
    fill(130, sat, bri, near > 0.05 ? 85 : 60);
    circle(p.x, p.y, size);
  }

  if (pointerActive && millis() - lastPing > 260) {
    pings.push({ x: mouseX, y: mouseY, life: 55, maxLife: 55 });
    lastPing = millis();
  }
  for (let i = pings.length - 1; i >= 0; i--) {
    const pg = pings[i];
    const t = pg.life / pg.maxLife;
    noFill();
    stroke(130, 20, 100, 60 * t);
    strokeWeight(1.6);
    circle(pg.x, pg.y, (1 - t) * POINTER_RADIUS * 1.4);
    pg.life -= 1;
    if (pg.life <= 0) pings.splice(i, 1);
  }

  for (let i = echoes.length - 1; i >= 0; i--) {
    const e = echoes[i];
    const t = e.life / e.maxLife;
    noFill();
    stroke(130, 30, 95, 30 * t);
    strokeWeight(1);
    circle(e.x, e.y, e.reach * 0.55 * (1.4 - t));
    e.life -= 1;
    e.strength = 0.9 * (e.life / e.maxLife);
    if (e.life <= 0) echoes.splice(i, 1);
  }

  if (pointerActive) {
    blendMode(ADD);
    noStroke();
    fill(130, 15, 100, 30);
    circle(mouseX, mouseY, 46);
    blendMode(BLEND);
    noFill();
    stroke(130, 15, 100, 85);
    strokeWeight(1.4);
    circle(mouseX, mouseY, 26);
    line(mouseX - 20, mouseY, mouseX - 8, mouseY);
    line(mouseX + 8, mouseY, mouseX + 20, mouseY);
    line(mouseX, mouseY - 20, mouseX, mouseY - 8);
    line(mouseX, mouseY + 8, mouseX, mouseY + 20);
    drawingContext.setLineDash([3, 4]);
    stroke(130, 25, 95, 35);
    circle(mouseX, mouseY, POINTER_RADIUS * 1.5);
    drawingContext.setLineDash([]);
  }

  blendMode(BLEND);
  drawScanlines();
  drawHUD(coherence);

  sweepAngle += 0.012;
  zoff += 0.0016;
}

function touchMoved() {
  lastMoveTime = millis();
  return false;
}
```
</details>

### Arreglo 3 — Corrección de errores en el HTML

Al integrar el sketch dentro del mockup del submarino aparecieron los siguientes errores:

![Error de integración](https://github.com/user-attachments/assets/8f35f7d0-3373-472d-958e-bd056d76864e)

Recordé que tenía que organizar bien el `.html` de la interfaz, ya que no coincidía por completo con lo que necesitaba el sketch (los IDs y la estructura de los contenedores no estaban alineados con lo que esperaba `sketch.js`). Al revisar y ajustar esa correspondencia entre `index.html`, `style.css` y `sketch.js`, se solucionó.  
<details>
<summary><strong>Código final</strong></summary>

```javascript
let particles = [];
const N = 240;
 
const NOISE_SCALE = 0.0032;
const FIELD_TURNS = 2.2;
let zoff = 0;
 
const HIGH_SIGMA = 2.6;
const LOW_SIGMA  = 0.10;
 
const LEVY_PROB  = 0.0011;
const LEVY_ALPHA = 1.25;
const LEVY_SCALE = 7;
 
let echoes = [];
let eventCount = 0;
 
let pointerActive = false;
let lastMoveTime = -9999;
let POINTER_RADIUS = 260;
let pings = [];
let lastPing = 0;
 
let seed;
let sweepAngle = 0;
let screenEl;
 
function setup() {
  screenEl = document.getElementById('screen');
  const cnv = createCanvasFit();
  cnv.parent('screen');
  pixelDensity(1);
  seed = floor(random(1000000));
  noiseSeed(seed);
  randomSeed(seed);
  colorMode(HSB, 360, 100, 100, 100);
  textFont('Courier New');
  background(0, 0, 1);
 
  for (let i = 0; i < N; i++) particles.push(makeParticle());
}
 
function makeParticle() {
  return { x: random(width), y: random(height), speed: random(0.6, 1.4) };
}
 
function windowResized() {
  createCanvasFit();
  background(0, 0, 1);
}
 
function createCanvasFit() {
  const w = screenEl.clientWidth;
  const h = screenEl.clientHeight;
  POINTER_RADIUS = min(w, h) * 0.45;
  return resizeOrCreate(w, h);
}
 
let _created = false;
function resizeOrCreate(w, h) {
  if (!_created) {
    _created = true;
    return createCanvas(floor(w), floor(h));
  } else {
    resizeCanvas(floor(w), floor(h));
    return null;
  }
}
 
function coherenceNow() {
  const slow = 0.5 + 0.42 * sin(frameCount * 0.0021);
  const drift = (noise(4000, frameCount * 0.0007) - 0.5) * 0.28;
  return constrain(slow + drift, 0.04, 0.97);
}
 
function updatePointerState() {
  const moving = (abs(mouseX - pmouseX) > 0.5 || abs(mouseY - pmouseY) > 0.5);
  if (moving) lastMoveTime = millis();
  const inside = mouseX >= 0 && mouseX <= width && mouseY >= 0 && mouseY <= height;
  pointerActive = inside && (millis() - lastMoveTime < 900);
}
 
function lerpAngle(a, b, t) {
  let diff = ((b - a + PI) % TWO_PI + TWO_PI) % TWO_PI - PI;
  return a + diff * t;
}
 
function fieldAngleAt(x, y) {
  let n = noise(x * NOISE_SCALE, y * NOISE_SCALE, zoff);
  let ang = n * TWO_PI * FIELD_TURNS;
 
  for (let e of echoes) {
    const dx = e.x - x, dy = e.y - y;
    const d = sqrt(dx * dx + dy * dy) + 0.001;
    if (d < e.reach) {
      const pull = (1 - d / e.reach) * e.strength;
      ang = lerpAngle(ang, atan2(dy, dx), pull * 0.5);
    }
  }
 
  if (pointerActive) {
    const dx = x - mouseX, dy = y - mouseY;
    const d = sqrt(dx * dx + dy * dy);
    if (d < POINTER_RADIUS) {
      const w = pow(1 - d / POINTER_RADIUS, 0.6);
      const swirl = atan2(dy, dx) + HALF_PI;
      ang = lerpAngle(ang, swirl, w * 0.95);
    }
  }
  return ang;
}
 
function localSigma(x, y, baseSigma) {
  if (!pointerActive) return baseSigma;
  const d = dist(x, y, mouseX, mouseY);
  if (d > POINTER_RADIUS) return baseSigma;
  const w = pow(1 - d / POINTER_RADIUS, 0.6);
  return lerp(baseSigma, HIGH_SIGMA * 1.3, w);
}
 
function pointerSpeedBoost(x, y) {
  if (!pointerActive) return 1;
  const d = dist(x, y, mouseX, mouseY);
  if (d > POINTER_RADIUS) return 1;
  const w = pow(1 - d / POINTER_RADIUS, 0.6);
  return lerp(1, 2.2, w);
}
 
function drawRadarGrid() {
  push();
  translate(width / 2, height / 2);
  noFill();
  stroke(130, 45, 30, 22);
  strokeWeight(1);
  const maxR = dist(0, 0, width / 2, height / 2);
  for (let r = maxR / 5; r <= maxR; r += maxR / 5) circle(0, 0, r * 2);
  for (let a = 0; a < TWO_PI; a += PI / 6) line(0, 0, cos(a) * maxR, sin(a) * maxR);
  pop();
 
  stroke(130, 40, 22, 10);
  strokeWeight(1);
  for (let x = 0; x < width; x += 34) line(x, 0, x, height);
  for (let y = 0; y < height; y += 34) line(0, y, width, y);
}
 
function drawSweep() {
  push();
  translate(width / 2, height / 2);
  const maxR = dist(0, 0, width / 2, height / 2);
  noStroke();
  for (let i = 0; i < 18; i++) {
    const a = sweepAngle - i * 0.02;
    fill(130, 60, 55, 3.2 * (1 - i / 18));
    beginShape();
    vertex(0, 0);
    vertex(cos(a) * maxR, sin(a) * maxR);
    vertex(cos(a - 0.02) * maxR, sin(a - 0.02) * maxR);
    endShape(CLOSE);
  }
  stroke(130, 25, 95, 45);
  strokeWeight(1.4);
  line(0, 0, cos(sweepAngle) * maxR, sin(sweepAngle) * maxR);
  pop();
}
 
function drawScanlines() {
  noStroke();
  fill(0, 0, 0, 7);
  for (let y = 0; y < height; y += 3) rect(0, y, width, 1);
  noFill();
  for (let i = 0; i < 24; i++) {
    stroke(0, 0, 0, 3);
    strokeWeight(i);
    rect(i / 2, i / 2, width - i, height - i);
  }
}
 
function drawHUD(coherence) {
  push();
  noStroke();
  fill(130, 20, 95, 85);
  textSize(11);
  textLeading(15);
  const estado = coherence > 0.62 ? 'NORMALIDAD' : (coherence > 0.32 ? 'TENDENCIA' : 'POSIBILIDAD');
  const txt =
`SISTEMA · NAVEGAR LA INCERTIDUMBRE
SEED       ${seed}
ESTADO     ${estado}
COHERENCIA ${nf(coherence, 1, 2)}
EXCEPC.    ${eventCount}
SENSOR     ${pointerActive ? 'ACTIVO' : 'EN ESPERA'}`;
  text(txt, 14, 20);
  pop();
}
 
function draw() {
  updatePointerState();
  const coherence = coherenceNow();
  const sigmaAngle = lerp(HIGH_SIGMA, LOW_SIGMA, coherence);
 
  blendMode(BLEND);
  noStroke();
  fill(0, 0, 1, 14);
  rect(0, 0, width, height);
 
  drawRadarGrid();
 
  blendMode(ADD);
  drawSweep();
 
  for (let p of particles) {
    const base = fieldAngleAt(p.x, p.y);
    const sigma = localSigma(p.x, p.y, sigmaAngle);
    let angle = base + randomGaussian() * sigma;
    const boost = pointerSpeedBoost(p.x, p.y);
 
    let stepLen;
    const isException = random() < LEVY_PROB;
 
    if (isException) {
      const u = random(0.015, 1);
      stepLen = min(pow(u, -1 / LEVY_ALPHA) * LEVY_SCALE, max(width, height) * 0.35);
      angle = random(TWO_PI);
    } else {
      stepLen = p.speed * abs(randomGaussian(1, 0.22)) * boost;
    }
 
    p.x += cos(angle) * stepLen;
    p.y += sin(angle) * stepLen;
 
    if (p.x < 0) p.x += width;
    if (p.x > width) p.x -= width;
    if (p.y < 0) p.y += height;
    if (p.y > height) p.y -= height;
 
    if (isException) {
      echoes.push({ x: p.x, y: p.y, life: 260, maxLife: 260, reach: 160, strength: 0.9 });
      eventCount++;
    }
 
    const near = pointerActive ? constrain(1 - dist(p.x, p.y, mouseX, mouseY) / POINTER_RADIUS, 0, 1) : 0;
    const bri = lerp(lerp(50, 92, coherence * 0.4 + 0.3), 100, near);
    const sat = lerp(60, 12, near);
    const size = lerp(isException ? 4.5 : 2.2, isException ? 7 : 5, near);
    fill(130, sat, bri, near > 0.05 ? 85 : 60);
    circle(p.x, p.y, size);
  }
 
  if (pointerActive && millis() - lastPing > 260) {
    pings.push({ x: mouseX, y: mouseY, life: 55, maxLife: 55 });
    lastPing = millis();
  }
  for (let i = pings.length - 1; i >= 0; i--) {
    const pg = pings[i];
    const t = pg.life / pg.maxLife;
    noFill();
    stroke(130, 20, 100, 60 * t);
    strokeWeight(1.6);
    circle(pg.x, pg.y, (1 - t) * POINTER_RADIUS * 1.4);
    pg.life -= 1;
    if (pg.life <= 0) pings.splice(i, 1);
  }
 
  for (let i = echoes.length - 1; i >= 0; i--) {
    const e = echoes[i];
    const t = e.life / e.maxLife;
    noFill();
    stroke(130, 30, 95, 30 * t);
    strokeWeight(1);
    circle(e.x, e.y, e.reach * 0.55 * (1.4 - t));
    e.life -= 1;
    e.strength = 0.9 * (e.life / e.maxLife);
    if (e.life <= 0) echoes.splice(i, 1);
  }
 
  if (pointerActive) {
    blendMode(ADD);
    noStroke();
    fill(130, 15, 100, 30);
    circle(mouseX, mouseY, 46);
    blendMode(BLEND);
    noFill();
    stroke(130, 15, 100, 85);
    strokeWeight(1.4);
    circle(mouseX, mouseY, 26);
    line(mouseX - 20, mouseY, mouseX - 8, mouseY);
    line(mouseX + 8, mouseY, mouseX + 20, mouseY);
    line(mouseX, mouseY - 20, mouseX, mouseY - 8);
    line(mouseX, mouseY + 8, mouseX, mouseY + 20);
    drawingContext.setLineDash([3, 4]);
    stroke(130, 25, 95, 35);
    circle(mouseX, mouseY, POINTER_RADIUS * 1.5);
    drawingContext.setLineDash([]);
  }
 
  blendMode(BLEND);
  drawScanlines();
  drawHUD(coherence);
 
  sweepAngle += 0.012;
  zoff += 0.0016;
}
 
function touchMoved() {
  lastMoveTime = millis();
  return false;
}
 
```
</details>

---

## Trabajo final

**Enlace al prototipo:** [editor.p5js.org/mafora12/full/nHWkOUVUg](https://editor.p5js.org/mafora12/full/nHWkOUVUg)  


## Autoevalución  
| **Criterio**                                                                                                                                      | **Cumplió** | **Evidencia**                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ------------------------------------------------------------------------------------------------------------------------------------------------- | :---------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Encargo completo:** interpreto los cinco momentos dentro de un mismo sistema visual.                                                            |     ✅ Sí    | [Proceso – Paso 1: sistema base](#proceso--paso-1-sistema-base), donde se explica la interpretación de los cinco momentos. También en [¿Por qué cumple las condiciones?](#por-qué-cumple-las-condiciones), donde se justifica que todos los momentos ocurren dentro de un mismo sistema visual.                                                                                                                                                                      |
| **Simulación con intención:** utilizo al menos tres conceptos de la unidad para comunicar las ideas del encargo.                                  |     ✅ Sí    | [Intención conceptual #1 — Laboratorio molecular](#intención-conceptual-1--laboratorio-molecular), donde se presentan los conceptos utilizados, y [¿Por qué cumple las condiciones?](#por-qué-cumple-las-condiciones), donde se explica que el sistema integra cuatro conceptos de la unidad.                                                                                                                                                                        |
| **Interacción significativa:** la interacción modifica el comportamiento o las probabilidades del sistema, que también funciona sin intervención. |     ✅ Sí    | [Intención conceptual #1 — Laboratorio molecular](#intención-conceptual-1--laboratorio-molecular), donde se describe la interacción inicial; [Proceso – Paso 1: sistema base](#proceso--paso-1-sistema-base), donde se explica cómo la interacción modifica el sistema; y [Arreglo 1 — Interacción más notoria](#arreglo-1--interacción-más-notoria), donde se documentan las mejoras realizadas.                                                                    |
| **Prototipo funcional:** la experiencia puede ejecutarse y recorrerse completa sin errores que impidan comprenderla.                              |     ✅ Sí    | [Control de versiones](#control-de-versiones), donde se documenta el error encontrado, el diagnóstico y la solución; además del [Código — corregido y funcional](#código--corregido-y-funcional), que evidencia el funcionamiento del prototipo.                                                                                                                                                                                                                     |
| **Proceso documentado:** la bitácora evidencia avances, decisiones, dificultades, soluciones, uso de IA y enlace al prototipo.                    |     ✅ Sí    | [Experimentos y versiones intermedias](#experimentos-y-versiones-intermedias), [Intención conceptual #2 — Estación de sonar / localizador](#intención-conceptual-2--estación-de-sonar--localizador), [Proceso – Paso 2: piel de instrumento científico](#proceso--paso-2-piel-de-instrumento-científico), [Arreglo 1 — Interacción más notoria](#arreglo-1--interacción-más-notoria) y [Arreglo 2 — Mockup visual (submarino)](#arreglo-2--mockup-visual-submarino). |  
| **Nota:** 5                 |  

