# Sistema RAG sobre cáncer de mama: ingesta, vectorización y búsqueda semántica en la nube

Proyecto ACA N.° 1 – **Computación en la Nube** · CUN · Especialización en Inteligencia Artificial
**Autor:** Cristian Fernando Marín Caro · **Docente:** Hamilton Rivera Flor · **Pilar:** Salud

Pipeline de datos no estructurados en la nube: ingesta de documentos clínicos, chunking, embeddings con Google Gemini y almacenamiento vectorial en **Qdrant Cloud**, integrado con el dataset estructurado UCI Breast Cancer Wisconsin (Diagnostic).

## Arquitectura

```
Documentos clínicos (PDF/TXT)     Dataset UCI WDBC (569 casos)
            │                                  │
            ▼                                  ▼
   Extracción y limpieza             Fichas textuales con metadatos
            └───────────────┬──────────────────┘
                            ▼
      Chunking (RecursiveCharacterTextSplitter, 1000 / 150)
                            ▼
   Embeddings Gemini (768 dim., RETRIEVAL_DOCUMENT)
                            ▼
   Batch upsert ──►  Qdrant Cloud (distancia coseno)  ◄── consulta (RETRIEVAL_QUERY)
                            ▼
                Top-k fragmentos + similitud coseno
```

## Stack tecnológico

Python · Google Colab · Qdrant Cloud (Free tier) · Google Gemini Embedding (`gemini-embedding-2`) · LangChain Text Splitters · pypdf · qdrant-client · python-dotenv · pandas · matplotlib

## Corpus documental

| Archivo (nombre usado en el notebook) | Documento | Fuente |
|---|---|---|
| `01_GPC_pacientes_MinSalud_2013.pdf` | GPC cáncer de mama, versión para pacientes y cuidadores (GPC-2013-19) | [MinSalud](https://www.minsalud.gov.co/sites/rid/Lists/BibliotecaDigital/RIDE/INEC/IETS/GPC_Ptes_Mama-3.pdf) |
| `02_OMS_cancer_de_mama.pdf` | Cáncer de mama, nota descriptiva | [OMS](https://www.who.int/es/news-room/fact-sheets/detail/breast-cancer) |
| `03_Guia_clinica_cancer_mama_2020.pdf` | Guía clínica de oncología: diagnóstico y manejo del cáncer de mama | [Repositorio Unisanitas](https://repositorio.unisanitas.edu.co/bitstreams/e1281275-63d5-4940-8eab-33c43e0f35f6/download) |
| `wdbc.names`, `wdbc.data` | Documentación y datos del dataset UCI WDBC | [UCI](https://archive.ics.uci.edu/dataset/17/breast+cancer+wisconsin+diagnostic) |

Los documentos no se incluyen en este repositorio; se descargan desde los enlaces anteriores.

## Cómo reproducirlo

1. Crea un clúster gratuito en [Qdrant Cloud](https://cloud.qdrant.io) y una API key en [Google AI Studio](https://aistudio.google.com/apikey).
2. Copia `env.example` como `.env` (o `env.txt`) y completa tus tres claves. **Nunca subas ese archivo al repositorio.**
3. Abre el notebook en Google Colab y ejecútalo con *Entorno de ejecución → Ejecutar todo*.
4. Cuando lo pida, sube el archivo de claves y los documentos del corpus.

## Resultados (ejecución en Colab)

- 157 fragmentos indexados (4 documentos y 12 fichas del dataset); 0,48 MB de 1000 MB de la capa gratuita.
- Validación con 10 preguntas en lenguaje natural: Hit@1 = Hit@3 = 100 %, MRR = 1,00, similitud coseno media del primer resultado = 0,758.
- Control negativo (preguntas fuera de dominio): similitud media 0,540; separación de 0,219.
- Limitaciones: relevancia medida por palabras clave en solo 10 preguntas; fichas del dataset muy parecidas entre sí; encabezados repetidos de los PDF.

## Seguridad

Las credenciales se leen exclusivamente desde variables de entorno (`.env`); no hay claves escritas en el notebook.

## Aviso

Prototipo académico de recuperación de información. No ofrece diagnóstico médico ni reemplaza el criterio de un profesional de la salud.

## Referencias

Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goyal, N., Küttler, H., Lewis, M., Yih, W., Rocktäschel, T., Riedel, S., & Kiela, D. (2020). Retrieval-augmented generation for knowledge-intensive NLP tasks. *Advances in Neural Information Processing Systems, 33*, 9459–9474. https://arxiv.org/abs/2005.11401

Wolberg, W., Street, N., & Mangasarian, O. (1995). *UCI Breast Cancer Wisconsin (Diagnostic)* [Conjunto de datos]. UCI Machine Learning Repository. https://archive.ics.uci.edu/dataset/17/breast+cancer+wisconsin+diagnostic
