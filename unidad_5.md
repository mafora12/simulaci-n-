# Bitácora · Unidad 5 · Sistemas de partículas

**Encargo:** presentación generativa para la charla *“Relevo generacional: la ventaja que nadie está aprovechando”* (Alma López · Centro de Eventos Fórum UPB · Future Leaders Forum 2026).

## Presentación funcionando

* Escritorio: https://mafora12.github.io/U5-particulas/
* Celular: https://mafora12.github.io/U5-particulas/movil.html

## Abrir en local

Doble clic en `index.html`, o con un servidor local:

```bash
py -m http.server 8123
```

y abrir `http://localhost:8123`. Se puede ir directo a una diapositiva con `#número`
(por ejemplo `http://localhost:8123/#7`).

## Controles

- `Espacio`, `→` o `Av Pág`: avanzar.
- `←` o `Re Pág`: volver.
- `F` o botón de esquinas: pantalla completa (funciona en Chrome, Edge y Firefox; si el
  navegador la bloquea aparece un aviso y se puede usar `F11`).
- `H`: mostrar / ocultar ayuda.
- `R`: volver al inicio.
- Engranaje → `Esp` / `Por`: cambiar idioma.
- En pantallas táctiles: deslizar a izquierda o derecha.

## Repositorio principal: https://github.com/mafora12/U5-particulas.git  

---

## 1. El concepto de la propuesta

Para empezar con la propuesta tomé como punto de partida una de las ideas principales que nos dieron para la charla: la Universidad creó un espacio pensado originalmente para los grados, pero con el tiempo ese espacio terminó siendo un lugar donde conviven diferentes generaciones.

También me llamó la atención la idea de que **los eventos no son realmente el objetivo, sino el impacto que pueden generar**.

A partir de eso decidí no representar directamente eventos, auditorios o escenarios. Preferí representar a las personas y, sobre todo, las relaciones que se pueden formar entre ellas.

Por eso usé partículas. Cada partícula representa una persona y, por sí sola, no tiene demasiado significado. Lo importante empieza a pasar cuando las partículas se acercan, se conectan, se separan o generan algo juntas.

De ahí salieron dos decisiones principales para todo el proyecto:

1. **Las partículas son las mismas durante toda la presentación.**
   No quería que cada diapositiva pareciera una animación completamente diferente. Las partículas permanecen y lo que cambia es la forma en la que se organizan y se relacionan. Esto se conecta con la idea del relevo generacional: las personas están ahí, pero las relaciones entre ellas pueden cambiar.

2. **El texto es el que genera el movimiento.**
   Cuando cambia una diapositiva, las partículas empiezan a aparecer desde el lugar donde está el texto o desde el panel que lo contiene y después se organizan para formar la siguiente escena. De esta manera, el movimiento no está simplemente acompañando la frase, sino que nace de ella.

La composición general de las 13 diapositivas, los fondos, los paneles de vidrio, la tipografía Rajdhani, la navegación y el selector de idioma siguen el diseño realizado previamente en Figma `juan_franco_u5`.

Mi aporte estuvo principalmente en construir el sistema de partículas, decidir cómo se iban a relacionar entre ellas y llevar esa idea a una presentación web que pudiera funcionar en pantalla completa, en dos idiomas y también en celular.

---

## 2. La gramática visual  

Link figma: https://www.figma.com/design/wINLpyUOSzYRDlTden2Q8A/juan_franco_u5?node-id=0-1&t=ifqUYOIkmIFDKH7V-1  

Para que las escenas no terminaran siendo simplemente animaciones diferentes, primero definí qué significaba cada elemento dentro del sistema.

La idea era que un mismo elemento mantuviera su significado durante toda la presentación y que fueran las relaciones entre ellos las que cambiaran.

| Elemento                                  | Qué representa                          | Cómo funciona                                                                  |
| ----------------------------------------- | --------------------------------------- | ------------------------------------------------------------------------------ |
| **Partícula**                             | Una persona                             | Tiene su propia posición, tamaño, color y brillo                               |
| **Generación “experiencia”**              | Personas que han construido Fórum       | Son más grandes, se mueven más lento y tienen más inercia. Usan violeta y azul |
| **Generación “joven”**                    | Estudiantes y nuevos talentos           | Son más pequeñas, rápidas y exploratorias. Usan cian y rosa                    |
| **Vínculo (línea)**                       | Una relación o confianza                | Aparece y se fortalece cuando las personas permanecen cerca                    |
| **Brillo**                                | Ser visto o valorado                    | Aparece cuando una persona recibe algún tipo de interacción o impacto          |
| **Onda**                                  | Impacto                                 | Se mueve por el espacio y activa las partículas que encuentra                  |
| **Rastro**                                | El camino recorrido                     | Deja una marca que muestra por dónde pasó una partícula                        |
| **Estructura (arco, árbol, ojo, puerta)** | Algo que las personas construyen juntas | Solo aparece cuando las partículas cumplen las condiciones necesarias          |

En total utilicé **250 partículas**: 70 representan la generación de experiencia, 130 representan a los jóvenes y 50 funcionan como partículas adicionales que aparecen en escenas donde se necesita mostrar crecimiento, como en el árbol o cuando aparecen nuevas personas después de una colaboración.

La paleta parte principalmente de los colores utilizados en el diseño: azul `#2457ff`, cian `#65e6e2`, violeta `#6655ee`, rosa `#ff4fa3` y blanco `#f4f3ef`.

Estos colores también buscan relacionarse con el remolino rosa, rojo y azul del logo de Future Leaders Forum, para que la identidad del evento y la de Fórum UPB no se sintieran separadas.  
<img width="1170" height="350" alt="image" src="https://github.com/user-attachments/assets/81cdab8d-772f-4c85-b299-e085f5598b3c" />  

Una regla que traté de mantener durante el desarrollo fue que **el movimiento no estuviera solamente para decorar**.

Cada escena debía tener una razón para moverse de cierta manera. Para comprobarlo, intentaba responder tres preguntas:

* ¿Qué relación existe entre las partículas?
* ¿Qué hace que esa relación aparezca, crezca o desaparezca?
* ¿Qué cambio visual produce esa relación?

Si no encontraba una respuesta clara, sentía que la escena todavía no estaba funcionando y tenía que modificarla.

---

## 3. Relaciones estructurales y movimiento, diapositiva por diapositiva

1. **Relevo generacional: la ventaja que nadie está aprovechando.**
   En esta primera escena la generación de experiencia ya está conectada alrededor del edificio, mientras que los jóvenes están dispersos y tienen menos presencia visual. No existe todavía una conexión entre los dos grupos. De vez en cuando una partícula de experiencia manda un pulso hacia un joven, haciendo que este brille por un momento y después vuelva a apagarse. La idea es mostrar que la posibilidad de conexión existe, pero todavía no se está aprovechando.

2. **¿Un gran auditorio solo para hacer grados?**
   En esta escena todas las personas están organizadas en filas y no hay una diferencia clara entre generaciones. La estructura representa una función muy específica: estar ahí para un evento. Después aparecen grupos de partículas que suben como si fueran birretes. Cada salto cambia su color y finalmente chocan con un límite invisible. La energía existe, pero está contenida.

3. **La Universidad decidió encontrarse con el mundo.**
   Aquí la esfera representa la Universidad y las partículas que vienen desde afuera representan al mundo exterior. Los dos grupos se mueven hacia la frontera y las conexiones solo pueden aparecer cuando los dos grupos se encuentran en ese punto. La idea era que el vínculo no ocurriera dentro de un solo grupo, sino precisamente en el encuentro entre ambos.

4. **Academia + Industria + Ciudad.**
   En esta escena aparecen tres grupos diferentes, cada uno con su propio color y ritmo. Cada grupo mantiene cierta organización interna, pero también aparecen algunas partículas que funcionan como puentes entre ellos. Estas son las que permiten las conexiones hacia otros grupos. No quería que todos se mezclaran completamente, sino mostrar que pueden relacionarse sin perder su propia identidad.

5. **Los eventos nunca fueron el objetivo. El impacto sí.**
   Los tres grupos se mueven hacia un mismo punto que representa el evento. Cuando llegan, aparece una onda que se extiende por toda la pantalla. Las nuevas conexiones no aparecen inmediatamente en todas las partículas, sino únicamente en aquellas que fueron alcanzadas por la onda. De esta manera el evento funciona como un punto de partida y el impacto es lo que continúa después.

6. **Un evento trae personas. Una comunidad trae transformación.**
   Primero las personas hacen fila y entran por una puerta pequeña. La puerta sigue siendo exactamente igual después de que todos pasan. Después las mismas partículas empiezan a utilizarse para construir una puerta mucho más grande. La intención es mostrar que una comunidad no solamente ocupa un espacio, sino que puede cambiarlo.

7. **El talento crece a la velocidad de la confianza.**
   Las partículas giran juntas y mantienen sus posiciones relativas, por lo que la cercanía se conserva durante un periodo de tiempo. Los vínculos no aparecen inmediatamente, sino que se van fortaleciendo poco a poco. A medida que aumenta la confianza, también cambian el tamaño y el brillo de las partículas. La idea es mostrar que el talento no aparece de la nada, sino que puede crecer a partir de una relación que se construye con tiempo.

8. **La experiencia construye el camino. Las nuevas generaciones descubren nuevas rutas.**
   En esta escena la generación de experiencia se mueve más lentamente y va creando el camino principal. Los jóvenes lo siguen a mayor velocidad, pero en determinado momento algunos toman caminos diferentes y dejan sus propios rastros. En esta diapositiva decidí que las partículas pasaran por encima del panel de vidrio para que el camino atravesara el espacio del discurso y no simplemente lo rodeara.

9. **Una visión. Dos generaciones.**
   Las dos generaciones construyen juntas un ojo. La generación de experiencia forma los párpados y el contorno, mientras que los jóvenes forman el iris que se mueve en el interior. Después el ojo parpadea para darle vida a la estructura. La idea es que ninguna de las dos generaciones pueda formar el ojo completamente por separado.

10. **El crecimiento ocurre cuando trabajan juntas.**
    Para esta escena utilicé un árbol. Las ramas van apareciendo poco a poco y cada parte es construida por partículas de ambas generaciones. Una rama no puede completarse si falta una de las dos partes. Cuando finalmente el árbol está completo, aparecen nuevas partículas en las puntas, como si fueran brotes. Así represento el crecimiento que aparece cuando las dos generaciones trabajan juntas.

11. **Los jóvenes no son el futuro. Son el presente que muchas organizaciones aún no ven.**
    En esta escena utilicé un radar. Los jóvenes están presentes desde el principio, pero tienen muy poca visibilidad. El barrido del radar los va encontrando y, cuando una partícula es detectada, se enciende y comienza a conectarse con las demás. Las organizaciones aparecen fuera del radar, porque la idea no es que los jóvenes lleguen en ese momento, sino que ya estaban presentes y simplemente no estaban siendo vistos.

12. **El futuro no se hereda. Se construye.**
    En lugar de mostrar una estructura que ya está terminada, hice que las dos generaciones fueran colocando sus partes poco a poco. El arco se construye desde la base hasta llegar a la parte superior. Las piezas solo pueden mantenerse cuando los dos extremos necesarios están colocados. Así la estructura representa algo que se construye entre ambas generaciones.

13. **Cierre.**
    Para terminar, todas las personas que aparecieron durante el recorrido se juntan y forman una red en espiral de tres brazos. La estructura continúa girando alrededor de los códigos QR. La intención del cierre es que la energía de la presentación no termine completamente con la última diapositiva, sino que dé la sensación de que la conversación puede continuar después de la charla.

### Decisiones técnicas que sostienen el discurso

* **Pantalla grande:** trabajé con un lienzo de 1920 × 1080 para que la presentación pudiera proyectarse en pantalla grande sin deformarse.

* **Transición entre diapositivas:** los paneles, la marca y los elementos de navegación cambian de posición entre escenas y los textos hacen una transición suave. La idea es que se sienta parecido al movimiento de un *Smart Animate* de Figma. Así, la transición también forma parte de la propuesta y no es simplemente un cambio de página.

* **Dos idiomas:** el contenido está separado del código para poder trabajar español y portugués. Como el diseño original está pensado en español, hice que los textos en portugués pudieran ajustarse automáticamente dependiendo de su ancho para intentar mantener una composición similar.

* **Celular:** hice una versión propia para celular que conserva las mismas escenas y relaciones, pero adapta la composición a formatos verticales y horizontales. La prioridad fue que las estructuras importantes no quedaran cortadas.

* **Publicación:** la presentación está publicada mediante GitHub Pages y se puede actualizar automáticamente cada vez que se hace un nuevo *push*. En la última diapositiva también hay un QR que lleva a la versión para celular.

---

## 4. Autoevaluación

### 1. Cumplimiento del encargo

Creo que la presentación logra cumplir con el encargo porque mantiene los 13 momentos principales del guion y los convierte en diferentes comportamientos del sistema de partículas.

No intenté hacer una presentación tradicional en la que cada diapositiva simplemente tuviera texto acompañado de una animación. La idea fue que las partículas fueran parte del discurso y que cada escena tuviera una relación diferente.

También logré que la presentación funcionara en pantalla completa desde el navegador, tuviera versión en español y portugués y contara con una adaptación para celular.

**Evidencia:** los dos enlaces publicados, `slides.js`, donde está organizado el contenido de los dos idiomas, y `particles.js`, donde están definidas las escenas.

### 2. Relaciones estructurales

Una de las cosas que considero importantes del proyecto es que las partículas no se conectan de cualquier manera.

Cada escena tiene diferentes reglas para decidir quién puede conectarse con quién, a qué distancia y durante cuánto tiempo debe mantenerse la relación.

Por ejemplo, en la escena 3 las conexiones solo pueden ocurrir entre la Universidad y el mundo exterior. En la 4 aparecen conexiones principalmente dentro de cada grupo, excepto por las partículas que funcionan como puentes. En la 5 la onda determina qué partículas pueden empezar a relacionarse y en la 11 primero es necesario que el radar encuentre a los jóvenes para que estos empiecen a conectarse.

Esto hace que las relaciones sean parte del significado y no solamente un efecto visual.

**Evidencia:** el campo `link` de las escenas en `particles.js` y la tabla de gramática del sistema visual.

### 3. Comportamiento y significado

Creo que la mayor parte de los movimientos tienen una relación con lo que está diciendo la presentación.

Por ejemplo, el límite invisible de la escena 2 representa una energía que existe pero que está contenida. La onda de la escena 5 representa un impacto que puede ir mucho más allá del evento. En la escena 7 los vínculos aparecen lentamente para representar que la confianza necesita tiempo.

También intenté que las estructuras de las escenas 6, 9, 10 y 11 fueran más literales: una puerta, un ojo, un árbol y un radar.

Me quito algunos puntos en este aspecto porque esas cuatro escenas no empezaron así. En una primera versión eran redes de partículas más genéricas y sentía que se parecían demasiado entre ellas y que realmente no estaban aportando mucho al significado de cada frase.

Tuve que rehacerlas y buscar una forma más directa de representar lo que estaba diciendo el texto. Creo que ese cambio ayudó bastante a que cada escena tuviera una identidad más clara.

**Evidencia:** la sección de relaciones y comportamiento por diapositiva en `DECISIONES_SISTEMA_VISUAL.md` y el historial de cambios de las escenas.

### 4. Explicación y demostración

La presentación se puede mostrar directamente desde el navegador, avanzar utilizando el teclado, cambiar entre idiomas y también enseñar la versión para celular.

Además, puedo mostrar cómo funciona el código y dónde está cada parte del sistema. El contenido está separado en diferentes archivos: `slides.js` para el guion, `particles.js` para las partículas y escenas, y `main.js` para la navegación.

Esto también me ayudó durante el proceso porque no tuve que modificar todo el proyecto cada vez que necesitaba cambiar un texto o ajustar una escena.

**Evidencia:** `DECISIONES_SISTEMA_VISUAL.md` para las decisiones del sistema y `GUIA_CAMBIOS_RAPIDOS.md` para los cambios de textos, enlaces y parámetros.

---

| Item                                      | nota                                    |
| ----------------------------------------- | --------------------------------------- |
| **Cumplimiento del encargo**              | 25/25                                   | 
| **Relaciones estructurales**              | 25/25                                   |
| **Comportamiento y significado**          | 25/25                                   |
| **Explicación y demostración**            | 25/25                                   |
|Total                                      |5                                        |
