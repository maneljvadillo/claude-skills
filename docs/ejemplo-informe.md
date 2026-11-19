# Ejemplo ficticio de SECURITY_AUDIT.md

Este documento ilustra el formato de entrega. El proyecto, el hallazgo y las rutas son ficticios. No se han ejecutado pruebas ni obtenido resultados reales para este ejemplo.

## 1. Contexto y modo de profundidad

- Proyecto ficticio: una API de facturas con dos clientes separados.
- Modo: estándar.
- Entorno propuesto: local, con cuentas y datos de prueba.
- Producción: fuera del alcance de las pruebas activas.

## 2. Modelo de amenazas

Un usuario autenticado del cliente A podría intentar leer una factura del cliente B cambiando su identificador. La revisión tendría que comprobar la autorización en el servidor y el acceso a los datos.

## 3. Plan de fases

En este ejemplo solo se ilustra un punto de la fase 4, autorización. El resto de las fases está pendiente de evaluación; no se considera revisado ni se marca como «no aplica».

## 4. Hallazgos

### SEC-001 · Comprobación de propiedad por verificar

| Campo | Ejemplo |
| --- | --- |
| Etiqueta | Riesgo probable |
| Gravedad propuesta | Alta, si una cuenta normal pudiera leer facturas de otro cliente |
| Archivo de ejemplo | `src/routes/invoices.ts:42`, ruta ficticia |
| Evidencia | Pendiente: inspeccionar el controlador real y las reglas de acceso a los datos |
| Impacto posible | Lectura de facturas de otro cliente cambiando el ID solicitado |
| Corrección propuesta | Comprobar en el servidor que la factura pertenece al cliente de la sesión |
| Rollback propuesto | Revertir el cambio concreto, sin alterar datos ni migraciones |
| Verificación propuesta | Una cuenta de A obtiene su propia factura y no puede obtener la de B; una petición anónima tampoco puede acceder |
| Estado | Propuesto; pendiente de evidencia, aprobación y pruebas |

No se etiqueta como vulnerabilidad confirmada: en este ejemplo faltan tanto la inspección como la reproducción del acceso indebido.

## 5. Decisiones del usuario

Ninguna. La propuesta no autoriza cambios por sí sola.

## 6. Acciones pendientes para el usuario

Confirmar el entorno autorizado y facilitar dos cuentas de prueba de clientes distintos, si el proyecto real dispone de ellas.

## 7. Registro de fases

Sin fases ejecutadas, commits ni resultados de pruebas. Un informe real debe registrar solo las acciones y verificaciones realizadas.
