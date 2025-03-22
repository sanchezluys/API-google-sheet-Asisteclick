# Guía Práctica: API de Búsqueda en Google Sheets para AsisteClick 🚀

## Índice 📑

1. [Descripción general ℹ️](#descripción-general)
2. [Configuración del Google Sheet 📊](#configuración-del-google-sheet)
3. [Parámetros del API 🔧](#parámetros-del-api)
4. [Casos de uso con ejemplos de cURL 💻](#casos-de-uso-con-ejemplos-de-curl)
   - [Búsqueda básica en modo PHRASE 🔎](#búsqueda-básica-en-modo-phrase)
   - [Búsqueda por palabras en modo WORD 📝](#búsqueda-por-palabras-en-modo-word)
   - [Búsqueda con resultado más similar (one_hot_field) 🎯](#búsqueda-con-resultado-más-similar-one_hot_field)
   - [Aleatorización de resultados 🎲](#aleatorización-de-resultados)
   - [Exclusión de columnas en la búsqueda 🙈](#exclusión-de-columnas-en-la-búsqueda)
   - [Resultados como texto plano 🗒️](#resultados-como-texto-plano)
   - [Resultados como enlaces markdown 🔗](#resultados-como-enlaces-markdown)
5. [Escenarios de implementación comunes 💡](#escenarios-de-implementación-comunes)
   - [Sistema de preguntas frecuentes (FAQ) ❓](#sistema-de-preguntas-frecuentes-faq)
   - [Buscador de productos 🛍️](#buscador-de-productos)
   - [Chatbot de atención al cliente 🤖](#chatbot-de-atención-al-cliente)
   - [Base de conocimiento con enlaces 📚](#base-de-conocimiento-con-enlaces)
6. [Combinaciones avanzadas de parámetros ⚙️](#combinaciones-avanzadas-de-parámetros)
7. [Solución de problemas comunes 🛠️](#solución-de-problemas-comunes)
8. [Limitaciones y consideraciones ⚠️](#limitaciones-y-consideraciones)

## Descripción general ℹ️

Este servicio permite realizar búsquedas en hojas de Google Sheets desde una aplicación, utilizando una API REST. El servicio realiza búsquedas flexibles, maneja la normalización de texto (eliminando acentos y caracteres especiales), y puede devolver resultados en diferentes formatos según las necesidades de tu aplicación.

## Configuración del Google Sheet 📊

### 1. Crear y preparar la hoja de cálculo 📝

1. Crea una hoja de cálculo en Google Sheets.
2. La **primera fila** debe contener los nombres de las columnas:
   - Estos nombres serán las claves en los objetos JSON devueltos.
   - Evita espacios o caracteres especiales en los nombres de columnas.
   - Ejemplo: `Nombre`, `Descripcion`, `Precio`, `Categoria`.
3. A partir de la **segunda fila**, añade tus datos.

### 2. Configurar permisos de acceso 🔑

1. Haz clic en el botón "Compartir" en la esquina superior derecha.
2. Cambia la configuración a "Cualquier persona con el enlace puede ver".
   - Si no quieres que sea público, usa una cuenta de servicio con acceso específico.

### 3. Obtener credenciales necesarias 🔐

#### ID del Spreadsheet

El ID se encuentra en la URL de tu hoja de cálculo:
```
https://docs.google.com/spreadsheets/d/YOUR_SPREADSHEET_ID/edit#gid=0
```

Por ejemplo:
```
https://docs.google.com/spreadsheets/d/YOUR_SPREADSHEET_ID/edit
```
El ID es: `YOUR_SPREADSHEET_ID`

#### Nombre de la hoja (sheet_id)

Es el nombre de la pestaña en tu hoja de cálculo (por defecto "Hoja 1" o "Sheet1").

#### Clave de API para Google Sheets

1. Ve a la [Consola de Google Cloud](https://console.cloud.google.com/).
2. Crea un nuevo proyecto o selecciona uno existente.
3. Busca y habilita la "Google Sheets API".
4. En el menú, ve a "Credenciales" y crea una clave de API.
5. (Opcional) Restringe la clave para que solo pueda acceder a la API de Google Sheets.

## Parámetros del API 🔧

El servicio espera un objeto JSON con los siguientes parámetros:

| Parámetro            | Tipo    | Requerido | Descripción                                                                                  |
|----------------------|---------|-----------|----------------------------------------------------------------------------------------------|
| `sheet_api_key`      | string  | Sí        | Clave de API para acceder a Google Sheets.                                                   |
| `spreadsheet_id`     | string  | Sí        | ID de la hoja de cálculo de Google.                                                          |
| `sheet_id`           | string  | Sí        | Nombre de la hoja (tab).                                                                      |
| `search`             | string  | Sí        | Texto a buscar.                                                                              |
| `search_mode`        | string  | Sí        | Modo de búsqueda: "PHRASE" (frase exacta) o "WORD" (cualquier palabra).                      |
| `one_hot_field`      | string  | No        | Columna para encontrar el resultado más similar.                                           |
| `shuffle_results`    | boolean | No        | Aleatoriza el orden de los resultados (por defecto: false).                                  |
| `result_typeof`      | string  | Sí        | Formato de resultados: "OBJECT", "STRING" o "LINK".                                          |
| `result_object_text` | string  | Sí*       | Columna para mostrar como texto (*requerido si result_typeof es "STRING" o "LINK").           |
| `result_object_link` | string  | Sí*       | Columna para usar como URL (*requerido si result_typeof es "LINK").                           |
| `exclude_columns`    | array   | No        | Lista de columnas a excluir de la búsqueda.                                                  |

## Casos de uso con ejemplos de cURL 💻

### Búsqueda básica en modo PHRASE 🔎

Busca una frase exacta y devuelve objetos JSON completos:

```bash
curl -X POST https://tudominio.com/api/sheets-search \
  -H "Content-Type: application/json" \
  -d '{
    "sheet_api_key": "YOUR_API_KEY",
    "spreadsheet_id": "YOUR_SPREADSHEET_ID",
    "sheet_id": "Productos",
    "search": "zapatos deportivos",
    "search_mode": "PHRASE",
    "result_typeof": "OBJECT"
  }'
```

Respuesta:
```json
[
  {
    "Producto": "Zapatos deportivos Nike",
    "Descripcion": "Zapatos deportivos para running",
    "Precio": "89.99",
    "Categoria": "Calzado"
  },
  {
    "Producto": "Zapatos deportivos Adidas",
    "Descripcion": "Zapatos deportivos para gimnasio",
    "Precio": "79.99",
    "Categoria": "Calzado"
  }
]
```

### Búsqueda por palabras en modo WORD 📝

Busca cualquiera de las palabras y devuelve objetos JSON:

```bash
curl -X POST https://tudominio.com/api/sheets-search \
  -H "Content-Type: application/json" \
  -d '{
    "sheet_api_key": "YOUR_API_KEY",
    "spreadsheet_id": "YOUR_SPREADSHEET_ID",
    "sheet_id": "Productos",
    "search": "zapatos rojos",
    "search_mode": "WORD",
    "result_typeof": "OBJECT"
  }'
```

Respuesta:
```json
[
  {
    "Producto": "Zapatos deportivos rojos",
    "Descripcion": "Zapatos deportivos color rojo",
    "Precio": "89.99",
    "Categoria": "Calzado"
  },
  {
    "Producto": "Zapatos formales",
    "Descripcion": "Zapatos formales para oficina",
    "Precio": "120.00",
    "Categoria": "Calzado"
  },
  {
    "Producto": "Pantalones rojos",
    "Descripcion": "Pantalones de algodón color rojo",
    "Precio": "45.99",
    "Categoria": "Ropa"
  }
]
```

### Búsqueda con resultado más similar (one_hot_field) 🎯

Busca y devuelve solo el resultado con mayor similitud en la columna especificada:

```bash
curl -X POST https://tudominio.com/api/sheets-search \
  -H "Content-Type: application/json" \
  -d '{
    "sheet_api_key": "YOUR_API_KEY",
    "spreadsheet_id": "YOUR_SPREADSHEET_ID",
    "sheet_id": "FAQ",
    "search": "como cambiar contraseña",
    "search_mode": "WORD",
    "one_hot_field": "Pregunta",
    "result_typeof": "OBJECT"
  }'
```

Respuesta:
```json
[
  {
    "Pregunta": "¿Cómo puedo cambiar mi contraseña?",
    "Respuesta": "Para cambiar tu contraseña, ve a Configuración > Seguridad > Contraseña y sigue las instrucciones."
  }
]
```

### Aleatorización de resultados 🎲

Busca y devuelve resultados en orden aleatorio:

```bash
curl -X POST https://tudominio.com/api/sheets-search \
  -H "Content-Type: application/json" \
  -d '{
    "sheet_api_key": "YOUR_API_KEY",
    "spreadsheet_id": "YOUR_SPREADSHEET_ID",
    "sheet_id": "Frases",
    "search": "motivación",
    "search_mode": "WORD",
    "shuffle_results": true,
    "result_typeof": "OBJECT"
  }'
```

### Exclusión de columnas en la búsqueda 🙈

Busca en todas las columnas excepto las especificadas:

```bash
curl -X POST https://tudominio.com/api/sheets-search \
  -H "Content-Type: application/json" \
  -d '{
    "sheet_api_key": "YOUR_API_KEY",
    "spreadsheet_id": "YOUR_SPREADSHEET_ID",
    "sheet_id": "Productos",
    "search": "rojo",
    "search_mode": "WORD",
    "exclude_columns": ["SKU", "ID", "CodigoInterno"],
    "result_typeof": "OBJECT"
  }'
```

### Resultados como texto plano 🗒️

Busca y devuelve solo el texto de una columna específica:

```bash
curl -X POST https://tudominio.com/api/sheets-search \
  -H "Content-Type: application/json" \
  -d '{
    "sheet_api_key": "YOUR_API_KEY",
    "spreadsheet_id": "YOUR_SPREADSHEET_ID",
    "sheet_id": "FAQ",
    "search": "devolución",
    "search_mode": "WORD",
    "result_typeof": "STRING",
    "result_object_text": "Respuesta"
  }'
```

Respuesta:
```
Para realizar una devolución, contacta con nuestro servicio de atención al cliente en el plazo de 14 días desde la recepción del producto.<br/><br/>La política de devoluciones permite cambios durante los primeros 30 días para productos no usados.
```

### Resultados como enlaces markdown 🔗

Busca y devuelve resultados como enlaces en formato markdown:

```bash
curl -X POST https://tudominio.com/api/sheets-search \
  -H "Content-Type: application/json" \
  -d '{
    "sheet_api_key": "YOUR_API_KEY",
    "spreadsheet_id": "YOUR_SPREADSHEET_ID",
    "sheet_id": "Recursos",
    "search": "tutorial",
    "search_mode": "WORD",
    "result_typeof": "LINK",
    "result_object_text": "Titulo",
    "result_object_link": "URL"
  }'
```

Respuesta:
```
[Tutorial de inicio rápido](https://ejemplo.com/tutoriales/inicio)<br/><br/>[Tutorial avanzado de la API](https://ejemplo.com/tutoriales/api-avanzado)
```

## Escenarios de implementación comunes 💡

### Sistema de preguntas frecuentes (FAQ) ❓

**Configuración del Google Sheet**:
- Columnas: `Pregunta`, `Respuesta`

**Ejemplo de solicitud**:
```bash
curl -X POST https://tudominio.com/api/sheets-search \
  -H "Content-Type: application/json" \
  -d '{
    "sheet_api_key": "YOUR_API_KEY",
    "spreadsheet_id": "YOUR_SPREADSHEET_ID",
    "sheet_id": "FAQ",
    "search": "¿Cómo puedo cancelar mi suscripción?",
    "search_mode": "WORD",
    "one_hot_field": "Pregunta",
    "result_typeof": "STRING",
    "result_object_text": "Respuesta"
  }'
```

Esta configuración:
1. Busca palabras clave de la pregunta del usuario.
2. Encuentra la pregunta más similar en la columna "Pregunta".
3. Devuelve solo el texto de la respuesta.

### Buscador de productos 🛍️

**Configuración del Google Sheet**:
- Columnas: `Producto`, `Descripcion`, `Precio`, `Categoria`, `ImagenURL`, `SKU`

**Ejemplo de solicitud**:
```bash
curl -X POST https://tudominio.com/api/sheets-search \
  -H "Content-Type: application/json" \
  -d '{
    "sheet_api_key": "YOUR_API_KEY",
    "spreadsheet_id": "YOUR_SPREADSHEET_ID",
    "sheet_id": "Productos",
    "search": "camiseta algodón azul",
    "search_mode": "WORD",
    "exclude_columns": ["SKU", "ImagenURL"],
    "result_typeof": "OBJECT"
  }'
```

Esta configuración:
1. Busca productos que contengan cualquiera de las palabras.
2. No busca en las columnas SKU e ImagenURL.
3. Devuelve objetos completos con todos los datos del producto.

### Chatbot de atención al cliente 🤖

**Configuración del Google Sheet**:
- Columnas: `Intento`, `Patrones`, `Respuesta`

**Ejemplo de solicitud**:
```bash
curl -X POST https://tudominio.com/api/sheets-search \
  -H "Content-Type: application/json" \
  -d '{
    "sheet_api_key": "YOUR_API_KEY",
    "spreadsheet_id": "YOUR_SPREADSHEET_ID",
    "sheet_id": "ChatbotRespuestas",
    "search": "Hola, quiero hacer un pedido",
    "search_mode": "WORD",
    "one_hot_field": "Patrones",
    "shuffle_results": true,
    "result_typeof": "STRING",
    "result_object_text": "Respuesta"
  }'
```

Esta configuración:
1. Busca palabras clave del usuario.
2. Encuentra el patrón más similar en la columna "Patrones".
3. Si hay múltiples respuestas, elige una al azar.
4. Devuelve solo el texto de la respuesta.

### Base de conocimiento con enlaces 📚

**Configuración del Google Sheet**:
- Columnas: `Tema`, `Descripcion`, `URLRecurso`

**Ejemplo de solicitud**:
```bash
curl -X POST https://tudominio.com/api/sheets-search \
  -H "Content-Type: application/json" \
  -d '{
    "sheet_api_key": "YOUR_API_KEY",
    "spreadsheet_id": "YOUR_SPREADSHEET_ID",
    "sheet_id": "BaseConocimiento",
    "search": "configuración cuenta",
    "search_mode": "WORD",
    "result_typeof": "LINK",
    "result_object_text": "Tema",
    "result_object_link": "URLRecurso"
  }'
```

Esta configuración:
1. Busca recursos relacionados con palabras clave.
2. Devuelve los resultados como enlaces en formato markdown.
3. Usa el Tema como texto del enlace y URLRecurso como la URL.

## Combinaciones avanzadas de parámetros ⚙️

### Chatbot con respuestas aleatorias

Para un chatbot que seleccione una respuesta aleatoria entre las más relevantes:

```bash
curl -X POST https://tudominio.com/api/sheets-search \
  -H "Content-Type: application/json" \
  -d '{
    "sheet_api_key": "YOUR_API_KEY",
    "spreadsheet_id": "YOUR_SPREADSHEET_ID",
    "sheet_id": "Dialogo",
    "search": "hola como estas",
    "search_mode": "WORD",
    "one_hot_field": "Usuario",
    "shuffle_results": true,
    "result_typeof": "STRING",
    "result_object_text": "Respuesta"
  }'
```

### Buscador multilingüe

Aprovecha la normalización de texto para buscar con o sin acentos:

```bash
curl -X POST https://tudominio.com/api/sheets-search \
  -H "Content-Type: application/json" \
  -d '{
    "sheet_api_key": "YOUR_API_KEY",
    "spreadsheet_id": "YOUR_SPREADSHEET_ID",
    "sheet_id": "Productos",
    "search": "café orgánico",
    "search_mode": "PHRASE",
    "result_typeof": "OBJECT"
  }'
```

Esta búsqueda encontrará "café orgánico", "cafe organico", "CAFÉ ORGÁNICO", etc.

### Sistema de recomendación

Para recomendar productos relacionados:

```bash
curl -X POST https://tudominio.com/api/sheets-search \
  -H "Content-Type: application/json" \
  -d '{
    "sheet_api_key": "YOUR_API_KEY",
    "spreadsheet_id": "YOUR_SPREADSHEET_ID",
    "sheet_id": "Productos",
    "search": "zapatos running hombre",
    "search_mode": "WORD",
    "shuffle_results": true,
    "result_typeof": "OBJECT"
  }'
```

## Solución de problemas comunes 🛠️

### No se obtienen resultados

Verifica:
1. Que los permisos del Google Sheet estén configurados correctamente.
2. Que la clave de API tenga acceso a la API de Google Sheets.
3. Que el ID del spreadsheet y el nombre de la hoja sean correctos.
4. Que los términos de búsqueda existan en los datos (prueba con términos más generales).
5. Que no estés excluyendo columnas donde podría estar el término buscado.

### Errores de formato en los resultados

Si los resultados en formato `STRING` o `LINK` no se muestran correctamente:
1. Verifica que los campos `result_object_text` y `result_object_link` correspondan exactamente a nombres de columnas existentes.
2. Comprueba que no hay espacios adicionales en los nombres de las columnas.

### Rendimiento lento

Si la API responde lentamente:
1. Limita el rango de datos solicitado en la API de Google Sheets (modifica el código).
2. Implementa caché para reducir las llamadas a Google Sheets.
3. Considera migrar a una base de datos más eficiente si el volumen de datos es muy grande.

## Limitaciones y consideraciones ⚠️

### Limitaciones de Google Sheets

- **Cuota de API**: Google Sheets API tiene límites de uso. Verifica las [cuotas actuales](https://developers.google.com/sheets/api/limits).
- **Tamaño de datos**: No es recomendable para conjuntos de datos muy grandes (>10,000 filas).
- **Concurrencia**: No está diseñado para alto número de peticiones simultáneas.

### Seguridad

- **Clave de API**: No expongas tu clave de API directamente en el cliente.
- **Datos sensibles**: No almacenes información confidencial en Google Sheets con acceso público.
- **Validación**: Siempre valida y sanitiza las entradas del usuario.

### Optimización

- **Columnas**: Usa solo las columnas necesarias.
- **Estructura**: Mantén los datos estructurados de manera consistente.
- **Exclusión**: Excluye columnas que no son relevantes para la búsqueda para mejorar el rendimiento.
