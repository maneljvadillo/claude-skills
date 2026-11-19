# Auditoría de seguridad por fases

Una skill en español para que Claude Code revise la seguridad de tu proyecto, documente lo que encuentra y proponga arreglos que puedas comprobar.

Pensada para desarrolladores y creadores de webs, aplicaciones y APIs que quieren revisar su código con un proceso que puedan retomar entre sesiones. El archivo principal está en [`auditoria-seguridad/SKILL.md`](auditoria-seguridad/SKILL.md).

## Cómo funciona

1. **Reconoce el proyecto.** Lee el código y la configuración, pregunta lo que falta y determina qué partes de la revisión aplican.
2. **Prioriza con evidencia.** Distingue vulnerabilidades confirmadas, riesgos probables y mejoras defensivas. Pide justificar la gravedad y citar archivos y líneas.
3. **Propone antes de cambiar.** Presenta el plan de cada fase, sus efectos y cómo deshacer los cambios. Espera tu aprobación.
4. **Comprueba los arreglos.** Exige pruebas o verificaciones y registra el trabajo en `SECURITY_AUDIT.md`, junto con lo pendiente y lo que no pudo comprobarse.

La skill contiene instrucciones para el agente. Los comandos, pruebas y cambios los ejecuta Claude Code con las herramientas y permisos de tu entorno.

## Qué revisa

| Área | Ejemplos del contenido de la skill |
| --- | --- |
| Secretos y dependencias | Claves expuestas, historial de Git y cadena de suministro |
| Autenticación y permisos | Sesiones, acceso a recursos ajenos y separación entre clientes |
| Datos y entradas | Reglas de acceso a la base de datos, inyecciones y subidas de archivos |
| Pagos y abuso | Validación de precios, webhooks y límites de uso |
| Despliegue y privacidad | Cabeceras, configuración de red, datos personales y logs |

Hay 13 fases, de la 0 a la 12. El recorrido se adapta al proyecto: una web estática puede omitir las fases de autenticación, base de datos y pagos.

## Instalar en Claude Code

Necesitas Git y acceso a Claude Code. La skill no incluye una suscripción ni créditos de IA. Algunas comprobaciones necesitan herramientas adicionales, que el agente debe identificar antes de instalarlas.

En macOS o Linux, para disponer de ella en todos tus proyectos:

```bash
git clone https://github.com/maneljvadillo/claude-skills.git "$HOME/claude-skills"
mkdir -p "$HOME/.claude/skills"
if [ -e "$HOME/.claude/skills/auditoria-seguridad" ] || [ -L "$HOME/.claude/skills/auditoria-seguridad" ]; then
  printf '%s\n' 'La skill ya existe; se conserva la instalación actual.'
else
  ln -s "$HOME/claude-skills/auditoria-seguridad" "$HOME/.claude/skills/auditoria-seguridad"
fi
```

Si ya tienes el repositorio clonado en esa ruta, omite `git clone`. El bloque de instalación conserva cualquier skill que ya exista en la carpeta de destino.

Para instalarla solo en un proyecto, ejecuta desde la raíz de ese proyecto, después de clonar el repositorio:

```bash
mkdir -p .claude/skills
cp -R "$HOME/claude-skills/auditoria-seguridad" .claude/skills/
```

Usa esta segunda opción solo si aún no existe `.claude/skills/auditoria-seguridad`, para conservar cualquier versión que ya hayas modificado.

Claude Code admite skills personales en `~/.claude/skills/` y skills del proyecto en `.claude/skills/`. Consulta la [documentación oficial de skills](https://code.claude.com/docs/en/skills).

## Empezar una revisión

Abre Claude Code en el proyecto y escribe:

```text
/auditoria-seguridad Revisa este proyecto en modo estándar. Empieza por la fase 0 y espera mi aprobación antes de modificar código.
```

| Modo | Recorrido indicado en la skill |
| --- | --- |
| Rápido | Reconocimiento, secretos, dependencias y despliegue; prioriza problemas críticos y altos |
| Estándar | Todas las fases aplicables y mejoras defensivas de gravedad media o superior |
| Exhaustivo | Añade mejoras de gravedad baja y revisión profunda del código |

El agente te preguntará por el entorno de pruebas, los datos que maneja la aplicación y los accesos disponibles. Las pruebas ofensivas se limitan a local o staging, con autorización y datos de prueba.

## Qué recibes

Un archivo `SECURITY_AUDIT.md` en tu proyecto con contexto, modelo de amenazas, fases, hallazgos, decisiones y acciones pendientes. Cada hallazgo debe indicar su evidencia, impacto, propuesta de arreglo, verificación y estado.

Puedes ver un [ejemplo ficticio del formato de informe](docs/ejemplo-informe.md).

## Estado y límites

Versión inicial, abierta a comentarios de uso. El alcance de la revisión depende del modelo, de las herramientas y del acceso al proyecto. No se ha publicado una evaluación comparativa de detección ni de falsos positivos.

Revisa los cambios y sus pruebas antes de desplegarlos. Un informe generado por IA puede omitir problemas o equivocarse; no acredita que una aplicación esté libre de vulnerabilidades ni certifica cumplimiento normativo.

## Ayuda a mejorarla

Si la pruebas, [abre una incidencia](https://github.com/maneljvadillo/claude-skills/issues/new) indicando el tipo de proyecto, el modo utilizado y qué instrucción ayudó o falló. Un ejemplo mínimo que reproduzca el problema facilita corregirlo.

Comparte solo ejemplos que tengas permiso para publicar. Evita incluir claves, datos personales, código privado o informes de vulnerabilidades de aplicaciones ajenas.

Autor: [@maneljvadillo](https://github.com/maneljvadillo).

## Licencia

[MIT](LICENSE). Puedes usar, modificar y redistribuir el contenido, también con fines comerciales, conservando el aviso de copyright y la licencia.
