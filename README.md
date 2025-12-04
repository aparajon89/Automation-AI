# Alerta de Temperatura – Workflow n8n (Pre-entrega)

## 🎯 Descripción del proyecto  
Este workflow automático en n8n monitorea la previsión del clima para varias sucursales del retailer Oscar Barbieri SA, detecta picos de temperaturas (calor o frío), y envía alertas por correo para que cada sucursal revise su stock de productos de climatización o calefacción con anticipación.  

El objetivo es anticipar aumentos de demanda y evitar quiebres de stock en contextos de clima extremo.

---

## 🧰 Estructura del workflow

| Nodo | Tipo / Función |
|------|----------------|
| **Schedule Trigger** | Trigger automático — dispara el workflow cada X tiempo (periodicidad configurada). |
| **FN_InicializarSucursales** | Crea la lista de sucursales con sus datos base: código, idSucursal, coordenadas, umbrales de calor/frío, email. |
| **HTTP_OpenMeteo_14dias** | Llama a una API pública de clima (sin autenticación) para obtener pronóstico diario de temperatura según lat/lon de cada sucursal. |
| **Merge** | Une (merge) los datos de sucursales con los datos de clima recibidos — de modo que cada sucursal tenga su pronóstico asociado. |
| **Code_analizadorDepicos** | Código custom: analiza temperaturas máximas y mínimas, calcula `maxTemp`, `minTemp`, detecta si hay pico de calor/frío, y almacena las fechas estimadas de pico. |
| **If — ¿Hay pico de Temp?** | Condicional: verifica si `hayPicoCalor == true` o `hayPicoFrio == true`. Si ninguno es true, finaliza sin enviar mail. |
| **If — ¿Picos simultáneos?** *(opcional / según diseño)* | Si ambos `hayPicoCalor` y `hayPicoFrio` son true, rama especial. |
| **If — Pico de calor o frío** | Divide entre los casos “solo calor” o “solo frío” para enviar correo correspondiente. |
| **Send a message (Gmail)** | Nodo de notificación: envía mail HTML con alerta de stock según el tipo de pico detectado. |

---

## ✅ Qué evalúa el condicional — y por qué  
- El primer IF verifica si hay **algún pico de temperatura** (alto o bajo) — para no generar alertas innecesarias.  
- Si hay pico, se evalúa si hay **pico simultáneo de calor y frío**, lo que puede requerir una alerta combinada.  
- Si no es simultáneo, se verifica cuál es el tipo de pico — calor o frío — y en base a eso se elige la plantilla de mail adecuada.  

Esto asegura que solo se envíen alertas cuando realmente hay un evento climático relevante, discriminando entre los distintos escenarios.  

---

## 📧 Configuración de la notificación  
- Se utiliza el nodo *Send a message* de n8n con conexión a cuenta Gmail (vía credenciales de n8n — nunca hardcodeadas).  
- El tipo de correo es HTML, con plantilla que incluye datos dinámicos: nombre de sucursal, idSucursal, temperatura pico, fecha pico, umbral, instrucciones de stock.  
- Para pruebas, todas las sucursales pueden apuntar a un correo de testing genérico (por ejemplo `tucorreo@dominio.com`).  

---

## 🛡 Buenas prácticas aplicadas  

- El trigger es automático (schedule), sin intervención manual.  
- Se usa una API pública sin autenticación — simplifica la configuración y evita exponer tokens.  
- Los datos se manejan internamente como JSON, con nodos “Function/Code + Merge + If” para estructurar y decidir.  
- Las credenciales de Gmail se gestionan mediante el gestor de credenciales de n8n — **no se exponen en el repositorio** (cumpliendo la consigna).  
- El código es modular: cada nodo tiene una función clara.  
- La lógica condicional evita falsos positivos (alertas innecesarias).  

---

## 📂 Estructura recomendada de la entrega  

