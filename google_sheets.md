# 📊 Especificación Técnica: Módulo Google Sheets (google_sheets_lib.py)

## 🎯 Objetivo

Crear una librería Python que permita leer y escribir datos de Google Sheets usando credenciales de Service Account, sin dependencias externas como `gspread`. Solo debe usar librerías estándar de Python (`urllib`, `json`, `datetime`) y `PyJWT` para la autenticación.

## 📋 Clase Principal

### `GoogleSheetsReader`

**Propósito**: Manejar todas las operaciones de lectura y escritura con Google Sheets usando la API REST de Google.

## 🔧 Constructor

```python
def __init__(self, creds_path, sheet_id):
```

**Parámetros:**
- `creds_path` (str): Ruta al archivo JSON con las credenciales del service account
- `sheet_id` (str): ID de la hoja de cálculo de Google (extraído de la URL)

**Funcionalidad:**
- Cargar y almacenar las credenciales del service account
- Guardar el ID de la hoja para uso posterior

## 🔐 Autenticación

### `get_access_token(self)`

**Retorna**: `str` - Token de acceso válido

**Funcionalidad:**
- Crear un JWT usando las credenciales del service account
- Usar el scope: `https://www.googleapis.com/auth/spreadsheets` (lectura y escritura)
- Intercambiar el JWT por un access token válido
- Token válido por 1 hora (3600 segundos)

**Implementación requerida:**
- Header JWT: `{'alg': 'RS256', 'typ': 'JWT'}`
- Payload debe incluir: `iss`, `scope`, `aud`, `iat`, `exp`
- Usar `PyJWT` para encodear con la private_key
- Hacer request POST a `token_uri` con grant_type JWT

## 📖 Lectura de Datos

### `read_columns(self, range_c, range_d, range_h, range_j=None, range_monto=None)`

**Parámetros:**
- `range_c` (str): Rango para columna C (nombres de empresa) ej: "Hoja 1!C2:C"
- `range_d` (str): Rango para columna D (número de factura) ej: "Hoja 1!D2:D"  
- `range_h` (str): Rango para columna H (fecha vencimiento) ej: "Hoja 1!H2:H"
- `range_j` (str, opcional): Rango para columna J (estado pago) ej: "Hoja 1!J2:J"
- `range_monto` (str, opcional): Rango para columna de monto ej: "Hoja 1!E2:E"

**Retorna**: `list[dict]` - Lista de diccionarios con estructura:
```python
{
    'id': int,          # Número de fila (índice + 2)
    'nombre': str,      # Nombre de empresa
    'factura': str,     # Número de factura
    'vencimiento': str, # Fecha de vencimiento
    'estado': str,      # Estado del pago (si range_j proporcionado)
    'monto': str        # Monto (si range_monto proporcionado)
}
```

**Funcionalidad:**
- Hacer múltiples requests a la API de Google Sheets para cada rango
- URL base: `https://sheets.googleapis.com/v4/spreadsheets/{sheet_id}/values/{rango_codificado}`
- **IMPORTANTE**: Codificar el rango con `urllib.parse.quote()` para manejar espacios
- Combinar todas las columnas por índice de fila
- Asignar ID único basado en número de fila (índice + 2)
- Manejar filas vacías o de diferentes longitudes

## ✏️ Escritura de Datos

### `actualizar_pagado(self, fila_id, sheet_name='Hoja 1', col_j='J')`

**Parámetros:**
- `fila_id` (int): Número de fila a actualizar
- `sheet_name` (str): Nombre de la hoja (por defecto 'Hoja 1')
- `col_j` (str): Columna a actualizar (por defecto 'J')

**Retorna**: `bytes` - Respuesta del servidor

**Funcionalidad:**
- Actualizar una celda específica con el valor "PAGADO"
- Usar método PUT con la API de Google Sheets
- URL: `https://sheets.googleapis.com/v4/spreadsheets/{sheet_id}/values/{rango_codificado}?valueInputOption=RAW`
- **IMPORTANTE**: Codificar el rango con `urllib.parse.quote()`
- Body JSON: `{"values": [["PAGADO"]]}`

## 🔍 Filtros de Datos

### `filtrar_proximos_vencimientos(self, rows, dias=5)`

**Parámetros:**
- `rows` (list[dict]): Lista de filas obtenidas con `read_columns()`
- `dias` (int): Número de días hacia el futuro a considerar (por defecto 5)

**Retorna**: `list[dict]` - Filas que vencen en los próximos N días

**Funcionalidad:**
- Parsear fechas en múltiples formatos: `%d/%m/%Y`, `%Y-%m-%d`, `%d-%m-%Y`, `%d/%m/%y`
- Comparar con fecha actual
- Filtrar filas donde: `0 <= (fecha_vencimiento - hoy).days <= dias`
- Ignorar filas con fechas inválidas o no parseables

### `filtrar_pendientes(self, rows)`

**Parámetros:**
- `rows` (list[dict]): Lista de filas obtenidas con `read_columns()`

**Retorna**: `list[dict]` - Filas con fecha de vencimiento >= hoy

**Funcionalidad:**
- Similar a `filtrar_proximos_vencimientos()` pero sin límite superior
- Filtrar filas donde: `(fecha_vencimiento - hoy).days >= 0`
- Incluye facturas que vencen hoy o en el futuro

## 🎨 Formateo de Datos

### `format_rows_bonito(self, rows)`

**Parámetros:**
- `rows` (list[dict]): Lista de filas a formatear

**Retorna**: `str` - Texto formateado para mostrar

**Formato esperado:**
```
ID: {id}
Empresa: {nombre}
Factura: {factura}
Monto: {monto}
Vencimiento: {vencimiento}
Estado: {estado}
-------------------------
```

**Funcionalidad:**
- Solo incluir filas que tengan al menos uno de: nombre, factura, vencimiento
- Separar cada factura con línea de guiones
- Retornar "No hay datos." si no hay filas válidas

## 🛠️ Consideraciones Técnicas

### Manejo de URLs
- **CRÍTICO**: Usar `urllib.parse.quote()` para codificar rangos que contengan espacios
- Ejemplo: `"Hoja 1!C2:C"` debe convertirse a `"Hoja%201!C2:C"`

### Manejo de Errores
- Capturar y manejar excepciones de red (`urllib.error.HTTPError`)
- Validar respuestas JSON de la API
- Manejar casos donde las columnas tienen diferentes longitudes

### Autenticación
- Los tokens tienen duración de 1 hora
- No es necesario implementar refresh automático (se genera nuevo token por llamada)
- Usar scope completo de Sheets, no readonly

### Formatos de Fecha
- Soportar múltiples formatos comunes
- Usar `.strip()` para limpiar espacios
- Ser robusto ante fechas inválidas

## 📁 Dependencias Requeridas

```python
import json
import urllib.request
import urllib.parse
from datetime import datetime, timedelta
import jwt  # PyJWT
```

## 🔗 Endpoints de Google API

### Lectura:
- GET `https://sheets.googleapis.com/v4/spreadsheets/{spreadsheetId}/values/{range}`

### Escritura:
- PUT `https://sheets.googleapis.com/v4/spreadsheets/{spreadsheetId}/values/{range}?valueInputOption=RAW`

### Autenticación:
- POST a `token_uri` del service account con JWT assertion

## ✅ Casos de Prueba

1. **Lectura básica**: Leer 3 columnas obligatorias (C, D, H)
2. **Lectura completa**: Leer 5 columnas (C, D, H, J, monto)
3. **Filtro de vencimientos**: Facturas que vencen en 5 días
4. **Filtro pendientes**: Todas las facturas futuras
5. **Actualización**: Marcar factura como "PAGADO"
6. **Manejo de espacios**: Hojas con nombres como "Hoja 1"
7. **Filas vacías**: Manejar columnas de diferentes longitudes

## 🚨 Puntos Críticos

1. **Codificación de URLs**: Sin esto, fallarán las hojas con espacios en el nombre
2. **Scope de permisos**: Debe ser escritura completa, no readonly
3. **Formato de JWT**: Header y payload exactos para Google
4. **Manejo de filas**: ID debe corresponder al número real de fila en Sheets
5. **Parsing de fechas**: Múltiples formatos para máxima compatibilidad
