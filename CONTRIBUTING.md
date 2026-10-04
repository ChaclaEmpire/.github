# Cómo trabajamos

## Flujo

1. Todo trabajo parte de un **issue**.
2. Crea una rama desde `main`: `tipo/descripcion-corta` (`feat/login`, `fix/save-crash`, `docs/readme`), en inglés.
3. Commits pequeños y con mensaje claro, en imperativo: `Agrega pantalla de login`.
4. Abre un **Pull Request** hacia `main` enlazando el issue (`Closes #12`).
5. Otra persona lo revisa antes de fusionarlo. Nadie aprueba su propio PR.
6. Se fusiona con *squash* y se borra la rama.

## Reglas

- No se hace push directo a `main`.
- Nunca se suben claves, tokens ni archivos `.env`. Usa `.env.example` para documentar variables.
- Si dudas del alcance de algo, pregunta en el issue antes de escribir código.

## Idioma

**Todo el código en inglés. Toda la documentación en español, respetando los nombres de código en inglés.**

- **Código en inglés:** identificadores, comentarios, mensajes de error y de log, nombres de pruebas, claves de configuración, flags de CLI, rutas de API, columnas de base de datos y nombres de ramas.
- **Documentación en español:** README, `docs/`, notas, issues, descripciones de PR y mensajes de commit.
- **Los nombres de código no se traducen** en la documentación: se escriben tal cual, entre comillas invertidas. Se dice "la función `getUserById`", no "`obtenerUsuarioPorId`".
- Los términos técnicos de uso común se dejan en inglés (*pull request*, *commit*, *merge*, *hook*). Los nombres propios y términos de dominio sin equivalente se escriben tal cual.
- **El texto que ve el usuario del producto** no es código ni documentación: va en el idioma de sus usuarios, en archivos de recursos con las claves en inglés (`login.title`).

Los proyectos creados desde `plantilla-software` ya traen esta regla en `docs/convenciones.md`, `AGENTS.md` y `CLAUDE.md`, y los agentes la aplican.
