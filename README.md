# NexusAgil

> Scrum + Spec-Driven Development para equipos que trabajan con agentes de IA.

---

## ¿Qué es NexusAgil?

NexusAgil es una metodología de desarrollo ágil diseñada para equipos donde los ejecutores son agentes de IA. Toma lo mejor de Scrum como contenedor de trabajo, y agrega Spec-Driven Development como motor de ejecución.

**Principio central:**

> Un agente sin spec es un agente sin rumbo. Un spec sin validación adversarial es un spec optimista. NexusAgil exige ambos.

---

## La pregunta que responde

*¿Por qué Scrum puro no funciona con agentes de IA?*

Scrum asume ejecutores humanos que pueden improvisar, pedir clarificaciones, y adaptar el trabajo en tiempo real. Los agentes de IA no improvisan bien — necesitan instrucciones precisas o alucinarán para llenar los vacíos.

NexusAgil resuelve esto con dos capas:

1. **Scrum** — el contenedor: sprints, backlog, review, retro
2. **Spec-Driven Development** — el motor: nada se ejecuta sin un spec aprobado

---

## Los 5 Gates Formales

Todo trabajo pasa por gates explícitos. El gate bloquea la ejecución hasta recibir aprobación.

```
HU_APPROVED
    ↓
SPEC_APPROVED
    ↓
SPRINT_APPROVED
    ↓
REVIEW_APPROVED
    ↓
RETRO_APPROVED
```

### HU_APPROVED
El Product Owner aprueba la Historia de Usuario. Define QUÉ se construye y POR QUÉ importa.
Sin este gate, no se escribe ningún spec.

### SPEC_APPROVED
El Product Owner aprueba el Story-Driven Document (SDD). Define CÓMO se construye.
El SDD es la fuente de verdad absoluta para el agente ejecutor.
Sin este gate, el agente no toca código.

### SPRINT_APPROVED
El Product Owner aprueba el plan completo del sprint. Fija el scope, las prioridades y el orden de ejecución.
Sin este gate, no se lanza ningún sub-agente.

### REVIEW_APPROVED
El Product Owner aprueba los entregables del sprint después de la demo.
Sin este gate, el sprint no se cierra y el trabajo no se considera DONE.

### RETRO_APPROVED
El Product Owner aprueba la retrospectiva. Captura lecciones aprendidas y acciones para el siguiente sprint.
Cierra el sprint formalmente.

---

## Clasificación de Trabajo

No todo el trabajo necesita todos los gates. La clasificación determina el camino:

| Tipo | Descripción | Gates requeridos |
|---|---|---|
| **FAST-FIX** | Corrección puntual, alcance mínimo, riesgo bajo | SPRINT_APPROVED → ejecuta directo |
| **HU-MINOR** | Feature pequeña, bien entendida, sin impacto arquitectural | SPRINT_APPROVED → ejecuta directo |
| **HU-MAJOR** | Feature significativa, requiere diseño previo | HU_APPROVED → SPEC_APPROVED → ejecuta |
| **QUALITY** | Cambio arquitectural, seguridad crítica, o decisión técnica de alto impacto | Todos los gates |

---

## Spec-Driven Development (SDD)

El corazón de NexusAgil es el Story-Driven Document (SDD). Es el spec que el agente ejecutor seguirá al pie de la letra.

### Estructura de un SDD

```markdown
# Story File — SDD #NNN: <Título>
**Sprint N | WAS-XX**
**Classification: QUALITY / HU-MAJOR**
**Source of truth: this file only. Read every file before modifying.**

## Context
Por qué existe esta tarea. Qué problema resuelve.

## Acceptance Criteria
Lista numerada de condiciones verificables. Sin ACs = sin DONE.

## Wave 0 — Pre-flight (obligatorio)
Verificaciones antes de ejecutar. Ver sección "Wave 0" abajo.

## Wave 1 — <Nombre>
Instrucciones paso a paso. Código de referencia cuando aplica.
**Build gate:** verificar que el build pasa antes de continuar.

## Wave 2 — <Nombre>
...
**Build gate:** verificar que el build pasa antes de continuar.

## Wave N — Commit + Push
Comando exacto de git.

## Rollback
Instrucciones para revertir si el fix causa regresión.

## Critical Constraints
Reglas que el agente NO puede ignorar.
```

### Principios del SDD

1. **Una sola fuente de verdad** — el agente lee el SDD y solo el SDD
2. **Wave 0 siempre primero** — verificar estado actual antes de tocar código
3. **Doble validación** — SM valida el SDD, sub-agente re-valida antes de ejecutar
4. **Waves en orden estricto** — nunca saltarse pasos
5. **Leer antes de modificar** — el agente lee cada archivo antes de tocarlo
6. **Build gate por wave** — si el build falla al final de una wave, STOP y reportar
7. **Test gate obligatorio** — si los tests fallan, el agente se detiene y reporta
8. **Rollback documentado** — todo SDD debe incluir cómo revertir

---

## Wave 0 — Pre-flight (obligatorio en todo SDD)

Wave 0 tiene dos momentos de ejecución:
1. **El SM lo ejecuta** antes de SPEC_APPROVED (para validar el SDD)
2. **El sub-agente lo ejecuta** antes de Wave 1 (como defensa contra errores del SM)

Esta doble ejecución crea redundancia: si el SM alucina en el SDD, el sub-agente lo detecta antes de tocar código.

### Pasos de Wave 0

| Paso | Verificación | Si falla |
|------|-------------|----------|
| **0.1** | ¿El fix ya existe en el codebase? (`grep`/búsqueda del cambio propuesto) | Marcar SDD como `ALREADY_IMPLEMENTED`, no ejecutar |
| **0.2** | ¿Los archivos referenciados en las waves existen y tienen la estructura esperada? | Corregir rutas/firmas en el SDD |
| **0.3a** | ¿El código de referencia del SDD compila contra los tipos/ABI reales del proyecto? | Corregir el código antes de aprobar |
| **0.3b** | ¿El encoding de datos es correcto? (indexed events → keccak256, structs packed, mappings) | Corregir el código de referencia en el SDD |
| **0.3c** | Para smart contracts: ¿hay riesgos de overflow, reentrancy, o front-running no documentados? | Agregar al SDD o crear constraint |
| **0.4** | ¿Las dependencias entre SDDs están resueltas? (ej: WAS-166 depende de WAS-162) | Bloquear ejecución hasta que la dependencia esté DONE |
| **0.5** | ¿El SDD tiene TODOs, placeholders o ambigüedades? | Completar antes de aprobar |

### Instrucción para el sub-agente (incluir en el task)

```
Antes de ejecutar Wave 1, ejecuta Wave 0:
Lee el SDD completo. Luego verifica:
1. ¿Los archivos mencionados existen? Lee cada uno.
2. ¿El código de referencia es compatible con los tipos reales?
3. ¿El fix ya está implementado?
Si encuentras discrepancias, STOP y reporta. No ejecutes.
```

**¿Por qué existe Wave 0?**

Nació de la retro del Sprint de Quality Fixes (marzo 2026), donde:
- 2 de 6 SDDs resultaron ya implementados (tokens y tiempo desperdiciados)
- 1 SDD tenía un error de encoding (`indexed string` retorna keccak256, no el string directo) que el agente corrigió por suerte, no por proceso

Wave 0 convierte esas lecciones en un paso obligatorio y redundante del proceso.

---

## Build Gate por Wave

Cada wave debe terminar con una verificación de build. No es opcional.

```
[Al final de cada Wave]
1. Ejecutar build (ej: tsc --noEmit, forge build, npm run build)
2. Si FALLA → STOP, reportar error, NO continuar a la siguiente wave
3. Si PASA → continuar
```

**¿Por qué?** Un agente que commitea código roto al final de 5 waves desperdició todo el trabajo de las waves 2-5. Detectar en Wave 1 que algo está roto ahorra el 80% del tiempo.

---

## Rollback Plan

Todo SDD debe incluir instrucciones de rollback. El formato mínimo:

```markdown
## Rollback
Si este fix causa regresión:
1. `git revert <commit-hash>`
2. Archivos afectados: [lista]
3. Dependencias que se rompen al revertir: [lista o "ninguna"]
4. Para smart contracts: el contrato anterior sigue deployado en [dirección]
```

**¿Por qué?** Sin rollback plan, una regresión en producción requiere debugging bajo presión. Con rollback plan, es un comando.

---

## SDD Amendments

Si el PO pide un cambio después de SPEC_APPROVED, no se modifica el SDD silenciosamente. Se crea un amendment formal:

```markdown
## Amendment A1 — <Título>
**Solicitado por:** PO
**Fecha:** YYYY-MM-DD
**Cambio:** Descripción del cambio
**Waves afectadas:** Wave 2 (modificada), Wave 3 (nueva)
**Impacto:** [bajo/medio/alto]
**Aprobación:** AMENDMENT_APPROVED por PO
```

El amendment se agrega al final del SDD original. Mantiene trazabilidad completa.

**Regla especial:** Amendments a smart contracts siempre son impacto ALTO y requieren re-ejecución de Wave 0.3c (verificación de seguridad).

---

## Reglas de Paralelismo

Los sub-agentes pueden ejecutar en paralelo **solo si** se cumplen todas estas condiciones:

| Condición | Verificación |
|-----------|-------------|
| **Archivos disjuntos** | Los SDDs no tocan los mismos archivos (verificar en Wave 0.2) |
| **Sin dependencias de import** | Los cambios de un SDD no son importados por otro SDD |
| **Sin dependencias de orden** | Los SDDs no tienen relación `depende de` |
| **Mismo branch base** | Todos parten del mismo commit |

Si alguna condición no se cumple → **ejecución secuencial** en orden de prioridad.

**¿Por qué?** 4 sub-agentes tocando el mismo repo en paralelo pueden generar merge conflicts silenciosos. En smart contracts, un merge conflict puede introducir vulnerabilidades.

---

## Security Gate (para QUALITY y contratos)

Los SDDs clasificados como QUALITY o que tocan smart contracts tienen un gate adicional post-ejecución:

```
Sub-agente commitea → SM revisa el diff real → Security Gate
```

### Checklist del Security Gate

1. **¿El diff introduce nueva superficie de ataque?** (nuevos endpoints, nuevas funciones públicas, nuevos permisos)
2. **¿Se mantiene el principio de menor privilegio?** (¿el cambio da más acceso del necesario?)
3. **¿Los inputs están validados?** (Zod, require, bounds checking)
4. **¿Hay secrets hardcodeados o expuestos?** (grep por API keys, private keys, tokens)
5. **¿El error handling es seguro?** (¿los mensajes de error exponen información interna?)

Si el SM encuentra un concern → **revert antes de push** o crear un hotfix SDD.

---

## El Ciclo de un Sprint

```
1. SM presenta backlog priorizado
2. PO aprueba HUs → HU_APPROVED
3. SM escribe SDDs para HU-MAJOR / QUALITY
4. SM ejecuta Wave 0 y validación adversarial
5. PO revisa SDDs → SPEC_APPROVED
6. SM presenta Sprint Plan (incluyendo reglas de paralelismo)
7. PO aprueba → SPRINT_APPROVED
8. Sub-agentes ejecutan Wave 0 (re-validación) + Waves 1..N
9. SM ejecuta Security Gate en cambios QUALITY/contratos
10. SM presenta Review con evidencia
11. PO aprueba → REVIEW_APPROVED
12. Retro rápida — lecciones + acciones
13. PO aprueba → RETRO_APPROVED
14. Sprint cerrado → siguiente sprint
```

---

## Integración con NexusAudit

Para trabajo de seguridad en smart contracts, NexusAgil se integra con NexusAudit:

```
NexusAudit (8 fases) → Findings clasificados
    ↓
Findings → Issues en Linear (HU-MAJOR / KNOWN-LIMITATION)
    ↓
Issues → SDDs en NexusAgil
    ↓
SDDs → Fixes + PoC tests invertidos
    ↓
Fixes → Deploy nueva versión del contrato
```

El Fix Loop de NexusAudit es compatible con NexusAgil — cada finding se convierte en una HU que pasa por los gates correspondientes según su clasificación.

---

## Validación Adversarial

Antes de que el PO apruebe un SDD, el SM hace una revisión adversarial:

1. **Ejecuta Wave 0 completo** — ¿el fix ya existe? ¿Los archivos, tipos y encoding son correctos?
2. **Valida código de referencia contra tipos reales** — ¿el código del SDD compila contra el ABI/interfaces del proyecto?
3. **Busca el gap crítico** — ¿qué puede salir mal que el SDD no contempla?
4. **Verifica dependencias ocultas** — ¿el SDD asume algo que no está documentado?
5. **Prueba la aritmética** — para contratos: cada operación matemática se verifica
6. **Valida el encoding** — para contratos: indexed strings → keccak256, structs packed, etc.
7. **Verifica paralelismo** — ¿este SDD puede correr en paralelo con otros del sprint?
8. **Valida el orden de waves** — ¿tiene sentido ejecutar en este orden?
9. **Verifica rollback** — ¿el SDD tiene instrucciones de rollback claras?

Si el SM encuentra un problema crítico, el SDD se corrige antes de SPEC_APPROVED.
Si el SM no encuentra problemas, dice explícitamente: "listo para SPEC_APPROVED".

---

## Roles

### Product Owner (PO)
- Define qué se construye y por qué
- Aprueba los gates (incluyendo amendments)
- Decide las prioridades del backlog
- Da el go/no-go en decisiones de negocio

### Scrum Master / Orquestador (SM)
- Escribe artefactos (Work Items, SDDs)
- **Nunca se auto-evalúa** — cada artefacto pasa por un sub-agente antes del PO
- Orquesta los 6 sub-agentes (define orden, paralelismo, re-ejecución)
- Presenta resultados + reportes de sub-agentes al PO
- Actualiza issue tracker y documentación

### Los 6 Sub-agentes Especializados

Cada sub-agente tiene un **role skill** (`roles/*.md`) con: misión, checklist, formato de output, y lo que NO debe hacer.

| # | Sub-agente | Misión | Cuándo | Role Skill |
|---|-----------|--------|--------|-----------|
| 1 | **Requirements Reviewer** | Encontrar lo que FALTA en el Work Item | Pre HU_APPROVED | `roles/requirements-reviewer.md` |
| 2 | **Spec Reviewer** | Encontrar errores técnicos en el SDD (Wave 0 + coherencia) | Pre SPEC_APPROVED | `roles/spec-reviewer.md` |
| 3 | **Builder** | Implementar EXACTAMENTE el SDD, nada más | Post SPRINT_APPROVED | `roles/builder.md` |
| 4 | **Logic Auditor** | ¿El código hace lo correcto? Buscar bugs lógicos | Post Builder | `roles/logic-auditor.md` |
| 5 | **Security Reviewer** | ¿El código es seguro? Buscar vulnerabilidades | Post Builder (solo QUALITY) | `roles/security-reviewer.md` |
| 6 | **QA Verifier** | ¿Los ACs se cumplen? Con evidencia archivo:línea | Post Auditor/Security | `roles/qa-verifier.md` |

**Principio clave: separación de responsabilidades**
- Quien **escribe** (SM) no valida su propio trabajo
- Quien **implementa** (Builder) no audita su código
- Quien **audita lógica** (Auditor) no busca vulnerabilidades (Security)
- Quien **verifica ACs** (QA) no opina sobre diseño

**Cuántos sub-agentes por clasificación:**

| Clasificación | Sub-agentes activos |
|--------------|-------------------|
| FAST-FIX | Solo Builder |
| HU-MINOR | Builder + QA |
| HU-MAJOR | Req Reviewer + Spec Reviewer + Builder + Logic Auditor + QA |
| QUALITY | Los 6 completos |

---

## Anti-patrones a evitar

| Anti-patrón | Consecuencia | Solución |
|---|---|---|
| Ejecutar sin SDD aprobado | El agente alucina el diseño | Esperar SPEC_APPROVED |
| SDD con TODOs | El agente infiere incorrectamente | Completar el SDD antes de aprobar |
| Saltarse Wave 0 | Se ejecutan SDDs para fixes que ya existen, o con encoding incorrecto | Wave 0 es obligatorio, siempre — doble ejecución (SM + sub-agente) |
| SDD sin verificar encoding | El agente compara datos con tipo incorrecto (ej: indexed string vs keccak256) | Wave 0.3a/b verifica tipos y encoding |
| No verificar build entre waves | El agente acumula errores y commitea código roto | Build gate al final de cada wave |
| SDD sin rollback plan | Regresión en prod requiere debugging bajo presión | Sección Rollback obligatoria |
| Cambio ad-hoc en smart contract | Cambio sin SDD bypassa validación adversarial | Todo cambio a contratos requiere SDD (mínimo amendment) |
| Paralelismo sin verificar archivos | Merge conflicts silenciosos, posibles vulnerabilidades | Reglas de paralelismo estrictas |
| Review sin evidencia | No se puede verificar si el trabajo está hecho | Exigir tests + build output |
| Sprint sin scope fijo | Scope creep infinito | SPRINT_APPROVED fija el scope |
| Gates implícitos | Nadie sabe en qué estado está el trabajo | Gates siempre explícitos y documentados |
| Confiar solo en el SM | El SM también alucina — sin redundancia, errores pasan | Doble validación: SM + sub-agente ejecutan Wave 0 |

---

## Historia

NexusAgil nació durante el desarrollo de WasiAI en marzo 2026, cuando quedó claro que Scrum puro no era suficiente para coordinar un equipo donde los ejecutores son agentes de IA. La metodología evolucionó sprint a sprint, incorporando lecciones de cada retro.

### Changelog

**v1.3** (marzo 2026) — Arquitectura de Sub-agentes Especializados
- 6 sub-agentes con role skills dedicados (`roles/*.md`)
- Orquestador nunca se auto-evalúa — cada artefacto validado por sub-agente independiente
- Requirements Reviewer valida Work Items antes de HU_APPROVED
- Spec Reviewer ejecuta Wave 0 + coherencia antes de SPEC_APPROVED
- Logic Auditor separado de Security Reviewer (corrección lógica vs vulnerabilidades)
- QA Verifier con evidencia archivo:línea obligatoria
- Tabla de sub-agentes por clasificación (FAST: 1, MINOR: 2, MAJOR: 5, QUALITY: 6)
- Comparación con MetaGPT/ChatDev/CrewAI documentada

**v1.2** (marzo 2026) — Post Quality Fixes Sprint
- Wave 0.3 ampliado: validación de código contra tipos reales (0.3a), encoding (0.3b), seguridad contratos (0.3c)
- Doble validación Wave 0: SM + sub-agente ejecutan independientemente
- Build gate obligatorio al final de cada wave
- Sección Rollback obligatoria en todo SDD
- SDD Amendments formales para cambios post-SPEC_APPROVED
- Reglas de paralelismo con verificación de archivos disjuntos
- Security Gate post-ejecución para QUALITY y smart contracts
- Instrucción explícita para sub-agentes: re-validar Wave 0 antes de ejecutar

**v1.1** (marzo 2026) — Post Quality Fixes Sprint
- Wave 0 obligatorio (pre-flight checks)
- Anti-patrones: saltarse Wave 0, encoding incorrecto, cambios ad-hoc

**v1.0** (marzo 2026) — Release inicial
- 5 gates formales, clasificación de trabajo, SDD structure
- Primera aplicación: WasiAI Marketplace (Sprints 7-13)

---

*NexusAgil es open-source. Úsalo, mejóralo, y contribuye lo que aprendas.*
