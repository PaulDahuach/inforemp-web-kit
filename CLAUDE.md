# inforemp-web-kit — contexto para Claude

Kit reutilizable para dar **front web PHP a sistemas legacy** (VB6/Access) leyendo la
**misma `.mdb`/`.accdb`** vía COM/ADODB. Es el patrón "passthrough" (patrón B): cero
migración de datos, el legacy de escritorio y la web conviven sobre el dato vivo.

- **Repo:** github.com/PaulDahuach/inforemp-web-kit.git (rama `master`)
- **Origen del patrón:** `C:\wamp64\www\RDN\` (PHP que abre `RDN.mdb` vía ACE).
- **Instancias reales construidas con este kit:** `produccion_ptp`, `supervisores_ptp`
  (cada una en su propia carpeta `C:\wamp64\www\` con su repo).

## Cómo se instancia
Copiar el kit a una carpeta nueva. **ÚNICO archivo a editar por sistema:**
`config/system.php` (copiar de `system.example.php`; está **gitignored**). Define:
`name`, `base_url`, `mdb_path`, `mdb_provider`, `mode` (readonly|readwrite), `auth`
(tabla/cols de login), `menu`, `dashboard` (KPIs), `sector_login` (opt-in), `deploy_key`.

## Estructura
- `includes/db.php` — `db_connect/db_query/db_row/db_lookup/db_exec/ado_val`,
  `db_begin/commit/rollback`, `next_number($ultCol)`, helpers de fecha. Convierte
  Windows-1252→UTF-8. `db_exec` tira excepción si `mode=readonly`.
- `includes/auth.php` — login 2 pasos; soporta `sector_login` (login por operario+sector).
- `includes/helpers.php`, `includes/layout.php` (`module_head/module_foot`), `bu()` (base_url).
- `app/` — login, index (dashboard sidebar+KPIs config-driven), sector.php.
- `api/auth.php`, `modules/_template/` (plantilla read-only), `modules/abm/` (CRUD genérico
  config-driven con hijos), `modules/reportes/` (listados DataTable + impresión), `deploy.php`.

## Reglas de oro (COM/ADO + ACE) — valen para TODA instancia
- **PHP 5.5 compatible** en el server del cliente PTP (Win 2008 R2 + WAMP 32-bit + PHP 5.5.12).
  NO usar `??`, `intdiv`, arrow fns, tipos, `match`, spread. (En JS de módulos ES6 sí va —
  usan Chrome, nadie usa IE.)
- **EOL = CRLF** (Windows/Bloc de notas). Hay `.gitattributes` (`* text=auto eol=crlf`).
- **PK legacy = Long MANUAL** (no autonum): usar `next_number('ULT<ENT>')` (lee/incrementa
  `[Rec Control]`), NUNCA `MAX+1` (colisiona con el desktop).
- **Fechas Access = serial OLE** (base 1899-12-30): convertir con los helpers; al escribir
  usar `#mm/dd/YYYY#`.
- **ACE NO soporta:** `UNION` dentro de subquery de FROM (hacer UNION ALL a nivel tope +
  agregar en PHP), `Count(DISTINCT)` (subconsulta agregada), `Nz()` (leer-calcular-escribir).
- Joins anidados a la izquierda estilo Access; contar bien los paréntesis de apertura.

## Git / deploy
- Cada repo con `git -C <ruta>` (el shell del Bash tool resetea cwd; un `git add -A` sin
  `-C` commitea el repo equivocado).
- Deploy a prod por `deploy.php` + curl (server del cliente: Windows + WAMP + ACE 32-bit +
  DDNS + Certbot). RAR de deploy en `Desktop\PTP_Deploy.rar`.

## Historia completa
El blow-by-blow de TODO el desarrollo (cada módulo, decisiones, validaciones) está en la
memoria del proyecto **inforemp_inside**:
`C:\Users\pauld\.claude\projects\C--wamp64-www-inforemp-inside\memory\inforemp_web_kit.md`.
Leelo con la herramienta Read si necesitás el detalle.
