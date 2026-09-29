# Monitor de precios de gasolina en México

Pipeline ELT que captura diariamente los precios de gasolina y diésel de las
estaciones de servicio de México, conserva su historia y los cruza con el tipo
de cambio.

## Preguntas que responde

- ¿En qué estados y municipios es más cara la gasolina?
- ¿Qué estaciones cobran consistentemente más que su zona?
- ¿Cómo se relacionan los precios con el tipo de cambio peso-dólar?

## Arquitectura

*Pendiente: diagrama.*

## Fuentes de datos

| Fuente | Qué aporta | Frecuencia |
|---|---|---|
| XML de precios (CNE) | Precio por estación y producto | Diaria |
| API de estaciones (CNE, no documentada) | Nombre, dirección, estado y municipio | Semanal |
| Catálogos de estados y municipios (CNE) | Nombres de estados y municipios | Ocasional |
| API SIE de Banxico (serie SF43718) | Tipo de cambio FIX | Diaria (días hábiles) |

## Stack

Python · Airflow · Google Cloud Storage · BigQuery · dbt · Terraform · GitHub Actions

## Hallazgos principales

- El XML de precios no incluye ubicación; esta se obtiene de una API no
  documentada de la CNE, unida por número de permiso.
- La API de estaciones responde con éxito aunque la consulta sea inválida, por
  lo que el pipeline valida el total de estaciones en cada corrida.
- Los IDs de estado y municipio llegan en formatos distintos según la fuente y
  se normalizan antes de unir tablas.

Detalle completo en [docs/decisiones.md](docs/decisiones.md).

## Cómo ejecutarlo

*Pendiente.*

## Estado

- [x] Exploración de fuentes
- [ ] Fase 1: ingesta
- [ ] Fase 2: orquestación
- [ ] Fase 3: modelado con dbt
- [ ] Fase 4: calidad de datos
- [ ] Fase 5: infraestructura y CI
- [ ] Fase 6: dashboard