# 🤖 Resumen del Notebook "Día 3 - IA conversacional - Chatbot" 🤖

Este notebook desarrolla un chatbot sencillo utilizando Gradio como interfaz y OpenAI para la generación de respuestas. A continuación se resumen los elementos clave:

## 🛠️ Configuración Inicial

- Importa las bibliotecas necesarias: `os`, `dotenv`, `OpenAI` y `gradio`
- Carga la API key desde un archivo `.env`
- Configura el cliente OpenAI para usar OpenRouter como intermediario
- Selecciona el modelo `gpt-4o-mini` para responder a las consultas

```python
# imports
import os
from dotenv import load_dotenv
from openai import OpenAI
import gradio as gr

# Cargar variables de entorno
load_dotenv()
api_key = os.getenv('OPENAI_API_KEY')

# Inicializar
openai = OpenAI(
  base_url="https://openrouter.ai/api/v1",
  api_key=api_key
)
MODEL = "openai/gpt-4o-mini"
```

## 💬 Funcionalidad del Chatbot

- Define un mensaje de sistema básico: "Eres un assistente muy útil"
- Implementa la función `chat()` que:
  - Maneja el historial de conversación
  - Procesa los mensajes nuevos
  - Envía todo al API de OpenAI
  - Recibe las respuestas en streaming para mostrarlas gradualmente

```python
def chat(message, history):
    messages = [{"role": "system", "content": system_message}] + history + [{"role": "user", "content": message}]

    stream = openai.chat.completions.create(model=MODEL, messages=messages, stream=True)

    response = ""
    for chunk in stream:
        response += chunk.choices[0].delta.content or ''
        yield response
```

## 🖥️ Interfaz con Gradio

- Crea una interfaz de chat simple y elegante con `gr.ChatInterface()`
- La interfaz muestra la conversación y permite enviar mensajes de forma interactiva

```python
# ¡La magia de Gradio en una sola línea!
gr.ChatInterface(fn=chat, type="messages").launch()
```

## 🎭 Personalización del Comportamiento

El notebook muestra cómo personalizar el comportamiento del chatbot:

1. Primero con un mensaje de sistema básico:
```python
system_message = "Eres un assistente muy útil"
```

2. Luego evoluciona a un escenario de tienda de ropa 👕👒:
```python
system_message = """Eres un asistente útil en una tienda de ropa. 
Debes tratar de alentar gentilmente al cliente a que pruebe los artículos que están en oferta.
Los sombreros tienen un 60 % de descuento y la mayoría de los demás artículos tienen un 50 % de descuento."""
```

3. Añadiendo comportamiento específico para productos:
```python
system_message += """
Si el cliente pide zapatos, debes responder que los zapatos no están en oferta hoy, 
¡pero recuérdale al cliente que mire los sombreros!"""
```

## 🔄 Demostración de Flujo de Trabajo

El notebook ilustra cómo modificar incrementalmente el comportamiento del chatbot:
- Cambiando el mensaje de sistema
- Añadiendo instrucciones específicas para casos particulares
- Modificando la función `chat()` para añadir contexto condicional:

```python
def chat(message, history):
    messages = [{"role": "system", "content": system_message}] + history + [{"role": "user", "content": message}]

    # Añadir contexto adicional basado en palabras clave
    if 'cinturon' in message:
        messages.append({"role": "system", "content": "Para mayor contexto, la tienda no vende cinturones,\
        pero asegúrate de señalar otros artículos en oferta."})
    
    # ... resto del código
```

## ✨ Conclusión

Esta implementación demuestra lo sencillo que es crear un chatbot funcional con Gradio y cómo personalizar su comportamiento mediante instrucciones en el mensaje de sistema, todo ello con apenas unas docenas de líneas de código. ¡Un excelente punto de partida para construir aplicaciones de IA conversacional! 🚀