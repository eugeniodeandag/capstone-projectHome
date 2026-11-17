# Actividad 2.13 — Organización y arquitectura del reto de agua

Equipo: Alejandro Vázquez Treviño, Javier Alejandro Chicuate Mendoza, Santiago Valencia Nájera y Eugenio De Anda Garza. Grupo 303. Profesor Servio Palacios. 17 de noviembre de 2026.

## Objetivo y usuarios

Diseñamos una solución BI para que Auditoría de Sustentabilidad de Coca-Cola FEMSA revise la consistencia de datos de descarga de agua, compare fuentes y priorice posibles inconsistencias. Usuarios: auditores y responsables ambientales por planta. Las alertas no constituyen un dictamen de incumplimiento y los originales no se corrigen automáticamente.

## Fuentes y alcance

Bitácoras Excel de lecturas/diferencias diarias y exportación mensual de Reporting Factory. Operación prevista en SharePoint de Microsoft 365; piloto con archivos autorizados del curso. Confirmar catálogo de plantas, unidades, tipos aplicables y periodos cerrados antes de definir la cobertura. No publicar los XLSX corporativos ni registros del socio en este repositorio; usar datos sintéticos para ejemplos y pruebas. El material de Pokémon es un ejemplo didáctico anterior, no el reto de agua.

## Preguntas y KPIs

1. ¿Qué plantas/meses pueden conciliarse? Cobertura = 100 × pares comparables / pares esperados.
2. ¿Dónde difieren las fuentes? Coincidencia = 100 × pares con |B-R| ≤ T / pares comparables. Diferencia absoluta de volumen = suma de |B-R|, en m³.
3. ¿Qué registros requieren revisión? Registros diarios con alerta = 100 × registros únicos alertados / registros diarios evaluables.

B: volumen mensual de bitácora. R: volumen de Reporting Factory. Par comparable: planta-mes-tipo único, ambas fuentes presentes, unidad y perímetro homologados y periodo completo. T = máximo(tolerancia absoluta en m³, tolerancia relativa × |B|); parámetros pendientes de acordar con Auditoría. Bitácora es referencia de conciliación, no verdad certificada. Denominador cero: No evaluable. No promediar porcentajes. Exponer numeradores, denominadores y excluidos.

## Arquitectura

Fuentes XLSX → ingesta Python batch (versiones, hashes y conteos) → Bronze Delta (originales/metadatos) → Silver Databricks/Python (claves, formato largo, unidades y cuarentena) → Gold Delta (agregación mensual, conciliación y reglas) → modelo semántico Power BI → dashboard y revisión humana.

Flujos: XLSX; originales/metadatos; tablas normalizadas m³; hechos de conciliación/alertas; consultas y medidas. Seguridad: permisos mínimos en orígenes y Unity Catalog previsto; credenciales fuera del código; datos corporativos en almacenamiento autorizado. Calidad: esquema, calendario, claves, duplicados, fuente/celda, periodos completos y registro de reglas. No convertir faltante en cero ni deduplicar sin evidencia.

Granos: lecturas por planta-fecha-tipo-medidor identificable; conciliación por planta-mes-tipo-versión; alerta por registro-regla-versión. Dimensiones Planta, Fecha, TipoDescarga y Regla; relaciones uno a muchos. No sumar lecturas acumuladas ni añadir el consolidado TOTAL al detalle.

Decisiones: batch porque las fuentes son archivos y cierres; Bronze/Silver/Gold por auditabilidad; reglas explicables antes que clasificación por falta de etiquetas confirmadas; Power BI por continuidad con el entorno descrito por el socio. Accesos y disponibilidad tecnológica pendientes de validar.

Semáforo: verde sin alertas en controles completos; amarillo para patrones que necesitan contexto; rojo para discrepancia fuera de tolerancia o error técnico confirmado; gris No evaluable. Colores priorizan revisión, no determinan cumplimiento.

## Trabajo y aceptación

### KAN-6 — Ingestar bitácora de descargas y Reporting Factory

Responsable: Santiago Valencia Nájera. Fecha acordada: 2026-11-18. Dependencia: Ninguna.

Aceptación: Registrar ambos XLSX, versión/fecha y SHA256; verificar hash de bitácora contra 66979994cacb55107c946b818f0e512fd744e5d7341d42be177727f1e1f6471d; inventariar hojas y conteos; conservar originales restringidos. No subir datos del socio al repositorio público.

### KAN-7 — Explorar cobertura, unidades y calidad del agua

Responsable: Alejandro Vázquez Treviño. Fecha acordada: 2026-11-18. Dependencia: KAN-6.

Aceptación: Documentar estructura de hojas, plantas, periodos, unidades, faltantes, duplicados, ceros y lecturas repetidas; distinguir lectura acumulada de volumen diario; confirmar la discrepancia de alcance 22/23 plantas y validar el catálogo sin contar totales consolidados como planta.

### KAN-8 — Preparar y homologar fuentes de descargas de agua

Responsable: Eugenio De Anda Garza. Fecha acordada: 2026-11-19. Dependencia: KAN-7.

Aceptación: Normalizar planta-fecha-tipo de descarga; documentar conversión de unidades confirmada y catálogo de alias; conservar valor original y trazabilidad hoja/celda; excluir subtotales del detalle y días inexistentes; separar faltante, cero y no aplica; no corregir anomalías sin evidencia.

### KAN-9 — Modelar conciliación y reglas de alertas

Responsable: Javier Alejandro Chicuate Mendoza. Fecha acordada: 2026-11-20. Dependencia: KAN-8.

Aceptación: Construir modelo diario y conciliación planta-mes-tipo; comparar solo periodos completos con unidades y perímetro homologados; parametrizar tolerancia absoluta/relativa, ceros, repeticiones y picos; registrar regla, versión y motivo; bloquear comparaciones ambiguas y excluirlas del KPI de coincidencia; no emitir dictamen de cumplimiento.

### KAN-10 — Diseñar dashboard de auditoría de descargas

Responsable: Alejandro Vázquez Treviño. Fecha acordada: 2026-11-20. Dependencia: KAN-9.

Aceptación: Mostrar cobertura de conciliación, coincidencia y registros con alertas; semáforo y filtros planta/mes/tipo/regla; detalle con valores de ambas fuentes y origen; mostrar no evaluable cuando falte evidencia; totales reconciliados con Gold y medidas documentadas.

### KAN-11 — Validar conciliación, KPIs y arquitectura de agua

Responsable: Javier Alejandro Chicuate Mendoza. Fecha acordada: 2026-11-20. Dependencia: KAN-9 y KAN-10.

Aceptación: Probar coincidencia, discrepancia, límites de tolerancia, unidad ambigua, cero válido, faltante, duplicado, reinicio de medidor y periodo incompleto; recalcular KPIs manualmente; revisión cruzada por Eugenio; conservar evidencia y preparar defensa de cada miembro. Aceptación del socio pendiente.

## Hitos y dependencias

18 nov: ingesta/perfilado. 19 nov: preparación. 20 nov: reglas, prototipo y validación. Hitos aprobados por el equipo el 17 de noviembre de 2026; no representan la fecha de entrega en Canvas. Dependencias: KAN-6 → KAN-7 → KAN-8 → KAN-9 → KAN-10 → KAN-11. Todas las tareas siguen por hacer. Revisión cruzada antes de cerrar cada entrega.

## Validación planeada

Probar coincidencia, límites de tolerancia, discrepancias, unidad ambigua, ceros válidos, repetición, reinicio, faltantes, duplicados y mes incompleto. Caso sintético: 10 pares esperados, 8 comparables y 6 coincidentes → cobertura 80%, coincidencia 75%. Dos registros alertados de 20 evaluables → 10%. Con unidad confirmada, 1,000 L = 1 m³. Reejecutar misma carga y reglas sin duplicar y contrastar KPIs con cálculo manual. Son pruebas propuestas, no resultados ejecutados.

## Defensa

Santiago: ingesta y conservación de originales. Alejandro: perfilado y mostrar cobertura junto a coincidencia. Eugenio: preparación, faltantes versus ceros y revisión cruzada. Javier: modelo y reglas explicables trazables. Cada integrante revisará su responsabilidad y explicará una decisión con sus palabras.

## Uso de IA

Codex apoyó planeación, alternativas y redacción. Incorporado al diseño: homologar grano planta-mes-tipo y separar capas para trazabilidad. Revisado: todos los ceros como errores; se sustituyó por alertas contextuales. Se descartó el borrador de fraude al confirmar el reto real. Diseño propuesto para implementación; las tolerancias y el alcance operativo requieren validación del socio.

## Recursos

- Repositorio: https://github.com/itd-messilovers/capstone-projectHome
- Jira: https://tec-083f9284.atlassian.net/jira/software/projects/KAN/boards/1
- Slack: https://app.slack.com/client/T0CFRNMP3EY/C0CF6RSDX4P

## Evidencias y acceso

- Reporte y capturas reales de Jira: https://docs.google.com/document/d/1NvBfUkq8SpPUJWpvSrvxvgBVL7wLOW52pmHf7xZKTec/edit
- Diagrama editable realizado en Lucidchart: https://lucid.app/lucidchart/0be30779-a25f-469b-8a4b-9fd18e1c0e57/edit
- Las dependencias KAN-6 → KAN-7 → KAN-8 → KAN-9 → KAN-10 → KAN-11 y KAN-9 → KAN-11 están vinculadas en Jira.

Javier compartirá el diagrama con el profesor. El reporte permite lectura por enlace y el repositorio es público. Confirmar con el socio el alcance 22/23 plantas, unidades y tolerancias antes de ejecutar comparaciones. No se ha entregado en Canvas. Esta documentación describe diseño propuesto, no una solución ya implementada.
