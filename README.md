# Forja de Retratos

Generador de retratos de personajes para un juego de supervivencia y fantasía. Todo se pinta con un
*fragment shader* de WebGL y ruido procedural: no hay imágenes ni dependencias. La misma semilla
produce siempre el mismo personaje.

Todo vive en un único archivo: `index.html`.

## Ejecutarlo

- **En línea:** https://claude.ai/artifact/JHReMqE2QrvLeUbRX7R8yN (privado; se comparte desde el
  menú *Share* de la página). Es el mismo `index.html` sin las etiquetas `<html>`, `<head>` y
  `<body>`, que pone claude.ai.
- **Local:** abre `index.html` en el navegador (doble clic). Si tu navegador bloquea WebGL en
  `file://`, sirve la carpeta: `python3 -m http.server` y entra en `http://localhost:8000`.
- **Enlace directo a una semilla:** `index.html#seed=kaen&race=dragon&cls=mage&hair=6&hat=2`.
  Claves: `seed`, `race`, `sex`, `age`, `cls`, `hair` (id de peinado), `hat` (id de sombrero).
  En la versión publicada como artefacto de claude.ai solo funciona la semilla sola: `...#kaen`.
- **Botón PNG:** descarga el retrato grande a 512×640.

## Cómo funciona

```
semilla (texto)
  └─ xmur3 → mulberry32            PRNG determinista (index.html, genCharacter)
       └─ genCharacter(seed, overrides)
            ├─ elige raza, sexo, edad, clase, equipo, peinado, ojos, colores…
            ├─ aplica overrides (selectores / presets) y DISABLED
            └─ devuelve p (parámetros) + colors
  └─ renderTo(): empaqueta p en uQ[13] (vec4) y los colores en uniforms vec3
       └─ shader: pinta por capas (fondo → objetos detrás → pelo trasero → cuello → ropa
                  → objetos delante → orejas → cabeza/cara → barba/máscara → pelo → sombrero
                  → capucha → cuernos → brillos → acabado pictórico)
```

## Combinaciones

La página muestra el recuento bajo el título. Hoy son **≈ 1,2 × 10¹³ combinaciones de rasgos**
(11.933.873.233.920). El recuento se calcula en `countCombinations()`, que respeta los kits de cada
clase y lo que desactives en `DISABLED`, así que se actualiza solo. El desglose:

| Bloque | Opciones | Qué incluye |
|---|---|---|
| Raza | 27 | humano, elfo, duende, y dragonoide × 4 cuernos × 3 orejas × escamas sí/no |
| Sexo | 3 | mujer, hombre con barba, hombre sin barba |
| Edad | 3 | joven, adulto, anciano |
| Clase + ropa + sombrero + objeto | 367 | solo las combinaciones que permite cada clase (`KITS`), con estampados de kimono y cuello de piel |
| Peinado | 121 | 13 peinados × 5 adornos × ahoge sí/no (el rapado cuenta 1) |
| Ojos | 60 | 5 formas × 3 pupilas × heterocromía × brillo |
| Cara | 768 | gafas/monóculo/parche, máscara, 6 marcas, cicatriz, pecas, lunar, colmillo |
| Cuerpo | 8 | amuleto, pendientes, bufanda |
| Luz | 3 | hoguera, luna, arcana |

No cuenta los colores (piel, pelo, ojos, ropa, metal, escamas…) ni las proporciones continuas de la
cara (ancho, mandíbula, separación de ojos, tamaño de nariz…), que multiplican la variedad hasta
hacerla prácticamente infinita. Con una configuración fija de selectores hay 2³² = 4.294.967.296
semillas distintas.

## Catálogo de ids

Los ids son el índice en cada lista de `index.html`. Se usan en `DISABLED`, en `KITS`, en los
presets y en los selectores.

| Categoría (clave en `DISABLED`) | Lista en el código | Ids |
|---|---|---|
| `races` | `RACE_ID` | `'human'`, `'elf'`, `'dragon'`, `'duende'` |
| `classes` | `KITS` | `'mage'`, `'warrior'`, `'ranger'`, `'survivor'` |
| `hairStyles` | `HAIR_STYLES` | 0 rapado · 1 corto · 2 largo · 3 moño · 4 hime · 5 odango · 6 coletas · 7 cola alta · 8 bob · 9 en punta · 10 trenza lateral · 11 moño samurái · 12 largo con flequillo |
| `hats` | `HATS` | 0 ninguno · 1 mago · 2 boina · 3 capucha · 4 diadema · 5 yelmo · 6 kasa · 7 hachimaki · 8 kabuto · 9 gorro con pluma · 10 gorro de piel · 11 corona de flores |
| `outfits` | `LABELS.outfit` | 0 túnica · 1 placas · 2 cuero · 3 pieles · 4 kimono · 5 samurái |
| `props` | `LABELS.prop` | 0 ninguno · 1 bastón · 2 espada · 3 carcaj · 4 katana · 5 arco · 6 escudo · 7 grimorio · 8 orbe · 9 hacha · 10 lanza |
| `eyeShapes` | `LABELS.eyeShape` | 0 normales · 1 afilados · 2 caídos · 3 grandes · 4 somnolientos |
| `pupils` | `LABELS.pupil` | 0 redonda · 1 reptil · 2 estrella |
| `ornaments` | `LABELS.ornament` | 0 ninguno · 1 lazo · 2 kanzashi · 3 horquillas · 4 flor |
| `marks` | `LABELS.mark` | 0 ninguna · 1 banda · 2 garras · 3 línea · 4 marca mística · 5 tatuaje élfico |
| `horns` | `LABELS.horns` | 0 ninguno · 1 hacia atrás · 2 altos · 3 cortos · 4 astas |
| `glasses` | `LABELS.glasses` | 0 ninguna · 1 gafas redondas · 2 monóculo |
| `patterns` | `LABELS.pattern` | 0 liso · 1 seigaiha · 2 ichimatsu · 3 sakura |
| `features` | nombres de `p` | `amulet` `earrings` `eyepatch` `glasses` `mask` `scarf` `fang` `mole` `freckles` `scar` `hetero` `glowEyes` `ahoge` `fur` `beard` `scales` |

## Deshabilitar elementos

Edita el objeto `DISABLED` al principio del generador (busca `// ---- Generator switches`):

```js
const DISABLED = {
  races: ['duende'],       // no más duendes
  hats: [8, 10],           // sin kabuto ni gorro de piel
  props: [0],              // todos llevan algún objeto
  hairStyles: [9],         // sin pelo en punta
  features: ['mask', 'eyepatch'],
  // …el resto de claves se deja en []
};
```

- Lo desactivado deja de generarse, desaparece de los selectores y el recuento de combinaciones se
  actualiza.
- Si desactivas todas las opciones de una categoría, se usa `FALLBACK` (normalmente "ninguno").
- Los presets fijados (la maga *sakura*) guardan sus valores exactos e ignoran `DISABLED`.
- Cambiar `DISABLED` cambia los personajes de muchas semillas (se reparten las probabilidades).
  Si un personaje concreto te gusta, fíjalo antes como preset (ver "Fijar un personaje").

### Ajustar probabilidades en vez de desactivar

- **Equipo por clase:** en `KITS` cada opción es `[id, peso]`. Sube el peso para que salga más,
  bájalo para que salga menos; peso `0` equivale a desactivarlo solo para esa clase.
- **Raza, peinado, forma de ojos, cuernos, adornos:** listas `wpick([[id, peso], …])` dentro de
  `genCharacter`.
- **Accesorios sí/no:** valores `chance(x)` en el objeto `p` de `genCharacter` (por ejemplo
  `scarf`, `mask`, `fang`, `freckles`). `x` es la probabilidad entre 0 y 1.

## Añadir elementos nuevos

### Mapa de coordenadas para dibujar

El shader trabaja en unidades donde la altura del cuadro es 0.84 (`q.y` va de −0.42 a 0.42 y
`q.x` de −0.336 a 0.336). `q` es la posición con la deformación de pincel; úsala para las formas.

| Referencia | Valor |
|---|---|
| Centro de la cabeza `hc` | (0, 0.05) |
| Radios de la cabeza `hr` | ≈ (0.128, 0.172) × `HEAD_W`/`HEAD_H` |
| Coordenadas de cara `fq = q - hc` | ojos en `fq.y = 0` (x = ±`EYE_SEP`), boca en −0.108, barbilla ≈ −0.17, frente ≈ 0.06–0.12 |
| Hombros | parte alta del cuerpo en `q.y` ≈ −0.22; elipse `bc = (0, -0.53)`, `br = (0.46·BODY_W, 0.31)` |
| Lado del objeto | `PS` = ±1 (`PROP_SIDE`), los objetos van en `x ≈ PS * 0.26…0.32` |

Funciones de dibujo disponibles en el shader:

- **Formas (distancia con signo):** `sdE` (elipse), `sdTaper` (segmento con grosor variable),
  `sdBox`, `sdRhomb`, `sdFlower`, `sdStar5`, `sdBow`.
- **Máscaras:** `fill(d, suavizado)` convierte una distancia en máscara. `band(v, a, b)` vale 1 cuando `a < v < b`.
- **Iluminación:** `shade(color, normal, L, jit)` es la luz pictórica (luces cálidas, sombras frías).
  `metal(...)` da brillo especular, `gem(...)` pinta gemas mágicas, `lamellar(...)` la armadura
  samurái y `scaleSkin(...)` las escamas.
- **Normales:** `ellN(p, radios)` da la normal de una elipse para sombrear volúmenes.
- **Brillo:** suma a `addGlow` lo que deba brillar; se aplica al final.

Patrón típico de una capa:

```glsl
if (eq(HAT, 12.0)) {
  vec2 cq = q - (hc + vec2(0.0, hr.y * 0.9));          // posición local
  float d = sdE(cq, vec2(hr.x * 1.1, 0.05));            // forma
  vec3 c = shade(uHat, ellN(cq, vec2(hr.x * 1.1, 0.05)), L, jit);  // color con luz
  col = mix(col, c, fill(d, soft));                     // pintar encima
}
```

### Sombrero nuevo

1. Añade el nombre al final de `HATS` en el JS; su índice es el nuevo id (p. ej. 12).
2. En el shader, dentro del bloque `// ---- hats`, añade `else if (eq(HAT, 12.0)) { … }`.
3. Si tapa toda la cabeza (como el yelmo), añádelo a `topCovered` para ocultar el pelo de arriba y
   las orejas.
4. Añádelo con un peso a las clases que lo usen, en `KITS[clase].hats`.
5. Si necesita un color propio, decídelo donde se calcula `hatC` en `genCharacter`.
6. Si debe contar en el recuento, no hace falta nada más: `countCombinations()` lee `KITS`.

### Objeto nuevo (arma, libro, farol…)

1. Añade el nombre al final de `LABELS.prop` (nuevo id).
2. En el shader dibújalo con `if (eq(PROP, 11.0)) { … }`: en el bloque
   `// ---- props behind the body` si va a la espalda, o en `// ---- props in front of the body`
   si se sostiene delante.
3. Añádelo a `KITS[clase].props` con su peso.

### Ropa nueva

1. Añade el nombre a `LABELS.outfit` (nuevo id).
2. En el shader, en `// ---- body / outfit`, añade una rama `else if (eq(OUTFIT, 6.0))` **antes**
   del `else` final (que es la armadura samurái). Pinta sobre `dBody`, y pon `brooch = false` si no
   quieres el broche.
3. Añádela a `KITS[clase].outfits`.

### Peinado nuevo

Muchos peinados se construyen combinando piezas que ya existen. Añade una entrada a `HAIR_STYLES`:

```js
{ name: 'Media melena', fringe: 1, sideIn: 0.78, side: () => -0.09, back: 3, extra: 0 },
```

- `fringe`: −1 sin pelo · 0 flequillo de lado · 1 recto · 2 en punta · 3 peinado hacia atrás.
- `side`: hasta dónde bajan los mechones laterales (`fq.y`). `sideIn`: dónde empiezan (fracción de `hr.x`).
- `back`: pelo detrás de la cabeza (0 nada · 1 suelto · 2 cortina recta · 3 bob).
- `extra`: 0 nada · 1 moño · 2 odango · 3 coletas · 4 cola alta · 5 trenza · 6 moño samurái.

Luego añádelo con un peso a las listas `hairStyle` de `genCharacter` (mujer y/o hombre). Si
necesitas una pieza nueva (p. ej. `extra: 7`, un moño con palillos), dibújala en el shader en
`// ---- hair on top` con `if (eq(EXTRA, 7.0))`.

### Parámetro nuevo (un accesorio, un rasgo)

1. Añade la clave al final de `PARAMS`. Cada 4 claves ocupan un `vec4` de `uQ`: la clave número
   `i` se lee en el shader como `uQ[i / 4]` componente `.x/.y/.z/.w` (`i % 4`). Hay 52 huecos
   (`uQ[13]`) y hoy se usan 49. Si necesitas más, sube `uQ[13]` en el shader y
   `new Float32Array(52)` en `genCharacter`.
2. Declara un `#define` en el shader, p. ej. `#define TIARA uQ[12].y`.
3. Dale valor en el objeto `p` de `genCharacter`, p. ej. `tiara: chance(0.1) ? 1 : 0`.
4. Añade la etiqueta a la lista `flags` (o a `LABELS`) para que salga en los rasgos.
5. Si es un sí/no, puedes desactivarlo desde `DISABLED.features` sin más cambios.

### Color nuevo

1. Añade la clave a `COLOR_KEYS` (p. ej. `'gold'`).
2. Declara `uniform vec3 uGold` junto a los demás uniforms del shader.
3. Dale valor en el objeto `colors` de `genCharacter`.

### Raza nueva

1. `RACE_ID` (nuevo id numérico), `LABELS.race` (nombres masculino y femenino),
   `NAME_PARTS` (sílabas de nombres) y una `<option>` en el selector `selRace`.
2. En `genCharacter`: su paleta de piel y sus rangos en `headW`, `jawBase`, `eyeBase`, `noseBase`,
   el tipo de oreja (`ear`) y el peso en la lista de razas.
3. Si tiene rasgos propios (cola, antenas…), añade un parámetro y dibújalo en el shader.

### Clase nueva

1. Añade una entrada a `KITS` con `outfits`, `hats`, `props` y `amulet`.
2. Añade sus nombres a `LABELS.cls`, una `<option>` en `selCls` y su color de ropa en `cloth`
   dentro de `genCharacter`.

### Fijar un personaje (preset)

Para que un personaje no cambie aunque modifiques el generador, haz como con *sakura*
(`SAKURA_PIN`): guarda su objeto `p` completo, sus `colors`, su `noiseSeed`, nombre y tipo de luz, y
añádelo como variante con `pin` en `PRESETS`. Para obtener esos valores, ejecuta
`genCharacter('semilla', overrides)` en la consola del navegador y copia el resultado.

## Comprobaciones al cambiar el generador

- Abre la página y mira la consola: si el shader no compila, el error aparece junto al nombre.
- Comprueba que *sakura* sigue igual (primera miniatura de "Maga de pelo rosa").
- Prueba el elemento nuevo con los selectores o con `renderTo(canvas, 'semilla', { set: { hat: 12 } })`
  desde la consola.
