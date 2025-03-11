# 🖥️ Creación de Interfaces con Gradio

## 📋 Descripción
Este proyecto demuestra la creación de interfaces de usuario interactivas utilizando Gradio, un framework sencillo pero potente para Python. El trabajo muestra una progresión desde conceptos básicos hasta implementaciones avanzadas que integran modelos de IA como GPT y Claude.

## 🔑 Tecnologías utilizadas
- **Gradio**: Framework para crear interfaces de usuario
- **OpenAI API**: Para interactuar con modelos GPT
- **Anthropic API**: Para interactuar con modelos Claude
- **BeautifulSoup**: Para extracción de datos web
- **OpenRouter**: Para gestionar diferentes modelos de IA con una única API

## 🎯 Objetivos
- Aprender a crear interfaces de usuario interactivas con Gradio
- Integrar modelos de lenguaje grandes (LLMs) en aplicaciones web
- Demostrar casos de uso prácticos combinando web scraping y generación de texto por IA

## 💻 Componentes principales

### 1. Configuración del entorno
```python
import os
import requests
from bs4 import BeautifulSoup
from typing import List
from dotenv import load_dotenv
from openai import OpenAI
import gradio as gr

load_dotenv()
api_key = os.getenv('OPENAI_API_KEY')  # OpenRouter maneja todos los modelos
```

### 2. Interfaces básicas
La primera interfaz muestra cómo conectar una función simple con una entrada y salida de texto:

```python
def shout(text):
    return text.upper()

gr.Interface(fn=shout, inputs="textbox", outputs="textbox").launch()
```

### 3. Integración con modelos de IA
Implementación de funciones para comunicarse con modelos de lenguaje:

```python
def message_gpt(prompt):
    messages = [
        {"role": "system", "content": system_message},
        {"role": "user", "content": prompt}
    ]
    completion = openai.chat.completions.create(
        model="openai/gpt-4o-mini",
        messages=messages,
    )
    return completion.choices[0].message.content
```

### 4. Streaming de respuestas
Mejora de la experiencia de usuario mediante streaming de respuestas:

```python
def stream_gpt(prompt):
    # Configuración de la llamada a la API
    stream = openai.chat.completions.create(
        model='gpt-4o-mini',
        messages=[
            {"role": "system", "content": system_message},
            {"role": "user", "content": prompt}
        ],
        stream=True
    )
    result = ""
    for chunk in stream:
        result += chunk.choices[0].delta.content or ""
        yield result
```

### 5. Selector de modelos
Interfaz con selector para elegir entre diferentes modelos de IA:

```python
def stream_model(prompt, model):
    if model=="GPT":
        result = stream_gpt(prompt)
    elif model=="Claude":
        result = stream_claude(prompt)
    else:
        raise ValueError("Modelo Desconocido")
    yield from result

view = gr.Interface(
    fn=stream_model,
    inputs=[
        gr.Textbox(label="Tu mensaje:"), 
        gr.Dropdown(["GPT", "Claude"], label="Selecciona un modelo:", value="GPT")
    ],
    outputs=[gr.Markdown(label="Respuesta:")],
    flagging_mode="never"
)
```

## 🚀 Aplicación práctica: Generador de folletos empresariales

La aplicación final combina web scraping con generación de texto por IA para crear folletos empresariales a partir del contenido de un sitio web:

```python
class Website:
    def __init__(self, url):
        self.url = url
        response = requests.get(url)
        self.body = response.content
        soup = BeautifulSoup(self.body, 'html.parser')
        self.title = soup.title.string if soup.title else "No se ha encontrado título de la página"
        for irrelevant in soup.body(["script", "style", "img", "input"]):
            irrelevant.decompose()
        self.text = soup.body.get_text(separator="\n", strip=True)

    def get_contents(self):
        return f"Título de la Web:\n{self.title}\nContenido de la Web:\n{self.text}\n\n"

def stream_brochure(company_name, url, model):
    prompt = f"Genera un folleto de la empresa {company_name}. Esta es su página de destino:\n"    
    prompt += Website(url).get_contents()
    if model=="GPT":
        result = stream_gpt(prompt)
    elif model=="Claude":
        result = stream_claude(prompt)
    else:
        raise ValueError("Modelo Desconocido")
    yield from result
```

## 📈 Conclusiones

Este proyecto demuestra cómo Gradio facilita la creación de interfaces de usuario para aplicaciones de IA, permitiendo:

- Rápido prototipado de aplicaciones web con mínimo código
- Fácil integración con modelos de lenguaje grandes
- Creación de aplicaciones prácticas que combinan diferentes tecnologías
- Exposición pública de interfaces para demostración o uso compartido

Esta implementación sirve como base para la creación de aplicaciones más complejas que requieran interacción con modelos de IA a través de interfaces amigables.