# Prompts — TP 1

El registro del proceso, en orden. Tres prompts en una sola conversación de Claude. El artefacto quedó terminado en el tercero.

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

**Qué intentaba lograr:** el artefacto entero de una sola vez, nombrando las cuatro capas — estructura con el orden de campos, estilo con las señales visuales suprimidas, comportamiento expresado como estado (valores, errores, enviado), y constraints de empaque (un solo archivo, sin dependencias).

**Qué devolvió:** el formulario completo funcionando: los campos en el orden ilógico pedido, la barra de progreso fija en el paso 1, la validación disparándose toda junta al hacer click en "Siguiente", el mensaje de error genérico, el borrado de las contraseñas al fallar, y el vaciado silencioso a los 60 segundos. Respetó los constraints: un solo archivo, vanilla JS, sin backend real.

**Qué hice con eso:** lo acepté. Pero el prompt tenía dos ambigüedades que no vi al escribirlo y que el modelo resolvió por su cuenta:
Pedí "al menos 20 opciones sin buscador y en un orden aleatorio" para el país, pero nunca especifiqué cuáles 20 países ni qué criterio de desorden usar — el modelo eligió una lista y un orden fijo por su cuenta. Pedí que el año "empiece en 2024 y baje de uno en uno", pero no dije hasta dónde — el modelo decidió arbitrariamente parar en 1900.

Ninguna de las dos rompe el ejercicio (ambas caen dentro del espíritu de "mal UX"), pero son decisiones que tomó el modelo y no yo, y por eso las dejo documentadas.

---

## 2 — Iterar sobre el estado: reordenar las canaletas

```
Cambia el selector de país por una lista de chips clickeables (los mismos 20 países, en el mismo orden aleatorio fijo en el HTML). Agrégale dos estados: `paises` (el array de 20 países, hoy fijos en el HTML) y `seleccionado` (el índice del chip tocado primero, o null). Click en un chip con `seleccionado` en null: pasa a ser ese índice y el chip se marca. Click en otro chip: se intercambian los dos países dentro de `paises`, `seleccionado` vuelve a null y se sacan las marcas. Click en el chip ya seleccionado: `seleccionado` vuelve a null sin intercambiar nada. Dos reglas: mientras `enviando` es true los clicks en los chips no hacen nada, y el valor que se guarda en `valores.pais` pertenece a la posición del chip, no al país que quedó ahí — al intercambiar, el índice no se mueve.
```

**Qué intentaba lograr:** devolverle agencia al usuario. Sin esto, el selector de país es una lista más: eliges de un <select> normal. Con esto, el usuario puede "corregir" el orden alfabético manualmente, intentando reordenar los países a su gusto — que es justamente donde se esconde la trampa.

**Por qué está escrito así:** las tres líneas de click son la ida y dos vueltas distintas — completar el intercambio, y cancelar la selección. Nombrar solo la ida deja al modelo inventando cómo se sale del estado, y lo más común es que no haya forma de deseleccionar un chip sin intercambiarlo.

Las dos reglas del final previenen bugs concretos. Sin la primera, intercambiar chips mientras enviando está en curso (por ejemplo, justo cuando se dispara la validación) deja el formulario guardando un país distinto del que el usuario vio al hacer click. Sin la segunda, el modelo mueve el valor guardado junto con el nombre del país al intercambiar, porque están renderizados en el mismo elemento — es la confusión clásica entre el estado y su reflejo en el DOM: el índice del chip debía quedarse fijo, y solo el texto visible debía moverse.

**Qué devolvió:** las tres transiciones correctas y las dos reglas respetadas. El valor guardado en valores.pais se quedó en la posición del chip al intercambiar, no en el país que quedó ahí.

---

## 3 — Envolver el captcha en una página anfitriona

```
Envolvé el formulario en una página que sea sobre otra cosa.

La página es la landing de una plataforma: <header> con el nombre de la
plataforma y un <button> "Crear cuenta", <main> que por defecto muestra
una descripción corta de la plataforma (dos o tres líneas), <footer> con
una línea de contacto.

Agregá un estado `paso` con tres valores: "bienvenida", "formulario" y
"confirmado".

- Arranca en "bienvenida": se ve la descripción y el botón "Crear cuenta",
  el formulario no.
- Al hacer click en "Crear cuenta": `paso` pasa a "formulario", la
  descripción se oculta y aparece el formulario completo (con el orden de
  campos, los chips de país intercambiables y toda la validación tal como
  están).
- Si la validación pasa: `paso` pasa a "confirmado" y se ve un mensaje de
  bienvenida con los datos que cargó el usuario.
- Si falla: se queda en "formulario" con el mensaje de error genérico.

El formulario no cambia por dentro: mismos campos, mismos estados, misma
lógica de validación y de los chips. Solo deja de ser la página y pasa a
ser un paso.
```

**Qué intentaba lograr:** que el formulario apareciera donde aparece un formulario de verdad — cortando una tarea que el usuario quiere terminar. Una página que es solo el formulario no frustra a nadie, porque nadie llegó ahí queriendo otra cosa; en cambio, alguien que solo quería "crear una cuenta" desde una landing normal se topa con todo el desastre sin haberlo buscado.

**Por qué la última línea:** un pedido estructural como este es el caso donde el modelo tiende a reescribir lo que ya funcionaba, y ahí se pierde el trabajo de los dos prompts anteriores (el orden de campos, la validación, los chips intercambiables). Decirlo explícito — "el formulario no cambia por dentro" — lo evitó.

**Qué devolvió:** los tres pasos funcionando, con el formulario intacto adentro del segundo. La landing quedó como una plataforma cualquiera pidiendo crear cuenta.

---
