# Model Support

pgai supports a wide range of AI models through various providers. This guide details the supported models, their capabilities, and how to use them.

## Supported Providers

| Provider | Tokenize | Embed | Chat Complete | Generate | Moderate | Classify | Rerank |
|----------|----------|-------|---------------|----------|-----------|-----------|---------|
| Ollama | | ✔️ | ✔️ | ✔️ | | | |
| OpenAI | ✔️ | ✔️ | ✔️ | | ✔️ | | |
| Anthropic | | | | ✔️ | | | |
| Cohere | ✔️ | ✔️ | ✔️ | | | ✔️ | ✔️ |
| Voyage AI | | ✔️ | | | | | |
| Huggingface | | ✔️ | | | | | |
| Mistral | | ✔️ | | | | | |
| Azure OpenAI | | ✔️ | | | | | |
| AWS Bedrock | | ✔️ | | | | | |
| Vertex AI | | ✔️ | | | | | |

## Configuration

### Ollama

```sql
-- Embedding
SELECT ai.embedding_ollama('all-minilm', 384);
SELECT ai.embedding_ollama('nomic-embed-text', 768);

-- Chat completion
SELECT ai.ollama_chat_complete('tinyllama', messages);

-- Generation
SELECT ai.ollama_generate('tinyllama', prompt);
```

### OpenAI

```sql
-- Set API key
SET ai.openai_api_key = 'your-api-key';

-- Embedding
SELECT ai.embedding_openai('text-embedding-3-small');

-- Chat completion
SELECT ai.openai_chat_complete('gpt-4', messages);

-- Moderation
SELECT ai.openai_moderate(text);
```

### Anthropic

```sql
-- Set API key
SET ai.anthropic_api_key = 'your-api-key';

-- Generation
SELECT ai.anthropic_generate('claude-3-sonnet', prompt);
```

### Cohere

```sql
-- Set API key
SET ai.cohere_api_key = 'your-api-key';

-- Embedding
SELECT ai.embedding_cohere('embed-english-v3.0');

-- Chat completion
SELECT ai.cohere_chat_complete('command', messages);

-- Classification
SELECT ai.cohere_classify(text, labels);

-- Reranking
SELECT ai.cohere_rerank(query, documents);
```

## Model Selection Guide

### Embedding Models

1. **OpenAI text-embedding-3-small**
   - High quality embeddings
   - Good for general purpose use
   - Cost-effective

2. **all-minilm**
   - Open source
   - Good quality
   - Free to use
   - Runs locally

3. **nomic-embed-text**
   - Higher dimensionality (768)
   - Good for specialized tasks
   - Runs locally

### Chat/Generation Models

1. **GPT-4**
   - Highest quality responses
   - Best for complex tasks
   - More expensive

2. **Claude 3 Sonnet**
   - High quality responses
   - Good for long-form content
   - Competitive pricing

3. **tinyllama**
   - Open source
   - Runs locally
   - Good for basic tasks
   - Free to use

## Best Practices

### API Key Management

1. Use environment variables:
```bash
export OPENAI_API_KEY='your-key'
export ANTHROPIC_API_KEY='your-key'
export COHERE_API_KEY='your-key'
```

2. Set in database:
```sql
SET ai.openai_api_key = 'your-key';
```

### Error Handling

```sql
-- Wrap API calls in exception handling
DO $$
BEGIN
    -- Your API call here
EXCEPTION WHEN OTHERS THEN
    -- Handle error
END $$;
```

### Rate Limiting

- Implement retries for rate limits
- Use connection pooling
- Consider batch processing

## Performance Optimization

### Caching

```sql
-- Create cache table
CREATE TABLE embedding_cache (
    text TEXT PRIMARY KEY,
    embedding vector(384),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Use cache
WITH embedding AS (
    SELECT embedding 
    FROM embedding_cache 
    WHERE text = 'query'
    UNION ALL
    SELECT ai.ollama_embed('all-minilm', 'query')
    WHERE NOT EXISTS (
        SELECT 1 
        FROM embedding_cache 
        WHERE text = 'query'
    )
)
SELECT * FROM embedding LIMIT 1;
```

### Batch Processing

```sql
-- Process in batches
SELECT ai.batch_embed(
    'SELECT text FROM documents',
    'all-minilm',
    batch_size => 10
);
```

## Troubleshooting

Common issues and solutions:

1. **API Errors**
   - Verify API keys
   - Check rate limits
   - Validate input format

2. **Performance Issues**
   - Use appropriate batch sizes
   - Implement caching
   - Monitor resource usage

3. **Quality Issues**
   - Try different models
   - Adjust prompt engineering
   - Validate input data

## Additional Resources

- [OpenAI Documentation](https://platform.openai.com/docs)
- [Anthropic Documentation](https://docs.anthropic.com)
- [Cohere Documentation](https://docs.cohere.com)
- [Ollama Documentation](https://ollama.ai/docs)
- [VoyageAI Documentation](https://docs.voyageai.com)