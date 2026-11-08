# Release OEFA · Antapaccay 2022 — 2022.1

Todo funciona en local. No hace falta red ni servidor.

## Abrir

- Doble clic en `visor.html`. Muestra procesos, tablas, inventario y cola de revisión.
- Para consultar los Parquet con SQL hace falta un servidor local:

```bash
cd /data/users/julio/apps/kipu-360/data/oefa/release
python -m http.server 8899
```

Y abrir `http://127.0.0.1:8899/visor.html`.

## Mapa de archivos

| Archivo | Qué es |
|---|---|
| `visor.html` | visor de una página, con todos los datos embebidos |
| `tablas/*.parquet` | las doce tablas del estándar |
| `archivos.parquet` | los 266 archivos del ZIP, con hash, rol y estado |
| `documentos.parquet` | los 104 PDF, con páginas, OCR y texto extraído |
| `imagenes.parquet` | las 8.639 imágenes extraídas, con su documento y página |
| `validacion.json` | claves, coherencia, bbox y motivos de bloqueo |
| `manifiesto.json` | conteos, SHA-256 de cada archivo y entorno |
| `fuentes.json` | SHA-256 de las 112 fuentes originales |
| `revision/cola.csv` | lo que necesita revisión humana, con motivo |
| `revision/resumen.json` | conteos de la cola por prioridad y responsable |
| `catalog.json` y `catalogo/` | STAC 1.0.0, válido con PySTAC |
| `docs/stations.geojson` | los 504 puntos para cualquier visor de mapas |
| `reconstruir.sh` | los cuatro comandos que regeneran todo |

Los originales quedan intactos en `../raw/`. Nada se borra.

## Los diez procesos

| N | Nombre | Qué hace | Deja |
|---|---|---|---|
| 1 | Recepción | guarda los ZIP y los descomprime | `raw/` |
| 2 | Inventario | pesa y hashea cada archivo | `archivos.parquet` |
| 3 | Clasificación | le da rol y estado a cada archivo | `archivos.parquet` |
| 4 | Extracción | PDF a texto e imágenes, OCR solo si falta texto | `text/`, `images/` |
| 5 | Digitalización | Excel de hoja ancha a una fila por celda | `tablas/celdas.parquet` |
| 6 | Normalización | celdas a las doce tablas | `tablas/*.parquet` |
| 7 | Validación | claves, coherencia, bbox, unidades | `validacion.json` |
| 8 | Cuarentena | lo dudoso va a una cola con motivo | `revision/cola.csv` |
| 9 | Publicación local | manifiesto, hash y STAC | `manifiesto.json` |
| 10 | Servicio local | esta página | `visor.html` |

## Los estados

| Estado | Significa |
|---|---|
| `VALID` | el dato se puede usar |
| `PENDING_REVIEW` | falta un dato de contexto, casi siempre la fecha |
| `EXCLUDED_FROM_ANALYSIS` | la celda decía "no aplica" |
| `CONTRATADO` | el archivo está indexado pero no convertido a tabla |

Cada archivo del ZIP tiene además su `disposicion`: `ingestado`, `indexado`,
`contraste`, `pendiente_extraccion`, `pendiente_ocr` o `pendiente_parser`.

## Lo que necesita revisión humana

- 65 items en `revision/cola.csv`, 42 de prioridad alta.
- 35 parámetros con más de una unidad.
- 17 documentos con páginas de OCR dudoso.
- 6.289 resultados sin fecha, porque la hoja no la trae.
- 160 archivos todavía sin procesar: 10 DOCX, 91 JPG y 53 BIN.
- 4 duplicados exactos y 6 libros derivados que no se usan como fuente.

## Lo que no se puede publicar todavía

- La licencia del expediente no está definida.
- Los PDF, DOCX y cadenas de custodia traen nombres y firmas.
- Los dos puntos anteriores los decide el cliente, no el pipeline.

## Reconstruir

```bash
bash reconstruir.sh
```

Al terminar, comparar `manifiesto.json` con la copia anterior. Si los conteos y
los hashes coinciden, el release es el mismo.
