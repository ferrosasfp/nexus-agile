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

## Wave 1 — <Nombre>
Instrucciones paso a paso. Código de referencia cuando aplica.

## Wave 2 — <Nombre>
...

## Wave N — Commit + Push
Comando exacto de git.

## Critical Constraints
Reglas que el agente NO puede ignorar.
```

### Principios del SDD

1. **Una sola fuente de verdad** — el agente lee el SDD y solo el SDD
2. **Waves en orden estricto** — nunca saltarse pasos
3. **Leer antes de modificar** — el agente lee cada archivo antes de tocarlo
4. **Build gate obligatorio** — si el build falla, el agente se detiene y reporta
5. **Test gate obligatorio** — si los tests fallan, el agente se detiene y reporta

---

## El Ciclo de un Sprint

```
1. SM presenta backlog priorizado
2. Fer aprueba HUs → HU_APPROVED
3. San escribe SDDs para HU-MAJOR / QUALITY
4. Fer revisa SDDs → SPEC_APPROVED
5. SM presenta Sprint Plan
6. Fer aprueba → SPRINT_APPROVED
7. Sub-agentes ejecutan (FAST-FIX y HU-MINOR directo, HU-MAJOR con SDD)
8. SM presenta Review con evidencia
9. Fer aprueba → REVIEW_APPROVED
10. Retro rápida — lecciones + acciones
11. Fer aprueba → RETRO_APPROVED
12. Sprint cerrado → siguiente sprint
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

Antes de que Fer apruebe un SDD, San hace una revisión adversarial:

1. **Busca el gap crítico** — ¿qué puede salir mal?
2. **Verifica dependencias ocultas** — ¿el SDD asume algo que no está documentado?
3. **Prueba la aritmética** — para contratos: cada operación matemática se verifica
4. **Valida el orden de waves** — ¿tiene sentido ejecutar en este orden?

Si San encuentra un problema crítico, el SDD se corrige antes de SPEC_APPROVED.
Si San no encuentra problemas, dice explícitamente: "listo para SPEC_APPROVED".

---

## Roles

### Product Owner (Fer)
- Define qué se construye y por qué
- Aprueba los gates
- Decide las prioridades del backlog
- Da el go/no-go en decisiones de negocio

### Scrum Master + Tech Lead (San)
- Facilita el proceso
- Escribe y valida SDDs
- Hace la revisión adversarial
- Orquesta los sub-agentes
- Actualiza Linear y la documentación

### Agentes Ejecutores (Sub-agentes especializados)
- Ejecutan una sola tarea a la vez
- Siguen el SDD al pie de la letra
- Reportan resultado y commit hash
- No toman decisiones de diseño — escalan si hay ambigüedad

---

## Anti-patrones a evitar

| Anti-patrón | Consecuencia | Solución |
|---|---|---|
| Ejecutar sin SDD aprobado | El agente alucina el diseño | Esperar SPEC_APPROVED |
| SDD con TODOs | El agente infiere incorrectamente | Completar el SDD antes de aprobar |
| Review sin evidencia | No se puede verificar si el trabajo está hecho | Exigir tests + build output |
| Sprint sin scope fijo | Scope creep infinito | SPRINT_APPROVED fija el scope |
| Gates implícitos | Nadie sabe en qué estado está el trabajo | Gates siempre explícitos y documentados |

---

## Historia

NexusAgil nació durante el desarrollo de WasiAI en marzo 2026, cuando quedó claro que Scrum puro no era suficiente para coordinar un equipo donde los ejecutores son agentes de IA. La metodología evolucionó sprint a sprint, incorporando lecciones de cada retro.

Versión actual: **1.0**
Primera aplicación: WasiAI Marketplace (Sprints 7-13)

---

*NexusAgil es open-source. Úsalo, mejóralo, y contribuye lo que aprendas.*
