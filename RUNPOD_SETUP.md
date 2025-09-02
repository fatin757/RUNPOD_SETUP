# Evaluation Pipeline Setup

This guide walks you through setting up and running the evaluation pipeline on RunPod. The setup involves configuring RunPod with SSH access, setting up the Python environment, installing Neo4j, and running the evaluation notebooks.

## Prerequisites

- RunPod account
- SSH key pair
- API keys for OpenAI, Cohere, and OpenRouter

## 1. RunPod Setup with SSH

### Generate SSH Key Pair
```bash
ssh-keygen -t ed25519 -C "dnth@runpod"
```

### Add SSH Key to RunPod
1. Copy your public key:
   ```bash
   cat /home/dnth/.ssh/id_ed25519.pub
   ```
2. Go to RunPod settings and paste the public key content
3. Create a new GPU machine instance
4. Copy the SSH over exposed TCP command from RunPod

### Connect via VS Code
1. Install the Remote SSH extension in VS Code
2. Add a new SSH connection using the command from RunPod
3. Connect to the machine

## 2. Environment Setup

### Install uv Package Manager
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Clone and Setup Project
```bash
# Navigate to workspace (or /root if preferred)
cd /workspace

# Clone the repository
git clone https://github.com/insourcedata/cxs-eval
cd cxs-eval

# Install dependencies
uv sync
```

## 3. Neo4j Database Setup

### Install Neo4j
```bash
# Add Neo4j GPG key and repository
curl -fsSL https://debian.neo4j.com/neotechnology.gpg.key | gpg --dearmor -o /etc/apt/trusted.gpg.d/neo4j.gpg
echo 'deb https://debian.neo4j.com stable latest' | tee /etc/apt/sources.list.d/neo4j.list

# Update package list and install Neo4j
apt update
apt install neo4j
```

### Import Database Dump
```bash
# Stop Neo4j service
neo4j stop
```

Download the database dump from https://github.com/insourcedata/cxs-eval/releases/tag/Database and place it in the current directory. 

```
# Import the database dump into neo4j
neo4j-admin database load --from-path=./ --overwrite-destination=true neo4j

# Start Neo4j service
neo4j start
```

### Configure Neo4j
1. Navigate to `http://localhost:7474` in your browser
2. Log in with default credentials (username: `neo4j`, password: `neo4j`)
3. Set a new password when prompted

**Useful Neo4j Commands:**
- `neo4j start` - Start the service
- `neo4j stop` - Stop the service  
- `neo4j status` - Check service status

## 4. Environment Configuration

Create a `.env` file in the project root with the following configuration:

```env
# Neo4j Configuration
NEO4J_URI=bolt://127.0.0.1:7687
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=your_neo4j_password_that_you_set_from_previous_step

# API Keys
OPENAI_API_KEY=your_openai_api_key_here
OPENROUTER_API_KEY=your_openrouter_api_key_here
COHERE_API_KEY=your_cohere_api_key_here
```

Replace the placeholder values with your actual Neo4j password and API keys.

## 5. Running the Evaluation Pipeline

With everything set up, you can now run the evaluation using just one line of code with `uv`.

The script supports three types of embedding models using a prefix-based system:

- **`sentence-transformers` models**: Prefix with `st:` (e.g., `st:all-MiniLM-L12-v2`)
- **Cohere models**: Prefix with `cohere:` (e.g., `cohere:embed-v4.0`)
- **OpenAI models**: Prefix with `openai:` (e.g., `openai:text-embedding-3-small`)

Here are some example commands

```bash
# Using sentence-transformers
uv run scripts/benchmark.py --embedding-model "st:sentence-transformers/all-MiniLM-L12-v2"

# Using Cohere
uv run scripts/benchmark.py --embedding-model "cohere:embed-v4.0"

# Using OpenAI
uv run scripts/benchmark.py --embedding-model "openai:text-embedding-3-small"
```

Other supported arguments:
- `--embedder-top-k` (default: 35) - Number of top results to retrieve
- `--reranker-top-k` (default: 10) - Number of top results after reranking
- `--limit` (default: None) - Limit the evaluation to a certain number of queries (useful for testing)
- `--no-embed` - Skip embedding the data (default: False - data will be embedded)
- `--experimental` - Use experimental hybrid search parameters (ranker='linear', alpha=0.7, effective_search_ratio=2) instead of defaults (ranker='naive', alpha=None, effective_search_ratio=1)
- `--use-query-expansion` - Use LLM-based query expansion to enhance retrieval
- `--debug` - Show all log levels (DEBUG, INFO, WARNING, ERROR, CRITICAL) instead of just INFO and above