# Directorio Monteleva — instrucciones para Claude

## Regla: registrar overrides manuales sin excepción

Cada vez que se haga un cambio manual al directorio que contradiga lo que dice el CSV (baja de un vecino, quitar descuento a petición, fusionar entradas duplicadas, etc.), se debe registrar en `data/overrides.json` **antes de hacer commit**.

Esto es crítico porque cuando Karen mande un nuevo CSV, el proceso de actualización lee `overrides.json` para no revertir esos cambios.

**Sin excepción. Si se hace el cambio, se registra el override.**

## Flujo para procesar un CSV nuevo

Ver skill `.claude/skills/procesar-csv-directorio/SKILL.md`.
