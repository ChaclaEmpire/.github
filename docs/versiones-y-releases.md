# Versiones y releases

Estándar de ChaclaEmpire para numerar versiones, registrar cambios y publicar releases. Está pensado para videojuegos (Unity),
pero vale para cualquier repo que entregue algo a usuarios. Nace de lo aprendido en Hero Duel CCG (TCGMaster) y Beyond
Destruction.

## 1. Tres números, cada uno con su trabajo

| Número | Formato | Dónde vive | Sube cuando… | Quién lo ve |
|---|---|---|---|---|
| **Versión** | `MAJOR.MINOR.PATCH` (SemVer) | Unity: *Player Settings → Version* (`bundleVersion`) | se publica un release | el jugador |
| **Protocolo** (solo juegos en línea) | entero (`1`, `2`…) | una constante compartida por cliente y servidor | el cliente y el servidor dejan de entenderse | el jugador, solo si tiene que actualizar |
| **Build** (solo tiendas) | entero creciente | `AndroidBundleVersionCode`, *Build* de iOS | con cada build que se sube a una tienda | la tienda |

No se mezclan. Un parche puede no tocar el protocolo y un cambio de protocolo no obliga a subir el MAJOR.

## 2. La versión del juego

- **Una sola fuente:** `bundleVersion`. La interfaz la lee con `Application.version` (menú principal, pantalla de carga, pantalla
  de error). Nunca se escribe a mano en otro sitio. Si el juego es en línea, se muestra junto al protocolo: `v0.9.0 · p5`.
- **Qué sube cada parte:**
  - **PATCH** (`0.9.0 → 0.9.1`): correcciones y detalles que no cambian cómo se juega.
  - **MINOR** (`0.9.1 → 0.10.0`): novedades. En un juego, eso incluye contenido nuevo (cartas, niveles, personajes), modos,
    pantallas y **cambios de balance o de reglas**: aunque sean pocas líneas, el jugador lo nota al jugar.
  - **MAJOR**: `1.0.0` es el lanzamiento público. Después, solo en un cambio que rompa el progreso o las partidas guardadas del
    jugador o lo obligue a empezar de nuevo.
- **Antes de la 1.0** (`0.x`) no hay promesa de estabilidad: es desarrollo, demos y pruebas cerradas.
- **La versión se decide al publicar**, no en cada PR. Los PR solo añaden su línea al changelog en «Sin publicar» (punto 5); el
  release sube `bundleVersion` y convierte esa sección en la versión. Así no hay conflictos en `ProjectSettings.asset`.
- **Pre-releases** con sufijo SemVer: `0.9.0-beta.1`, `0.9.0-rc.1`. En Unity se escribe igual en `bundleVersion`.

## 3. El protocolo (juegos en línea)

Es la puerta entre cliente y servidor: si no coinciden, el juego dice «actualiza el juego» en vez de fallar de formas raras.

- **Sube** cuando una versión vieja del cliente ya no puede jugar con el servidor nuevo, o al revés:
  - cambia un contrato (una respuesta, un campo, un enum que el cliente deserializa de forma estricta);
  - en juegos con servidor autoritativo que **repite las partidas desde su registro** (Beyond), cualquier cambio de reglas que
    haga que una partida se repita distinto.
- **El servidor acepta un rango** `MinProtocol..CurrentProtocol`. Subir solo `CurrentProtocol` mantiene jugando a los clientes
  viejos; subir `MinProtocol` obliga a actualizar. Hay que escribirlo en el changelog: «actualización obligatoria».
- **Servidor y cliente se publican juntos, el servidor primero.** Nunca se reparte un cliente que exige un protocolo que el
  servidor desplegado no acepta. El script de release lo comprueba antes de publicar (punto 8).
- Comentario en la constante: qué cambió en cada número (como `MatchRecord.CurrentProtocol` en Beyond).

## 4. Etiquetas y ramas

- **Etiqueta por release:** `vMAJOR.MINOR.PATCH` (con sufijo si es pre-release), anotada, sobre el commit de `main` que se
  publica, y siempre igual a `bundleVersion`. Ejemplo: `v0.9.0`, `v0.9.0-beta.1`.
- Las etiquetas no se mueven ni se borran: un error se corrige con otra versión.
- Los releases salen de `main`. Solo si hay que corregir una versión ya publicada mientras `main` avanza, se crea
  `release/0.9` desde su etiqueta y de ahí sale `v0.9.1`; la corrección vuelve a `main` por PR.
- Las ramas de trabajo siguen `CONTRIBUTING.md` (`tipo/descripcion-corta`, en inglés).

## 5. Changelog

`CHANGELOG.md` en la raíz del repo, en español y escrito **para el jugador**: qué hay de nuevo, qué se corrigió, qué cambió en
el balance. Las notas técnicas van en los PR, los commits o un registro técnico aparte (`docs/notas/` u otro).

```markdown
# Changelog

## [Sin publicar]

### Novedades
- …
### Cambios
- …
### Correcciones
- …

## [0.9.0] - 2026-11-15

Protocolo 5 (compatible con 0.8.x).

### Novedades
- …
```

- Cada PR que el jugador note añade su línea en **[Sin publicar]**. Si el PR no se nota jugando (herramientas, pruebas,
  refactor), no la añade.
- Secciones posibles: **Novedades**, **Cambios** (incluye balance), **Correcciones**, **Eliminado**. Las vacías no se ponen.
- Al publicar, «[Sin publicar]» pasa a `[X.Y.Z] - AAAA-MM-DD` con una línea sobre el protocolo si el juego es en línea.

## 6. GitHub Releases

- **Título:** `<Juego> vX.Y.Z` (p. ej. `Beyond Destruction v0.9.0`).
- **Notas:** la sección de esa versión del changelog, más el protocolo y si el servidor ya está desplegado.
- **Archivos:** `<Juego>-<versión>-<Plataforma>.zip` (`BeyondDestruction-0.9.0-Windows.zip`), cada uno con su `LEEME.txt`, más
  `SHA256SUMS.txt` con la suma de cada ZIP.
- **Pre-release:** marcarlo solo si la versión tiene sufijo (`-beta.1`, `-rc.1`). Las `0.x` sin sufijo son releases normales:
  así «Latest» siempre apunta a la última demo.
- Nada de builds en el repo: los ejecutables viven en los releases (`Builds/` va en `.gitignore`).

## 7. La receta de cada release

1. `main` limpio y al día; pruebas en verde (las de la CI y las del editor de Unity).
2. **Servidor** (si es en línea): si cambió el protocolo o el código del servidor, desplegarlo primero y comprobar que acepta el
   protocolo del cliente.
3. Subir `bundleVersion` y convertir «[Sin publicar]» en la versión, en un commit `Release vX.Y.Z` por PR.
4. Build de cada plataforma **desde una copia limpia**, no desde la carpeta en la que se trabaja.
5. Prueba de humo sobre el ejecutable, no sobre el editor: arranca, se juega una partida y, si es en línea, se conecta.
6. Etiqueta y release con los ZIP y `SHA256SUMS.txt`.
7. Avisar al equipo con el enlace.

Cada repo automatiza esta receta con un script `tools/release.ps1` (o `Tools/`), que se niega a publicar si falla un paso
(pruebas, servidor sin desplegar, etiqueta que ya existe, rama sucia). La primera vez, y siempre que se dude, se ejecuta con
`-DryRun`. Implementación de referencia: `Tools/release.ps1` de
[beyond-destruction-tcg](https://github.com/ChaclaEmpire/beyond-destruction-tcg) (PowerShell, Unity en modo batch desde una
copia aislada, comprobación del protocolo contra UGS, ZIP con `LEEME.txt` y `SHA256SUMS.txt`).

## 8. Lo que GitHub no impone (todavía)

Las reglas de GitHub (*rulesets*: quién crea etiquetas `v*`, prohibir borrarlas, exigir CI antes de fusionar en `main`) necesitan
GitHub Team o que el repo sea público. Con los repos privados del plan gratuito, este estándar se cumple **por convención y por
el script de release**. Si la organización pasa a Team, activar:

- ruleset de etiquetas `v*`: solo administradores las crean; nadie las borra ni las mueve;
- ruleset de `main`: PR obligatorio, CI en verde y una aprobación.

## 9. Adoptarlo en un proyecto que ya existe

1. Poner en `bundleVersion` la versión real (la del último release publicado) y mostrarla en el menú con `Application.version`.
2. Crear `CHANGELOG.md` en la raíz con «[Sin publicar]» y, si se quiere, el historial de los releases anteriores. Si ya hay un
   changelog para jugadores en otra ruta, se mueve a la raíz (o se deja un `CHANGELOG.md` que enlace a él).
3. Las etiquetas antiguas no se renombran: el estándar empieza en la próxima versión. El changelog puede anotar «publicada como
   `v0.8-demo`».
4. Escribir el script de release y probarlo con `-DryRun` antes del primer release real.
5. Juegos en línea: protocolo como rango `MinProtocol..CurrentProtocol` y comprobación del servidor en el script.
6. Repos que viven en **Plastic SCM** (con GitHub como espejo): la etiqueta de cada release es un *label* de Plastic con el mismo
   nombre (`v0.9.0`), y el espejo la replica como etiqueta de git. El release de GitHub se crea igual.

## 10. En los PR

La plantilla de PR pregunta:

- ¿El jugador nota el cambio? → línea en `CHANGELOG.md`, en «[Sin publicar]».
- ¿Rompe la compatibilidad entre cliente y servidor? → subir el protocolo en este PR y decir en el changelog si la actualización
  es obligatoria.
