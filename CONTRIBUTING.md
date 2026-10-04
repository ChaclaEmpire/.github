# Cómo trabajamos

## Flujo

1. Todo trabajo parte de un **issue**.
2. Crea una rama desde `main`: `tipo/descripcion-corta` (`feat/login`, `fix/crash-al-guardar`, `docs/readme`).
3. Commits pequeños y con mensaje claro, en imperativo: `Agrega pantalla de login`.
4. Abre un **Pull Request** hacia `main` enlazando el issue (`Closes #12`).
5. Otra persona lo revisa antes de fusionarlo. Nadie aprueba su propio PR.
6. Se fusiona con *squash* y se borra la rama.

## Reglas

- No se hace push directo a `main`.
- Nunca se suben claves, tokens ni archivos `.env`. Usa `.env.example` para documentar variables.
- Si dudas del alcance de algo, pregunta en el issue antes de escribir código.
