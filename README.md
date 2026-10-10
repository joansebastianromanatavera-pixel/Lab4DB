[README.md](https://github.com/user-attachments/files/33283558/README.md)
<div align="center">

# Práctica 4 — Seguridad en Bases de Datos
### Usuarios · Privilegios · Roles · Auditoría

*UMB — Teoría de Bases de Datos · Guía No. 4 · Eje temático: Seguridad en BD*

</div>

---

## Índice

- [Sobre esta guía](#sobre-esta-guía)
- [Estado actual](#estado-actual)
- [Estructura de carpetas](#estructura-de-carpetas)
- [Sistema base — Guia_3](#sistema-base--guia_3)
- [Preparación — tablas y datos](#preparación--tablas-y-datos)
- [Tarea previa — GRANT vs REVOKE](#tarea-previa--grant-vs-revoke)
- [Actividad 1 — Usuarios y privilegios directos](#actividad-1--usuarios-y-privilegios-directos)
- [Actividad 2 — Roles corporativos](#actividad-2--roles-corporativos)
- [Seguridad — cuenta ROOT](#seguridad--cuenta-root)
- [Qué falta](#qué-falta)

---

## Sobre esta guía

La práctica pone al estudiante en el rol de **DBA**: configurar la seguridad del motor (usuarios, roles, privilegios), prevenir inyección SQL y construir auditoría con triggers, sobre el sistema ya construido en las Prácticas 1-3 (`Guia_3`).

| | |
|---|---|
| **Resultado de aprendizaje** | Implementar mecanismos de seguridad en BD: usuarios, privilegios, roles y auditoría |
| **Actividades** | 1. Usuarios y privilegios · 2. Roles corporativos · 3. Prevención de inyección SQL · 4. Trigger de auditoría · 5. Informe de seguridad |
| **Entregable final** | `Practica4_NombreEquipo.pdf` (vía Moodle UMB) |

### Rúbrica (ponderación)

| Criterio | % | Qué evalúa |
|---|:-:|---|
| Procedimental | 35% | Organización y precisión de los resultados |
| Cognitiva | 35% | Aplicación de conceptos, relación teoría-práctica |
| Socio-afectiva | 10% | Participación y responsabilidad |
| Comunicativa | 10% | Calidad del informe |
| Investigativa | 10% | Fuentes citadas (APA 7) |

---

## Estado actual

| Bloque | Script | Capturas | Estado |
|---|---|:-:|:-:|
| Preparación (tablas + datos) | [`Scripts/01_Tablas.sql`](Scripts/01_Tablas.sql), [`Scripts/02_Datos.sql`](Scripts/02_Datos.sql) | — | ✔ Completo |
| Tarea previa — GRANT vs REVOKE | [`Actividad-tarea-previa/Script/00_TareaPrevia_GRANT_vs_REVOKE.sql`](Actividad-tarea-previa/Script/00_TareaPrevia_GRANT_vs_REVOKE.sql) | 21 (`Actividad-tarea-previa/Imagenes/`) | ✔ Completo |
| Actividad 1 — Usuarios y privilegios | [`Actividad-1/Script/01_Actividad1_Usuarios_Privilegios.sql`](Actividad-1/Script/01_Actividad1_Usuarios_Privilegios.sql) | 28 (`Actividad-1/Imagenes/`) | ✔ Completo |
| Actividad 2 — Roles corporativos | [`Actividad-2/Scripts/02_Actividad2_Roles_Corporativos.sql`](Actividad-2/Scripts/02_Actividad2_Roles_Corporativos.sql) | 49 (`Actividad-2/Imagenes/`) | ✔ Completo |
| Actividad 3 — Prevención de inyección SQL | [`Actividad 3/Scripts/vulnerable-ej.py`](Actividad%203/Scripts/vulnerable-ej.py), [`Segura.py`](Actividad%203/Scripts/Segura.py) | — | ✔ Completo (faltan capturas de ejecución) |
| Actividad 4 — Trigger de auditoría | [`Actividad 4/Scripts/triggers.sql`](Actividad%204/Scripts/triggers.sql), [`testeo logs.sql`](Actividad%204/Scripts/testeo%20logs.sql) | 2 (`Actividad 4/Capturaa/`) | ⚠ Re-ejecutar en `Guia_3` con `USER()` |
| Actividad 5 — Informe de seguridad | [`Actividad 5/Informe_Seguridad.md`](Actividad%205/Informe_Seguridad.md), [`05_Actividad5_Mapa_Seguridad.sql`](Actividad%205/Scripts/05_Actividad5_Mapa_Seguridad.sql) | — | ✔ Redactado (faltan capturas 5.1–5.9) |

---

## Estructura de carpetas

```
Guia4/
├── README.md                              <- este documento
├── Scripts/
│   ├── 01_Tablas.sql                      <- crea la BD Guia_3 y sus 6 tablas
│   └── 02_Datos.sql                       <- puebla las tablas con datos reales
├── Actividad-tarea-previa/
│   ├── Script/00_TareaPrevia_GRANT_vs_REVOKE.sql
│   └── Imagenes/   (21 capturas)
├── Actividad-1/
│   ├── Script/01_Actividad1_Usuarios_Privilegios.sql
│   └── Imagenes/   (28 capturas)
└── Actividad-2/
    ├── Scripts/02_Actividad2_Roles_Corporativos.sql
    └── Imagenes/   (49 capturas)
```

**Orden de ejecución:** `Scripts/01_Tablas.sql` → `Scripts/02_Datos.sql` → `Actividad-1/.../01_...sql` → `Actividad-2/.../02_...sql` → `Actividad-tarea-previa/.../00_...sql` (este último depende de que ya existan los usuarios y roles de las dos actividades anteriores).

---

## Sistema base — Guia_3

Base de datos de las Prácticas 1-3 (sistema académico): 6 tablas relacionadas.

| Tabla | Llave primaria | Relación |
|---|---|---|
| `Programa` | `IdPrograma` | — |
| `Estudiante` | `IdEstudiante` | — |
| `Profesor` | `IdProfesor` | — |
| `Materia` | `IdMateria` | → `Programa` |
| `Grupo` | `IdGrupo` | → `Materia`, `Profesor` |
| `Matricula` | `IdMatricula` | → `Estudiante`, `Grupo` |
| `auditoria_log` | `id` | tabla nueva de esta práctica (Actividad 1, usada por el trigger de Actividad 4) |

---

## Preparación — tablas y datos

- **`01_Tablas.sql`**: crea la base `Guia_3` (si no existe) y las 6 tablas con sus llaves foráneas.
- **`02_Datos.sql`**: trunca e inserta datos reales — 5 programas, 5 estudiantes, 5 profesores, 5 materias, 5 grupos, 13 matrículas.

> Ejecutar siempre en ese orden y antes que cualquier otro script de la práctica — las Actividades 1, 2 y la tarea previa asumen que estos datos ya existen (por ejemplo, usan `IdEstudiante = 1031422990` e `IdGrupo = 1` en sus pruebas).

---

## Tarea previa — GRANT vs REVOKE

<table>
<tr><th></th><th><code>GRANT</code></th><th><code>REVOKE</code></th></tr>
<tr><td><b>Qué hace</b></td><td>Otorga (autoriza) un privilegio</td><td>Retira un privilegio otorgado</td></tr>
<tr><td><b>Efecto</b></td><td>Suma capacidades</td><td>Resta capacidades</td></tr>
<tr><td><b>Requisito</b></td><td colspan="2" align="center">Quien lo ejecuta necesita <code>GRANT OPTION</code> sobre ese privilegio</td></tr>
<tr><td><b>Cuándo aplica</b></td><td colspan="2" align="center">Inmediato — no requiere <code>FLUSH PRIVILEGES</code> ni reconexión</td></tr>
<tr><td><b>Analogía</b></td><td>Entregar una llave</td><td>Quitar una llave ya entregada</td></tr>
</table>

### Los 5 ejemplos

Código completo en [`00_TareaPrevia_GRANT_vs_REVOKE.sql`](Actividad-tarea-previa/Script/00_TareaPrevia_GRANT_vs_REVOKE.sql).

**1 · Privilegio puntual sobre una tabla — el caso más granular**
```sql
GRANT  SELECT ON Guia_3.Estudiante TO 'analista'@'localhost';
REVOKE SELECT ON Guia_3.Estudiante FROM 'analista'@'localhost';
```

**2 · Privilegio de escritura — el REVOKE es selectivo**
```sql
GRANT  INSERT ON Guia_3.Matricula TO 'operador'@'localhost';
REVOKE INSERT ON Guia_3.Matricula FROM 'operador'@'localhost';
-- operador conserva SELECT/UPDATE: el REVOKE solo quita lo que se nombra
```

**3 · Esquema completo + `GRANT OPTION` — la delegación es un privilegio aparte**
```sql
GRANT  ALL PRIVILEGES ON Guia_3.* TO 'admin_bd'@'localhost' WITH GRANT OPTION;
REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'admin_bd'@'localhost';
-- si no se nombra GRANT OPTION en el REVOKE, la capacidad de delegar NO se retira
```

**4 · Privilegio sobre un ROL — efecto en cascada**
```sql
GRANT  UPDATE ON Guia_3.Grupo TO 'rol_operador';
REVOKE UPDATE ON Guia_3.Grupo FROM 'rol_operador';
-- afecta a TODOS los usuarios con rol_operador a la vez, no uno por uno
```

**5 · Rol completo asignado a un usuario**
```sql
GRANT  'rol_consultor' TO 'analista'@'localhost';
REVOKE 'rol_consultor' FROM 'analista'@'localhost';
-- analista gana o pierde TODOS los privilegios heredados de un solo golpe
```

### Comparativa de los 5 ejemplos

| # | Privilegio / objeto | Alcance | Destinatario | Efecto del `REVOKE` |
|:-:|---|---|:-:|---|
| 1 | `SELECT` | 1 tabla | Usuario | Pierde solo lectura de esa tabla |
| 2 | `INSERT` | 1 tabla | Usuario | Pierde solo escritura; conserva el resto |
| 3 | `ALL` + `GRANT OPTION` | Esquema completo | Usuario | Pierde todo, incluida la delegación |
| 4 | `UPDATE` | 1 tabla | **Rol** | Afecta a todos los usuarios del rol, a la vez |
| 5 | Rol completo | Todos sus privilegios | Usuario | Pierde de un solo golpe todo lo heredado |

> **Conclusión:** `GRANT`/`REVOKE` son siempre inversos y simétricos; lo que cambia es el **alcance** (puntual vs. total), el **destinatario** (usuario vs. rol) y el **tipo de objeto** (un privilegio vs. un rol completo).

---

## Actividad 1 — Usuarios y privilegios directos

Script: [`01_Actividad1_Usuarios_Privilegios.sql`](Actividad-1/Script/01_Actividad1_Usuarios_Privilegios.sql)

| Usuario | Contraseña (lab) | Alcance otorgado |
|---|---|---|
| `admin_bd` | `AdminBD_2026!` | `ALL PRIVILEGES` sobre `Guia_3.*` (incluye `auditoria_log`), con `GRANT OPTION` |
| `analista` | `Analista_2026!` | `SELECT` sobre las 6 tablas operativas, tabla por tabla — **sin** `auditoria_log` |
| `operador` | `Operador_2026!` | `SELECT`, `INSERT`, `UPDATE` sobre las 6 tablas operativas — **sin** `auditoria_log` |
| `auditor` | `Auditor_2026!` | `SELECT` sobre `Guia_3.auditoria_log` **únicamente** |

> `analista` y `operador` reciben sus privilegios **tabla por tabla** (nunca con `Guia_3.*`): un comodín incluiría `auditoria_log` automáticamente, y solo `auditor` debe poder leerla.

### Matriz de permisos por usuario

| Usuario | SELECT | INSERT | UPDATE | DELETE | SELECT `auditoria_log` | GRANT OPTION |
|---|:-:|:-:|:-:|:-:|:-:|:-:|
| `admin_bd` | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| `analista` | ✔ | ✘ | ✘ | ✘ | ✘ | ✘ |
| `operador` | ✔ | ✔ | ✔ | ✘ | ✘ | ✘ |
| `auditor`  | ✘ | ✘ | ✘ | ✘ | ✔ | ✘ |

> `auditor` marca ✘ en SELECT general porque su único privilegio es sobre `auditoria_log`.

### Pruebas de acceso documentadas (28 capturas en `Actividad-1/Imagenes/`)

| Usuario | Prueba | Resultado |
|---|---|:-:|
| `admin_bd` | cualquier operación / `GRANT ...` | OK |
| `analista` | `SELECT * FROM Estudiante;` | OK |
| `analista` | `INSERT INTO Programa ...` | `ERROR 1142` |
| `operador` | `DELETE FROM Matricula ...` | `ERROR 1142` |
| `auditor` | `SELECT * FROM auditoria_log;` | OK |
| `auditor` | `SELECT * FROM Estudiante;` | `ERROR 1142` |

---

## Actividad 2 — Roles corporativos

Script: [`02_Actividad2_Roles_Corporativos.sql`](Actividad-2/Scripts/02_Actividad2_Roles_Corporativos.sql)

| Rol | Privilegios asignados |
|---|---|
| `rol_admin` | `ALL PRIVILEGES` sobre `Guia_3.*` con `GRANT OPTION` (incluye `auditoria_log`) |
| `rol_consultor` | `SELECT` sobre las 6 tablas operativas, tabla por tabla (sin `auditoria_log`) |
| `rol_operador` | `SELECT`, `INSERT`, `UPDATE` sobre las 6 tablas operativas (sin `DELETE`, sin `auditoria_log`) |

### Diagrama de asignación

```
admin_bd@localhost ──▶ rol_admin ─────▶ ALL PRIVILEGES ON Guia_3.* WITH GRANT OPTION
analista@localhost ──▶ rol_consultor ─▶ SELECT sobre tablas operativas (sin auditoria_log)
operador@localhost ──▶ rol_operador ──▶ SELECT, INSERT, UPDATE (sin DELETE, sin auditoria_log)

auditor@localhost  ┄┄▶ (privilegio DIRECTO, sin rol) ┄┄▶ SELECT ON Guia_3.auditoria_log
```

| Usuario | Rol | Vía |
|---|---|:-:|
| `admin_bd` | `rol_admin` | heredado |
| `analista` | `rol_consultor` | heredado |
| `operador` | `rol_operador` | heredado |
| `auditor` | — | **directo** |

### Privilegio directo vs. heredado por rol

- `SHOW GRANTS FOR 'auditor'@'localhost';` → el `SELECT` sobre `auditoria_log` aparece **directamente**.
- `SHOW GRANTS FOR 'analista'@'localhost';` → solo muestra `GRANT rol_consultor TO analista`; con `... USING 'rol_consultor'` se ven los privilegios reales heredados.
- Un `REVOKE` sobre un **rol** afecta a todos sus usuarios a la vez; un privilegio **directo** (como el de `auditor`) nunca se ve afectado por cambios en ningún rol.
- `SET DEFAULT ROLE` es obligatorio para que el rol quede activo al iniciar sesión.

### Pruebas de acceso documentadas (49 capturas en `Actividad-2/Imagenes/`)

| Usuario | Prueba | Resultado |
|---|---|:-:|
| `operador` | `DELETE FROM Matricula ...` | `ERROR 1142` |
| `analista` | `INSERT INTO Programa ...` | `ERROR 1142` |
| `analista` | `SELECT * FROM Estudiante;` | OK |
| `auditor` | `SELECT * FROM Estudiante;` | `ERROR 1142` |
| `auditor` | `SELECT * FROM auditoria_log;` | OK |
| `admin_bd` | cualquier operación / `GRANT ...` | OK |

---

## Seguridad — cuenta ROOT

Ninguno de los 4 usuarios operativos usa `root`. Operar con `root` en producción es un riesgo porque:

- **Sin trazabilidad** — todas las acciones quedan bajo la misma identidad.
- **Sin límites** — un error o una inyección SQL compromete toda la instancia.
- **Rompe el menor privilegio** — el principio que exige esta práctica.

---

## Qué falta

| Actividad | Entregable según la guía |
|---|---|
| **3 — Prevención de inyección SQL** | 2 consultas vulnerables (código Python + MySQL Connector), reescritas con prepared statements, explicación de por qué protegen |
| **4 — Trigger de auditoría** | Triggers `AFTER UPDATE` y `AFTER DELETE` sobre una tabla de `Guia_3` que inserten en `auditoria_log` (la tabla ya existe desde la Actividad 1); capturas del log poblado |
| **5 — Informe de seguridad** | Documento final: mapa de usuarios/roles/privilegios/auditoría + recomendaciones para producción, consolidando todo lo anterior |
