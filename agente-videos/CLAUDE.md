# Agente creador de vídeos

Proyecto de Hugo para generar vídeos cortos automáticamente (unos 50 s por defecto; admite más largos si Hugo los pide). Hugo no es programador: explica las cosas en español y de forma sencilla.

- `render.py`: convierte un guion JSON en MP4 (Pillow dibuja los fotogramas, Kokoro genera la voz gratis en local, ffmpeg monta el vídeo).
- `esquema.py`: el estilo "esquema técnico 2D" (ver más abajo).
- `agente.py`: escribe el guion con la API de Claude y llama a render.py (necesita ANTHROPIC_API_KEY).
- `instrucciones-guion.md`: reglas para escribir buenos guiones.
- Tipos de escena: portada, dato, lista, frase, cierre. Para añadir uno nuevo, añade un `elif` en `dibujar_escena` de render.py y documéntalo en instrucciones-guion.md.

**En el ordenador de Hugo el comando es `py`, no `python`** (Python 3.14 está instalado pero no en el PATH).

## Dos estilos visuales

- **`carteles`** (el de siempre, por defecto): cada escena es un cartel de texto a pantalla completa sobre un degradado, y se sustituyen unos a otros.
- **`esquema`**: todas las escenas viven en un mismo dibujo grande y la cámara viaja entre ellas **sin ningún corte**, con estética de esquema técnico (fondo casi negro, contornos naranjas, etiquetas con línea guía). Se activa poniendo `"estilo": "esquema"` en el guion. Lo dibuja `esquema.py`, que tiene una función por tipo de escena.
  - Solo funciona con **formas geométricas** (relojes, anillos, rejillas, caminos, corazones). No sirve para objetos reales reconocibles.
  - Al añadir una estación nueva, **dibuja su esqueleto completo desde el primer fotograma** y limítate a iluminarlo según avanza; si no, quedan fotogramas vacíos al arrancar la escena.
  - Tarda unas 3,5 veces más en renderizar que el estilo de carteles.

## Vídeos cuidados: lee la skill `videos-hyperframes`

Para un vídeo que Hugo vaya a publicar, **usa HyperFrames**, no `render.py` (que queda
para pruebas rápidas). Es HTML renderizado a MP4, va por `npx hyperframes <comando>`,
renderiza en local gratis y es mucho más expresivo.

**Antes de tocar la voz, el guion, el `index.html`, el sonido o de renderizar, invoca la
skill `videos-hyperframes`.** Ahí está todo: el flujo paso a paso, las reglas visuales que
Hugo ha pedido explícitamente, la escala de volúmenes de la mezcla y el masterizado. No
improvises ninguna de esas cosas de memoria: son unas 180 líneas de detalle que se
aprendieron a base de equivocarse, y saltárselas se nota en el resultado.

Las skills de HyperFrames que hacen falta: `hyperframes` (el contrato), `hyperframes-cli`
(comandos), `hyperframes-registry` (bloques ya hechos) y `media-use` (sonido).

## Sector del canal: IA y Claude para la comunidad tech hispanohablante

Hugo ha enfocado el canal en este sector (23/10/2026) para poder monetizarlo después
desde una web vinculada a su Instagram. Cambia una cosa de forma radical respecto a los
vídeos de curiosidades:

- **NO ESCRIBAS NI UN DATO DE MEMORIA.** El conocimiento del modelo llega a mayo de 2026
  y este sector va de la actualidad. Toda versión, fecha, precio, nombre de función o
  límite de plan **se verifica antes de escribir el guion**, con `WebSearch` y la skill
  `last30days`. Si no se puede verificar, no entra en el vídeo.
- **Ve a la fuente oficial, no a los blogs.** Al buscar novedades de octubre de 2026
  salieron tres fuentes con tres fechas distintas para la misma versión. Para versiones y
  funciones: el changelog oficial y `code.claude.com`. Los recopilatorios de blogs sirven
  para saber de qué se habla, no como dato.
- **Un dato mal hunde el canal.** En curiosidades un error pasa desapercibido; aquí la
  audiencia es técnica, lo detecta y lo dice en comentarios. La credibilidad se quema una
  sola vez. Ante la duda, quita la cifra y deja la idea.
- **Di lo que no sabes.** Si una cifra no se pudo confirmar, avísale a Hugo **antes** de
  renderizar, no después. Es mejor un guión con un dato menos que uno con un dato falso.
- **Caduca rápido.** Un vídeo de "lo nuevo de este mes" sirve semanas. Antes de reutilizar
  un guión viejo o volver a publicar, comprueba si sigue siendo verdad.
- **El diferencial de Hugo es el "cómo lo hago", no los "top 5".** Listas de trucos de
  Claude hay cientos; un pipeline real que genera vídeos y que se puede enseñar
  funcionando, no. Los vídeos de trucos traen alcance; los de "cómo hago estos vídeos"
  son los que venden. Aprovéchalo: cuando el tema lo permita, enséñale este proyecto.

## Cuando Hugo pida un vídeo ("crea un vídeo sobre...")

1. **Pregunta solo lo que falte**: tema y, si no está claro, duración. Por defecto:
   vertical para redes, unos 40 s, y la voz de **Enzo** (`4L2A19qwD3pMcNlt484F`).
   Si el vídeo es un **diálogo** (el búho explicándole a un personaje, que es el formato
   que mejor funciona), el personaje lleva la voz de **Sara Martin**
   (`KHCvMklQZZo0O30ERnVn`): se pone una `"voz"` por escena en el guion.
2. **Verifica los datos antes de escribir** si el tema es del sector de arriba (IA,
   Claude, actualidad tech). Ningún dato de memoria.
3. **Lee `instrucciones-guion.md` antes de escribir** el guion. Esta lectura es obligatoria:
   contiene las reglas sobre ganchos virales, retención y estructura que dan forma a cada
   guion. No las improvises de memoria.
4. **Decide la categoría**: 1 a 3 palabras, minúsculas, con guiones, sin acentos ni eñes.
   Mira primero las carpetas de `videos/` y reutiliza una si encaja. Las que hay:
   `crear-con-ia` (el sector actual), `ciencia-curiosa`, `curiosidades-de-la-mente`.
5. **Escribe el guion** aplicando las reglas recién leídas (ojo: la tabla de palabras
   depende de la voz, Enzo lee a 2,3 palabras/segundo) y guárdalo en
   `guiones/<nombre-corto>.json`:
   `{"titulo", "categoria", "formato", "paleta", "voz", "escenas": [{"tipo", "titulo", "texto", "numero", "puntos", "narracion"}]}`
6. **Si el vídeo es para publicar** (lo normal), invoca la skill `videos-hyperframes` y
   sigue su flujo. **No uses `render.py` para un vídeo que Hugo vaya a subir.**
7. **Solo para una prueba rápida y descartable**, `render.py` sirve:
   `py render.py guiones/<nombre>.json videos/<categoria>/<nombre>.mp4`
8. **Al entregar**, di:
   - En qué categoría lo guardaste (y si era nueva).
   - **Qué tipo de gancho usaste** (curiosidad, contraintuitivo, error común, dato impactante o historia breve).
   - Recuérdale que lo abra **desde el Explorador de Windows**: la vista previa de VS Code no reproduce el sonido.
   - Ofrécele cambiar textos, colores o voz.
