**Advanced Retrieval-Augmented Generation (RAG) with Metadata Augmentation & Persistent Storage**

**Description**
This repository demonstrates a highly customizable RAG pipeline built on [LlamaIndex](https://github.com/jerryjliu/llama_index). It ingests unstructured documents, applies advanced metadata-based transformations to improve retrieval accuracy, and persists vector indices to avoid redundant re-embedding.

Key features:

* **Unstructured Data Ingestion** via `SimpleDirectoryReader`
* **Custom Metadata Templates** & selective key exclusion for both LLM prompts and embeddings
* **LLM-Driven Transformations**: title extraction, question-answer pair generation
* **Multiple Embedding Backends**: HuggingFace models or Chroma vector store
* **Persistent Storage**: save/load vector indices to disk (via LlamaIndex’s `StorageContext`)
* **Seamless Querying** with OpenAI LLMs (GPT-3.5/GPT-4)

---

## 🚀 Features

* **Flexible Data Extraction**

  * Load entire directories of files
  * Use filenames as document IDs if desired

* **Advanced Transformations**

  * Exclude sensitive metadata keys from embedding and LLM views
  * Define custom metadata/text templates
  * **TitleExtractor**: auto-generate document titles
  * **QuestionsAnsweredExtractor**: pinpoint key Q\&A pairs

* **Embeddings & Vector Stores**

  * **HuggingFaceEmbedding**: e.g. `BAAI/bge-small-en-v1.5`
  * **ChromaVectorStore**: persist vectors in a local Chroma database

* **Persistent Indexing**

  * Store vectors on disk to speed up future loads
  * Leverage `StorageContext.from_defaults(persist_dir="./vectors")`

* **Query Engine**

  * Swap between GPT-3.5-turbo & GPT-4 for high-quality responses
  * Easily rehydrate an index from storage and continue querying

---

## 📦 Installation

1. **Clone the repo**

   ```bash
   git clone https://github.com/your-username/advanced-rag.git
   cd advanced-rag
   ```

2. **Create a virtual environment** (recommended)

   ```bash
   python -m venv .venv
   source .venv/bin/activate    # macOS/Linux
   .\.venv\Scripts\activate     # Windows
   ```

3. **Install dependencies**

   ```bash
   pip install --upgrade pip
   pip install nest_asyncio llama-index llama-index-llms-openai openai \
               llama-index-embeddings-huggingface chromadb llama-index-vector-stores-chroma
   ```

---

## ⚙️ Configuration

* **OpenAI API Key**

  ```bash
  export OPENAI_API_KEY="your_openai_key"
  ```

  Or, if you prefer prompts:

  ```python
  import getpass, os
  os.environ["OPENAI_API_KEY"] = getpass.getpass("Enter your OpenAI API key: ")
  ```

* **Chroma DB Path** (optional)
  By default, Chroma will persist to `./chroma_db`. You can override `path` in `PersistentClient()`.

---

## 📑 Usage

1. **Load & Inspect Documents**

   ```python
   from llama_index.core import SimpleDirectoryReader
   docs = SimpleDirectoryReader(input_dir="./data").load_data()
   ```

2. **Customize Metadata Templates**

   ```python
   from llama_index.core.schema import MetadataMode

   for doc in docs:
       doc.excluded_embed_metadata_keys.append("page_label")
       doc.text_template = "Metadata:\n{metadata_str}\n---\nContent:\n{content}"
   ```

3. **Apply LLM-Driven Transformations**

   ```python
   from llama_index.llms.openai import OpenAI
   from llama_index.core.extractors import TitleExtractor, QuestionsAnsweredExtractor
   from llama_index.core.node_parser import SentenceSplitter
   from llama_index.core.ingestion import IngestionPipeline

   llm_transform = OpenAI(model="gpt-3.5-turbo", temperature=0.0)
   pipeline = IngestionPipeline(transformations=[
       SentenceSplitter(separator=" ", chunk_size=1024, chunk_overlap=128),
       TitleExtractor(llm=llm_transform, nodes=5),
       QuestionsAnsweredExtractor(llm=llm_transform, questions=3),
   ])
   nodes = pipeline.run(documents=docs, in_place=True, show_progress=True)
   ```

4. **Generate Embeddings & Build Index**

   ```python
   from llama_index.embeddings.huggingface import HuggingFaceEmbedding
   from llama_index.core import VectorStoreIndex

   hf_emb = HuggingFaceEmbedding(model_name="BAAI/bge-small-en-v1.5")
   index = VectorStoreIndex(nodes, embed_model=hf_emb)
   ```

5. **Query with OpenAI**

   ```python
   from llama_index.llms.openai import OpenAI

   llm_query = OpenAI(model="gpt-4", temperature=0.0)
   response = index.as_query_engine(llm=llm_query).query(
       "What are challenges faced in the Oil industry?"
   )
   print(response)
   ```

6. **Persist & Reload Index**

   ```python
   # Save
   index.storage_context.persist(persist_dir="./vectors")

   # Later, reload
   from llama_index.core import StorageContext, load_index_from_storage
   storage = StorageContext.from_defaults(persist_dir="./vectors")
   index = load_index_from_storage(storage, embed_model=hf_emb)
   ```

7. **(Optional) Use Chroma Vector Store**

   ```python
   import chromadb
   from llama_index.vector_stores.chroma import ChromaVectorStore
   from llama_index.core import StorageContext, VectorStoreIndex

   db = chromadb.PersistentClient(path="./chroma_db")
   coll = db.get_or_create_collection("YourCollectionName")
   store = ChromaVectorStore(chroma_collection=coll)
   storage = StorageContext.from_defaults(vector_store=store)
   index  = VectorStoreIndex(nodes, storage_context=storage, embed_model=hf_emb)
   ```

---

## 📂 Project Structure

```
advanced-rag/
├── data/                     # Your raw documents
├── chroma_db/                # Local Chroma database (after first run)
├── vectors/                  # Persisted vector index files
├── README.md                 # This file
└── rag_pipeline.py           # Example script tying everything together
```

---

## 🤝 Contributing

1. Fork the repo
2. Create a feature branch (`git checkout -b feature-name`)
3. Commit your changes (`git commit -m "Add feature"`)
4. Push to your branch (`git push origin feature-name`)
5. Open a Pull Request

---

## 📜 License

This project is released under the MIT License. Feel free to use, modify, and distribute!
