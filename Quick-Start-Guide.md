# Quick Start Guide

This guide will walk you through getting started with pgai using Docker and Ollama. We'll cover the basic features and show you how to:
- Set up your environment
- Create and manage vector embeddings
- Perform semantic search
- Implement basic RAG

## Prerequisites

- Docker and Docker Compose installed
- At least 4GB of free disk space
- Basic knowledge of SQL and PostgreSQL

## Step 1: Initial Setup

1. Download and start the Docker environment:
```bash
# Download docker-compose.yml
curl -O https://raw.githubusercontent.com/timescale/pgai/main/examples/docker_compose_pgai_ollama/docker-compose.yml

# Start the containers
docker compose up -d
```

2. Download required models:
```bash
# Download embedding model
docker compose exec ollama ollama pull all-minilm

# Download LLM for reasoning
docker compose exec ollama ollama pull tinyllama
```

## Step 2: Create a Test Database

1. Connect to the database:
```bash
docker compose exec -it db psql
```

2. Enable pgai:
```sql
CREATE EXTENSION IF NOT EXISTS ai CASCADE;
```

## Step 3: Create and Load Test Data

1. Create a sample table:
```sql
CREATE TABLE wiki (
    id      TEXT PRIMARY KEY,
    url     TEXT,
    title   TEXT,
    text    TEXT
);
```

2. Load sample data:
```sql
SELECT ai.load_dataset(
    'wikimedia/wikipedia',
    '20231101.en',
    table_name=>'wiki',
    batch_size=>5,
    max_batches=>1,
    if_table_exists=>'append'
);
```

## Step 4: Create a Vectorizer

Set up automatic vector embedding creation:
```sql
SELECT ai.create_vectorizer(
    'wiki'::regclass,
    destination => 'wiki_embeddings',
    embedding => ai.embedding_ollama('all-minilm', 384),
    chunking => ai.chunking_recursive_character_text_splitter('text')
);
```

Check vectorizer status:
```sql
SELECT * FROM ai.vectorizer_status;
```

## Step 5: Perform Semantic Search

Search for semantically similar content:
```sql
SELECT title, chunk
FROM wiki_embeddings 
ORDER BY embedding <=> ai.ollama_embed('all-minilm', 'properties of light')
LIMIT 1;
```

## Step 6: Test Data Updates

Add new data and watch automatic embedding:
```sql
INSERT INTO wiki (id, url, title, text)
VALUES (
    11,
    'https://en.wikipedia.org/wiki/Pgai',
    'pgai - Power your AI applications with PostgreSQL',
    'pgai is a tool to make developing RAG and other AI applications easier...'
);
```

## Step 7: Implement Basic RAG

Create a simple RAG function:
```sql
CREATE OR REPLACE FUNCTION generate_rag_response(query_text TEXT)
RETURNS TEXT AS $$
DECLARE
   context_chunks TEXT;
   response TEXT;
BEGIN
   -- Get relevant context
   SELECT string_agg(title || ': ' || chunk, E'\n')
   INTO context_chunks
   FROM (
       SELECT title, chunk
       FROM wiki_embeddings
       ORDER BY embedding <=> ai.ollama_embed('all-minilm', query_text)
       LIMIT 3
   ) AS relevant_chunks;

   -- Generate response
   SELECT ai.ollama_chat_complete(
       'tinyllama',
       jsonb_build_array(
           jsonb_build_object('role', 'system', 'content', 'You are a helpful assistant'),
           jsonb_build_object('role', 'user', 'content', 
               query_text || E'\nUse this context:\n' || context_chunks
           )
       )
   )->'message'->>'content'
   INTO response;

   RETURN response;
END;
$$ LANGUAGE plpgsql;
```

Use the RAG function:
```sql
SELECT generate_rag_response('What can you tell me about light?');
```

## Next Steps

- Explore different [embedding models](Model-Support)
- Learn about [advanced vectorizer options](Vectorizer-Documentation)
- Check out the [Features Overview](Features-Overview)
- Join the community and [contribute](Contributing-Guide)