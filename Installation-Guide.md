# Installation Guide

There are several ways to install and use pgai. Choose the method that best suits your needs:

## 1. Docker Installation (Recommended)

The easiest way to get started with pgai is using Docker:

1. Download the docker compose file:
```bash
curl -O https://raw.githubusercontent.com/timescale/pgai/main/examples/docker_compose_pgai_ollama/docker-compose.yml
```

2. Start the containers:
```bash
docker compose up -d
```

This will start:
- A PostgreSQL instance with pgai installed
- An Ollama instance for model serving

## 2. Timescale Cloud

For a managed solution:

1. Create a free trial account on [Timescale Cloud](https://www.timescale.com/cloud)
2. Follow the cloud setup instructions
3. pgai will be pre-installed and ready to use

## 3. Installing on Existing PostgreSQL (Linux/MacOS)

To install pgai on an existing PostgreSQL instance:

1. Install prerequisites:
```bash
# Required development packages
sudo apt-get install -y \
    build-essential \
    postgresql-server-dev-15 \
    git
```

2. Clone and build pgai:
```bash
git clone https://github.com/timescale/pgai.git
cd pgai
make
sudo make install
```

3. Enable the extension in your database:
```sql
CREATE EXTENSION IF NOT EXISTS ai CASCADE;
```

## Post-Installation Setup

After installation, you'll need to:

1. Download required models (if using Ollama):
```bash
docker compose exec ollama ollama pull all-minilm
docker compose exec ollama ollama pull tinyllama
```

2. Configure environment variables (if using cloud services):
```bash
# Example for OpenAI
export OPENAI_API_KEY='your-api-key'
```

3. Test the installation:
```sql
-- Connect to your database
CREATE EXTENSION IF NOT EXISTS ai CASCADE;
SELECT ai.ollama_embed('all-minilm', 'test');
```

## Troubleshooting

Common issues and solutions:

1. **Connection Issues**
   - Verify PostgreSQL is running
   - Check port configurations
   - Ensure proper credentials

2. **Model Download Problems**
   - Verify internet connection
   - Check disk space
   - Ensure proper permissions

3. **Extension Loading Errors**
   - Verify PostgreSQL version compatibility
   - Check system requirements
   - Review installation logs

For more detailed troubleshooting, consult the [pgai GitHub issues](https://github.com/timescale/pgai/issues).