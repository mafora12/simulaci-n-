## Unidad 1  

## Actividad 7   

## Intención conceptual

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

Esto parece un resto de código que se quedó pegado fuera de la clase Molecule (ya está manejado correctamente dentro del constructor()).

Falta una llave de cierre } al final del archivo — la que cierra la clase Molecule. Por eso el error es "Unexpected end of input": el intérprete llega al final del archivo esperando todavía cerrar la clase.



