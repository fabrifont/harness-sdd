# Arquitectura — Qué significa "hacer un buen trabajo"

> Este documento define el estándar de calidad. Los agentes revisores
> evalúan código contra este archivo. Si no está aquí, no es un requisito.

## Principios

1. **Capas claras.** Define las capas de tu proyecto aquí. Ejemplo:
   - `storage.py` — persistencia.
   - `models.py` — modelo de dominio.
   - `api.py` / `cli.py` — interfaz de usuario.
   No introducir capas adicionales (servicios, repositorios, ORMs) hasta que
   haya una razón concreta documentada en `feature_list.json`.

2. **Dependencias controladas.** Lista las dependencias externas permitidas.
   Si una feature requiere una dependencia nueva, primero se discute
   (estado `blocked`).

3. **Errores explícitos.** Las funciones que pueden fallar lanzan
   excepciones nombradas, no devuelven `None`.

4. **Inmutabilidad por defecto.** Los datos de dominio preferentemente
   inmutables. Modificar = crear una nueva instancia.

5. **Atomicidad en disco.** Toda escritura a archivos críticos se hace
   primero en un archivo temporal y luego se renombra. Nunca dejar un
   archivo a medio escribir.

## Flujo de datos

```
usuario  ─→  interfaz (api/cli)
               │
               ├─ construye/valida entidades del dominio
               │
               └─→  capa de persistencia
                        │
                        └─→  almacenamiento (disco/db)
```

## Qué NO hacer

- No usar `print()` para errores. Usa `sys.stderr` y exit code != 0.
- No mezclar IO con lógica de dominio dentro de los modelos.
- No leer/escribir el mismo recurso en cada operación dentro de un bucle.
  Carga al inicio, modifica en memoria, guarda al final.
- No añadir un sistema de configuración sin justificación. Las rutas y
  parámetros se pasan explícitamente o usan constantes por defecto.
