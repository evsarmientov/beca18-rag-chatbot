# Beca 18 RAG Chatbot

Chatbot de recuperación aumentada por generación (RAG) que responde preguntas sobre el reglamento oficial de **Beca 18** — Resolución Directoral Ejecutiva N.° 033-2026-MINEDU/VMGI-PRONABEC — sin depender del conocimiento paramétrico del modelo.

## Pipeline

El sistema extrae el texto del PDF página por página (insertando marcadores `[PAGE N]`), lo divide en fragmentos de 400 tokens con 60 de solapamiento usando `RecursiveCharacterTextSplitter`, genera embeddings con `gemini-embedding-001` (768 dimensiones) diferenciando entre documentos y consultas mediante `task_type`, y los almacena en una colección ChromaDB persistente con distancia coseno. Ante una pregunta del usuario, el pipeline embebe la consulta, recupera los `k` fragmentos más relevantes y los pasa como contexto al LLM `gemini-2.5-flash`, que está instruido a responder únicamente desde ese contexto y a citar la página fuente.

## Instalación

**Requisitos:** Python 3.10+

```bash
pip install -r requirements.txt
```

## Configuración de la API Key

1. Obtén una clave gratuita en [Google AI Studio](https://aistudio.google.com/app/apikey).
2. Copia `.env.example` a `.env`:
   ```bash
   cp .env.example .env
   ```
3. Edita `.env` y coloca tu clave:
   ```
   GEMINI_API_KEY=tu_clave_aqui
   ```

El archivo `.env` está en `.gitignore` y **nunca** debe commitearse.

## Cómo ejecutar el notebook

Abre `notebooks/beca18_rag_chatbot.ipynb` en Jupyter Lab, Jupyter Notebook o Google Colab y ejecuta las celdas en orden:

| Step | Descripción |
|------|-------------|
| 0    | Setup: carga de paquetes y API key |
| 1    | Extracción de texto del PDF con marcadores de página |
| 2    | Conteo de tokens y construcción de chunks |
| 3    | Funciones de embedding con manejo de rate-limit |
| 4    | Indexado idempotente en ChromaDB |
| 5    | Búsqueda semántica (`semantic_search`) |
| 6    | Generación fundamentada (`answer_with_context`) + pruebas |
| 7    | Interfaz de chat interactiva |

En Google Colab, descomenta la celda de instalación (`!pip install ...`) al inicio del Step 0.

## Uso de la interfaz de chat

Al ejecutar la última celda (Step 7) aparece la interfaz:

- **Caja de texto**: escribe tu pregunta sobre el reglamento de Beca 18.
- **Botón "Preguntar"**: recupera los chunks y genera la respuesta.
- **Botón "Limpiar"**: reinicia la interfaz.
- **Slider k**: controla cuántos fragmentos del documento se recuperan (1–10).
- **Acordeón "Fragmentos fuente"**: expándelo para ver los fragmentos exactos, página y distancia coseno.

El modelo responde **"El documento no contiene información sobre este tema."** cuando la pregunta está fuera del alcance del reglamento.

## Estructura del repositorio

```
beca18-rag-chatbot/
├── data/
│   └── beca18_reglamento.pdf
├── notebooks/
│   └── beca18_rag_chatbot.ipynb
├── .env.example
├── .gitignore
├── requirements.txt
└── README.md
```
