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

