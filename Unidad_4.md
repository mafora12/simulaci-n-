# CORAZÓN — Unidad 4 / Oscilación


https://editor.p5js.org/mafora12/full/7P6bokuM8

---

## 2. Mapa de la ecuación dentro del código

En la parte de arriba de `sketch.js` también dejé este mapa. Las marcas entre corchetes aparecen tal cual en el código, así que se pueden buscar con **Ctrl+F** para encontrar cada sección rápido.

### La función

```
dθᵢ/dt = ωᵢ + (K/W) · Σⱼ wⱼ · sin(θⱼ − θᵢ)
```

La ecuación original usa `(K/N)·Σⱼ sin(...)`. En este proyecto cambié eso por el peso `wⱼ` y el divisor fijo `W`. La razón de este cambio la explico en la §6.

### Dónde está cada parte

| Marca | Parte de la función | Dónde está |
|---|---|---|
| `[A]` | **ωᵢ** — la frecuencia natural | `Agente.omega()` |
| `[B]` | **(K/W) · Σⱼ wⱼ sin(θⱼ − θᵢ)** — el término de acoplamiento, con peso | dentro de `draw()` |
| `[C]` | **dθᵢ/dt** — la derivada completa | dentro de `draw()` |
| `[D]` | **θᵢ += dθᵢ/dt · dt** — la integración (Euler) | dentro de `draw()` |
| `[E]` | **R y ψ** — el parámetro de orden | dentro de `draw()` |

Las cinco partes están juntas dentro de `draw()`. Las marqué con un recuadro que dice `AQUI SE APLICA EL MODELO DE KURAMOTO, UNA VEZ POR FRAME`. También dejé la sumatoria escrita con el `for`, en vez de usar directamente `K·R·sin(ψ − θᵢ)`, para que se pueda ver cómo se aplica la ecuación en el código.

### Dónde se APLICA la función

Esta parte es importante porque el modelo realmente afecta lo que pasa en la pantalla y en el sonido. Todo termina dependiendo de θ.

| Marca | Qué produce | Dónde está |
|---|---|---|
| `[F]` | θ → **eventos sonoros** (el evento dispara cuando θ cruza un umbral) | `Agente.revisarDisparos()` |
| `[G]` | θ → **contracción de la cámara** (sístole/diástole) | `dibujarCamara()` |
| `[H]` | R → **forma del corazón** (las partículas se organizan) | `actualizarParticulas()` |
| `[I]` | δᵢ = θᵢ − ψ → **notas del vals** | `tocarTiempo()` |

---

## 3. Qué representa cada variable

| Variable | En el modelo | En CORAZÓN |
|---|---|---|
| **θᵢ** | fase del oscilador | El reloj interno del agente. Decide **cuándo** dispara su evento y en qué punto de la contracción está su cámara. |
| **ωᵢ** | frecuencia natural | El ritmo propio del agente si nadie lo influyera: su **marcapasos individual**. Lo controla el usuario global (`D`, `T`) y cámara por cámara (arrastrando). |
| **K** | acoplamiento | Cuánto se escuchan entre sí. Es **lo que separa la fibrilación del ritmo sinusal**. En MODO SOLO además controla la tensión armónica. |
| **wᵢ** | peso del agente *(variación)* | Cuánto pesa su voz en el acoplamiento de los demás. Ventrículo 5, aurícula 0.4. |
| **W** | peso del órgano sano *(variación)* | Es el divisor **fijo** del acoplamiento. Lo importante es que, si se apaga una cámara, se pierde su peso del numerador pero `W` sigue igual. |
| **N** | número de osciladores | Las cámaras activas × 2. Apagar una cámara la saca del sistema dinámico. |
| **R, ψ** | parámetro de orden | **R** decide el estado, **cuánto se ha armado el órgano** y con cuánta fuerza golpean las cámaras. **ψ** es el latido del colectivo: cada vuelta suya es un tiempo del compás en MODO SOLO. |
| **δᵢ = θᵢ − ψ** | desfase de enganche | En el estado enganchado cada agente conserva un desfase fijo. De ahí salen las notas del vals. |

---

## 4. Anatomía: 8 agentes, 4 cámaras, 4 tonos

**Son 8 agentes en total: 4 cámaras y 2 roles por cada cámara.**

| Cámara | Nota | Color | Papel |
|---|---|---|---|
| VENTRÍCULO IZQ | A1 — 55.0 Hz | rojo (sangre oxigenada) | el golpe más grave, el que marca el latido |
| VENTRÍCULO DER | D2 — 73.4 Hz | azul (sangre venosa) | grave, complementa |
| AURÍCULA DER | A2 — 110.0 Hz | azul claro | medio |
| AURÍCULA IZQ | D3 — 146.8 Hz | rojo claro | agudo |

**Los dos roles dentro de cada cámara:**

- **IMPULSO** — el disparo eléctrico. Chasquido corto y seco (ruido de banda estrecha + cuadrada muy breve). Se dibuja como una chispa blanca en el borde superior de la cámara, donde estaría el nodo.
- **PARED** — la contracción muscular. El golpe con cuerpo (seno con caída de tono + ruido filtrado). Visualmente **encoge la cámara**: sístole cuando se contrae, diástole cuando se suelta.

Con esto se puede entender visualmente el acoplamiento sin tener que explicarlo solamente con teoría:

> **Con K bajo, el IMPULSO y la PARED de una misma cámara van desfasados: se ve
> la chispa en un momento y la contracción en otro. La cámara tiembla en vez de
> latir. Eso es literalmente una fibrilación.
> Con K alto se alinean y aparece el latido limpio.**

Debajo de cada cámara hay dos relojitos de fase, uno por rol, para ver
exactamente si van juntos o separados.

### Un detalle anatómico

Las aurículas disparan cuando θ = 0 y los ventrículos lo hacen 1 rad después. A 1.15 ciclos/s eso da aproximadamente 0.14 s, que corresponde al **intervalo PR real** de un corazón. Por eso, cuando todo se sincroniza, se escucha el «lub-dub» y no un solo golpe.

En la sustentación hay que aclarar que este desfase **no hace parte de la ecuación de Kuramoto**. Es una decisión sobre el punto de θ en el que dispara cada cámara. Aun así, el momento del disparo sigue dependiendo de θ, que sí viene del modelo.

---

## 5. Las partículas y el corazón anatómico

Alrededor de las cámaras puse 900 partículas. Cada una puede tener **dos destinos**:

- uno **libre**: deriva en órbita alrededor del sistema,
- uno **anatómico**: un punto concreto dentro de la silueta del corazón.

`forma = map(media de R, 0.60, 0.93, 0, 1)` decide cuánto pesa cada destino, y
las partículas van a un resorte amortiguado entre ambos. El HUD muestra ese
valor como **`órgano = N%`**.

La forma que aparece no es simplemente el símbolo ♥. Intenté hacer una silueta de corazón más anatómica, con ventrículos y aurículas, y con el ápex apuntando hacia abajo a la izquierda. También agregué cuatro vasos grandes como tubos: **arco aórtico, tronco pulmonar, vena cava superior y vena cava inferior**. Las partículas que caen en las zonas venosas se muestran azules y las demás rojas.

Además, el órgano **late**: cada vez que ocurre una sístole, toda la silueta aumenta su escala un 5%.

> Es la lectura más directa del parámetro de orden: **la forma del corazón *es* R
> hecho imagen.** Si el sistema se desincroniza, el órgano se deshace en una nube.

---

## 6. La variación del modelo: el acoplamiento lleva peso

Esta es **la modificación de Kuramoto que lleva el proyecto**, y hay que poder
sustentarla. El original es:

```
dθᵢ/dt = ωᵢ + (K/N) · Σⱼ sin(θⱼ − θᵢ)
```

### Por qué lo cambié

Con la ecuación original, **apagar una cámara no hace que el sistema se desordene**. Al quitar un oscilador de una red donde todos están conectados, incluso puede ser más fácil que los que quedan se sincronicen. Eso no funciona con la idea del proyecto, porque un corazón al perder un ventrículo no debería latir mejor. Con la ecuación original obtuve:

| | media de R |
|---|---|
| las 4 cámaras | 0.988 |
| apagando cualquiera de las 4 | 0.99 |

Prácticamente no cambia nada. Matemáticamente tiene sentido, pero para la representación del corazón no funciona: un corazón al que le falta un ventrículo no debería funcionar mejor.

### Qué cambié

Le asigné un peso `wᵢ` a cada agente. La suma se divide por el peso del órgano **completo** (`W`), que se mantiene constante, y no solamente por el peso de los agentes que siguen activos:

```
dθᵢ/dt = ωᵢ + (K/W) · Σⱼ wⱼ · sin(θⱼ − θᵢ)        (j = agentes activos)
```

Los ventrículos tienen `w = 5` y las aurículas `w = 0.4`. Lo escogí pensando en la función de cada parte: el ventrículo es el que bombea la sangre, así que su ausencia debería afectar mucho más al sistema. Una aurícula que falla también afecta, pero no debería tener el mismo peso.

Cuando apago un ventrículo, su peso deja de estar en el numerador, pero `W` sigue igual. Por eso **el acoplamiento efectivo que recibe el resto baja de 8 a 3.9** y el sistema queda por debajo del umbral crítico. Si dividiera por el peso activo, apagar una cámara prácticamente no tendría este efecto. Esa es la diferencia principal.

### Qué pasa en las pruebas, usando 20 corridas por cada caso

| | media de R | llega a sinusal | estado |
|---|---|---|---|
| las 4 cámaras | 0.90 | 20/20 | ritmo sinusal |
| sin **VENTRÍCULO IZQ** | 0.45 | 0/20 | **fibrilación** |
| sin **VENTRÍCULO DER** | 0.45 | 0/20 | **fibrilación** |
| sin AURÍCULA DER | 0.92 | 20/20 | ritmo sinusal |
| sin AURÍCULA IZQ | 0.92 | 20/20 | ritmo sinusal |

Entonces, perder una aurícula puede desajustar un poco el sistema, pero no lo hace caer. En cambio, perder un ventrículo sí lo lleva a fibrilación. Esto es lo que quería representar con el comportamiento del corazón.

También hice que las conexiones entre cámaras tengan diferente grosor: las de los ventrículos son más gruesas y las de las aurículas más delgadas. Así el peso de cada parte también se nota visualmente.

---

## 7. ¿Se sincroniza solo, sin tocar nada?

**Sí. El proyecto está pensado para empezar así.** Los valores iniciales son `K = 8` y `D = 1.00`.

Es importante entender por qué **no** puede arrancar en `K = 0`: con K en cero el
término de acoplamiento vale exactamente cero, así que la ecuación se reduce a
dθᵢ/dt = ωᵢ. Cada oscilador gira a su ritmo y **nada** puede juntarlos, por mucho
que se espere. El umbral crítico está entre K=6 y K=7, medido sobre 8 corridas
de 50 s por valor:

| K | media de R | ¿llega a ritmo sinusal? |
|---|---|---|
| 0 – 4 | 0.32 – 0.47 | **0 de 8** — nunca |
| 5 | 0.66 | 0 de 8 |
| 6 | 0.76 | 0 de 8 |
| **7** | 0.86 | 8 de 8, a los ~10 s |
| **8** | 0.90 | 8 de 8, a los ~6.8 s |
| 12 | 0.96 | 8 de 8, a los ~5.3 s |

Este cambio tan marcado no es un error. Corresponde a la **transición de fase de Kuramoto**, es decir, al umbral Kc a partir del cual el sistema puede sincronizarse.

Con los valores iniciales, las fases empiezan al azar, así que el sistema comienza en fibrilación y después el mismo modelo las va acercando. En 20 corridas obtuve:

| | |
|---|---|
| ARRITMIA | ~2.7 s |
| RITMO SINUSAL | ~6.8 s (20/20) |
| las 4 cámaras caen dentro de | ~55 ms |

### ¿Las cuatro cámaras se activan exactamente al mismo tiempo?

Casi, pero no del todo, y eso también es parte del modelo. Cuando los osciladores se sincronizan, cada uno mantiene un pequeño desfase δᵢ = arcsin((ωᵢ − ω̄)/(K·R)). Los que tienen una ωᵢ más alejada del promedio quedan un poco adelante o atrás. Esta diferencia cambia según K y D:

| | K=6 | K=8 | K=10 | K=12 |
|---|---|---|---|---|
| **D=1.00** | no engancha | **55 ms** | 41 ms | 32 ms |
| **D=0.70** | 50 ms | 34 ms | 27 ms | 22 ms |
| **D=0.50** | 32 ms | 23 ms | 18 ms | 15 ms |
| **D=0.30** | 18 ms | 14 ms | 11 ms | 9 ms |

Con 55 ms prácticamente se escuchan como un solo golpe. Para juntarlas más se puede subir K o bajar D. Pero hay un problema: **si bajo demasiado D, apagar un ventrículo deja de tumbar el sistema**, porque al ser las ωᵢ casi iguales los agentes todavía tienen suficiente acoplamiento para sincronizarse. Por eso las dos cosas están relacionadas.

### Modo UNÍSONO — tecla `U`

**Hay un desfase que no viene directamente de la ecuación:** los ventrículos disparan 1 rad después de las aurículas (aprox. 139 ms). Ese desfase representa el intervalo PR y ayuda a generar el «lub-dub». Con la tecla `U` lo puedo quitar en tiempo real para que las cuatro cámaras disparen en el mismo punto de θ. En ese caso deja de ser una representación tan realista y se convierte en una sincronía más pura.

Con el desfase en cero y los valores de arranque:

| | separación |
|---|---|
| los **dos ventrículos** — el golpe grave, el «pum» | **8 ms** |
| las cuatro cámaras | 55 ms |

O sea que **el pum ya es un solo golpe**: los ventrículos caen prácticamente
encima. Los 55 ms son la cola de las aurículas, que son agudas y suaves.

### Si quiero que estén todavía más juntos

Para reducir más la separación tendría que subir K o bajar D. El problema es que eso entra en conflicto con otra parte importante del proyecto. Los resultados que medí fueron:

| D | K | separación de las 4 | ¿apagar un ventrículo lo tumba? |
|---|---|---|---|
| **1.00** | **8** | **55 ms** | **sí** — cae a R=0.37, fibrilación franca |
| 1.00 | 12 | 32 ms | sí, pero flojo (R=0.76, solo arritmia) |
| 0.80 | 10 | 31 ms | sí, al límite (R=0.80) |
| 0.80 | 12 | 25 ms | **no** |
| 0.60 | 12 | 18 ms | no |
| 0.30 | 12 | 9 ms | no |

**El piso es ~31 ms** si quieres conservar el colapso al apagar un ventrículo. Por
debajo de eso hay que renunciar a él, porque con las ωᵢ casi iguales sobra
acoplamiento para sincronizar aun con la mitad del peso. Las dos propiedades
tiran en direcciones opuestas y esa tabla es la frontera.

Por eso dejé como valores iniciales K=8 / D=1.00. Con ellos el colapso al apagar un ventrículo se nota bastante y el pum ya suena casi como uno solo. Si para la presentación quiero mostrar un unísono más extremo, puedo usar `U` y después subir K o bajar D con los sliders.

---

## 8. Los tres estados colectivos

El PDF pide diferenciar tres estados: desorden, organización parcial y organización estable. En el proyecto los muestro con un nombre más relacionado con el corazón y también con el término del enunciado entre paréntesis:

| En pantalla | Término del enunciado | media de R |
|---|---|---|
| **FIBRILACIÓN** | desorden | < 0.62 |
| **ARRITMIA** | organización parcial | 0.62 – 0.85 |
| **RITMO SINUSAL** | organización estable | ≥ 0.85, sostenido 0.6 s |

**Por qué los umbrales no empiezan en 0:** como solo hay 8 osciladores, R no llega a cero aunque estén completamente desordenados. En ese caso fluctúa alrededor de 1/√N ≈ 0.35. Por eso el estado se determina usando una **media lenta de R (≈2 s)** y los umbrales que medí. También usé una histéresis de 0.035 para evitar que la etiqueta cambie constantemente cuando está cerca del límite.

Para suavizar R uso una **constante de tiempo real** (τ ≈ 2.1 s), en vez de un valor que dependa de los frames. Así el cambio de estado tarda aproximadamente lo mismo a 60 fps que a 15 fps.

**Por qué las ωᵢ no están repartidas de forma uniforme:** uso el perfil `[-1, 0.06, -0.42, 0.20, -0.20, 0.42, -0.06, 1]`, que tiene un grupo más concentrado y dos extremos. Si las ωᵢ fueran uniformes, el cambio entre estados sería demasiado brusco y casi no aparecería el estado intermedio. En las pruebas saltaba de R≈0.32 a R≈0.85 entre K=5 y K=6. Esto **no cambia la ecuación**; la distribución de ωᵢ es un parámetro del modelo. Además, repartí los agentes para que cada cámara tenga uno del núcleo y otro de los extremos, haciendo posible que todas las cámaras puedan entrar en fibrilación.

---

## 9. MODO SOLO — la recompensa por sincronizar

Cuando el sistema mantiene el ritmo sinusal durante unos ~2.5 s, aparece un **piano en menor armónica y con ritmo de vals** por encima del latido. Las cámaras siguen sonando, pero bajan al 55%.

El vals no está programado como una melodía fija. Todo se genera a partir del estado del sistema:

| Elemento | De dónde sale |
|---|---|
| **Tempo** | cada vuelta de **ψ** es un tiempo. Medido: 61 bpm, compases de 2.93 s |
| **Bajo (tiempo 1)** | el agente con el \|δᵢ\| más pequeño |
| **Acorde (tiempos 2 y 3)** | los dos siguientes en fidelidad |
| **Melodía (tiempos 1 y 3)** | recorre los agentes ordenados por δᵢ; cada δᵢ es un grado de la escala menor |
| **Tensión armónica** | **K** |

Esa última parte también la medí y se puede mostrar durante la presentación:

| K | R | rango de la melodía | notas distintas |
|---|---|---|---|
| 6 | 0.905 | grados −5 a +5 | 7 |
| 8 | 0.956 | grados −3 a +3 | 5 |
| 12 | 0.982 | grados −2 a +2 | 5, casi todas la tónica |

Esto sale de la relación δᵢ = arcsin((ωᵢ − ω̄)/(K·R)): **cuando K aumenta, el desfase disminuye**. Por eso el slider K funciona como un control de tensión armónica. No es algo puesto de manera arbitraria, sino que viene del comportamiento del modelo.

---

## 10. Controles

**Controles globales**
- Slider `K` — acoplamiento
- Slider `D` — dispersión de las ωᵢ
- Slider `T` — tempo base
- Tecla `S` — **SCATTER**: induce la fibrilación aleatorizando todas las fases
- Tecla `U` — **unísono**: anula el desfase anatómico y junta las cuatro cámaras en un solo golpe
- Tecla `0` — reinicia fases y ωᵢ

**Controles individuales**
- Clic sobre una cámara — la activa / desactiva (entra o sale del sistema)
- Arrastrar verticalmente sobre una cámara — cambia **su** ωᵢ
- `A` con el cursor sobre una cámara — intercambia sus roles (impulso ↔ pared)

---

## 11. Cómo cumple el proyecto con los requisitos mínimos

| Requisito | Dónde |
|---|---|
| 8 agentes gobernados por el sistema dinámico | `N_MAX = 8` (4 cámaras × 2 roles), todos integran la ecuación en `draw()` |
| 4 personalidades audiovisuales diferentes | las 4 cámaras: cada una con su tono, color, posición anatómica, tamaño y registro; y dentro de cada una, dos roles con comportamiento sonoro y visual distinto |
| Manifestación visual y sonora por agente | IMPULSO = chispa + chasquido; PARED = contracción + golpe |
| ≥2 variables del modelo en tiempo real, K obligatorio | `K`, dispersión de ωᵢ, tempo base, y ωᵢ cámara por cámara |
| ≥2 formas de interacción performativa | los tres sliders + `S` (global); clic para activar/desactivar una cámara, arrastre para cambiar su ωᵢ, `A` para intercambiar sus roles (individual) |
| ≥1 mecanismo de perturbación | dos: `S` = SCATTER dispersa todas las fases (global), y apagar una cámara con un clic, que en el caso de un ventrículo tumba el sistema entero (individual) |
| 3 estados colectivos reconocibles | umbrales medidos sobre R (§8) |
| ≥1 forma perceptible de comunicar el estado | **tres:** la forma del órgano, la etiqueta central con la media de R, y el anillo de fases con el vector R |

---

## 12. Calificación  
- Revisé el proyecto y considero que cumple con los requisitos mínimos de la unidad: sí — 25 puntos.  
- Puedo explicar qué representa cada variable del modelo de Kuramoto dentro de mi proyecto: sí — 25 puntos.  
- Puedo explicar cómo las variables del modelo generan el comportamiento que se ve en el proyecto: sí — 25 puntos.  
- Puedo mostrar que el proyecto cumple con los objetivos de la unidad: sí — 25 puntos.  
calificación:5.0 


---  

# Bitácora — Unidad 4 / Oscilación

Proyecto: **CORAZÓN**, instrumento audiovisual performativo sobre el modelo de
Kuramoto. p5.js + Web Audio API.

En este documento cuento el proceso que fui siguiendo durante las diez versiones del proyecto,
qué fui probando en cada una, qué resultados obtuve y por qué fui tomando cada decisión. Los datos no son
estimaciones — cada una salió de correr el modelo y contar.

---

# PARTE 1 — HISTORIAL DE VERSIONES

---

## V1 · ENJAMBRE — primera versión funcional

**Punto de partida.** Primero revisé el enunciado y empecé por construir una versión básica que cumpliera con lo principal:
8 agentes gobernados por Kuramoto, 4 personalidades audiovisuales, tres estados
colectivos, controles en tiempo real.

**Qué se construyó.** Hice 8 agentes repartidos en 4 personalidades (dos de cada una):
PULSO, HILO, CHISPA y CAMPANA. Anillo de fases central con el vector R, drone de
fondo que se abría según R, y síntesis con **Web Audio API nativa** en vez de
`p5.sound` (más confiable y sin dependencias).

El corazón del sketch, que no cambió en toda la vida del proyecto:

```js
// dTheta_i/dt = omega_i + (K/N) * SUM_j sin(theta_j - theta_i)
const derivadas = [];
for (const a of act) {
  let acoplamiento = 0;
  for (const b of act) acoplamiento += sin(b.theta - a.theta);
  acoplamiento = (K / n) * acoplamiento;
  derivadas.push(a.omega(SPREAD) + acoplamiento);
}
// se aplica DESPUES del bucle: todos usan las fases del mismo instante
for (let i = 0; i < n; i++) act[i].theta += derivadas[i] * dt;
```

**Tres problemas que encontré al probarlo** (detalle en E1, E2, E3):

*Uno.* Con las ωᵢ repartidas de forma pareja, la transición era un salto de
R≈0.32 a R≈0.85 entre K=5 y K=6. **La organización parcial casi no aparecía.**

```js
// antes — reparto uniforme
const desvio = (this.i - (N_MAX - 1) / 2) / ((N_MAX - 1) / 2);  // -1 .. 1
return TEMPO * TWO_PI * this.wBase * (1 + desvio * dispersion);

// después — nucleo denso con dos colas
const PERFIL = [-1, -0.42, -0.20, -0.06, 0.06, 0.20, 0.42, 1];
return TEMPO * TWO_PI * this.wBase * (1 + PERFIL[this.i] * dispersion);
```

*Dos.* Con solo 8 osciladores R nunca cae a cero: fluctúa alrededor de 1/√N ≈
0.35. Los umbrales no podían arrancar en 0. Quedaron en 0.62 y 0.85, decididos
sobre una media lenta de R.

*Tres.* La etiqueta parpadeaba justo en la frontera. Se añadió histéresis:

```js
const H = 0.035;                      // hay que cruzar el umbral con margen
let nuevoEstado = estado;
if (estado === 0 && Rsuave > 0.62) nuevoEstado = 1;
else if (estado === 1) {
  if (Rsuave < 0.62 - H) nuevoEstado = 0;
  else if (Rsuave > 0.85) nuevoEstado = 2;
} else if (estado === 2 && Rsuave < 0.85 - H) nuevoEstado = 1;
```

**También:** el rango de K se subió de 6 a 12, porque con el máximo en 6 el
sistema apenas alcanzaba a engancharse.

---

## V2 · Ocho personalidades distintas + MODO SOLO

**Qué quería lograr.** Que los tipos no se repitieran, y un detalle grande: que al
sincronizarse sonara *Victor's Piano Solo*.

**Sobre la pieza.** No se transcribió, por dos razones. Tiene derechos de autor,
y el enunciado descarta explícitamente que el sistema pueda reemplazarse por
«una secuencia predeterminada»: una melodía fija disparada al sincronizar es
exactamente eso, y habría debilitado el criterio que más pesa. En su lugar se
construyó el **carácter** que buscaba —piano solitario, modo menor, vals lento—
pero con el material generado por el modelo.

**Las ocho personalidades**, cada una definida por su relación con la fase, no
por color:

| | Relación con θ |
|---|---|
| PULSO | 1 evento por ciclo |
| ARCO | continuo, envolvente (1−cos θ)/2, sin ataque |
| CAMPANA | 1 evento cada 2 ciclos (subarmónico) |
| HILO | continuo, sin(θ) modula el filtro |
| ECO | 1 evento + 3 ecos a 1/6 y 1/3 de su periodo |
| CHISPA | 2 eventos por ciclo (subdivide) |
| GRIETA | suena según \|θᵢ − ψ\|: protesta cuando el grupo está suelto |
| GOTA | 3 eventos por ciclo (tresillo) |

**El MODO SOLO.** Al sostener la organización estable entra un vals de piano en
menor armónica. Nada está guardado en una partitura: el tempo lo pone ψ (cada
vuelta suya es un tiempo del compás) y las **notas salen de los desfases de
enganche** δᵢ = θᵢ − ψ, que en Kuramoto valen arcsin((ωᵢ − ω̄)/(K·R)):

```js
// desfase -> grado de la escala menor
function gradoDesde(delta) {
  return constrain(Math.round(map(delta, -0.9, 0.9, -5, 5)), -5, 5);
}
```

**Lo que pasó al medirlo fue:** como δᵢ ∝ 1/K, subir K comprime la melodía hacia la
tónica. **K es literalmente el control de tensión armónica**, y sale de la física
del modelo, no de un cableado arbitrario.

| K | R | rango de la melodía | notas distintas |
|---|---|---|---|
| 6 | 0.905 | grados −5 a +5 | 7 |
| 8 | 0.956 | grados −3 a +3 | 5 |
| 12 | 0.982 | grados −2 a +2 | 5, casi todas la tónica |

---

## V3 · CORAZÓN — rediseño completo

**Qué quería lograr.** Cuatro pulsos en tonos distintos, como un corazón. Partículas
alrededor que se organicen en la forma de un corazón **real, anatómico**, no el
símbolo, cuando estén completamente sincronizados. Y que el código marque dónde
está cada parte de la función y dónde se aplica.

**El problema de los agentes.** El enunciado pide 8 agentes, y yo pedía 4
pulsos. Se resolvió partiendo cada cámara en **dos roles**:

```
8 agentes = 4 CAMARAS x 2 ROLES
  IMPULSO = el disparo electrico (chispa + chasquido)
  PARED   = la contraccion muscular (encoge la camara + golpe con cuerpo)
```

Eso cumple los dos requisitos y además regaló el efecto más legible del
proyecto: **con K bajo, el impulso y la pared de una misma cámara se desfasan y
la cámara tiembla en vez de latir.** Es literalmente una fibrilación, y hace
visible el acoplamiento sin tener que explicarlo.

**Las cuatro cámaras y sus tonos:**

```js
const CAMARAS = [
  { nom: "VENTRICULO IZQ", freq:  55.00, col: [255,  70,  84], desfase: 1.00 },
  { nom: "VENTRICULO DER", freq:  73.42, col: [ 92, 150, 255], desfase: 1.00 },
  { nom: "AURICULA DER",   freq: 110.00, col: [132, 196, 255], desfase: 0.00 },
  { nom: "AURICULA IZQ",   freq: 146.83, col: [255, 132, 122], desfase: 0.00 }
];
```

Rojo el lado izquierdo (sangre oxigenada), azul el derecho (venosa). El
`desfase` de 1 rad en los ventrículos es el **intervalo PR real** (~139 ms a
1.15 Hz): las aurículas se contraen primero y los ventrículos después. Es lo que
produce el «lub-dub».

**La silueta anatómica, en dos intentos** (E10). El primero definía el órgano
como unión de elipses rotadas: salió una mancha amorfa, con los grandes vasos
enterrados dentro de las aurículas. El segundo separó las dos cosas — el
miocardio como polígono cerrado con el ápex abajo-izquierda, y los vasos como
polilíneas con grosor:

```js
const CUERPO = [
  [-0.33, 0.60], [-0.36, 0.44], [-0.39, 0.26], [-0.37, 0.08], [-0.34, -0.08],
  [-0.35, -0.20], [-0.29, -0.30], [-0.17, -0.33], [-0.09, -0.29],
  /* ... 21 vertices en total ... */
];
const VASOS = [
  { g: 0.050, azul: false, p: [[0.04,-0.30],[0.04,-0.50],[0.12,-0.65],
                               [0.26,-0.68],[0.34,-0.58],[0.36,-0.44]] },  // arco aortico
  { g: 0.046, azul: true,  p: [[0.15,-0.34],[0.06,-0.47],[-0.06,-0.55],
                               [-0.21,-0.56]] },                            // tronco pulmonar
  /* cava superior e inferior */
];
```

**Las partículas.** Cada una tiene dos destinos —uno libre en órbita y uno
anatómico— y R decide cuánto pesa cada uno:

```js
// [H] R -> FORMA DEL CORAZON
forma = constrain(map(Rsuave, 0.60, 0.93, 0, 1), 0, 1);
// ...
const destX = lerp(libreX, corX, forma);
const destY = lerp(libreY, corY, forma);
```

Es la lectura más directa del parámetro de orden: **la forma del órgano es R
hecho imagen.** Si el sistema se desincroniza, el órgano se deshace en una nube.

**Los estados se renombraron** manteniendo el término del enunciado al lado:
FIBRILACIÓN (desorden), ARRITMIA (organización parcial), RITMO SINUSAL
(organización estable).

**El mapa del código**, que fue lo otro que pedí. Va en el encabezado y cada
marca existe literalmente en el archivo:

```
     [A]  omega_i .................. frecuencia natural   -> Agente.omega()
     [B]  (K/N) * SUM sin(...) ..... termino de acoplamiento -> draw()
     [C]  dTheta_i/dt .............. la derivada completa  -> draw()
     [D]  theta_i += dTheta_i/dt*dt  integracion (Euler)   -> draw()
     [E]  R y PSI .................. parametro de orden    -> draw()

   DONDE SE APLICA theta:
     [F]  theta -> EVENTOS SONOROS ....... Agente.revisarDisparos()
     [G]  theta -> CONTRACCION DE LA CAMARA ... dibujarCamara()
     [H]  R     -> FORMA DEL CORAZON ..... actualizarParticulas()
     [I]  theta -> TRAZO DEL ECG ......... actualizarECG()
     [J]  delta = theta - PSI -> NOTAS DEL VALS ... tocarTiempo()
```

**También se añadió un ECG**: una tira cuyo trazo es la suma de las envolventes
de los ocho agentes. Con las fases dispersas da picos pequeños e irregulares;
con las fases juntas, un solo pico alto y periódico.

---

## V4 · Interfaz limpia

**Qué quería lograr.** Quitar la leyenda de cámaras del lado derecho, el
electrocardiograma y las dos líneas explicativas de K. Dejar solo el estado
centrado con los controles, las teclas y el anillo de fases.

**Qué eliminé.** El ECG completo (`actualizarECG`, `dibujarECG`, el buffer), la
leyenda y las líneas de texto. La marca `[I]` del mapa quedó libre y el vals pasó
de `[J]` a `[I]`.

**Y apareció un bug real.** Probándolo, el sistema llegaba a K=12 y seguía
marcando FIBRILACIÓN. El suavizado de R usaba un factor **por frame**, no por
segundo: si el navegador bajaba los FPS, el estado tardaba proporcionalmente más
en cambiar.

```js
// antes — depende del framerate
Rsuave = lerp(Rsuave, R, 0.008);

// después — constante de tiempo real
const TAU_R = 2.1;                       // segundos de memoria de la media
Rsuave += (R - Rsuave) * (1 - Math.exp(-dt / TAU_R));
```

Lo mismo se aplicó a la entrada del MODO SOLO y a la amortiguación de las
partículas. Ahora tarda lo mismo a 15 que a 60 fps.

---

## V5 · Se sincroniza solo

**Qué quería comprobar.** Si el modelo estaba bien implementado, y si las cuatro cámaras
podían llegar a sonar juntas después de un rato **sin tocar nada**.

**Respuesta a lo primero:** sí. sí. Las tres cosas que suelen fallar estaban bien —
el parámetro de orden como promedio de vectores unitarios, la sumatoria dividida
por N, y sobre todo las derivadas calculadas todas antes de aplicarse (si se
actualizara θᵢ dentro del mismo bucle, los agentes tardíos verían fases ya
adelantadas y dejaría de ser Kuramoto).

**Respuesta a lo segundo:** no. y no era cuestión de esperar. Arrancaba en `K=0`,
y con K en cero el término de acoplamiento vale **exactamente cero**: la ecuación
se reduce a dθᵢ/dt = ωᵢ. Medido, 0 de 8 corridas de 60 s con K = 0, 2, 4 y 5.

```js
/* Valores de arranque. K empieza POR ENCIMA del umbral critico a proposito:
   con K = 0 el termino de acoplamiento es exactamente 0 y el sistema no puede
   sincronizarse nunca, por mucho que se espere. Arrancando en K = 8 las fases
   parten al azar (o sea, en fibrilacion) y el propio modelo las junta solo. */
let K = 8.0;          // antes: 0.0
let SPREAD = 0.30;
```

Las fases igual parten al azar, así que abre en fibrilación y se organiza sola:
no se pierde el arco de la presentación.

---

## V6 · Sin el sintetizador de fondo

**Qué quería lograr.** Quitar el drone que sonaba de forma continua.

**Qué eliminé.** Los dos osciladores sierra con filtro que se abrían según R
(`crearDrone`, `actualizarDrone` y su llamada por frame). Ahora **lo único que
suena son los golpes de las cámaras** y el piano del MODO SOLO. En silencio
absoluto significa que ninguna cámara está disparando.

El reverb se quedó: lo usan la pared del ventrículo y el piano, no era parte del
zumbido.

Las formas de comunicar el estado bajaron de cuatro a tres —la forma del órgano,
la etiqueta central y el anillo de fases—, muy por encima del mínimo de una.

---

## V7 · Versión web con link

**Qué quería lograr.** Poder abrirlo desde un link, sin copiar y pegar.

**Qué hice.** Una página autocontenida que trae el sketch dentro y carga p5.js
desde un CDN, publicada con URL propia.

**Un problema de codificación.** La página llevaba acentos y símbolos griegos
directos, y si el servidor no declara la codificación el navegador los rompe
(`CORAZÃ³N`, `cÃ¡maras`). Se reescribió con entidades HTML, de modo que el
archivo es **ASCII puro** y se ve igual sin importar cómo lo sirvan:

```html
<h1><span class="lat">Coraz&oacute;n</span> de Kuramoto</h1>
<span class="ley">d&theta;&#7522;/dt = &omega;&#7522; + (K/W) &middot;
                  &Sigma;&#11388; w&#11388; &middot; sin(&theta;&#11388;
                  &minus; &theta;&#7522;)</span>
```

---

## V8 · Acoplamiento con peso — la variación del modelo

**Primero hice unos arreglos.** El editor de p5 tropezaba con el desestructurado
dentro de `for...of`. Lo cambié por `forEach`:

```js
// no funcionaba en el editor de p5
for (const [m, gg] of [[1,1],[2.001,0.42],[3.004,0.20],[4.01,0.09],[5.02,0.045]]) {
  const o = AC.createOscillator(); o.frequency.value = f * m;
  ...
}

// reemplazo
const armonicos = [[1,1],[2.001,0.42],[3.004,0.20],[4.01,0.09],[5.02,0.045]];
armonicos.forEach(function (h) {
  const o = AC.createOscillator(); o.frequency.value = f * h[0];
  ...
});
```

El mismo patrón quedaba en `dibujarCamara`; se convirtió también, porque iba a
fallar después.

**Qué quería conseguir.** Que apagar un ventrículo debería desordenar el sistema, porque
es un desbalance: un solo ventrículo no sostiene la sincronía del conjunto.

**Lo que encontré al probarlo.** Con la ecuación intacta era **imposible**:

| | media de R |
|---|---|
| las 4 cámaras | 0.988 |
| quitando cualquiera | 0.99 |

No solo no se desordenaba: sincronizaba **mejor**. Y no es un error de
implementación, es una propiedad del modelo — quitar un oscilador de una red
todos-con-todos elimina una fuente de discrepancia y no elimina nada que
sostuviera al grupo.

**La modificación.** Cada agente lleva un peso, y la suma se normaliza por el peso
del órgano **completo** (constante), no por el de los que quedan:

```js
// antes — Kuramoto estandar
let acoplamiento = 0;
for (const b of act) acoplamiento += sin(b.theta - a.theta);
acoplamiento = (K / n) * acoplamiento;

// después — con peso y divisor constante
let acoplamiento = 0;
for (const b of act) acoplamiento += b.peso * sin(b.theta - a.theta);
acoplamiento = (K / W_ORGANO) * acoplamiento;
```

```js
// ventriculo 5, auricula 0.4
const W_ORGANO = CAMARAS.reduce(function (a, c) { return a + c.peso * 2; }, 0);
```

**La clave está en mantener constante el divisor.** Al apagar un ventrículo su peso
sale del numerador pero W no cambia, así que el acoplamiento efectivo cae de 8 a
3.9 y el sistema se hunde bajo el umbral crítico. Si se dividiera por el peso
activo, apagar cámaras no cambiaría nada.

**Un problema que tuve que corregir.** Al principio el ventrículo
derecho colapsaba pero el izquierdo no: el perfil de ωᵢ le había tocado al
izquierdo el oscilador más extremo, y al apagarlo desaparecía el agente más
difícil de enganchar, compensando la pérdida de peso.

```js
// antes
const PERFIL = [-1, 0.06, -0.42, 0.20, -0.20, 0.42, -0.06, 1];

// después — ventriculos apretados, auriculas dispersas
const PERFIL = [-0.30, 0.24, -0.24, 0.30, -0.72, 0.36, -0.36, 0.72];
//               VENT IZQ      VENT DER       AURICULA DER   AURICULA IZQ
```

Además de arreglar la asimetría tiene sentido anatómico: la variabilidad del
marcapasos vive en las aurículas.

**Resultado final**, después de hacer 20 pruebas por caso:

| | media de R | sinusal | estado |
|---|---|---|---|
| las 4 cámaras | 0.90 | 20/20 | ritmo sinusal |
| sin ventrículo izq | 0.45 | 0/20 | **fibrilación** |
| sin ventrículo der | 0.45 | 0/20 | **fibrilación** |
| sin aurícula der | 0.92 | 20/20 | ritmo sinusal |
| sin aurícula izq | 0.92 | 20/20 | ritmo sinusal |

Las vías de conducción dibujadas llevan el peso en el grosor: las de los
ventrículos se ven gruesas. El desbalance se ve antes de tocarlo.

---

## V9 · Modo unísono

**Qué quería comprobar.** Si podía sonar todo en un mismo «pum», súper sincronizado,
aunque no fuera natural.

**Lo que encontré al medir.** Con el desfase anatómico en cero, los **dos
ventrículos caen a 8 ms uno del otro** — el golpe grave ya era uno solo. Los
55 ms que se oían eran la cola de las aurículas, que son agudas y suaves. Lo
único que partía el golpe en dos era el desfase de 139 ms puesto a mano.

```js
// tecla U: anula el desfase anatomico
revisarDisparos() {
  const desfase = unisono ? 0 : this.camara.desfase;
  const c = Math.floor((this.theta - desfase) / TWO_PI);
  ...
}
```

**También encontré una limitación del modelo** (E9): apretar más los golpes y que
perder un ventrículo tumbe el sistema son **incompatibles** más allá de ~31 ms,
porque ambas dependen de la relación entre K y la dispersión de ωᵢ, en sentidos
opuestos.

| D | K | separación | ¿tumba? |
|---|---|---|---|
| 1.00 | 8 | 55 ms | sí, R=0.37 |
| 0.80 | 10 | 31 ms | sí, al límite |
| 0.80 | 12 | 25 ms | **no** |
| 0.30 | 12 | 9 ms | no |

Por eso el unísono quedó en una tecla en vez de cambiar los valores de arranque:
así se pueden mostrar los dos comportamientos en la presentación.

---

## V10 · Controles simplificados

**Qué quería lograr.** Quitar las teclas 1–8, las flechas y el clic derecho.

**Verificación previa.** Antes de quitarlas se comprobó que los mínimos del
enunciado seguían cumpliéndose:

- **≥1 mecanismo de perturbación:** quedan dos — la tecla `S` (dispersa todas las fases) y apagar una cámara, que en el caso de un ventrículo tumba el sistema entero.
- **Interacción global e individual:** global son los sliders y `S`; individual, el clic para activar/desactivar, el arrastre para cambiar su ωᵢ y `A` para intercambiar roles.

Las flechas no perdían nada porque duplicaban los sliders. También se quitó el
bloqueo del menú contextual, que solo existía para el clic derecho.

**El panel de ayuda quedó en ocho líneas:**

```
TECLAS   S ... dispersa todas las fases
         U ... unisono: las 4 camaras en un solo golpe
         A ... intercambia los roles de la camara bajo el cursor
         0 ... reiniciar fases

MOUSE    click en una camara .. la activa o desactiva
         arrastrar arriba/abajo .. cambia su omega
         los sliders de abajo .... K, dispersion y tempo
```

---

# PARTE 2 — EXPERIMENTOS Y MEDICIONES

Los experimentos que sustentan las decisiones de arriba. Cada uno: qué se
preguntó, cómo se midió, qué salió.

### E1 — ¿Existe la organización parcial?

Correr 12 s con ωᵢ uniformes, promediando R sobre 5 corridas por valor de K.

| K | media de R |
|---|---|
| 0 – 5 | ~0.32 |
| 6 | 0.85 |
| 8 | 0.94 |

**Resultado.** Salto de 0.32 a 0.85 sin nada en medio. → Perfil núcleo + colas.

### E2 — ¿Dónde poner los umbrales?

Con 8 osciladores R nunca cae a cero: fluctúa alrededor de 1/√N ≈ 0.35. Con
memoria corta (τ≈0.37 s) el rango a K=0 iba de 0.07 a 0.79 — la etiqueta habría
parpadeado sin parar. Con τ≈2 s se cerró a 0.20–0.45. → Umbrales 0.62 y 0.85
sobre una media lenta.

### E3 — El parpadeo en la frontera

Simulando 75 s: la etiqueta pasaba a PARCIAL en t=11.4 s, volvía a DESORDEN en
13.2 s (media 0.619) y regresaba en 13.4 s (media 0.621). → Histéresis 0.035.

### E4 — Una hipótesis que resultó falsa

**Hipótesis:** el paso de Euler se vuelve inestable cuando los FPS bajan y `dt`
llega a su tope de 0.05 s.

| dt | ≈fps | R final |
|---|---|---|
| 0.0083 | 120 | 0.988 |
| 0.0167 | 60 | 0.988 |
| 0.0500 | 20 | 0.988 |

**Resultado: era falsa.** La integración aguanta perfectamente y no se tocó nada. Pero buscando
la causa real apareció E5.

### E5 — Un error real: suavizado por frame

La media de R se actualizaba con un factor fijo por frame. En una prueba el
sistema llegó a K=12 y seguía marcando FIBRILACIÓN. → Constante de tiempo real.

### E6 — ¿Se sincroniza solo?

8 corridas de 60 s por valor de K.

| K | ¿llega a ritmo sinusal? |
|---|---|
| 0, 2, 4, 5 | 0 de 8 — nunca |
| 6 | 8 de 8, ~6.6 s |
| 8 | 12 de 12, ~4.9 s |

Con la variación de V8 el umbral crítico subió a entre K=6 y K=7, y los valores
de arranque quedaron en K=8 / D=1.00: arritmia a los ~2.7 s, ritmo sinusal a los
~6.8 s, 20/20 corridas.

### E7 — Apagar una cámara no hacía nada

Ver V8. Con la ecuación intacta, quitar cualquier cámara subía R de 0.988 a 0.99.

### E8 — La variación y el colapso asimétrico

Ver V8. Primer intento: el derecho colapsaba (R=0.56), el izquierdo no (R=0.86).
→ Reordenar el perfil de ωᵢ.

### E9 — Dos propiedades incompatibles

Ver V9. El piso es ~31 ms si se quiere conservar el colapso del ventrículo.

### E10 — La silueta del corazón

Ver V3. Unión de elipses → mancha amorfa. Polígono + polilíneas → se lee. Con 540
partículas se veía rala; se subió a 900 y se bajó la opacidad de las cámaras.

---

# PARTE 3 — DECISIONES Y SU JUSTIFICACIÓN

| Decisión | Por qué |
|---|---|
| **8 agentes = 4 cámaras × 2 roles** | Cumple los 8 agentes y las 4 personalidades a la vez, y regala el efecto más legible: con K bajo el impulso y la pared se desfasan y la cámara fibrila. |
| **Peso en el acoplamiento** | Única forma de que apagar un ventrículo desordene el sistema (E7). Justificación anatómica: el ventrículo bombea. |
| **Divisor constante W** | Es la pieza que hace funcionar lo anterior. Con divisor activo, apagar cámaras no cambiaría nada. |
| **Desfase de 139 ms** | Es el intervalo PR real y da el «lub-dub». **No es un término del modelo**, es puesta en escena — por eso se apaga con `U`. Hay que decirlo así en la sustentación. |
| **No reproducir la pieza pedida** | Derechos de autor, y el enunciado descarta la «secuencia predeterminada». El vals se genera desde los desfases de enganche. |
| **Retirar el drone** | Sonaba continuo y tapaba los golpes. El estado ya lo comunican el órgano, la etiqueta y el anillo. |
| **Retirar el ECG y la leyenda** | Interfaz más limpia; quedaban tres formas de comunicar el estado. |
| **Retirar teclas 1–8, flechas y clic derecho** | Las flechas duplicaban los sliders; los mínimos se verificaron antes de quitar las patadas de fase. |

---

# PARTE 4 — REFLEXIONES

**Lo que más me costó no fue programar el modelo, sino conseguir que el comportamiento se sintiera bien al usarlo.** Kuramoto
son tres líneas. Que los tres estados se puedan *tocar* —que exista un rango de K
donde vivir la organización parcial, que la etiqueta no parpadee, que el desorden
se distinga del ruido estadístico de tener solo 8 osciladores— tomó mucho más
trabajo que la ecuación.

**Una de las cosas que más aprendí fue que el modelo no siempre se comporta como uno espera.** La intuición
de que apagar un ventrículo debía desordenar el sistema era correcta desde la
anatomía, pero el modelo hacía lo contrario: sincronizaba mejor. Descubrir *por
qué* fue más útil que si hubiera funcionado a la primera, porque obligó a
entender qué parte de la ecuación había que tocar y a poder defenderlo.

**También encontré que hay cosas que el sistema no puede hacer al mismo tiempo.** Que los golpes caigan
súper juntos y que perder un ventrículo lo tumbe son incompatibles más allá de
cierto punto, y no por una limitación de la implementación sino por la estructura
del modelo. Tener la frontera medida (~31 ms) vale más que haber elegido un lado
sin saberlo.

**Las pruebas fueron cambiando mis decisiones.** Varias veces lo que parecía obvio resultó falso
al contarlo: que la integración se rompía a FPS bajos (era otra cosa), que las
cuatro cámaras sonaban desparramadas (los ventrículos ya caían a 8 ms), que
bastaba esperar para que se sincronizara (con K=0 era imposible).

**Sobre el trabajo con IA.** Las decisiones de diseño —el corazón, las cuatro
cámaras, el desbalance del ventrículo, la limpieza de la interfaz— salieron de
mí. Lo que aportó la herramienta fue implementarlas y, sobre todo, **medir si
funcionaban**: varias veces devolvió que lo que yo pedía no era posible tal cual
y hubo que decidir otra cosa. Esa fricción es la parte que más sirvió.
