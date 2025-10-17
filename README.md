🧠 Proyecto Chatbot IA con LLaMA + FastAPI + Chroma + Streamlit + Docker
📋 Descripción general

Este proyecto implementa un chatbot con inteligencia artificial basado en modelos LLaMA ejecutados localmente, con capacidades de búsqueda contextual mediante una base de datos vectorial (Chroma) y una API desarrollada con FastAPI.
Además, incluye una interfaz visual en Streamlit para probar el sistema fácilmente.

La arquitectura sigue el patrón RAG (Retrieval-Augmented Generation):

Se ingestan documentos o textos.

Se convierten en embeddings vectoriales.

Se almacenan en la base vectorial.

El modelo LLaMA genera respuestas contextualizadas.

🧰 Tecnologías utilizadas
Componente	Tecnología	Descripción
🧠 Modelo IA	LLaMA / llama-cpp-python
	Modelo de lenguaje ejecutado localmente.
🔍 Base Vectorial	ChromaDB
	Almacenamiento y búsqueda semántica de embeddings.
🧩 Embeddings	Sentence-Transformers
	Convierte texto a vectores numéricos.
⚙️ Backend	FastAPI
	API moderna y rápida en Python.
💻 Interfaz	Streamlit
	Panel web para visualizar y probar el chatbot.
🐳 Contenedores	Docker
	Aislamiento y despliegue consistente del proyecto.

⚙️ Variables de entorno (.env)

Crea un archivo .env en la raíz con:

MODEL_PATH=/app/models/your-model.bin
CHROMA_DB_PATH=/app/chroma_db
FASTAPI_PORT=8000
STREAMLIT_PORT=8501

🐳 Configuración de Docker
Dockerfile.api
# Imagen para el backend (FastAPI)
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY ./src ./src
COPY ./models ./models
COPY .env .

EXPOSE 8000

CMD ["uvicorn", "src.app.main:app", "--host", "0.0.0.0", "--port", "8000"]

Dockerfile.ui
# Imagen para la interfaz Streamlit
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY ./frontend ./frontend
COPY .env .

EXPOSE 8501

CMD ["streamlit", "run", "frontend/streamlit_app.py", "--server.port=8501", "--server.address=0.0.0.0"]

docker-compose.yml
version: "3.9"

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile.api
    container_name: chatbot_api
    env_file: .env
    volumes:
      - ./models:/app/models
      - ./src:/app/src
      - ./chroma_db:/app/chroma_db
    ports:
      - "${FASTAPI_PORT}:8000"
    restart: always

  ui:
    build:
      context: .
      dockerfile: Dockerfile.ui
    container_name: chatbot_ui
    env_file: .env
    depends_on:
      - api
    ports:
      - "${STREAMLIT_PORT}:8501"
    restart: always

🚀 Ejecución con Docker
1️⃣ Construir e iniciar los contenedores
docker compose up --build


Esto levantará:

Servicio	URL	Descripción
🧩 API FastAPI	http://localhost:8000/docs
	Endpoints para ingestión y consulta.
💻 Interfaz Streamlit	http://localhost:8501
	Interfaz web del chatbot.
2️⃣ Apagar los contenedores
docker compose down

3️⃣ Ver logs en tiempo real
docker compose logs -f

📦 Instalación sin Docker (modo manual)

Si prefieres probar el proyecto sin contenedores:

python -m venv .venv
source .venv/bin/activate  # o .venv\Scripts\activate en Windows
pip install -r requirements.txt
uvicorn src.app.main:app --reload


Luego, en otra terminal:

streamlit run frontend/streamlit_app.py

🧩 Endpoints principales
POST /ingest

Ingesta textos o documentos en la base vectorial.

{
  "texts": ["SAP ABAP es un lenguaje de programación empresarial."]
}

POST /query

Consulta al chatbot con recuperación contextual.

{
  "query": "¿Qué es SAP ABAP?",
  "top_k": 3
}

💡 Recomendaciones

Usa modelos quantizados (Q4/Q5) para menor consumo de RAM.

En producción, considera usar Qdrant o Milvus como vector DB externa.

Si vas a correr en servidor con GPU, prueba vLLM como alternativa más eficiente al ejecutar modelos grandes.

Autor

Proyecto creado por Juan Pablo Velásquez
