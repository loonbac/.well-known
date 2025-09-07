# 📋 ESPECIFICACIONES PARA DESARROLLO COLABORATIVO

Este directorio contiene un sistema completo de especificaciones técnicas para que puedas reconstruir `google_sheets_lib.py` desde cero, manteniendo 100% de compatibilidad con el bot existente.

## 🎯 OBJETIVO

Reconstruir la librería Google Sheets con:
- ✅ **Tu propio estilo de codificación**
- ✅ **Compatibilidad 100% garantizada** 
- ✅ **APIs oficiales de Google**
- ✅ **Funcionamiento silencioso**

## 📁 ARCHIVOS INCLUIDOS

### 📖 `SPECIFICATION.md`
**Documentación técnica completa**
- Arquitectura y requisitos del sistema
- Especificaciones detalladas de cada método
- Ejemplos de uso y comportamiento esperado
- Guías de implementación con APIs oficiales

### 🔌 `interface_contract.py`
**Contrato de interfaz abstracta**
- Clase base abstracta que define la interfaz exacta
- Validadores automáticos de tipos y formatos
- Documentación de cada método requerido
- Hereda de esta clase para garantizar compatibilidad

### 📊 `sample_data.py`
**Datos de prueba y resultados esperados**
- Datos mock para testing sin conexión real
- Resultados esperados para cada método
- Casos de prueba representativos
- Permite desarrollo offline completo

### 🧪 `test_compatibility.py`
**Suite de tests de compatibilidad**
- Tests unitarios completos
- Validación automática de compatibilidad
- Casos edge y manejo de errores
- Ejecuta con: `python test_compatibility.py`

### 🛠️ `validation_tool.py`
**Herramienta de validación en tiempo real**
- Validación automática durante desarrollo
- Monitoreo de cambios en archivos
- Feedback inmediato sobre problemas
- Tres modos de uso disponibles

## 🚀 CÓMO EMPEZAR

### 1️⃣ **Lee las especificaciones**
```bash
# Abre y lee completamente
SPECIFICATION.md
```

### 2️⃣ **Crea tu implementación**
```bash
# Crea tu archivo (puedes nombrarlo como quieras)
my_google_sheets_lib.py
```

### 3️⃣ **Hereda del contrato**
```python
from dev_specs.interface_contract import GoogleSheetsReaderContract

class GoogleSheetsReader(GoogleSheetsReaderContract):
    # Tu implementación aquí
    pass
```

### 4️⃣ **Desarrolla con validación**
```bash
# Validación rápida
python dev_specs/validation_tool.py

# Monitoreo automático (detecta cambios)
python dev_specs/validation_tool.py watch

# Validación básica
python dev_specs/validation_tool.py quick
```

### 5️⃣ **Tests de compatibilidad**
```bash
# Tests completos
python dev_specs/test_compatibility.py

# Tests específicos
python -m pytest dev_specs/test_compatibility.py::test_read_columns -v
```

## 🔧 HERRAMIENTAS DE DESARROLLO

### 🛠️ Validation Tool - Tres Modos

**Modo Normal** (validación única):
```bash
python dev_specs/validation_tool.py
```

**Modo Watch** (monitoreo continuo):
```bash
python dev_specs/validation_tool.py watch
```

**Modo Quick** (validación rápida):
```bash
python dev_specs/validation_tool.py quick
```

### 🧪 Testing System

**Tests completos**:
```bash
python dev_specs/test_compatibility.py
```

**Tests específicos**:
```bash
# Solo test de formato
python -c "from dev_specs.test_compatibility import *; test_format_rows_bonito()"

# Solo test de filtros
python -c "from dev_specs.test_compatibility import *; test_filtrar_proximos_vencimientos()"
```

### 📊 Sample Data Access

```python
from dev_specs.sample_data import *

# Ver datos de ejemplo
print("Datos de entrada:", SAMPLE_RAW_DATA)
print("Resultado esperado:", EXPECTED_READ_COLUMNS_RESULT)
```

## 📝 PROCESO DE DESARROLLO RECOMENDADO

### Paso 1: Estructura Básica
1. Crea `my_google_sheets_lib.py`
2. Implementa la clase heredando del contrato
3. Define todos los métodos (aunque estén vacíos)
4. Ejecuta `python dev_specs/validation_tool.py` para verificar estructura

### Paso 2: Implementación por Métodos
1. Implementa un método a la vez
2. Usa `sample_data.py` para testing offline
3. Valida con `validation_tool.py` después de cada cambio
4. Ejecuta tests específicos para el método implementado

### Paso 3: Integración y Testing
1. Implementa todos los métodos
2. Ejecuta la suite completa de tests
3. Corrige cualquier incompatibilidad detectada
4. Valida que todos los tests pasen

### Paso 4: Verificación Final
1. Ejecuta `python dev_specs/test_compatibility.py`
2. Verifica que todos los tests pasen ✅
3. Tu implementación está lista para usar

## 🎨 LIBERTAD CREATIVA

Puedes implementar:
- ✅ **Tu propio estilo de código**
- ✅ **Tu estructura interna preferida**
- ✅ **Tus propias optimizaciones**
- ✅ **Tu manejo de errores**
- ✅ **Tus propios comentarios**

**Lo único que DEBE mantenerse**:
- 🔒 **Nombres de métodos exactos**
- 🔒 **Firmas de métodos (parámetros)**
- 🔒 **Tipos de retorno**
- 🔒 **Comportamiento funcional**

## 🆘 SOLUCIÓN DE PROBLEMAS

### ❌ "No se puede importar my_google_sheets_lib"
- Asegúrate de que el archivo esté en el directorio correcto
- Verifica que la clase se llame exactamente `GoogleSheetsReader`

### ❌ "Tests fallan"
- Revisa `SPECIFICATION.md` para comportamiento exacto esperado
- Usa `sample_data.py` para verificar entrada/salida
- Ejecuta `validation_tool.py` para diagnóstico detallado

### ❌ "Errores de API de Google"
- Instala dependencias: `pip install google-api-python-client google-auth`
- Revisa la sección de APIs oficiales en `SPECIFICATION.md`
- Usa los ejemplos de autenticación proporcionados

### ❌ "Problemas de formato"
- Revisa `EXPECTED_FORMAT_RESULT` en `sample_data.py`
- El formato debe ser exactamente como se especifica
- Usa los casos de prueba para verificar

## 🏆 ÉXITO GARANTIZADO

Si sigues este proceso y todos los tests pasan, tu implementación será **100% compatible** con el bot existente. ¡Tu amigo podrá usar tu código directamente sin modificaciones!

## 📞 NECESITAS AYUDA?

1. **Lee primero**: `SPECIFICATION.md` tiene respuestas detalladas
2. **Ejecuta herramientas**: `validation_tool.py` te dirá qué está mal
3. **Revisa tests**: `test_compatibility.py` muestra comportamiento esperado
4. **Usa sample data**: `sample_data.py` tiene ejemplos completos

¡Buena suerte con tu implementación! 🚀
