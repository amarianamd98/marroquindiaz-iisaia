# Prompts — TP 1

El registro del proceso, en orden. Tres prompts en una sola conversación de Gemini Canvas. El artefacto quedó terminado en el tercero.

---

## 1 — Prompt inicial

```
Construye un formulario de registro de datos personales que sea
intencionalmente un desastre de UX, como ejercicio didáctico.
Estructura:
- <header> con el título "Registro de usuario" y, debajo, una barra de
  progreso de 3 pasos que nunca se actualiza (siempre se ve en el paso 1,
  sin importar en qué parte del formulario esté el usuario).
- <main> con los campos, en este orden exacto (deliberadamente ilógico):
  contraseña, confirmar contraseña, correo electrónico, teléfono,
  apellido, país (select), nombre, fecha de nacimiento (tres selects:
  año empezando en 2024 y bajando de uno en uno, mes, día), checkbox
  "Acepto términos y condiciones" (sin marcar) y checkbox "Suscribirme
  al boletín publicitario" (premarcado). El campo "nombre" no debe
  tener <label>, solo un ícono de usuario genérico. El campo "teléfono"
  debe tener como único texto de ayuda "Número" sin especificar de qué
  tipo ni formato.
- <footer> con un <button> "Siguiente" y, debajo, un área para mensajes
  de error.
Estilo:
- Fondo blanco, bordes de los inputs en gris muy claro (#eee) casi
  invisibles sobre el fondo.
- Tipografía pequeña (11px) en todas las etiquetas.
- Botón "Siguiente" y el texto plano del formulario deben verse
  visualmente idénticos (mismo color, mismo peso), para que no quede
  claro qué es clickeable.
- Cero indicación de foco visible al navegar con teclado.
Comportamiento:
- Estado: valores (objeto con los campos), errores (array), enviado
  (booleano).
- El botón "Siguiente" no avanza a ningún paso 2 real: al hacer click,
  ejecuta la validación de todos los campos de una sola vez (nunca en
  tiempo real mientras el usuario escribe).
- Si hay errores, mostrar un único mensaje genérico "Error" en el área
  de mensajes, sin decir cuál campo falló ni por qué. Además, borrar
  por completo el contenido de los campos de contraseña (no solo
  marcarlos) obligando a reescribirlos.
- La validación de contraseña exige mayúscula, número, símbolo y
  mínimo 12 caracteres, pero ese requisito solo se muestra después de
  que el usuario falla la primera vez, nunca antes.
- El campo "teléfono" debe rechazar el símbolo "+" y los espacios, sin
  explicar por qué en ningún mensaje.
- El selector de país debe tener al menos 20 opciones sin buscador y
  en un orden aleatorio (no alfabético).
- Si todos los campos pasan la validación, no mostrar ninguna
  confirmación visual: la pantalla se queda igual, como si no hubiera
  pasado nada.
- Agregar un temporizador: a los 60 segundos de cargada la página, sin
  ningún aviso previo, vaciar todos los campos del formulario.
- El botón "Siguiente" no debe deshabilitarse tras hacer click, para
  permitir múltiples envíos seguidos.
Constraints:
- Un solo archivo HTML, con el CSS en un <style> y el JS en un
  <script>.
- Vanilla JS, sin frameworks ni dependencias externas.
- No necesita conectarse a ningún backend real; el envío puede
  simularse con JavaScript.
```

**Qué intentaba lograr:** el artefacto entero de una sola vez, nombrando las cinco capas — estructura con etiquetas semánticas, estilo, comportamiento expresado como estado, y constraints de empaque.

**Qué devolvió:** el tablero funcionando, con las 4 filas de pegs, las 5 canaletas y la animación de caída. Respetó los tres constraints: un solo archivo, sin dependencias, y pegs y bola como elementos del DOM en lugar de `<canvas>`.

**Qué hice con eso:** lo acepté. Pero el prompt tenía dos ambigüedades que no vi al escribirlo y que el modelo resolvió por su cuenta — están detalladas en el README, porque son lo más interesante de esta entrega.

---

## 2 — Iterar sobre el estado: reordenar las canaletas

```
Agregale al captcha dos estados: `letras` (el array de 5 letras de las
canaletas, hoy fijas en el HTML) y `seleccionada` (el índice de la canaleta
tocada primero, o null).

Click en una canaleta con `seleccionada` en null: pasa a ser ese índice y la
canaleta se marca.
Click en otra canaleta: se intercambian las dos letras dentro de `letras`,
`seleccionada` vuelve a null y se sacan las marcas.
Click en la canaleta ya seleccionada: `seleccionada` vuelve a null sin
intercambiar nada.

Dos reglas: mientras `cayendo` es true los clicks en canaletas no hacen
nada, y los porcentajes pertenecen a la posición, no a la letra — al
intercambiar, los números no se mueven.
```

**Qué intentaba lograr:** devolverle agencia al usuario. Sin esto el captcha es una tragamonedas: mirás caer la bola y no podés hacer nada. Con esto podés poner en el centro la letra que necesitás, que es donde la probabilidad es más alta.

**Por qué está escrito así:** las tres líneas de click son la ida y **dos** vueltas distintas — completar el intercambio, y cancelar la selección. Nombrar solo la ida deja al modelo inventando cómo se sale del estado, y lo más común es que no haya forma de cancelar.

Las dos reglas del final previenen bugs concretos. Sin la primera, reordenar con la bola en el aire la hace aterrizar sobre una letra distinta de la que había cuando soltaste. Sin la segunda, el modelo mueve el porcentaje junto con la letra, porque están renderizados en el mismo elemento — es la confusión clásica entre el estado y su reflejo en el DOM.

**Qué devolvió:** las tres transiciones correctas y las dos reglas respetadas. Los porcentajes se quedaron en su posición al intercambiar.

---

## 3 — Envolver el captcha en una página anfitriona

```
Envolvé el captcha en una página que sea sobre otra cosa.

La página es un formulario para reservar un turno: <header> con el nombre
del lugar, <main> con un <form> de nombre, email y fecha y un <button>
"Reservar turno", <footer> con una línea de contacto.

Agregá un estado `paso` con tres valores: "formulario", "captcha" y
"confirmado".
- Arranca en "formulario": se ve el form, el captcha no.
- Al enviar el form: `paso` pasa a "captcha", el form se oculta y aparece
  el tablero de Galton.
- Si la verificación pasa: `paso` pasa a "confirmado" y se ve el turno
  reservado con los datos que cargó.
- Si falla: se queda en "captcha" con un objetivo nuevo.

El captcha no cambia por dentro: mismo tablero, mismos estados, misma
lógica. Solo deja de ser la página y pasa a ser un paso.
```

**Qué intentaba lograr:** que el captcha apareciera donde aparece un captcha de verdad — cortando una tarea que el usuario quiere terminar. Una página que es solo el captcha no frustra a nadie, porque nadie llegó ahí queriendo otra cosa.

**Por qué la última línea:** un pedido estructural como este es el caso donde el modelo tiende a reescribir lo que ya funcionaba, y ahí se pierde el trabajo de los dos prompts anteriores. Decirlo explícito lo evitó.

**Qué devolvió:** los tres pasos funcionando, con el captcha intacto adentro del segundo. El formulario quedó como un centro médico pidiendo turno.

---

## Conversación completa

Una sola conversación de Gemini Canvas, sin reiniciar el hilo. El artefacto final tiene 828 líneas en un archivo.
