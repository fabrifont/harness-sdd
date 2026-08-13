# Verificación — Cómo demostrar que el trabajo funciona

> Regla de oro: **el agente no dice "funciona", lo demuestra**.
> Toda feature termina con evidencia ejecutable, no con afirmaciones.

## Niveles de verificación

### Nivel 1 — Tests unitarios (obligatorio)

Toda función pública en `src/` tiene al menos un test en `tests/` que:

1. Cubre el camino feliz.
2. Cubre al menos un camino de error si la función puede fallar.

Comando:
```bash
{{TEST_COMMAND}}
```

### Nivel 2 — Tests de integración (obligatorio para features de UI/API)

Las features que añaden comandos o endpoints se verifican ejecutando la
interfaz real contra un entorno temporal:

```python
import subprocess, tempfile, os
with tempfile.TemporaryDirectory() as d:
    env = {**os.environ, "CONFIG_VAR": os.path.join(d, "config.json")}
    out = subprocess.check_output(
        ["{{ENTRY_POINT}}", "arg1", "arg2"],
        env=env, text=True,
    )
    assert "resultado_esperado" in out
```

*(Adapta este ejemplo al tipo de interfaz de tu proyecto.)*

### Nivel 3 — Smoke test manual (opcional pero recomendado)

Antes de cerrar la sesión, ejecuta un flujo end-to-end con datos temporales
en `/tmp`.

### Nivel 4 — Trazabilidad de requirements (obligatorio para features con `"sdd": true`)

Cada `R<n>` de `specs/<name>/requirements.md` debe poder mapearse a al
menos un test concreto en `tests/`. El reviewer rechaza si falta cobertura.

El implementer documenta el mapa en `progress/impl_<name>.md`:

```markdown
## Trazabilidad
- R1 → `test_feature_default_behavior`
- R2 → `test_feature_invalid_input`
- R3 → `test_feature_edge_case`
```

## Anti-patrones (no hacer)

- ❌ "He añadido el comando, debería funcionar." → falta test ejecutable.
- ❌ Test que solo verifica que la función no lanza excepción. → tiene que
  comprobar el resultado concreto.
- ❌ `mock` del filesystem o de la red cuando se puede usar un entorno real. →
  usa temporales y procesos reales.
- ❌ Marcar la feature como `done` sin pasar `./init.sh`.

## Verificación final antes de cerrar

```bash
./init.sh           # debe terminar con [OK] Entorno listo
```

Si `./init.sh` está rojo, **no** marques nada como `done`. Anota el bloqueo
en `progress/current.md` con estado `blocked` en `feature_list.json`.
