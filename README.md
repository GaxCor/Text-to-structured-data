# PlanetaFiscal

Script en Python que lee archivos con texto desordenado (correos, facturas, quejas) y los convierte en JSON estructurado listo para base de datos. Usa la API de OpenAI para interpretar el contenido.

---

## Formatos soportados

| Formato | Librería |
|---------|----------|
| `.txt`  | Built-in (detección automática de encoding) |
| `.pdf`  | [pdfplumber](https://github.com/jsvine/pdfplumber) |
| `.docx` | [python-docx](https://python-docx.readthedocs.io/) |
| `.xlsx` / `.xls` | [openpyxl](https://openpyxl.readthedocs.io/) |

---

## Instalación

```bash
pip install -r requirements.txt
```

Crear o editar el archivo `.env` en la raíz del proyecto:

```env
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxx
```

---

## Uso

1. Colocar los archivos a procesar en la carpeta `datos_entrada/`.
2. Ejecutar el script:

```bash
python procesador.py
```

3. Los resultados se generan automáticamente en `datos_salida/`:
   - **Un JSON por archivo** procesado (`*_resultado.json`)
   - **Un resumen consolidado** (`resumen_completo.json`)

---

## Salida JSON

Cada archivo procesado genera un JSON con esta estructura:

```json
{
  "nombre_cliente": "Roberto Medina",
  "monto": 45000,
  "fecha": "2026-03-15",
  "tipo_solicitud": "Venta"
}
```

### Campos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `nombre_cliente` | `string` | Nombre de la persona o empresa |
| `monto` | `number \| null` | Monto principal. `null` si no se menciona |
| `fecha` | `string` | Fecha relevante en formato `YYYY-MM-DD` |
| `tipo_solicitud` | `string` | Solo: `Venta`, `Queja` o `Factura` |

---

## Validación

- Se verifica que la respuesta sea JSON válido con todos los campos requeridos.
- Si la respuesta viene mal formateada, se reintenta hasta 3 veces.
- Si falla después de los reintentos, se registra el error y continúa con el siguiente archivo.

---

## Inserción en SQL

Los datos extraídos se pueden insertar en una tabla SQL con queries parametrizados:

```python
cursor.execute(
    "INSERT INTO solicitudes (nombre_cliente, monto, fecha, tipo_solicitud) VALUES (%s, %s, %s, %s)",
    (datos["nombre_cliente"], datos["monto"], datos["fecha"], datos["tipo_solicitud"])
)
```

---

## Estructura del proyecto

```
PlanetaFiscal/
├── .env                  # API key (no se versiona)
├── .env.example          # Plantilla de referencia
├── .gitignore
├── requirements.txt
├── procesador.py         # Script principal
├── datos_entrada/        # Archivos de entrada (cualquier formato soportado)
└── datos_salida/         # JSON generados automáticamente
```
