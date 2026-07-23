## Unidad 1  

## Actividad 7   

## Intención conceptual #1

El objetivo de este proyecto fue desarrollar una simulación inspirada en el comportamiento de un laboratorio molecular, donde un conjunto de partículas cambia su comportamiento según las condiciones del sistema. La intención fue representar visualmente cómo la combinación de reglas simples puede generar comportamientos complejos.

Para construir la simulación utilicé diferentes conceptos vistos durante la unidad:

- **Ruido Perlin**, para generar movimientos suaves y continuos.
- **Caminata aleatoria**, para introducir variaciones impredecibles en la trayectoria de las moléculas.
- **Distribución normal**, haciendo que la mayoría de las moléculas tiendan a formar pequeños grupos mediante una atracción temporal.
- **Lévy Flight**, incorporando desplazamientos largos y poco frecuentes que modifican la distribución del sistema.

La interacción permite que el usuario modifique el comportamiento del sistema sin detener la simulación. El movimiento vertical del mouse controla la temperatura y cada clic cambia el nivel de aleatoriedad del movimiento de las moléculas.  

---

# Experimentos y versiones intermedias

## Versión 1

se añadieron todos los requisitos propuestos:  
- Posibilidad: Este momento está representado por la caminata aleatoria, ya que cada molécula puede moverse en cualquier dirección.
  let randomWalk = p5.Vector.random2D();

randomWalk.mult(randomWalkStrength * temperature);

this.vel.add(randomWalk);  

¿Qué representa?  

Cada actualización genera una dirección completamente aleatoria, por lo que cualquier trayectoria es posible.  

- Tendencia: Este momento corresponde al Ruido Perlin.
  let angle = map(
  noise(this.tx, this.ty),
  0,
  1,
  0,
  TWO_PI
);

let perlin = p5.Vector.fromAngle(angle);

perlin.mult((0.35 - randomWalkStrength * 0.4) * temperature);

this.tx += 0.003;
this.ty += 0.003;

this.vel.add(perlin);  

¿Qué representa?  

Aunque el movimiento cambia constantemente, el ruido Perlin hace que exista una pequeña tendencia a continuar en una dirección similar, generando trayectorias suaves.  

- Normalidad: Está representada por la parte donde la mayoría de las moléculas intenta permanecer cerca de otra.
  if(random() < 0.85){

    this.groupTarget =
    molecules[
      int(random(molecules.length))
    ];

}
else{

    this.groupTarget = null;

}  

if(this.groupTarget != null){

    let attraction = p5.Vector.sub(
        this.groupTarget.pos,
        this.pos
    );

    attraction.normalize();

    attraction.mult(
        map(
            attraction.mag(),
            0,
            200,
            0,
            0.12
        )
    );

    this.vel.add(attraction);

}  
¿Qué representa?  

Existe un 85 % de probabilidad de que una molécula permanezca cerca de otra, por lo que el comportamiento más frecuente es mantenerse agrupada.  

- Excepción: Corresponde al Lévy Flight.
let levyProbability = levyProbabilityBase;

levyProbability *= temperature;

if(random() < levyProbability){

    let jump = p5.Vector.random2D();

    jump.mult(random(80,180));

    this.pos.add(jump);

}  

-  Influencia: Dos casos

  - El usuario modifica la energía del sistema moviendo el mouse.

temperature = map(
    mouseY,
    height,
    0,
    0.5,
    2.2
);  

- Cada clic cambia el nivel de aleatoriedad de las moléculas, modificando directamente su comportamiento.
  
  function mousePressed() {

    randomLevel = (randomLevel + 1) % RANDOMNESS.length;

    randomWalkStrength = RANDOMNESS[randomLevel];

}  

## Control de verisones   

### Versión 1  
El codigo era este:


let molecules = [];

const NUM_MOLECULES = 250;


let temperature = 1;


let globalTime = 0;


const LINK_DISTANCE = 55;

this.groupTarget = null;


this.changeTimer = int(random(60,180));


this.energy = 0;

function setup(){

  createCanvas(windowWidth, windowHeight);

  colorMode(HSB,360,100,100,100);

  noStroke();

  for(let i=0;i<NUM_MOLECULES;i++){

    molecules.push(new Molecule());

  }

}

function draw(){

  background(220,30,6,18);

  globalTime += 0.003;



  temperature = map(
    mouseY,
    height,
    0,
    0.5,
    2.2
  );

  drawTemperatureIndicator();



  for(let molecule of molecules){

    molecule.update();

  }


  drawConnections();



  for(let molecule of molecules){

    molecule.display();

  }

}

function drawConnections(){

  for(let i=0;i<molecules.length;i++){

    for(let j=i+1;j<molecules.length;j++){

      let d = dist(

        molecules[i].pos.x,
        molecules[i].pos.y,

        molecules[j].pos.x,
        molecules[j].pos.y

      );

      if(d < LINK_DISTANCE){

stroke(

180,

20,

100,

map(

d,

0,

LINK_DISTANCE,

80,

0

)

);

       strokeWeight(
map(
d,
0,
LINK_DISTANCE,
2,
0.3
));

line(

molecules[i].pos.x,
molecules[i].pos.y,

molecules[j].pos.x,
molecules[j].pos.y

);

      }

    }

  }

}

function drawTemperatureIndicator(){

  noStroke();

  fill(15,80,100);

  rect(25,25,25,160);

  let h = map(

    temperature,

    0.5,
    2.2,

    0,
    160

  );

  fill(0,80,100);

  rect(

    25,

    185-h,

    25,

    h

  );

  fill(0,0,100);

  textSize(15);

fill(0,0,100);

textSize(14);

textAlign(LEFT);

text("Temperatura", 20, 210);

text(
nf(temperature,1,2),
20,
230
);

}

function windowResized(){

  resizeCanvas(windowWidth,windowHeight);

}


class Molecule {

  constructor() {

    // Posición inicial
    this.pos = createVector(
      random(width),
      random(height)
    );

    // Velocidad
    this.vel = createVector();

    // Variables para Perlin Noise
    this.tx = random(1000);
    this.ty = random(5000);

    // Tamaño
    this.size = random(5, 9);

    // Color
    this.hue = random(170, 220);

    // Dirección aleatoria inicial
    this.angle = random(TWO_PI);

  }

 update(){



  let angle = map(
    noise(this.tx,this.ty),
    0,
    1,
    0,
    TWO_PI
  );

  let perlin = p5.Vector.fromAngle(angle);

  perlin.mult(0.25*temperature);

  this.tx+=0.003;
  this.ty+=0.003;



  let randomWalk=p5.Vector.random2D();

  randomWalk.mult(0.08*temperature);



  this.changeTimer--;

  if(this.changeTimer<=0){

      // La mayoría permanece
      // cerca de otra molécula.

      if(random()<0.85){

          this.groupTarget=
          molecules[
          int(random(molecules.length))
          ];

      }

      else{

          this.groupTarget=null;

      }

      this.changeTimer=int(random(80,200));

  }

  if(this.groupTarget!=null){

      let attraction=p5.Vector.sub(
      this.groupTarget.pos,
      this.pos
      );

      let d=attraction.mag();

      attraction.normalize();

      attraction.mult(
      map(
      d,
      0,
      200,
      0,
      0.12
      ));

      this.vel.add(attraction);

  }



  let levyProbability=0.0006;

  // La temperatura modifica
  // la probabilidad

  levyProbability*=temperature;

  if(random()<levyProbability){

      let jump=p5.Vector.random2D();

      jump.mult(random(80,180));

      this.pos.add(jump);

  }



  this.vel.add(perlin);

  this.vel.add(randomWalk);

  this.vel.limit(2.5*temperature);

  this.pos.add(this.vel);

  this.vel.mult(0.96);



  if(this.pos.x<0)this.pos.x=width;

  if(this.pos.x>width)this.pos.x=0;

  if(this.pos.y<0)this.pos.y=height;

  if(this.pos.y>height)this.pos.y=0;

}  

Pero no se dejaba ejecuta por este error   

<img width="925" height="927" alt="image" src="https://github.com/user-attachments/assets/72b01c62-6d8c-48ce-84c6-77ef1404ec0e" />  

Luegos se descubrio el error 

Líneas sueltas al inicio (antes de setup()) que usan this fuera de cualquier función o clase — eso es inválido en JavaScript:
js
this.groupTarget = null;
this.changeTimer = int(random(60,180));
this.energy = 0;

Se descubrio que hubo un error en la clase Molecule, (ya está manejado correctamente dentro del constructor()) y adicional, faltaba una llave de cierre } al final del archivo, se organizo y se analizo. Quedadno de esta manera el cdigo:  
let molecules = [];
const NUM_MOLECULES = 250;

let temperature = 1;
let globalTime = 0;

const LINK_DISTANCE = 55;

const RANDOMNESS = [
  0.05,
  0.10,
  0.18,
  0.30,
  0.50
];

let randomLevel = 0;
let randomWalkStrength = RANDOMNESS[randomLevel];

const LEVY_PROBABILITIES = [
  0.0002, 
  0.0006,  
  0.003, 
  0.01,   
  0.03   
];

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

  temperature = map(
    mouseY,
    height,
    0,
    0.5,
    2.2
  );

  drawTemperatureIndicator();

  for (let molecule of molecules) {
    molecule.update();
  }

  drawConnections();

  for (let molecule of molecules) {
    molecule.display();
  }
}

function drawConnections() {
  for (let i = 0; i < molecules.length; i++) {
    for (let j = i + 1; j < molecules.length; j++) {
      let d = dist(
        molecules[i].pos.x,
        molecules[i].pos.y,
        molecules[j].pos.x,
        molecules[j].pos.y
      );

      if (d < LINK_DISTANCE) {
        stroke(
          180,
          20,
          100,
          map(
            d,
            0,
            LINK_DISTANCE,
            80,
            0
          )
        );

        strokeWeight(
          map(
            d,
            0,
            LINK_DISTANCE,
            2,
            0.3
          )
        );

        line(
          molecules[i].pos.x,
          molecules[i].pos.y,
          molecules[j].pos.x,
          molecules[j].pos.y
        );
      }
    }
  }
}

function drawTemperatureIndicator() {
  noStroke();

  fill(15, 80, 100);
  rect(25, 25, 25, 160);

  let h = map(
    temperature,
    0.5,
    2.2,
    0,
    160
  );

  fill(0, 80, 100);
  rect(
    25,
    185 - h,
    25,
    h
  );

  fill(0, 0, 100);
  textSize(14);
  textAlign(LEFT);

  text("Temperatura", 20, 210);
  text(
    nf(temperature, 1, 2),
    20,
    230
  );
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

    this.pos = createVector(
      random(width),
      random(height)
    );


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
    let angle = map(
      noise(this.tx, this.ty),
      0,
      1,
      0,
      TWO_PI
    );

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
      let attraction = p5.Vector.sub(
        this.groupTarget.pos,
        this.pos
      );

      let d = attraction.mag();

      attraction.normalize();
      attraction.mult(
        map(
          d,
          0,
          200,
          0,
          0.12
        )
      );

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

#### Resultados  
<img width="1996" height="1083" alt="image" src="https://github.com/user-attachments/assets/c595bbe2-dad0-4c6f-b947-abea34d2b19b" />  
<img width="2003" height="1145" alt="image" src="https://github.com/user-attachments/assets/3ced8439-8eb7-498d-9f4a-fab219cf3bdd" />  

Observando el resultado, me parecio algo aburrido para una feria de ciencias e innovación, asi que decidi pensar en otra idea más acorde. Porque solo mostrar moleculas moviendose y uniendose aleatoriamente, no me parecio tan conceptual, si no más experimental ya que la forma de demostrarlo en la interaccion con la temperatura no era lo mas ideal, ni divertido. 
Basicamente odie el resultado y senti que pude dar mucho más. 

## Interaccion conceptual #2  

Pense bastante en la idea de algo más que interactivo fuera parte de una experiencia más grande, asi que despues de repasar a varias ferías que he ido, algo que me llamo mucho la atención fue que normalmente las ferias de ciencias no tienen buenos ejemplificadores de como podria ser en la vida real un lugar como un submarino, un laboratorio, o demas espacios que solo te quedas con tu imaginación. Asi que debido a esa idea decidi crear una especie de localizador que combinara lo visto en clase con un diseño que tenga estilo y utilidad, teniendo encuenta que para un trabajo anterior de diseño grafico, que me inspito en el arte. 

### Proceso (Paso 1)  

Empece con algo sencillo pero que lo tuviera todo, con puntos brillantes moviéndose en un fondo oscuro, con un solo campo de flujo de ruido Perlin que evoluciona en el tiempo, más reglas de paso que combinan random walk, distribución normal y Lévy flight, combinando los estados en un mismo parámetro global ("coherencia") que sube y baja lentamente.

#### Cómo es cada momento?

Posibilidad → Al comenzar la simulación, todas las moléculas se mueven en direcciones completamente aleatorias. Ninguna dirección tiene más importancia que otra, por lo que cualquier recorrido puede ocurrir.  
Tendencia → A medida que avanza la simulación, el ruido Perlin hace que las moléculas empiecen a seguir trayectorias más suaves y parecidas entre sí. Poco a poco se forma una tendencia en el movimiento, aunque cada molécula sigue teniendo pequeñas variaciones.  
Normalidad → La mayor parte del tiempo las moléculas mantienen un comportamiento similar: permanecen cerca de otras partículas y siguen el flujo general del sistema. Esto hace que el comportamiento más común sea mantenerse agrupadas y moverse de forma estable.  
Excepción → De vez en cuando ocurre un evento poco probable en el que una molécula realiza un salto mucho más largo de lo normal gracias al Lévy Flight. Esto hace que explore nuevas zonas y rompa temporalmente el patrón que sigue el resto de las moléculas.   
Influencia → la posición del visitante (mouse/touch) no mueve partículas directamente: cambia las probabilidades  de dirección, no controla trayectorias.    

#### Por qué cumple las condiciones?    

- **Combina 4 conceptos de la unidad:** random walk, ruido Perlin (flow field), distribución normal (sigma del paso) y Lévy flight (excepción) — supera el mínimo de 3.  
- **Una sola pieza continua:** todo vive en el mismo sistema de partículas y noise field, nunca cambias de "sketch".  
- **Sigue funcionando sin visitante:** el noise field evoluciona solo con el tiempo (t como tercera dimensión del Perlin).  
- **Variación entre ejecuciones:** semilla aleatoria distinta cada vez (posición inicial, seed del noise), pero la identidad visual (paleta oscura, trazos tipo luciérnaga, comportamiento de cauce) se mantiene.  
  
<img width="637" height="1154" alt="image" src="https://github.com/user-attachments/assets/ac1bfe3f-2ac4-4a29-a96e-347f66d4f86d" />  
#### Codigo  
let particles = [];
const N = 260;
 
const NOISE_SCALE = 0.0032;
const FIELD_TURNS = 2.2;      // cuántas vueltas completas mapea el noise
let zoff = 0;
 
const HIGH_SIGMA = 2.6;       // posibilidad: casi caminata aleatoria pura
const LOW_SIGMA  = 0.10;      // normalidad: muy pegado al campo
 
const LEVY_PROB   = 0.0011;   // probabilidad de excepción por partícula/frame
const LEVY_ALPHA  = 1.25;     // exponente de la cola pesada
const LEVY_SCALE  = 7;
 
let echoes = [];              // huellas de las excepciones (deforman el campo)
 
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
  // Mantiene proporción 9:16 real, centrado, con barras si la
  // ventana no coincide exactamente (pantalla de kiosco/festival).
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
  // Oscila lentamente entre estados de posibilidad y normalidad,
  // con un componente de ruido para que nunca sea perfectamente
  // periódico.
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
 
  // Influencia de los "ecos" (huellas de excepciones pasadas):
  // curvan el campo hacia sí mismos mientras tienen vida.
  for (let e of echoes) {
    const dx = e.x - x, dy = e.y - y;
    const d = sqrt(dx * dx + dy * dy) + 0.001;
    if (d < e.reach) {
      const pull = (1 - d / e.reach) * e.strength;
      const toEcho = atan2(dy, dx);
      ang = lerpAngle(ang, toEcho, pull * 0.5);
    }
  }
 
  // Influencia del visitante: remolino alrededor del puntero.
  if (pointerActive) {
    const dx = x - mouseX, dy = y - mouseY;
    const d = sqrt(dx * dx + dy * dy);
    if (d < POINTER_RADIUS) {
      const w = 1 - d / POINTER_RADIUS;
      const swirl = atan2(dy, dx) + HALF_PI; // tangencial: remolino, no atracción directa
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
  // Cerca del visitante se abre localmente la incertidumbre.
  return lerp(baseSigma, HIGH_SIGMA, w * 0.85);
}
 
function draw() {
  updatePointerState();
  const coherence = coherenceNow();
  const sigmaAngle = lerp(HIGH_SIGMA, LOW_SIGMA, coherence);
 
  // Desvanecido del fondo (deja estela) en modo normal.
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
      // EXCEPCIÓN: cola pesada tipo Lévy -> salto largo, dirección libre.
      const u = random(0.015, 1);
      stepLen = min(pow(u, -1 / LEVY_ALPHA) * LEVY_SCALE, max(width, height) * 0.35);
      angle = random(TWO_PI);
    } else {
      // Paso típico: distribución normal alrededor de 1.
      stepLen = p.speed * abs(randomGaussian(1, 0.22));
    }
 
    p.x += cos(angle) * stepLen;
    p.y += sin(angle) * stepLen;
 
    // continuidad toroidal: el sistema no "se acaba" en los bordes
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
 
  // Dibujar y envejecer ecos
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
 
  // Halo suave del visitante (abstracto, no un cursor literal)
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
  return false; // evita el scroll de la página en móvil
}  


## Proceso (paso 2)   
Luego de ver el resultado me parecio que no estaba del todo terminado, le flataba más, así que decidi preguntarle a la IA el añadirle algo más ordenado y bello. 
A lo que yo le dije como lo queria o me lo imaginaba: 

- **Verde fosforecente** monocromático.
- **Grid tipo radar:** círculos concéntricos y líneas radiales desde el centro, más una cuadrícula fina tipo radar de submarino.
- **Barrido de radar** rotando lento (como si buscara), con estela que demuestre escaneo ferorzando la idea de "busqueda" en vez de decoración.
-  **Ecos** anillos que se expande y se apaga, como si el instrumento detectara un evento o objeto raro.
- **Textura tipo pantalla**  para que se sienta un monitor, no lienzo de arte.
- **Señalización con datos en vivo arriba a la izquierda:** seed, estado (POSIBILIDAD/TENDENCIA/NORMALIDAD según coherencia), contador de excepciones, y si el sensor (visitante) está activo, algo que trasmita "aquí ves que el sistema cambió de estado", pero tambien se sienta como información que se le da al udsuario.  
- **El puntero** hace el papel de un sensor que indica si puede o no atacar esos visitantes.
  
  <img width="642" height="1154" alt="image" src="https://github.com/user-attachments/assets/3ea9acf9-8f79-4ea7-8302-2c8719c8348b" />

  #### Codigo
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

// ---------- capas de instrumento ----------

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

  

### Arreglo 1  

Veía la interacción un poco aburrida y plana nada interesante asi que con ayuda de la IA, pq no tenia muy claro como hacerlo, decidi modificar el puntero para que sea mas interactivo, asi  si este sensor se acerca a las particulas estas se iluminan y se mueven de una manera rapida simulando un proximo ataque, pero si este se queda quieto desaparece la amenaza, junto con el sensor. Ademas se hicieron varios pequeños arreglos independientes, como:  

- **Radio de influencia más grande (230 → 300px)** y la caída del efecto es más lenta (pow(..., 0.6)), así que se siente en más área, no solo pegado al cursor.  
- **Remolino mucho más fuerte:** el peso subió de 0.6 a 0.95, así que cerca del puntero las partículas casi obedecen por completo al giro, en vez de apenas curvarse.  
- **Pulsos de sensor:** cada ~260ms el puntero emite un anillo que se expande y se apaga, como un radar detectando presencia activa — feedback constante mientras se mueve, no solo un halo estático.  
- **Retícula con núcleo brillante** (antes era solo líneas finas casi invisibles).

<img width="646" height="1085" alt="image" src="https://github.com/user-attachments/assets/e75f5237-6422-472d-82ac-6df99f6b8343" />  

#### Codigo  
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
let pings = []; // pulsos que emite el sensor mientras está activo
let lastPing = 0;
 
let seed;
let sweepAngle = 0;
let mono; // fuente monoespaciada
 
function preload() {
  // Fuente del sistema como fallback monoespaciado (sin depender de red externa)
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
      const w = pow(1 - d / POINTER_RADIUS, 0.6); // cae más lento -> efecto notorio en más área
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
  return lerp(1, 2.2, w); // cerca del visitante todo se agita más rápido
}
 
// ---------- capas de instrumento ----------
 
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
 
  // grid cartesiano fino (tipo osciloscopio)
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
  // viñeta leve
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
 
    // Cerca del puntero: partículas más brillantes, más grandes y con
    // un leve corrimiento hacia blanco, para que el efecto sea inconfundible.
    const near = pointerActive ? constrain(1 - dist(p.x, p.y, mouseX, mouseY) / POINTER_RADIUS, 0, 1) : 0;
    const bri = lerp(lerp(50, 92, coherence * 0.4 + 0.3), 100, near);
    const sat = lerp(60, 12, near); // hacia blanco cerca del visitante
    const size = lerp(isException ? 4.5 : 2.2, isException ? 7 : 5, near);
    fill(130, sat, bri, near > 0.05 ? 85 : 60);
    circle(p.x, p.y, size);
  }
 
  // Pulsos que emite el sensor mientras está activo (feedback inmediato de presencia)
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
 
  // ecos como blips de radar (anillo que se expande y se apaga)
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
 
  // reticle de sensor donde está el visitante
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


### Arreglo 2  
Y ya por último quise hacer un poquito de diseño y arregle la parte visual tipo mockup para que se tuviera un poco de imaginación de como se vería, cosa que me ayudo la IA un poco ya que no sabia muy bien el lenguaje de los colores y formas para construirlo de una manera optima:  

- **Formato horizontal 16:9** en vez de vertical.
- **Panel metálico con remaches**, tuberías laterales decorativas, y una placa con el nombre "ESTACIÓN DE SONAR — DECK 2".
- **Pantalla** empotrada con bisel oscuro y un brillo diagonal tipo vidrio curvo, para que se sienta como un monitor físico y no un canvas.
- **Diales y switches decorativos** debajo de la pantalla (profundidad, presión, oxígeno) — puramente estéticos, no funcionales, solo para vender la idea de "consola real".

#### Codigo 

/* ============================================================
   Navegar la incertidumbre — sketch de p5.js
   ------------------------------------------------------------
   Campo de flujo Perlin + caminata aleatoria + distribución
   normal + Lévy flight + campo deformado por ecos e influencia
   del visitante. Formato horizontal 16:9, montado dentro del
   div #screen (definido en index.html / style.css) como
   prototipo de pantalla de sonar de submarino.

   Mapeo de los 5 momentos:
   1. Posibilidad -> sigma angular alta (poco peso del campo)
   2. Tendencia   -> el campo de Perlin empieza a pesar más
   3. Normalidad  -> sigma baja, pasos ~normales alrededor del flujo
   4. Excepción   -> salto tipo Lévy, detectado como "blip" de radar
   5. Influencia  -> el puntero curva el campo (remolino), acelera
      las partículas cercanas y abre localmente la incertidumbre.
   ============================================================ */

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

### Arreglo 3  
Me aparecieron estos errores:  
<img width="1160" height="477" alt="image" src="https://github.com/user-attachments/assets/8f35f7d0-3373-472d-958e-bd056d76864e" />  
Recorde que tenia que organizar el .html pq este no era completo a lo que yo necesitaba pero al final lo pude solucionar 


## Trabajo Final link  

<iframe src="https://editor.p5js.org/mafora12/full/nHWkOUVUg"></iframe>  




  
  
