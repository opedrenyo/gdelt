# dbt_gdelt

Proyecto dbt para modelar datos de eventos de **GDELT Event Database 1.0**.

El objetivo es transformar la tabla raw de eventos GDELT en modelos analiticos tipados y documentados, apoyandose en seeds estaticos con codigos de referencia de GDELT/CAMEO.

## Estructura del proyecto

```text
.
├── dbt_project.yml
├── packages.yml
├── docs/
│   ├── CAMEO.Manual.1.1b3.pdf
│   ├── GDELT-Data_Format_Codebook.pdf
│   └── data_model.drawio
├── macros/
├── models/
│   └── staging/
│       ├── gdelt_event_1_0.sql
│       ├── schema.yml
│       └── sources.yml
└── seeds/
    ├── actortype_codes_static.csv
    ├── cameo_country_codes_static.csv
    ├── ethnic_codes_static.csv
    ├── event_codes_static.csv
    ├── fips_country_codes_static.csv
    ├── geotype_codes_static.csv
    ├── keds_code_static.csv
    ├── knowngroup_codes_static.csv
    ├── quad_class_static.csv
    └── religion_codes_static.csv
```

## Fuente de datos

La fuente principal esta definida en `models/staging/sources.yml`:

- Source: `GDELT`
- Database: `DEV_GDELT`
- Schema: `LANDING`
- Tabla raw: `GDELT_EVENT_DATABASE_1_0_RAW`
- Campo de frescura: `INGESTED_AT`

La tabla raw representa eventos geopoliticos publicados por GDELT. El proyecto valida que `GLOBALEVENTID` sea unico y no nulo, y que `SQLDATE` no sea nulo.

## Modelos

### `gdelt_event_1_0`

Modelo de staging que limpia y tipa la tabla raw de GDELT 1.0:

- Convierte identificadores y metricas a tipos numericos.
- Convierte fechas `YYYYMMDD` a `DATE`.
- Renombra columnas raw a nombres mas legibles en formato snake case.
- Se materializa como incremental con `unique_key='GLOBALEVENTID'` y estrategia `merge`.

El modelo incluye tests sobre:

- `GLOBAL_EVENT_ID`: `unique`, `not_null`
- `EVENT_DATE`: `not_null`

## Seeds

Los seeds contienen catalogos estaticos de referencia para enriquecer o validar campos codificados de GDELT:

- Tipos de actor
- Paises CAMEO y FIPS
- Codigos etnicos, religiosos y de grupos conocidos
- Codigos de eventos CAMEO/KEDS
- Tipos geograficos
- Clases `quad_class`

Por configuracion, los seeds se construyen en el schema con sufijo `dbt`.

## Dependencias

El proyecto usa:

```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: ">=1.3.0"
```

Instala o actualiza dependencias con:

```bash
dbt deps
```

## Uso

Configura primero el perfil `dbt_gdelt` en `~/.dbt/profiles.yml` apuntando al warehouse donde exista la tabla raw `DEV_GDELT.LANDING.GDELT_EVENT_DATABASE_1_0_RAW`.

Comandos habituales:

```bash
# Instalar dependencias
dbt deps

# Cargar tablas seed
dbt seed

# Ejecutar modelos y tests
dbt build

# Ejecutar solo staging
dbt build --select staging

# Comprobar frescura de la fuente
dbt source freshness
```

## Documentacion incluida

La carpeta `docs/` contiene material de apoyo para entender el modelo de datos:

- `GDELT-Data_Format_Codebook.pdf`: formato de datos de GDELT.
- `CAMEO.Manual.1.1b3.pdf`: manual de codificacion CAMEO.
- `data_model.drawio`: diagrama del modelo de datos.

## Notas de desarrollo

- `target/`, `logs/`, `dbt_packages/`, `dbt_internal_packages/` y `.env` estan excluidos del control de versiones.
- La configuracion activa `+static_analysis: strict`.
- El proyecto todavia no define modelos de marts; la capa actual esta centrada en staging y tablas de referencia.
