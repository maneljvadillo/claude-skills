---
name: "auditoria-seguridad"
description: "Auditoría y hardening de seguridad por fases con aprobación entre cada una. Úsala cuando pidan auditar, revisar o endurecer la seguridad de un proyecto, web, app o API."
---

# Auditoría de seguridad por fases

Audita y endurece el proyecto POR FASES, con aprobación del usuario entre cada una. Trabaja en el branch `security-hardening`.

**Cómo usar esta skill:** en cada fase aplica las REGLAS DE TRABAJO + solo la sección de la fase actual. No intentes aplicar el checklist de todas las fases a la vez.

## Modo de profundidad
Si el usuario no lo dice, pregúntalo en la Fase 0 (por defecto: ESTÁNDAR).
- **RÁPIDO:** Fases 0, 1, 2 y 8, más las críticas y altas que veas por el camino. Sin hardening bajo.
- **ESTÁNDAR:** todas las fases que apliquen. Hardening solo de gravedad media o superior.
- **EXHAUSTIVO:** todo, incluido el hardening bajo y la revisión línea a línea de la Fase 12.
Si el modo no encaja con lo que ves (por ejemplo, RÁPIDO en una app con pagos), dilo y recomienda otro.

**Proyecto simple** (web estática, sin backend propio, sin login y sin datos personales más allá de un formulario de contacto): si la Fase 0 lo confirma, propón el recorrido corto: Fases 1, 2, 8 y 9 (solo cookies y scripts de terceros). El resto va como "no aplica" en bloque, con un motivo de una línea por fase. No crees backend, auth ni tests de integración que no se necesitan.

---

# REGLAS DE TRABAJO (todas las fases)

## Flujo y aprobaciones
- Una fase cada vez. No encadenes fases.
- Por fase hay **un solo punto de aprobación para empezar**: el resumen de cierre de la fase anterior va junto con el PLAN de la siguiente (qué has encontrado y qué vas a cambiar). Un OK aprueba las dos cosas. Solo entonces tocas código.
- Si algo requiere una decisión del usuario, para y pregunta: opciones con pros, contras y tu recomendación.
- Las funcionalidades nuevas (2FA, passkeys, alertas, captcha, WAF, cambio de proveedor de auth...) se PROPONEN y no se implementan sin OK explícito. Arreglar una vulnerabilidad no es añadir una funcionalidad.

## Proporcionalidad
- Prioriza por riesgo real para ESTE proyecto (impacto × probabilidad). Primero las vulnerabilidades confirmadas críticas y altas, después el hardening.
- Si un punto no aplica: "no aplica" y por qué, en una línea.

## Alcance
- Prohibido sin OK explícito: refactors grandes, migraciones de BD, cambios de arquitectura, sustituir librerías o proveedores, y cambiar APIs públicas o contratos con el frontend o la app.
- Cada plan incluye el rollback. Si un cambio sensible se puede poner tras una variable de entorno o un flag, hazlo así.

## No romper nada
- Antes de empezar: `git status` limpio. Si hay cambios sin commitear, para y pregunta. Después, crea el branch.
- Cambios pequeños y reversibles.
- Antes de cada cambio que afecte al comportamiento (CSP, CORS, rate limit, cookies, sesiones, cabeceras, redirecciones): di qué puede romperse y cómo lo has comprobado.
- Nada de dependencias nuevas sin justificarlo. Si añades una, que sea mantenida, popular y fijada en el lockfile.
- Herramientas del sistema (gitleaks, trufflehog, etc.): si no están instaladas, pide permiso para instalarlas o usa una alternativa y dilo. Nada de instalar herramientas globales sin preguntar.
- Nunca reescribas el historial de Git ni borres nada irreversible sin preguntar.

## Honestidad y verificación (anti-alucinación)
- Solo puedes afirmar lo que has visto en esta sesión.
- No cites un archivo, una línea, una función o una configuración que no hayas abierto. El código citado se copia tal cual.
- No des el resultado de un comando o test que no hayas ejecutado: pon el comando y su salida real (recortada y sin secretos).
- No supongas versiones, proveedores ni la configuración del hosting: o lo has leído, o es "no verificable".
- "No lo he comprobado" es una respuesta válida. "Parece que funciona" no lo es.
- Nunca imprimas secretos completos (enmascara: `sk-...abcd`).
- Prohibido inventar vulnerabilidades, recomendar controles genéricos sin comprobar si aplican o dar por resuelto lo no probado.

## Etiquetas (una por punto)
- **VULNERABILIDAD CONFIRMADA:** hay evidencia de que es explotable aquí.
- **RIESGO PROBABLE:** hay indicios; di qué falta para confirmarlo.
- **HARDENING:** mejora defensiva. No la presentes nunca como fallo.
- **NO APLICA:** con el motivo.
- **NO VERIFICABLE DESDE AQUÍ:** con lo que tiene que comprobar el usuario.

## Gravedad
- **CRÍTICA:** explotable ya y con daño grave (secretos de producción expuestos, RCE, admin sin auth, acceso masivo a datos ajenos, manipulación de pagos, SSRF a metadata cloud, BaaS sin RLS con datos reales).
- **ALTA:** explotable con poco esfuerzo o con una cuenta normal (IDOR/BOLA, bypass de autorización, inyección, sesión secuestrable, webhook de pago falsificable, XSS almacenado, subida de ejecutables).
- **MEDIA:** necesita condiciones concretas o el impacto es limitado (sin rate limit en el login, enumeración de usuarios, CSRF de poco impacto, XSS reflejado mitigado, dependencia vulnerable no alcanzable).
- **BAJA:** debilita la defensa sin un ataque directo (cabeceras que faltan, versiones expuestas, errores verbosos sin datos sensibles).
El contexto (datos reales, usuarios, dinero) sube o baja la gravedad. Justifícala en una línea.

## Formato de cada hallazgo
ID (SEC-001) · etiqueta · gravedad · archivo:línea · evidencia · impacto concreto y facilidad de explotación · corrección y rollback · test o verificación · estado (pendiente / propuesto / aprobado / implementado / verificado / aplazado / no verificable).

## Tests y verificación
- **Arreglos de comportamiento** (auth, autorización, inyecciones, validación, pagos, rate limit, subidas): test automático que reproduce el ataque real, FALLA sin el arreglo y PASA con él. Compruébalo y dilo. Incluye también el caso legítimo (el usuario correcto sí puede).
- **Arreglos de configuración** (cabeceras, .gitignore, CI, Dependabot, security.txt, DNS): basta con una verificación por comando con su salida real antes y después (por ejemplo, `curl -I` en local). No montes un framework de tests solo para esto.
- Si no se puede automatizar, explica por qué y da los pasos manuales.
- Si el proyecto tiene backend o lógica y no tiene tests, propón en la Fase 0 el framework mínimo para el stack y móntalo tras el OK, antes de arreglar nada. En un proyecto simple basta con un script de verificación (`scripts/security-check.sh` o similar).

## Cambios sensibles
CSP, CORS, cookies, sesiones, JWT, rate limits, webhooks, pagos, redirecciones, cabeceras, auth:
- Antes de aplicarlos: qué puede romperse (login, pagos, webhooks, integraciones, apps móviles, iframes, scripts de terceros), cómo lo compruebas y cómo se deshace.
- Pruébalos primero en local o staging. La CSP, primero en modo Report-Only.
- Lo que puede echar a todos los usuarios (invalidar sesiones, cambiar el secreto JWT, cambiar cookies) solo se hace con OK y avisando del efecto.

## Pruebas de ataque: solo en local o staging
- Todo lo ofensivo (fuerza bruta, payloads, subidas maliciosas, reenvío de webhooks, CSRF...) se hace SOLO en local o staging.
- Contra producción, solo comprobaciones pasivas de solo lectura (`curl -I`, TLS).
- Nunca uses claves, cuentas ni pasarelas reales: usa el modo test del proveedor.

## SECURITY_AUDIT.md (única fuente de verdad)
Vive en la raíz del repo y tiene estas secciones fijas:
1. Contexto y modo de profundidad
2. Modelo de amenazas
3. Plan de fases (aplica / no aplica / hecha / pendiente)
4. Hallazgos (en el formato de arriba)
5. Decisiones del usuario
6. Acciones pendientes para el usuario
7. Registro de fases (fecha, commit, resumen)

- Al EMPEZAR cada fase, vuelve a leerlo y trabaja a partir de lo que dice. Si no coincide con lo que recuerdas, manda el archivo y avisa.
- Lo que no está en el archivo no se ha decidido ni hecho.
- Debe poder retomarse en una sesión nueva leyendo solo el archivo y esta skill. Recomienda `/clear` entre fases si el contexto va cargado.
- Para no saturar el contexto, lee solo los archivos relevantes para cada fase y resume las salidas largas.

## Fase terminada = se cumple TODO esto
- Cada hallazgo aprobado está "verificado" o "aplazado" con OK.
- El proyecto compila y arranca, y los flujos críticos siguen funcionando.
- Todos los tests pasan. Si alguno ya fallaba antes, dilo; no lo "arregles" borrándolo.
Si algo no se cumple, la fase no está terminada: dilo en vez de cerrarla.

## Cierre de cada fase (en este orden)
1. Comprueba que el proyecto compila y arranca.
2. Ejecuta todos los tests y verificaciones y da el resultado.
3. Actualiza SECURITY_AUDIT.md.
4. Haz un commit separado: `security: fase N - <nombre>`.
5. Resumen de 3-5 líneas (qué encontraste, qué arreglaste, qué queda y qué necesitas del usuario) + el PLAN de la siguiente fase.
6. PARA y espera el OK.

---

# FASES

## FASE 0 · Reconocimiento y plan (sin tocar código)
**Paso 1 · Contexto.** Saca del repo todo lo que puedas (package.json o equivalente, configuración, .env.example, CI, Dockerfile, rutas) y prepara una ficha de lo DEDUCIDO, indicando de qué archivo sale cada dato. Después pregunta SOLO lo que falte, numerado.

Preguntas bloqueantes:
1. ¿Hay staging o un entorno de pruebas aparte de local? ¿Con qué URL?
2. ¿Qué datos personales o sensibles maneja? ¿Hay usuarios reales en producción?
3. ¿Qué roles hay y cuál tiene más privilegio?
4. ¿A qué tienes acceso y a qué no (hosting, DNS, panel de auth, panel de pagos, BD, logs)?
5. Modo de profundidad (si no se ha dicho).

Preguntas opcionales (si no hay respuesta, asume lo más prudente y apúntalo como supuesto):
6. Servicios externos que no aparecen en el código.
7. Comando de tests y qué cubren.
8. Funcionalidades que no se pueden romper bajo ningún concepto.

Espera las respuestas.

**Paso 2 · Análisis.** Stack, tipo de proyecto y entornos. Modelo de amenazas (puntos de entrada, datos sensibles, roles, servicios externos, dónde vive cada secreto). Barrido rápido con una lista preliminar de hallazgos por gravedad, con etiqueta y evidencia. Qué fases aplican y cuáles no, y si conviene cambiar el orden. Estado de los tests y framework propuesto. Crea SECURITY_AUDIT.md y PARA.

## FASE 1 · Secretos y configuración
- Escanea el árbol actual Y el historial completo de Git (gitleaks o trufflehog).
- Lista lo comprometido (enmascarado) y cómo rotar cada secreto. La rotación es obligatoria: limpiar el historial no basta. No reescribas el historial sin OK.
- Cero secretos en el código, en el frontend o en el binario. Las claves privadas, solo en el backend o en las variables de entorno del hosting.
- .env en .gitignore, .env.example sin valores reales. Revisa las variables públicas (NEXT_PUBLIC_, PUBLIC_, VITE_, EXPO_PUBLIC_...) por si hay algo privado expuesto.
- Pre-commit hook y paso en CI que bloqueen secretos.
- Debug OFF en producción, sin stack traces ni páginas de error que revelen versiones o rutas.

## FASE 2 · Dependencias, cadena de suministro y superficie expuesta
- Audit del gestor (npm/pnpm audit, pip-audit, composer audit...). Actualiza sin romper, priorizando críticas y altas. Lockfile.
- Paquetes sin uso o sospechosos (typosquatting, abandonados, postinstall raros).
- CI/CD: permisos mínimos, actions fijadas por versión o hash, secretos con el mínimo alcance y sin exponerse en logs ni en PRs de forks.
- Dependabot o Renovate.
- Archivos expuestos: .env, .git, backups, dumps, .DS_Store, source maps en producción, logs, paneles de debug, phpinfo, swagger sin proteger, listados de directorios, robots.txt o sitemap que revelen rutas sensibles.
- /.well-known/security.txt con un contacto.

## FASE 3 · Autenticación y sesiones
(Si la auth es de un proveedor como Supabase, Clerk, Auth0 o Firebase: di exactamente qué activar y dónde, en su panel.)
- Librería o proveedor probado; nada de criptografía propia.
- Contraseñas: argon2id (o bcrypt con coste ≥12), mínimo 12 caracteres, comprobación HIBP (k-anonymity) y sin un máximo absurdo.
- 2FA (PROPONER): TOTP como mínimo y passkeys si se puede. Obligatoria para admins. Códigos de recuperación de un solo uso guardados hasheados. Reverificación en acciones críticas. Nada de SMS como único factor.
- Fuerza bruta: rate limit por IP y por cuenta, bloqueo progresivo, errores genéricos y nada de enumeración en login, registro y recuperación.
- Cookies HttpOnly + Secure + SameSite, prefijo __Host- si se puede, rotación de la sesión al hacer login o subir privilegios, expiración por inactividad y absoluta, invalidación al cambiar la contraseña, y listado y cierre de sesiones.
- JWT: algoritmo fijado, expiración corta, refresh rotativo, verificación de iss/aud, sin datos sensibles dentro y revocables.
- Tokens de recuperación y verificación: aleatorios, de un solo uso, hasheados en BD y con caducidad de 15-30 minutos.
- CSRF: token, o SameSite + verificación de Origin, en las acciones que modifican datos.
- Cambio de email: confirmación en el antiguo y en el nuevo. Comparaciones de secretos en tiempo constante.

## FASE 4 · Autorización y control de acceso
- Denegar por defecto y comprobar siempre en el servidor.
- IDOR/BOLA: inventario de endpoints con ID y la comprobación de propiedad o rol de cada uno.
- /admin: rol en el servidor, 2FA, rate limit estricto, log de auditoría y restricción por IP si se puede.
- Mass assignment: lista blanca de campos en cada create/update.
- Mínimo privilegio en usuarios de BD, API keys y cuentas de servicio.
- Multi-tenant: aislamiento total entre clientes, verificado con tests.

## FASE 5 · Base de datos y datos
- Consultas 100 % parametrizadas u ORM. Revisa también NoSQL injection.
- BaaS: RLS o reglas en TODAS las tablas, colecciones y buckets. Prueba con un usuario normal y con uno anónimo. La service_role key nunca en el cliente.
- BD no accesible desde internet abierto (allowlist, red privada, SSL obligatorio).
- Cifrado en reposo de lo sensible y TLS en las conexiones.
- Backups automáticos con restauración probada (si no se puede verificar, va a las acciones del usuario).

## FASE 6 · Entradas, salidas y archivos
- Validación en el servidor con un esquema (zod, pydantic, joi...).
- XSS: escapado por defecto. Nada de innerHTML, dangerouslySetInnerHTML ni v-html con contenido no confiable (DOMPurify si hace falta). CSP con nonces o hashes, primero en Report-Only.
- Command injection, path traversal, template injection, header injection, open redirect y SSRF (allowlist, bloqueo de IPs internas y de metadata).
- Nada de deserializar datos no confiables con formatos que ejecutan código.
- Subidas: allowlist por magic bytes, límite de tamaño, nombres aleatorios, fuera del webroot o en un bucket privado con URLs firmadas, sin ejecución, y quitar el EXIF con datos personales.
- LLM: prompt injection directa e indirecta, sin secretos en los prompts, salidas validadas, herramientas limitadas y límite de coste por usuario.

## FASE 7 · Pagos y lógica de negocio
- Precios, descuentos, totales, stock y permisos, siempre en el servidor.
- Webhooks: firma verificada, validación con la API del proveedor cuando proceda e idempotencia.
- Race conditions: transacciones, bloqueos o restricciones únicas.
- Límites ante abusos (cupones, registros masivos, emails, APIs de pago por uso).
- Flujos que se pueden saltar (pasos, pruebas gratuitas infinitas).

## FASE 8 · Red, cabeceras y despliegue
- HTTPS forzado + HSTS con includeSubDomains (preload solo con confirmación, porque es difícil de deshacer). TLS 1.2+.
- CORS con orígenes concretos, nunca "*" con credenciales.
- CSP, HSTS, nosniff, frame-ancestors, Referrer-Policy, Permissions-Policy, COOP/CORP. Quita X-Powered-By y las versiones del servidor.
- Rate limiting global y por endpoint (login, registro, recuperación, contacto, búsqueda, endpoints caros). Propón Cloudflare u otro WAF si tiene sentido.
- Formularios públicos: Turnstile y honeypot (PROPONER).
- SRI en scripts de terceros y el mínimo de scripts externos.
- SPF, DKIM, DMARC, DNS, puertos, paneles expuestos, subdomain takeover y buckets públicos: compruébalo si hay acceso y, si no, da los pasos exactos.

## FASE 9 · Privacidad y protección de datos
- Normativa: en Andorra, la Llei 29/2021 qualificada de protecció de dades personals; con usuarios de la UE o España, también el RGPD. Indica cuál parece aplicar según los datos y los usuarios, y señala que lo confirme un profesional.
- Inventario de datos personales: qué, dónde, para qué y cuánto tiempo. No guardes lo que no se necesita.
- Borrado de cuenta real (storage y servicios externos incluidos) y exportación si aplica.
- Cookies y scripts de terceros (analítica, píxeles, chat): que no se carguen antes del consentimiento cuando la ley lo exija.
- Nada de enviar datos personales a terceros (analítica, IA, logs externos) sin necesidad.

## FASE 10 · Logs, monitorización y respuesta
- Eventos de seguridad: logins fallidos y exitosos, cambios de credenciales o 2FA, accesos denegados, acciones de admin, picos y 401/403/429 repetidos, subidas rechazadas.
- Logs sin contraseñas, tokens, tarjetas ni datos personales innecesarios, y protegidos contra la inyección de logs.
- Alertas por email o Telegram ante actividad sospechosa (PROPONER).
- SECURITY.md corto de respuesta a incidentes: a quién avisar, cómo rotar claves, cómo revocar sesiones y cómo poner la app en mantenimiento.

## FASE 11 · Específico según el tipo (solo lo que aplique)
- **WordPress/CMS:** todo actualizado, quitar lo que no se usa, xmlrpc y editor de archivos desactivados, 2FA en wp-admin, permisos 644/755, sin usuario "admin", bloquear la enumeración (?author=, /wp/v2/users) y backups.
- **App móvil:** tokens en Keychain o Keystore, pinning si procede, deep links validados, sin secretos en el binario, ofuscación, permisos mínimos y solo HTTPS.
- **Escritorio/CLI/bots:** firma del código, actualizaciones firmadas, nada de comandos con input sin sanear y credenciales en el almacén del sistema operativo.
- **Servidor propio:** SSH solo con clave y sin root, firewall mínimo, fail2ban, actualizaciones automáticas, y contenedores sin privilegios, sin root y sin secretos en la imagen.

## FASE 12 · Revisión profunda y pruebas de ataque (solo local o staging)
- Revisión del código buscando lo que no esté en las fases anteriores (línea a línea solo en EXHAUSTIVO).
- Pruebas reales con scripts o tests: fuerza bruta, saltarse el rate limit, admin sin permiso, cambiar IDs, XSS/SQLi, subir .php o .exe, archivos gigantes o con la extensión falsificada, manipular precios, reutilizar tokens, reenviar webhooks y CSRF desde otro origen.
- En producción, solo `curl -I` y TLS. Explica cómo pasar securityheaders.com y SSL Labs.
- Tests de seguridad en el repo con un comando único (y en CI si existe).
- Lista de pruebas manuales antes de fusionar a producción.

---

# ENTREGA FINAL (en SECURITY_AUDIT.md + resumen en el chat)
1. Resumen ejecutivo del antes y el después, para alguien no técnico.
2. Hallazgos por gravedad, con archivo y línea.
3. Lo arreglado, con su commit y su test o verificación.
4. Lo no arreglado, lo no verificable y las acciones del usuario, con pasos exactos.
5. Resultado de cada prueba de ataque (bloqueado / no bloqueado).
6. Lo que no aplica.
7. Riesgos residuales y checklist de mantenimiento mensual.