# Quick Flow — NexusAgil

> Triage maneja cambios triviales que no merecen el pipeline completo.
> Activar con: "quick flow", "cambio trivial", "patch rapido".

---

## Cuando aplica Quick Flow

Quick Flow es para cambios que cumplen TODAS estas condiciones:

| Criterio | Limite |
|----------|--------|
| Archivos afectados | 1-2 |
| Lineas de cambio | <30 |
| Base de datos | No toca BD |
| Logica nueva | No crea logica nueva |
| Seguridad | No afecta auth/permisos |
| Tests | No requiere tests nuevos |

### Ejemplos que SI califican
- Corregir un typo en un label
- Cambiar un color o spacing
- Actualizar texto estatico
- Corregir una URL hardcodeada
- Ajustar un padding/margin
- Cambiar un icono por otro del mismo set

### Ejemplos que NO califican
- Agregar un campo a un formulario (logica de validacion)
- Cambiar un flujo de navegacion (multiples archivos)
- Corregir un bug con logica condicional (requiere test)
- Cualquier cambio que toque la base de datos
- Cambios que afectan autenticacion/autorizacion

---

## Qualification Check

Triage ejecuta este check antes de aceptar un cambio en Quick Flow:

```
QUICK FLOW QUALIFICATION:
[ ] Maximo 2 archivos afectados
[ ] Maximo 30 lineas de cambio
[ ] No toca base de datos
[ ] No crea logica nueva (sin if/else, sin loops, sin calculos)
[ ] No afecta autenticacion ni autorizacion
[ ] No requiere tests nuevos
[ ] El cambio es verificable con typecheck/build (sin test manual)
```

**Si CUALQUIER check falla**: Triage escala a pipeline completo (F0).

---

## Pipeline Abreviado (4 pasos)

### Paso 1: Intake rapido

```markdown
## Quick Flow — [titulo corto]

| Campo | Valor |
|-------|-------|
| **Tipo** | patch |
| **Objetivo** | [1 oracion] |
| **Archivos** | `[path1]`, `[path2]` |
| **Cambio** | [descripcion en 1-2 oraciones] |
```

Triage presenta al humano. Si el humano aprueba, continuar.

### Paso 2: Codebase Grounding ligero

1. Leer los archivos que se van a modificar
2. Verificar que el cambio es tan simple como se espera
3. Si descubre complejidad oculta: **UPGRADE** a pipeline completo

### Paso 3: Implementar + Verificar

1. Hacer el cambio
2. Ejecutar verificacion minima:
   - typecheck
   - build (si es rapido)
3. Si falla: corregir o escalar

### Paso 4: Cerrar

1. Actualizar `doc/sdd/_INDEX.md`:

```markdown
| NNN | YYYY-MM-DD | [titulo] | patch | quick-flow | DONE | patch/NNN-titulo |
```

2. Presentar resumen al humano:

```markdown
## Quick Flow Completado

- **Cambio**: [que se hizo]
- **Archivos**: `[path1]` (N lineas)
- **Verificacion**: typecheck PASS
- **Branch**: patch/NNN-titulo
```

---

## Regla de Upgrade

Triage DEBE escalar a pipeline completo si DURANTE el Quick Flow descubre que:

1. El cambio toca mas de 2 archivos
2. El cambio requiere mas de 30 lineas
3. Hay logica condicional involucrada
4. Necesita tocar base de datos
5. Afecta flujos de autenticacion
6. El typecheck/build falla por razones no triviales
7. El cambio tiene efectos secundarios no previstos

### Formato de Upgrade

```markdown
## UPGRADE: Quick Flow -> Pipeline Completo

**Razon**: [por que Quick Flow no es suficiente]
**Recomendacion**: SDD_MODE [mini/full/bugfix]
**Archivos descubiertos**: [lista de archivos que necesitan cambio]

Procediendo con F0...
```

---

## Reglas del Quick Flow

1. **Triage califica, no el humano**. El humano pide un cambio, Triage determina si califica.
2. **En caso de duda, escalar**. Es mejor usar el pipeline completo que romper algo con un quick fix.
3. **Sin SDD ni Story File**. Quick Flow no genera artefactos formales (solo entrada en _INDEX.md).
4. **Sin Adversarial Review**. Cambios triviales no necesitan AR (pero SI typecheck/build).
5. **El humano puede forzar pipeline completo**. Si dice "usa el pipeline completo", Triage obedece.
6. **El humano puede forzar Quick Flow**. Si dice "solo hazlo rapido" y Triage califica, proceder.
7. **Auto-Blindaje aplica**. Si un Quick Flow causa un error, documentar.
