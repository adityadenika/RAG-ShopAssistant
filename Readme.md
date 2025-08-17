# Shop Recommendation
Repository ini adalah shop recommendation yang digunakan untuk Bootcamp pada dibimbing.  
# Prerequisite
Install library berikut:
```bash
haystack-ai==2.15.2
mongodb-atlas-haystack==3.3.0
sentence-transformers==5.0.0
pandas==2.3.1
pymongo==4.13.2
streamlit==1.47.1
python-dotenv==1.1.1
ipykernel==6.29.5
```

# 1) common_info_store.ipynb (Seeder & Storing)

Tujuan: menulis data “Common Information” (FAQ pembelian/pengiriman/refund) ke MongoDB Atlas dengan embedding.

Inti langkah:

Load env (MONGO_CONNECTION_STRING).

Siapkan seed_data (Q/A + kategori + tags).

Buat MongoDBAtlasDocumentStore (DB: depato_store, koleksi: common_information).

Inisialisasi SentenceTransformersDocumentEmbedder("all-MiniLM-L6-v2") + warm_up().

Konversi ke Document, embed, lalu write_documents() ke MongoDB.

Catatan: di Atlas, buat Vector Search Index vector_index_common_info (field: embedding, dim: 384, cosine) dan opsional FTS index search_index_common_info.

# 2) common_info_rag.ipynb (RAG Pipeline)

Tujuan: membangun pipeline RAG untuk menjawab pertanyaan umum dari koleksi di atas.

Komponen:
SentenceTransformersTextEmbedder → MongoDBAtlasEmbeddingRetriever(top_k=6) → ChatPromptBuilder → OpenAIChatGenerator.

Inti langkah:

Load env (OPENAI_API_KEY, MONGO_CONNECTION_STRING).

Hubungkan ke MongoDBAtlasDocumentStore (koleksi common_information).

Rakit pipeline, pipeline.warm_up(), lalu fungsi run(query):

Embed query → retrieve dokumen → bangun prompt → generate jawaban singkat berbasis dokumen.

Output: string jawaban (contoh query: “Bagaimana proses refund di toko ini?”).

# 3) agent_with_common_info.ipynb (Agent + Tool + Routing)

Tujuan: menambahkan tool common_information_tool ke Agent dan routing otomatis.

Inti langkah:

Buat CommonInfoToolPipeline (mini RAG seperti di atas) + pipeline.warm_up().

Bungkus jadi Haystack Tool: common_information_tool.

Buat Agent (OpenAIChatGenerator) dengan system_prompt:

Jika pertanyaan pembelian/pengiriman/refund/retur/tracking → pakai common_information_tool.

Jika pertanyaan rekomendasi produk → arahkan ke tool rekomendasi (plug-in bila ada).

agent.warm_up(), lalu contoh panggilan user → Agent memilih tool yang tepat.

Hasil: Agent siap merutekan intent user ke tool Common Info (atau tool produk, jika disambungkan).
