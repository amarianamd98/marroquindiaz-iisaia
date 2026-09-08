# TP 1

## Qué construí

Un formulario de registro de usuario intencionalmente mal diseñado en términos de UX, como ejercicio para evidenciar anti-patrones comunes de interfaz. El resultado final es una página de landing con tres pasos (bienvenida → formulario → confirmado) donde:

- El formulario tiene el orden de campos ilógico (contraseña antes que correo, nombre sin label, etc.), validación que se ejecuta toda de una vez al final y borra las contraseñas al fallar, un selector de país reemplazado por chips donde intercambiar dos países desincroniza lo que se ve del valor que realmente se guarda (el valor pertenece a la posición del chip, no al país que quedó ahí), y un vaciado silencioso de todos los campos a los 60 segundos.
- Todo el flujo vive dentro de una landing simple que oculta y muestra el formulario según el estado `paso`, sin cambiar la lógica interna del formulario.

## Cómo lo dirigí

Empecé pidiendo ideas generales de anti-patrones de UX (navegación, formularios, visual, feedback, accesibilidad) para tener un catálogo de opciones antes de elegir. De ahí elegí trabajar sobre un formulario de datos personales y pedí más ideas específicas para esa pieza.

Para construirlo formalicé cada iteración como un prompt estructurado en cuatro secciones (Estructura, Estilo, Comportamiento, Constraints), en vez de pedir el HTML directo en lenguaje suelto. Cada prompt nuevo lo armé ajustando el anterior:
1. Prompt base del formulario completo (campos, estilo, validación).
2. Prompt para reemplazar el selector de país por chips intercambiables con estado propio (`paises`, `seleccionado`).
3. Prompt para envolver el formulario en una landing con pasos (`bienvenida` / `formulario` / `confirmado`), sin tocar la lógica interna.

En cada paso definí yo el estado (variables y sus transiciones) y las reglas de comportamiento antes de pedir el código, para que la IA generara exactamente esa lógica y no una versión genérica de "formulario con validación".

## Decisiones que tomé

- Separar el problema en tres prompts en lugar de uno solo: primero el formulario, después el mecanismo de los chips, después el wrapper de la landing. Esto hizo cada iteración más fácil de revisar y corregir sin rehacer todo desde cero.
- Definir explícitamente el modelo de estados (`valores`, `errores`, `enviado`, `paises`, `seleccionado`, `paso`) en el prompt en vez de dejar que la IA decidiera la estructura de datos, para tener control sobre cómo se comporta cada anti-patrón.
- Especificar que el valor guardado en `valores.pais` pertenece a la posición del chip y no al país mostrado ahí — la regla central del anti-patrón de los chips — para que quedara claro y no ambiguo en la implementación.
- Mantener la lógica interna del formulario intacta al envolverlo en la landing, pidiendo explícitamente "el formulario no cambia por dentro", para aislar el cambio de contexto (página vs. paso) del comportamiento ya construido.

## Qué salió mal y cómo lo corregí
 
Al revisar el HTML generado encontré que el campo "nombre" quedó con `aria-label=""` vacío en el input, en vez de simplemente no tener ningún atributo de accesibilidad. Pedí explícitamente que el campo no tuviera `<label>` y solo un ícono genérico, pero el resultado fue más severo de lo esperado: un lector de pantalla no anuncia nada en absoluto para ese campo (ni siquiera "campo de texto sin nombre"), en vez de simplemente omitir la etiqueta visual. Quedó documentado como parte del anti-patrón de accesibilidad, pero es un matiz que no había considerado al escribir el prompt.

## Cómo se ejecuta

```bash
# Abrir el archivo HTML directamente en el navegador
open index.html
```

## Archivos

- `prompts.md`: registro de los prompts usados, sin editar.
