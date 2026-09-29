# Registro de exploración y decisiones

## Fuentes de datos

1. **Precios por estación (XML oficial):** se descarga desde el portal de la
   Comisión Nacional de Energía (CNE):
   https://www.cne.gob.mx/ConsultaPrecios/GasolinasyDiesel/GasolinasyDiesel.html
   Según la página, se actualiza diariamente a las 18:00 h (GMT-6).
2. **Estaciones (API no documentada de la CNE):** devuelve nombre, dirección,
   estado, municipio, producto, subproducto y precio de cada estación.
3. **Catálogos de estados y municipios (API de la CNE):** traducen los IDs a nombres.
4. **Tipo de cambio FIX (API SIE de Banxico):** serie SF43718, pesos por dólar.

## Precios (XML)

- El XML agrupa la información por estación. Cada estación se identifica con su
  número de **permiso**.
- El archivo no incluye fecha. La fecha de cada registro corresponde a la fecha
  de descarga, por lo que cada descarga se guarda en su propia carpeta con la fecha.
- Conviven dos formatos de permiso: `PL/.../EXP/ES/AAAA` y `CNE/PL/.../EXP/ES/2025`.
  **Hipótesis (sin verificar):** los permisos con prefijo `CNE/` fueron emitidos o
  reexpedidos por el nuevo regulador, por lo que una misma estación podría tener
  distintos identificadores a lo largo del tiempo.
- No todas las estaciones venden los mismos productos. Los tres productos son
  gasolina regular, gasolina premium y diésel; una estación puede vender los tres
  o solo algunos. La ausencia de un producto significa que no se vende, no que el
  precio sea cero.
- Los precios vienen como texto con decimales variables (`"27"`, `"26.9"`).
- **Supuesto:** los precios están en pesos mexicanos por litro. La fuente no
  indica las unidades.

## Estaciones (API no documentada)

El XML no incluye información geográfica. Con las herramientas de desarrollador
del navegador (pestaña Red) se identificaron las peticiones que usa la página:

- `api-catalogo.cne.gob.mx/entidadesfederativas`
- `api-catalogo.cne.gob.mx/municipios?EntidadFederativaId={id}`
- `api-reportediario.cne.gob.mx/api/EstacionServicio/Petroliferos?entidadId={id}&municipioId={id}`

Hallazgos:

1. Es obligatorio consultar por estado **y** municipio; la API no permite
   consultar un estado completo. Recorrer el país requiere unas 2,400 peticiones.
2. La API no tiene documentación pública (`/swagger` y `/help` no existen).
3. Los ceros a la izquierda no afectan: `entidadId=010` equivale a `entidadId=10`.
   Sin embargo, los catálogos devuelven los IDs como texto con ceros (`"010"`) y
   la API de estaciones como número (`10`). Hay que normalizarlos antes de unir tablas.
4. **Falla silenciosa:** con un estado o municipio inválido, la API responde
   `Success: true` y una lista vacía. No se distingue entre "no hay estaciones" y
   "la consulta no es válida". Mitigación: usar solo IDs de los catálogos y validar
   el total de estaciones de cada corrida.
5. El JSON viene envuelto en `Success`, `Errors` y `Value`; hay que revisar
   `Success` antes de procesar `Value`.
6. Los subproductos no son consistentes entre estaciones (por ejemplo, premium
   aparece como "mínimo de 92 octanos" y como "índice de octano mínimo de 91").
   Se requiere una tabla de mapeo a tres categorías.

## Vinculación entre fuentes

- El permiso es idéntico en el XML y en la API (verificado con
  `PL/6820/EXP/ES/2015`), y los precios coinciden entre ambas fuentes.
- La API de estaciones se une a los catálogos mediante `EntidadFederativaId`
  y `MunicipioId`.

## Pendientes

- Verificar si los permisos con prefijo `CNE/` aparecen en la API de estaciones.
- Decidir entre la serie FIX por fecha de determinación o por fecha de publicación.
- Definir qué tipo de cambio se asigna a fines de semana y días festivos.
- Verificar si las claves de estado y municipio coinciden con las del INEGI.