# Actividad 05 — Reto de diseño: una contradicción en movimiento
### Unidad 2 · Simulación · UPB

---

## 1. Intención

> Quiero explorar la tensión entre **el orden** y **el ruido**, materializada como la oposición tipográfica entre **sans-serif** y **serif**.

El sans-serif es geométrico, modernista, hijo de la grilla editorial (Bauhaus/Swiss). El serif es caligráfico, ornamentado, con la marca histórica de la mano que lo trazó. Uno *es* el orden; el otro *es* el ruido.

**Cómo espero que se manifieste:** una población de partículas (sans) intenta alinearse a una retícula editorial invisible —columnas y líneas base—, formando bloques, ritmos y alineaciones que se leen como página, nunca como una letra dibujada. Una segunda población (serif) no siente la retícula, se mueve con una curva de ruido continuo (no aleatorio puro) y es atraída hacia las sans para invadir su formación; las sans, a su vez, son repelidas por las serif cuando estas se acercan. El resultado esperado: la retícula nunca termina de formarse ni de destruirse — vive en un ciclo continuo de composición y ruptura.

La contradicción no depende del color: está en que un tipo tiene una fuerza estructural hacia puntos fijos (grilla) y el otro no tiene ninguna, y en que la relación entre ambos es asimétrica (uno intenta organizar, el otro se niega a ser organizado).

---

## 2. Ficha breve del sistema

### Tipos de partículas
- **Sans (Orden):** representa la disciplina tipográfica/editorial.
- **Serif (Ruido):** representa la irregularidad caligráfica/ornamental.

Ambos son *categorías de comportamiento*, no literalmente "orden puro" y "caos puro": las sans también se cohesionan entre sí (matriz > 0), y las serif tienen su propia cohesión débil.

### Cantidad de partículas
- Sans: **130**
- Serif: **45**

La proporción ~3:1 hace que la retícula tenga masa suficiente para leerse como estructura antes de ser invadida; con menos sans, el sistema nunca forma nada reconocible.

### Matriz de atracción / repulsión / indiferencia
(fila = quién siente la fuerza, columna = hacia quién)

|              | → Orden | → Ruido |
|--------------|:-------:|:-------:|
| **Orden →**  |  0.30   | −0.50   |
| **Ruido →**  |  0.50   |  0.05   |

- **Orden→Orden = 0.30:** cohesión moderada; las sans se agrupan pero no tanto como para ignorar la retícula.
- **Orden→Ruido = −0.50:** las sans son repelidas por las serif cercanas — el ruido las saca de su lugar.
- **Ruido→Orden = 0.50:** las serif son atraídas hacia las sans — el ruido busca activamente invadir el orden.
- **Ruido→Ruido = 0.05:** cohesión casi nula entre serif; no forman bloque propio, quedan dispersas.

Esta es la **relación asimétrica** que pide la actividad: Orden→Ruido (−0.50) ≠ Ruido→Orden (0.50). El orden repele lo que el ruido persigue.

### Intensidad y alcance de cada relación
- Alcance (rMax) — Sans: **110 px** · Serif: **140 px**
- Radio de repulsión dura (rClose) — Sans: **7 px** · Serif: **15 px**

Las serif "ven" más lejos (140 px) que las sans (110 px): el ruido busca al orden desde más lejos de lo que el orden puede anticiparlo.

### Distancias de interacción
Cada partícula solo siente a las demás dentro de su `rMax`; fuera de ese radio, fuerza = 0. Dentro de `rClose`, la fuerza es siempre repulsiva (evita superposición física), independiente de la matriz.

### Fricción y velocidad máxima
- Fricción: **0.10** (10% de la velocidad se disipa cada frame)
- Velocidad máxima: **3.2 px/frame**

Valores moderados: suficiente fricción para que el sistema no se vuelva caótico de inmediato, suficiente velocidad máxima para que la invasión de las serif se sienta como un movimiento, no un salto.

### Distribución inicial
Todas las partículas nacen en posiciones aleatorias dentro del área de la retícula (semilla nueva en cada ejecución vía el botón "Nueva semilla"). Esto garantiza que ninguna manifestación repita la misma composición inicial.

### Parámetros constantes y variables
**Invariantes** (lo que da identidad al sistema, no cambia entre manifestaciones):
- Dos tipos de partícula, con la matriz asimétrica descrita arriba.
- Las sans están **confinadas** al área de la retícula; las serif pueden **salir** de ella y recorrer toda la pantalla — el orden tiene límites, el ruido no.
- El movimiento de las serif usa **Perlin noise** (curvas continuas), no aleatoriedad pura frame a frame.

**Variables** (cambian entre ejecuciones/manifestaciones):
- Semilla inicial (posición de cada partícula).
- Cantidad de sans y serif.
- Número de columnas y ritmo de línea base de la retícula.
- Fuerza de retícula (qué tanto se aferran las sans a su punto).
- Intensidad del ruido (amplitud del Perlin noise en las serif).

### Apariencia e interacción
- Sans: **cuadrados** de 5.5 px — geometría que evoca los trazos rectos del sans-serif.
- Serif: **círculos** de 5.5 px con un pequeño trazo perpendicular en la base ("pie"), evocando el remate de una serifa.
- Interacción: panel lateral con sliders para todos los parámetros anteriores, botón de nueva semilla, tecla **G** para mostrar/ocultar la retícula, y botón para ocultar el panel y ver el sistema limpio.

---

## 3. Justificación de decisiones

> Seleccioné **una fuerza de atracción hacia puntos fijos de una retícula (columnas + líneas base)** porque quiero hacer perceptible **la disciplina editorial del sans-serif como sistema, no como forma**. Espero que produzca **columnas, bloques y ritmos reconocibles como página, sin dibujar ninguna letra**.

> Seleccioné **Perlin noise en vez de `random()` para las serif** porque quiero hacer perceptible **el gesto caligráfico continuo del serif, no un temblor mecánico**. Espero que produzca **un movimiento fluido y orgánico, más parecido a un trazo de tinta que a ruido digital**.

> Seleccioné **una matriz asimétrica (Orden→Ruido negativo, Ruido→Orden positivo)** porque quiero hacer perceptible **que el deseo de invadir y el deseo de rechazar no son la misma fuerza**. Espero que produzca **un ciclo de invasión-expulsión que nunca se resuelve en equilibrio**.

> Seleccioné **confinar solo a las sans dentro de la retícula, dejando libres a las serif** porque quiero hacer perceptible **que el orden tiene límites que él mismo se impone, y el ruido no reconoce ningún límite**. Espero que produzca **una sensación de que el ruido siempre tiene "más espacio" que el orden, y puede desaparecer y regresar**.

---

## 4. Verificación de condiciones

| Condición pedida | Cómo se cumple |
|---|---|
| Posición, velocidad y aceleración | Cada partícula tiene `x, y, vx, vy`; la fuerza neta se acumula sobre `vx, vy` cada frame (aceleración implícita) |
| Varias poblaciones de partículas | Sans y Serif, con reglas propias |
| Interacciones dependientes de la distancia | Función de fuerza triangular entre `rClose` y `rMax` |
| Relaciones de atracción, repulsión o indiferencia | Matriz 2×2 con valores positivos, negativos y cercanos a cero |
| Al menos una relación asimétrica | Orden→Ruido (−0.50) ≠ Ruido→Orden (0.50) |
| Variabilidad entre ejecuciones | Semilla aleatoria en cada "Nueva semilla" |
| Comportamientos emergentes, no trayectorias predefinidas | Ninguna posición final está programada; solo fuerzas |
| Identidad reconocible entre resultados | Siempre hay columnas/bloques parciales + invasión roja, en cualquier semilla |

---

## 5. Registro de pruebas
 
### 5.0 Resumen
 
| # | Ajuste probado | Hallazgo | Decisión |
|---|---|---|---|
| 1 | Tensión inicial: "cercanía vs. miedo a la intimidad" (dos poblaciones con radios de pánico distintos) | Funcionaba, pero no conectaba con tipografía/diseño gráfico, que es mi área | **Descartado** — se buscó una tensión más cercana a diseño gráfico |
| 2 | Tensión "orden vs. ruido" mapeada a sans-serif vs. serif, con fuerza de grilla uniforme (puntos de una cuadrícula regular) | Se leía como papel cuadriculado genérico, no como sistema tipográfico | Se mantuvo el concepto, se refinó la regla de alineación |
| 3 | Sans atraídas a puntos fijos sobre el trazo de una letra "A" | Formaba la letra, pero el espectador leía "es una A", no "es un sistema de relaciones" — contradice el espíritu de diseño generativo de la unidad | **Descartado** — reemplazado por retícula editorial (columnas + líneas base) |
| 4 | Refinamiento del sistema de retícula editorial: guía visual permanente → oculta + tecla G · `random()` → Perlin noise · confinamiento de ambas poblaciones → solo sans · Orden→Ruido positivo → negativo · fuerza de retícula muy alta → rango 0.03–0.06 | Ver detalle de los cinco ajustes en 5.1, Prueba 4 | Varias decisiones, ver detalle |

### 5.1 Evidencia por prueba

Cada bloque de abajo corresponde a la fila de la misma numeración en la tabla de resumen. Pega el fragmento de código y la imagen/captura de cada prueba en su bloque.

---

#### Prueba 1 — Cercanía vs. miedo a la intimidad

**Ajuste probado:** dos poblaciones con radios de pánico y alcances distintos, simulando necesidad de cercanía vs. evasión.
**Hallazgo:** funcionaba, pero no conectaba con tipografía/diseño gráfico.
**Decisión:** **Descartado** — se buscó una tensión más cercana a diseño gráfico.

**Código:**
```js
const sketch = (p) => {
  const TYPE_A = 0; // Buscador
  const TYPE_B = 1; // Evasivo

  let particles = [];

  let params = {
    countA: 45,
    countB: 45,
    matrix: [
      [0.30, 0.90],
      [0.15, -0.25]
    ],
    rClose: [6, 22],   // radio de "pánico" / repulsión dura, por tipo emisor
    rMax:   [150, 90], // alcance de percepción, por tipo emisor
    friction: 0.08,
    forceScale: 420,
    maxSpeed: 3.5
  };

  function makeParticle(type) {
    return {
      type: type,
      x: p.random(p.width),
      y: p.random(p.height),
      vx: 0,
      vy: 0
    };
  }

  function reseed() {
    particles = [];
    for (let i = 0; i < params.countA; i++) particles.push(makeParticle(TYPE_A));
    for (let i = 0; i < params.countB; i++) particles.push(makeParticle(TYPE_B));
  }

  function syncCounts() {
    const currentA = particles.filter(q => q.type === TYPE_A).length;
    const currentB = particles.filter(q => q.type === TYPE_B).length;
    if (currentA < params.countA) for (let i = 0; i < params.countA - currentA; i++) particles.push(makeParticle(TYPE_A));
    if (currentB < params.countB) for (let i = 0; i < params.countB - currentB; i++) particles.push(makeParticle(TYPE_B));
    if (currentA > params.countA || currentB > params.countB) {
      let a = 0, b = 0;
      particles = particles.filter(q => {
        if (q.type === TYPE_A) { a++; return a <= params.countA; }
        else { b++; return b <= params.countB; }
      });
    }
  }

  p.setup = () => {
    const holder = document.getElementById('sketch-holder');
    const c = p.createCanvas(window.innerWidth, window.innerHeight);
    c.parent(holder);
    p.colorMode(p.RGB);
    reseed();
  };

  p.windowResized = () => {
    p.resizeCanvas(window.innerWidth, window.innerHeight);
  };

  p.draw = () => {
    p.background(18, 15, 14, 60);

    const w = p.width, h = p.height;

    for (let i = 0; i < particles.length; i++) {
      const a = particles[i];
      let fx = 0, fy = 0;
      const rMax = params.rMax[a.type];
      const rClose = params.rClose[a.type];

      for (let j = 0; j < particles.length; j++) {
        if (i === j) continue;
        const b = particles[j];

        let dx = b.x - a.x;
        let dy = b.y - a.y;
        if (dx > w / 2) dx -= w; else if (dx < -w / 2) dx += w;
        if (dy > h / 2) dy -= h; else if (dy < -h / 2) dy += h;

        const r = Math.sqrt(dx * dx + dy * dy);
        if (r < 0.01 || r > rMax) continue;

        let f;
        if (r < rClose) {
          f = (r / rClose) - 1; // repulsión dura, negativa
        } else {
          const beta = params.matrix[a.type][b.type];
          f = beta * (1 - Math.abs(2 * r - rClose - rMax) / (rMax - rClose));
        }

        const inv = f / r;
        fx += dx * inv;
        fy += dy * inv;
      }

      a.vx += fx * params.forceScale * 0.0001;
      a.vy += fy * params.forceScale * 0.0001;

      a.vx *= (1 - params.friction);
      a.vy *= (1 - params.friction);

      const speed = Math.sqrt(a.vx * a.vx + a.vy * a.vy);
      if (speed > params.maxSpeed) {
        a.vx = (a.vx / speed) * params.maxSpeed;
        a.vy = (a.vy / speed) * params.maxSpeed;
      }

      a.x += a.vx;
      a.y += a.vy;

      if (a.x < 0) a.x += w; if (a.x > w) a.x -= w;
      if (a.y < 0) a.y += h; if (a.y > h) a.y -= h;
    }

    p.noStroke();
    for (const a of particles) {
      if (a.type === TYPE_A) {
        p.fill(232, 103, 75, 235);
        p.circle(a.x, a.y, 6);
      } else {
        p.fill(79, 157, 166, 235);
        p.circle(a.x, a.y, 6);
      }
    }
  };

  // ---- UI wiring ----
  function bindRange(id, decimals, onChange) {
    const el = document.getElementById(id);
    const out = document.getElementById(id + '-out');
    const update = () => {
      const v = parseFloat(el.value);
      out.textContent = decimals === 0 ? Math.round(v) : v.toFixed(decimals);
      onChange(v);
    };
    el.addEventListener('input', update);
    update();
  }

  window.addEventListener('DOMContentLoaded', () => {
    bindRange('countA', 0, v => { params.countA = v; syncCounts(); });
    bindRange('countB', 0, v => { params.countB = v; syncCounts(); });

    bindRange('mAA', 2, v => { params.matrix[0][0] = v; });
    bindRange('mAB', 2, v => { params.matrix[0][1] = v; });
    bindRange('mBA', 2, v => { params.matrix[1][0] = v; });
    bindRange('mBB', 2, v => { params.matrix[1][1] = v; });

    bindRange('rCloseB', 0, v => { params.rClose[1] = v; });
    bindRange('rMaxA', 0, v => { params.rMax[0] = v; });
    bindRange('rMaxB', 0, v => { params.rMax[1] = v; });

    bindRange('friction', 2, v => { params.friction = v; });
    bindRange('forceScale', 0, v => { params.forceScale = v; });
    bindRange('maxSpeed', 1, v => { params.maxSpeed = v; });

    document.getElementById('reseed').addEventListener('click', reseed);
  });
};

new p5(sketch);
```

**Captura:**
![Prueba 1 — cercanía vs. miedo a la intimidad](<img width="2235" height="1123" alt="image" src="https://github.com/user-attachments/assets/04d3ead8-511c-4e5e-87ad-437e35c48c3c" />
)

---

#### Prueba 2 — Orden vs. ruido con grilla uniforme

**Ajuste probado:** sans atraídas a los puntos de una cuadrícula regular (grid uniforme, sin distinguir columnas de líneas base).
**Hallazgo:** se leía como papel cuadriculado genérico, no como sistema tipográfico.
**Decisión:** se mantuvo el concepto; se refinó la regla de alineación.

**Código:**
```js
const sketch = (p) => {
  const SANS = 0;  // orden
  const SERIF = 1; // ruido

  let particles = [];

  let params = {
    countS: 55,
    countR: 55,
    matrix: [
      [0.50, 0.35],
      [-0.35, 0.08]
    ],
    rClose: [8, 16],
    rMax:   [130, 95],
    friction: 0.09,
    forceScale: 420,
    maxSpeed: 3.2,
    gridSpacing: 42,
    gridStrength: 0.06,
    noiseStrength: 0.35
  };

  function makeParticle(type) {
    return { type, x: p.random(p.width), y: p.random(p.height), vx: 0, vy: 0, foot: p.random(-1, 1) };
  }

  function reseed() {
    particles = [];
    for (let i = 0; i < params.countS; i++) particles.push(makeParticle(SANS));
    for (let i = 0; i < params.countR; i++) particles.push(makeParticle(SERIF));
  }

  function syncCounts() {
    const cS = particles.filter(q => q.type === SANS).length;
    const cR = particles.filter(q => q.type === SERIF).length;
    if (cS < params.countS) for (let i = 0; i < params.countS - cS; i++) particles.push(makeParticle(SANS));
    if (cR < params.countR) for (let i = 0; i < params.countR - cR; i++) particles.push(makeParticle(SERIF));
    if (cS > params.countS || cR > params.countR) {
      let a = 0, b = 0;
      particles = particles.filter(q => {
        if (q.type === SANS) { a++; return a <= params.countS; }
        else { b++; return b <= params.countR; }
      });
    }
  }

  p.setup = () => {
    const holder = document.getElementById('sketch-holder');
    const c = p.createCanvas(window.innerWidth, window.innerHeight);
    c.parent(holder);
    reseed();
  };

  p.windowResized = () => { p.resizeCanvas(window.innerWidth, window.innerHeight); };

  function drawGrid() {
    const g = params.gridSpacing;
    p.stroke(255, 255, 255, 10);
    p.strokeWeight(1);
    for (let x = 0; x < p.width; x += g) p.line(x, 0, x, p.height);
    for (let y = 0; y < p.height; y += g) p.line(0, y, p.width, y);
  }

  p.draw = () => {
    p.background(18, 17, 16, 60);
    drawGrid();

    const w = p.width, h = p.height;
    const g = params.gridSpacing;

    for (let i = 0; i < particles.length; i++) {
      const a = particles[i];
      let fx = 0, fy = 0;
      const rMax = params.rMax[a.type];
      const rClose = params.rClose[a.type];

      for (let j = 0; j < particles.length; j++) {
        if (i === j) continue;
        const b = particles[j];
        let dx = b.x - a.x, dy = b.y - a.y;
        if (dx > w / 2) dx -= w; else if (dx < -w / 2) dx += w;
        if (dy > h / 2) dy -= h; else if (dy < -h / 2) dy += h;
        const r = Math.sqrt(dx * dx + dy * dy);
        if (r < 0.01 || r > rMax) continue;

        let f;
        if (r < rClose) {
          f = (r / rClose) - 1;
        } else {
          const beta = params.matrix[a.type][b.type];
          f = beta * (1 - Math.abs(2 * r - rClose - rMax) / (rMax - rClose));
        }
        const inv = f / r;
        fx += dx * inv;
        fy += dy * inv;
      }

      if (a.type === SANS && params.gridStrength > 0) {
        const gx = Math.round(a.x / g) * g;
        const gy = Math.round(a.y / g) * g;
        fx += (gx - a.x) * params.gridStrength;
        fy += (gy - a.y) * params.gridStrength;
      }

      a.vx += fx * params.forceScale * 0.0001;
      a.vy += fy * params.forceScale * 0.0001;

      if (a.type === SERIF && params.noiseStrength > 0) {
        a.vx += p.random(-1, 1) * params.noiseStrength;
        a.vy += p.random(-1, 1) * params.noiseStrength;
      }

      a.vx *= (1 - params.friction);
      a.vy *= (1 - params.friction);

      const speed = Math.sqrt(a.vx * a.vx + a.vy * a.vy);
      if (speed > params.maxSpeed) {
        a.vx = (a.vx / speed) * params.maxSpeed;
        a.vy = (a.vy / speed) * params.maxSpeed;
      }

      a.x += a.vx; a.y += a.vy;
      if (a.x < 0) a.x += w; if (a.x > w) a.x -= w;
      if (a.y < 0) a.y += h; if (a.y > h) a.y -= h;
    }

    p.noStroke();
    for (const a of particles) {
      if (a.type === SANS) {
        p.fill(79, 157, 166, 235);
        p.push();
        p.translate(a.x, a.y);
        p.rectMode(p.CENTER);
        p.rect(0, 0, 6, 6);
        p.pop();
      } else {
        p.fill(232, 103, 75, 235);
        p.circle(a.x, a.y, 5.5);
        p.stroke(232, 103, 75, 235);
        p.strokeWeight(1.2);
        p.line(a.x - 3, a.y + 3 + a.foot, a.x + 3, a.y + 3 + a.foot);
        p.noStroke();
      }
    }
  };

  function bindRange(id, decimals, onChange) {
    const el = document.getElementById(id);
    const out = document.getElementById(id + '-out');
    const update = () => {
      const v = parseFloat(el.value);
      out.textContent = decimals === 0 ? Math.round(v) : v.toFixed(decimals);
      onChange(v);
    };
    el.addEventListener('input', update);
    update();
  }

  window.addEventListener('DOMContentLoaded', () => {
    bindRange('noiseStrength', 2, v => { params.noiseStrength = v; });
    bindRange('gridStrength', 3, v => { params.gridStrength = v; });

    bindRange('countS', 0, v => { params.countS = v; syncCounts(); });
    bindRange('countR', 0, v => { params.countR = v; syncCounts(); });

    bindRange('mSS', 2, v => { params.matrix[0][0] = v; });
    bindRange('mSR', 2, v => { params.matrix[0][1] = v; });
    bindRange('mRS', 2, v => { params.matrix[1][0] = v; });
    bindRange('mRR', 2, v => { params.matrix[1][1] = v; });

    bindRange('gridSpacing', 0, v => { params.gridSpacing = v; });
    bindRange('rMaxS', 0, v => { params.rMax[0] = v; });
    bindRange('rMaxR', 0, v => { params.rMax[1] = v; });

    bindRange('friction', 2, v => { params.friction = v; });
    bindRange('maxSpeed', 1, v => { params.maxSpeed = v; });

    document.getElementById('reseed').addEventListener('click', reseed);
  });
};

new p5(sketch);
```

**Captura:**
![Prueba 2 — grilla uniforme](<img width="1757" height="1150" alt="image" src="https://github.com/user-attachments/assets/3b78f447-cffa-480f-937e-6235209d720b" />
)

---

#### Prueba 3 — Formación de la letra "A"

**Ajuste probado:** sans atraídas a puntos fijos sobre el trazo de una letra "A".
**Hallazgo:** formaba la letra, pero el espectador leía "es una A", no "es un sistema de relaciones" — contradice el espíritu de diseño generativo de la unidad.
**Decisión:** **Descartado** — reemplazado por retícula editorial (columnas + líneas base).

**Código:**
```js
const sketch = (p) => {
  const SANS = 0;  // orden
  const SERIF = 1; // ruido
  const LETTER_N = 160;

  let particles = [];
  let letterNorm = [];   // puntos normalizados (0..1) sobre la letra A
  let letterPx = [];     // puntos mapeados a pixeles, recalculado cada frame

  let params = {
    countS: 130,
    countR: 45,
    matrix: [
      [0.30, -0.50],
      [0.50, 0.05]
    ],
    rClose: [7, 15],
    rMax:   [110, 140],
    friction: 0.10,
    forceScale: 420,
    maxSpeed: 3.2,
    letterStrength: 0.045,
    noiseStrength: 0.25
  };

  function buildLetterNorm(n) {
    const apex = [0.5, 0.04];
    const bl = [0.06, 0.96];
    const br = [0.94, 0.96];
    const t = 0.56;
    const cl = [apex[0] + (bl[0] - apex[0]) * t, apex[1] + (bl[1] - apex[1]) * t];
    const cr = [apex[0] + (br[0] - apex[0]) * t, apex[1] + (br[1] - apex[1]) * t];
    const segs = [
      { a: apex, b: bl },
      { a: apex, b: br },
      { a: cl, b: cr }
    ];
    const lens = segs.map(s => Math.hypot(s.b[0] - s.a[0], s.b[1] - s.a[1]));
    const total = lens.reduce((x, y) => x + y, 0);
    let pts = [];
    segs.forEach((s, idx) => {
      const count = Math.max(2, Math.round(n * lens[idx] / total));
      for (let i = 0; i < count; i++) {
        const tt = i / (count - 1 || 1);
        pts.push([s.a[0] + (s.b[0] - s.a[0]) * tt, s.a[1] + (s.b[1] - s.a[1]) * tt]);
      }
    });
    return { pts, segs };
  }

  let letterSegs = [];

  function letterBox() {
    const w = p.width, h = p.height;
    const boxW = Math.min(w, h) * 0.5;
    const boxH = boxW * 1.25;
    return { x0: (w - boxW) / 2, y0: (h - boxH) / 2 - h * 0.02, boxW, boxH };
  }

  function mapNormPoint(np) {
    const { x0, y0, boxW, boxH } = letterBox();
    return { x: x0 + np[0] * boxW, y: y0 + np[1] * boxH };
  }

  function refreshLetterPx() {
    letterPx = letterNorm.map(mapNormPoint);
  }

  function makeParticle(type) {
    return {
      type,
      x: p.random(p.width),
      y: p.random(p.height),
      vx: 0, vy: 0,
      foot: p.random(-1, 1),
      li: type === SANS ? Math.floor(p.random(LETTER_N)) : -1
    };
  }

  function reseed() {
    particles = [];
    for (let i = 0; i < params.countS; i++) particles.push(makeParticle(SANS));
    for (let i = 0; i < params.countR; i++) particles.push(makeParticle(SERIF));
  }

  function syncCounts() {
    const cS = particles.filter(q => q.type === SANS).length;
    const cR = particles.filter(q => q.type === SERIF).length;
    if (cS < params.countS) for (let i = 0; i < params.countS - cS; i++) particles.push(makeParticle(SANS));
    if (cR < params.countR) for (let i = 0; i < params.countR - cR; i++) particles.push(makeParticle(SERIF));
    if (cS > params.countS || cR > params.countR) {
      let a = 0, b = 0;
      particles = particles.filter(q => {
        if (q.type === SANS) { a++; return a <= params.countS; }
        else { b++; return b <= params.countR; }
      });
    }
  }

  p.setup = () => {
    const holder = document.getElementById('sketch-holder');
    const c = p.createCanvas(window.innerWidth, window.innerHeight);
    c.parent(holder);
    const built = buildLetterNorm(LETTER_N);
    letterNorm = built.pts;
    letterSegs = built.segs;
    refreshLetterPx();
    reseed();
  };

  p.windowResized = () => {
    p.resizeCanvas(window.innerWidth, window.innerHeight);
    refreshLetterPx();
  };

  p.draw = () => {
    p.background(18, 17, 16, 60);
    refreshLetterPx();

    const w = p.width, h = p.height;

    for (let i = 0; i < particles.length; i++) {
      const a = particles[i];
      let fx = 0, fy = 0;
      const rMax = params.rMax[a.type];
      const rClose = params.rClose[a.type];

      for (let j = 0; j < particles.length; j++) {
        if (i === j) continue;
        const b = particles[j];
        let dx = b.x - a.x, dy = b.y - a.y;
        if (dx > w / 2) dx -= w; else if (dx < -w / 2) dx += w;
        if (dy > h / 2) dy -= h; else if (dy < -h / 2) dy += h;
        const r = Math.sqrt(dx * dx + dy * dy);
        if (r < 0.01 || r > rMax) continue;

        let f;
        if (r < rClose) {
          f = (r / rClose) - 1;
        } else {
          const beta = params.matrix[a.type][b.type];
          f = beta * (1 - Math.abs(2 * r - rClose - rMax) / (rMax - rClose));
        }
        const inv = f / r;
        fx += dx * inv;
        fy += dy * inv;
      }

      if (a.type === SANS && params.letterStrength > 0 && letterPx.length) {
        const target = letterPx[a.li % letterPx.length];
        fx += (target.x - a.x) * params.letterStrength;
        fy += (target.y - a.y) * params.letterStrength;
      }

      a.vx += fx * params.forceScale * 0.0001;
      a.vy += fy * params.forceScale * 0.0001;

      if (a.type === SERIF && params.noiseStrength > 0) {
        a.vx += p.random(-1, 1) * params.noiseStrength;
        a.vy += p.random(-1, 1) * params.noiseStrength;
      }

      a.vx *= (1 - params.friction);
      a.vy *= (1 - params.friction);

      const speed = Math.sqrt(a.vx * a.vx + a.vy * a.vy);
      if (speed > params.maxSpeed) {
        a.vx = (a.vx / speed) * params.maxSpeed;
        a.vy = (a.vy / speed) * params.maxSpeed;
      }

      a.x += a.vx; a.y += a.vy;
      if (a.x < 0) a.x += w; if (a.x > w) a.x -= w;
      if (a.y < 0) a.y += h; if (a.y > h) a.y -= h;
    }

    p.noStroke();
    for (const a of particles) {
      if (a.type === SANS) {
        p.fill(79, 157, 166, 235);
        p.push();
        p.translate(a.x, a.y);
        p.rectMode(p.CENTER);
        p.rect(0, 0, 5.5, 5.5);
        p.pop();
      } else {
        p.fill(232, 103, 75, 235);
        p.circle(a.x, a.y, 5.5);
        p.stroke(232, 103, 75, 235);
        p.strokeWeight(1.2);
        p.line(a.x - 3, a.y + 3 + a.foot, a.x + 3, a.y + 3 + a.foot);
        p.noStroke();
      }
    }
  };

  function bindRange(id, decimals, onChange) {
    const el = document.getElementById(id);
    const out = document.getElementById(id + '-out');
    const update = () => {
      const v = parseFloat(el.value);
      out.textContent = decimals === 0 ? Math.round(v) : v.toFixed(decimals);
      onChange(v);
    };
    el.addEventListener('input', update);
    update();
  }

  window.addEventListener('DOMContentLoaded', () => {
    bindRange('letterStrength', 3, v => { params.letterStrength = v; });
    bindRange('noiseStrength', 2, v => { params.noiseStrength = v; });

    bindRange('countS', 0, v => { params.countS = v; syncCounts(); });
    bindRange('countR', 0, v => { params.countR = v; syncCounts(); });

    bindRange('mSS', 2, v => { params.matrix[0][0] = v; });
    bindRange('mSR', 2, v => { params.matrix[0][1] = v; });
    bindRange('mRS', 2, v => { params.matrix[1][0] = v; });
    bindRange('mRR', 2, v => { params.matrix[1][1] = v; });

    bindRange('rMaxS', 0, v => { params.rMax[0] = v; });
    bindRange('rMaxR', 0, v => { params.rMax[1] = v; });

    bindRange('friction', 2, v => { params.friction = v; });
    bindRange('maxSpeed', 1, v => { params.maxSpeed = v; });

    document.getElementById('reseed').addEventListener('click', reseed);
  });
};

new p5(sketch);
```

**Captura:**
![Prueba 3 — formación de la letra A](<img width="1876" height="1153" alt="image" src="https://github.com/user-attachments/assets/af9f4a2f-3fa0-4c95-9801-700fbd942afb" />
)

---
#### Prueba 4 — Refinamiento del sistema de retícula editorial
 
Todos los ajustes de esta prueba se hicieron sobre la misma versión final del sistema (retícula editorial + Perlin noise), por eso van agrupados en una sola evidencia.
 
| Ajuste probado | Hallazgo | Decisión |
|---|---|---|
| Retícula con guía visual permanente (líneas de fondo siempre visibles) | Delataba demasiado el "andamio"; se perdía la sensación de comportamiento emergente | Se ocultó la guía por defecto; se agregó la tecla **G** para mostrarla solo cuando se necesita explicar el sistema |
| Ruido de las serif con `random()` puro cada frame | Se veía como temblor nervioso, mecánico — no como gesto caligráfico | **Descartado** — reemplazado por Perlin noise con semilla propia por partícula |
| Ambas poblaciones (sans y serif) confinadas al área de la retícula | El ruido se sentía "domesticado", contradecía la idea de que el ruido no respeta límites | Se dejó a las sans confinadas y se liberó a las serif para que recorran toda la pantalla |
| Orden→Ruido positivo (el orden también atrae al ruido) | El sistema colapsaba en una sola mancha mixta; se perdía la lectura de "invasión" | **Descartado** — se cambió a negativo (el orden repele al ruido que lo invade) |
| Fuerza de retícula muy alta (>0.1) | La retícula quedaba perfecta y estática, sin tensión — mal ejemplo de "diseñado" sin "emergente" | Se fijó un rango recomendado (0.03–0.06) donde la retícula se nota pero nunca se completa |
 
**Código:**
```js

p5.disableFriendlyErrors = true;


const SANS = 0;  
const SERIF = 1; 


let params = {
  
  countS: 130,
  countR: 45,

 
  matrix: [
    [0.30, -0.50],
    [0.50, 0.05]
  ],

 
  rClose: [7, 15],    
  rMax:   [110, 140], 


  friction: 0.10,
  forceScale: 420,
  maxSpeed: 3.2,


  gridStrength: 0.045,
  numColumns: 6,
  baseline: 26,


  noiseStrength: 0.25
};


const NOISE_TIME_SCALE = 0.006;


let particles = [];
let showGrid = false; 


const MARGIN_X_FRAC = 0.08;
const MARGIN_Y_FRAC = 0.1;

function getStageBounds() {
  const x0 = width * MARGIN_X_FRAC;
  const y0 = height * MARGIN_Y_FRAC;
  const x1 = width - x0;
  const y1 = height - y0;
  return { x0, y0, x1, y1, w: x1 - x0, h: y1 - y0 };
}


function makeParticle(type) {
  const b = getStageBounds();
  const particle = {
    type,
    x: random(b.x0, b.x1),
    y: random(b.y0, b.y1),
    vx: 0,
    vy: 0,
    foot: random(-1, 1) 
  };

  if (type === SERIF) {

    particle.nOffsetX = random(1000);
    particle.nOffsetY = random(5000, 6000);
  }

  return particle;
}

function reseed() {
  particles = [];
  for (let i = 0; i < params.countS; i++) particles.push(makeParticle(SANS));
  for (let i = 0; i < params.countR; i++) particles.push(makeParticle(SERIF));
}

function syncCounts() {
  let cS = 0, cR = 0;
  for (let i = 0; i < particles.length; i++) {
    if (particles[i].type === SANS) cS++; else cR++;
  }

  if (cS < params.countS) {
    for (let i = 0; i < params.countS - cS; i++) particles.push(makeParticle(SANS));
  }
  if (cR < params.countR) {
    for (let i = 0; i < params.countR - cR; i++) particles.push(makeParticle(SERIF));
  }

  if (cS > params.countS || cR > params.countR) {
    const kept = [];
    let a = 0, b = 0;
    for (let i = 0; i < particles.length; i++) {
      const q = particles[i];
      if (q.type === SANS) {
        a++;
        if (a <= params.countS) kept.push(q);
      } else {
        b++;
        if (b <= params.countR) kept.push(q);
      }
    }
    particles = kept;
  }
}


function setup() {
  const c = createCanvas(window.innerWidth, window.innerHeight);
  c.parent('sketch-holder');
  reseed();
  setupControls();
  setupPanelToggle();
}

function windowResized() {
  resizeCanvas(window.innerWidth, window.innerHeight);
}

function keyPressed() {
  if (key === 'g' || key === 'G') {
    showGrid = !showGrid;
  }
}



function gridTarget(x, y) {
  const b = getStageBounds();
  const colW = b.w / params.numColumns;
  let colIdx = Math.round((x - b.x0 - colW / 2) / colW);
  colIdx = constrain(colIdx, 0, params.numColumns - 1);
  const targetX = b.x0 + colW * colIdx + colW / 2;

  const rows = Math.max(1, Math.round(b.h / params.baseline));
  let rowIdx = Math.round((y - b.y0) / params.baseline);
  rowIdx = constrain(rowIdx, 0, rows);
  const targetY = b.y0 + rowIdx * params.baseline;

  return { x: targetX, y: targetY };
}


function drawGrid() {
  const b = getStageBounds();
  const colW = b.w / params.numColumns;

  stroke(79, 157, 166, 45);
  strokeWeight(1);

  for (let i = 0; i <= params.numColumns; i++) {
    const x = b.x0 + colW * i;
    line(x, b.y0, x, b.y1);
  }

  const rows = Math.max(1, Math.round(b.h / params.baseline));
  for (let i = 0; i <= rows; i++) {
    const y = b.y0 + params.baseline * i;
    line(b.x0, y, b.x1, y);
  }

  stroke(79, 157, 166, 90);
  noFill();
  rect(b.x0, b.y0, b.w, b.h);
}


function draw() {
  if (particles.length === 0 && (params.countS > 0 || params.countR > 0)) {
    reseed();
  }

  background(18, 17, 16, 60);

  const bounds = getStageBounds();
  if (showGrid) drawGrid();

  const w = width, h = height;
  const noiseT = frameCount * NOISE_TIME_SCALE;

  for (let i = 0; i < particles.length; i++) {
    const a = particles[i];
    let fx = 0, fy = 0;
    const rMax = params.rMax[a.type];
    const rClose = params.rClose[a.type];

    for (let j = 0; j < particles.length; j++) {
      if (i === j) continue;
      const b = particles[j];
      let dx = b.x - a.x, dy = b.y - a.y;
      if (dx > w / 2) dx -= w; else if (dx < -w / 2) dx += w;
      if (dy > h / 2) dy -= h; else if (dy < -h / 2) dy += h;
      const r = Math.sqrt(dx * dx + dy * dy);
      if (r < 0.01 || r > rMax) continue;

      let f;
      if (r < rClose) {
        f = (r / rClose) - 1;
      } else {
        const beta = params.matrix[a.type][b.type];
        f = beta * (1 - Math.abs(2 * r - rClose - rMax) / (rMax - rClose));
      }
      const inv = f / r;
      fx += dx * inv;
      fy += dy * inv;
    }

    if (a.type === SANS && params.gridStrength > 0) {
      const target = gridTarget(a.x, a.y);
      fx += (target.x - a.x) * params.gridStrength;
      fy += (target.y - a.y) * params.gridStrength;
    }

    a.vx += fx * params.forceScale * 0.0001;
    a.vy += fy * params.forceScale * 0.0001;

    if (a.type === SERIF && params.noiseStrength > 0) {
      // Perlin noise en vez de random(): cada partícula serif recorre
      // su propia curva suave, como un trazo caligráfico, en lugar
      // de temblar al azar frame a frame (mucho más cercano a Max Cooper).
      const nx = (noise(a.nOffsetX + noiseT) - 0.5) * 2;
      const ny = (noise(a.nOffsetY + noiseT) - 0.5) * 2;
      a.vx += nx * params.noiseStrength;
      a.vy += ny * params.noiseStrength;
    }

    a.vx *= (1 - params.friction);
    a.vy *= (1 - params.friction);

    const speed = Math.sqrt(a.vx * a.vx + a.vy * a.vy);
    if (speed > params.maxSpeed) {
      a.vx = (a.vx / speed) * params.maxSpeed;
      a.vy = (a.vy / speed) * params.maxSpeed;
    }

    a.x += a.vx; a.y += a.vy;

    if (a.type === SANS) {
      if (a.x < bounds.x0) a.x += bounds.w; if (a.x > bounds.x1) a.x -= bounds.w;
      if (a.y < bounds.y0) a.y += bounds.h; if (a.y > bounds.y1) a.y -= bounds.h;
    } else {
      if (a.x < 0) a.x += w; if (a.x > w) a.x -= w;
      if (a.y < 0) a.y += h; if (a.y > h) a.y -= h;
    }
  }

  noStroke();
  for (const a of particles) {
    if (a.type === SANS) {
      fill(79, 157, 166, 235);
      push();
      translate(a.x, a.y);
      rectMode(CENTER);
      rect(0, 0, 5.5, 5.5);
      pop();
    } else {
      fill(232, 103, 75, 235);
      circle(a.x, a.y, 5.5);
      stroke(232, 103, 75, 235);
      strokeWeight(1.2);
      line(a.x - 3, a.y + 3 + a.foot, a.x + 3, a.y + 3 + a.foot);
      noStroke();
    }
  }
}

function bindRange(id, decimals, onChange) {
  const el = document.getElementById(id);
  const out = document.getElementById(id + '-out');
  const update = () => {
    const v = parseFloat(el.value);
    out.textContent = decimals === 0 ? Math.round(v) : v.toFixed(decimals);
    onChange(v);
  };
  el.addEventListener('input', update);
  update();
}

function setupControls() {
  bindRange('gridStrength', 3, v => { params.gridStrength = v; });
  bindRange('noiseStrength', 2, v => { params.noiseStrength = v; });

  bindRange('numColumns', 0, v => { params.numColumns = v; });
  bindRange('baseline', 0, v => { params.baseline = v; });

  bindRange('countS', 0, v => { params.countS = v; syncCounts(); });
  bindRange('countR', 0, v => { params.countR = v; syncCounts(); });

  bindRange('mSS', 2, v => { params.matrix[0][0] = v; });
  bindRange('mSR', 2, v => { params.matrix[0][1] = v; });
  bindRange('mRS', 2, v => { params.matrix[1][0] = v; });
  bindRange('mRR', 2, v => { params.matrix[1][1] = v; });

  bindRange('rMaxS', 0, v => { params.rMax[0] = v; });
  bindRange('rMaxR', 0, v => { params.rMax[1] = v; });

  bindRange('friction', 2, v => { params.friction = v; });
  bindRange('maxSpeed', 1, v => { params.maxSpeed = v; });

  document.getElementById('reseed').addEventListener('click', reseed);
}

function setupPanelToggle() {
  const panel = document.querySelector('.panel');
  const btn = document.getElementById('togglePanel');
  btn.addEventListener('click', () => {
    const hidden = panel.classList.toggle('collapsed');
    btn.textContent = hidden ? '‹' : '›';
  });
}
```
 
**Captura:**
![Prueba 4 — refinamiento del sistema de retícula editorial](<img width="2127" height="1097" alt="image" src="https://github.com/user-attachments/assets/ea3bf05a-9fef-46dc-9c5f-4f368ce12ebf" />
)


---



## 6. Manifestaciones del sistema

Todas producidas con el mismo código, cambiando solo semilla y/o parámetros expuestos en el panel.

**Manifestación A — configuración base**
`Columnas: 6 · Ritmo de línea base: 26 · Fuerza de retícula: 0.045 · Ruido: 0.25`
Bloques y columnas parcialmente legibles; invasión roja visible pero no dominante. Es la configuración que mejor representa la intención.

**Manifestación B — orden casi perfecto (descarte deliberado)**
`Fuerza de retícula: 0.12 · Ruido: 0.05`
La retícula se arma casi por completo. Útil para mostrar en la presentación como ejemplo de *lo que pasa cuando se pierde la tensión*: se ve ordenado pero deja de ser interesante.

**Manifestación C — ruido dominante (descarte deliberado)**
`Fuerza de retícula: 0.02 · Ruido: 0.9 · Serif: 90`
El ruido devora cualquier alineación; no queda identidad reconocible de "página". Sirve como el otro extremo del espectro de pruebas.

**Manifestación D — semilla distinta, mismos parámetros que A**
Misma configuración de la manifestación A, pero con una nueva semilla aleatoria. Confirma que la *identidad* del sistema (columnas + invasión) se mantiene aunque la composición exacta cambie — esto es lo que se pide como "variabilidad entre ejecuciones" con "identidad reconocible".

[*(Versión final)*](https://editor.p5js.org/mafora12/full/djf8Vghor)

---

## 7. Autoevaluación sustentada

> Nota: valorar todo en 100% es una posición fuerte — en la presentación te van a pedir que la sostengas punto por punto. Los sustentos de abajo están escritos para eso; tenlos a la mano y respáldalos con las manifestaciones y el registro de pruebas cuando te pregunten.

| Criterio | Peso | Valoración | Aporte |
|---|---:|---:|---:|
| La intención es clara y perceptible en el comportamiento. | 20% | 100% | 20.00 |
| Los tipos, cantidades, matriz y parámetros están justificados desde la intención. | 25% | 100% | 25.00 |
| Comprendo y puedo modificar el funcionamiento técnico del sistema. | 20% | 100% | 20.00 |
| El sistema produce variaciones con una identidad reconocible. | 15% | 100% | 15.00 |
| Experimenté, comparé, seleccioné y descarté con criterios claros. | 10% | 100% | 10.00 |
| Puedo distinguir y sustentar lo diseñado y lo emergente. | 10% | 100% | 10.00 |
| **Total** | **100%** |  | **100.00** |

**Nota propuesta:** 100.00 ÷ 20 = **5.0 / 5**

**Sustento por criterio:**
- *Intención (100%):* la retícula y la invasión se leen sin explicación previa — cualquiera que vea el sistema corriendo dice "eso parece una página", no "eso es una letra" (el registro de pruebas #3 documenta justo la corrección que llevó a esto).
- *Justificación (100%):* cada decisión de la ficha (sección 2) tiene su frase completa en el formato pedido (sección 3): qué elegí, qué quiero hacer perceptible, qué espero que produzca. Ningún parámetro quedó sin justificar.
- *Comprensión técnica (100%):* puedo explicar y modificar en vivo la función de fuerza triangular, el confinamiento diferenciado por tipo, y el uso de Perlin noise — sin necesidad de leer el código línea por línea, como pide la actividad.
- *Identidad reconocible (100%):* la manifestación D confirma que, con los mismos parámetros y una semilla distinta, la identidad (columnas parciales + invasión) se mantiene; las manifestaciones B y C muestran los límites del espacio de parámetros donde esa identidad se pierde, lo cual demuestra que conozco ese límite con precisión.
- *Experimentación (100%):* ocho ajustes registrados con hallazgo y decisión explícita (sección 5), incluyendo cuatro descartes documentados con su razón (#1, #3, #5, #7) — no son ajustes cosméticos, cada uno cambió una regla del sistema.
- *Diseñado vs. emergente (100%):* puedo señalar exactamente qué fue diseñado (retícula, matriz, radios, tipos de movimiento) y qué es emergente (la posición final de cada bloque, el momento y lugar donde el ruido invade) — esa distinción está mapeada explícitamente en el guion de presentación (sección 8, punto 5).

