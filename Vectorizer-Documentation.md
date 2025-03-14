# Vectorizer Documentation

The pgai Vectorizer is a powerful tool that automatically creates and maintains vector embeddings for your data. This guide covers its features, configuration options, and best practices.

## Overview

The Vectorizer:
- Automatically creates embeddings for new data
- Updates embeddings when data changes
- Manages chunking and formatting
- Supports multiple embedding models
- Provides status monitoring

## Basic Usage

### Creating a Vectorizer

Basic syntax:
```sql
SELECT ai.create_vectorizer(
    source_table::regclass,
    destination => destination_table,
    embedding => embedding_config,
    chunking => chunking_config,
    [formatting => formatting_config]
);
```

Example:
```sql
SELECT ai.create_vectorizer(
    'documents'::regclass,
    destination => 'document_embeddings',
    embedding => ai.embedding_ollama('all-minilm', 384),
    chunking => ai.chunking_recursive_character_text_splitter('content', 512, 50)
);
```

## Configuration Options

### 1. Embedding Models

Available models:
```sql
-- Ollama
ai.embedding_ollama('all-minilm', 384)
ai.embedding_ollama('nomic-embed-text', 768)

-- OpenAI
ai.embedding_openai('text-embedding-3-small')

-- Cohere
ai.embedding_cohere('embed-english-v3.0')

-- VoyageAI
ai.embedding_voyage('voyage-2')
```

### 2. Chunking Methods

Text splitting options:
```sql
-- Basic character splitter
ai.chunking_character_text_splitter(
    column_name,
    chunk_size => 512,
    chunk_overlap => 50
)

-- Recursive character splitter
ai.chunking_recursive_character_text_splitter(
    column_name,
    chunk_size => 512,
    chunk_overlap => 50
)

-- Custom chunking
ai.chunking_custom(
    chunk_query => 'SELECT id, content FROM my_chunks'
)
```

### 3. Formatting Options

Text formatting:
```sql
-- Python template
ai.formatting_python_template('$chunk')

-- Custom template
ai.formatting_python_template('Title: $title\nContent: $chunk')
```

## Monitoring and Management

### Check Vectorizer Status

```sql
-- View all vectorizers
SELECT * FROM ai.vectorizers;

-- Check processing status
SELECT * FROM ai.vectorizer_status;

-- View processing logs
SELECT * FROM ai.vectorizer_logs;
```

### Managing Vectorizers

```sql
-- Pause a vectorizer
SELECT ai.pause_vectorizer('my_vectorizer');

-- Resume a vectorizer
SELECT ai.resume_vectorizer('my_vectorizer');

-- Delete a vectorizer
SELECT ai.drop_vectorizer('my_vectorizer');
```

## Best Practices

1. **Chunking Strategy**
   - Use smaller chunks for precise retrieval
   - Use larger chunks for context preservation
   - Consider overlap for coherence

2. **Performance Optimization**
   - Index your embedding columns
   - Monitor vectorizer status
   - Use batch processing for large datasets

3. **Error Handling**
   - Monitor vectorizer logs
   - Set up error notifications
   - Handle failed items appropriately

## Example Workflows

### 1. Document Search System

```sql
-- Create document table
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    title TEXT,
    content TEXT
);

-- Set up vectorizer
SELECT ai.create_vectorizer(
    'documents'::regclass,
    destination => 'document_embeddings',
    embedding => ai.embedding_ollama('all-minilm', 384),
    chunking => ai.chunking_recursive_character_text_splitter('content'),
    formatting => ai.formatting_python_template('Title: $title\n$chunk')
);

-- Perform semantic search
SELECT title, chunk
FROM document_embeddings
ORDER BY embedding <=> ai.ollama_embed('all-minilm', 'search query')
LIMIT 5;
```

### 2. Multi-Column Vectorization

```sql
-- Create product table
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    description TEXT,
    specifications TEXT
);

-- Set up vectorizer with multiple columns
SELECT ai.create_vectorizer(
    'products'::regclass,
    destination => 'product_embeddings',
    embedding => ai.embedding_ollama('all-minilm', 384),
    chunking => ai.chunking_recursive_character_text_splitter(
        'description || E''\n'' || specifications'
    ),
    formatting => ai.formatting_python_template(
        'Product: $name\nDetails: $chunk'
    )
);
```

## Troubleshooting

Common issues and solutions:

1. **Slow Processing**
   - Check system resources
   - Optimize chunk size
   - Review batch settings

2. **Failed Items**
   - Check error logs
   - Verify API keys
   - Review formatting templates

3. **Memory Issues**
   - Adjust batch size
   - Monitor system resources
   - Consider chunking strategy

## Additional Resources

- [Model Support Documentation](Model-Support)
- [Performance Tuning Guide](Performance-Tuning)
- [Error Handling Guide](Error-Handling)