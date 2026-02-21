# LiteLLM Proxy Setup

## Requirements

- Python 3.11
- A Supabase (or other Postgres) database
- An OpenAI API key

## Environment Variables

Create a `.env` file in your project root:

```
DATABASE_URL=postgresql://postgres:yourpassword@db.xxxx.supabase.co:5432/postgres
LITELLM_MASTER_KEY=sk-yourkey
OPENAI_API_KEY=sk-your-openai-key
```

> ⚠️ Add `.env` to your `.gitignore` — never commit secrets!

## Config

Create a `config.yaml` file. You can either hardcode values or reference env vars:

```yaml
model_list:
  - model_name: gpt-4
    litellm_params:
      model: openai/gpt-4

general_settings:
  master_key: sk-yourkey
  database_url: "postgresql://postgres:yourpassword@db.xxxx.supabase.co:5432/postgres"
```

## Installation

```bash
pip install 'litellm[proxy]'
pip install 'litellm[extra_proxy]'
pip install prisma
```

## Generate Prisma Client

LiteLLM uses Prisma to connect to the database. You need to run this once:

```bash
prisma generate --schema=$(python3 -c "import litellm; import os; print(os.path.dirname(litellm.__file__))")/proxy/schema.prisma
```

Or find the schema path manually:

```bash
find ~/.pyenv -name "schema.prisma" 2>/dev/null
# Then run:
prisma generate --schema=/path/to/schema.prisma
```

## Start the Proxy

```bash
source .env && litellm --config ./config.yaml
```

The proxy runs at: **http://localhost:4000**  
Admin UI: **http://localhost:4000/ui**

## Generate a Virtual Key

Virtual keys let you give others access to your proxy without exposing your master key or OpenAI key. Run this in a new terminal tab:

```bash
curl -X POST http://0.0.0.0:4000/key/generate \
  -H 'Authorization: Bearer sk-YOUR-MASTER-KEY' \
  -H 'Content-Type: application/json' \
  -d '{"models": ["gpt-4"], "metadata": {"user": "test"}}'
```

You'll get back a new `sk-...` virtual key. Save it!

## Make a Chat Completion

Using your master key or a virtual key:

```bash
curl -X POST http://0.0.0.0:4000/chat/completions \
  -H 'Authorization: Bearer sk-YOUR-KEY' \
  -H 'Content-Type: application/json' \
  -d '{"model": "gpt-4", "messages": [{"role": "user", "content": "hello"}]}'
```

## Use in Python

LiteLLM is fully OpenAI-compatible, so you can use it as a drop-in replacement:

```python
import openai

client = openai.OpenAI(
    api_key="sk-your-virtual-key",
    base_url="http://0.0.0.0:4000"
)

response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "hello"}]
)
```

## Key Concepts

| Thing | What it is |
|---|---|
| Master key | Your admin key — you set this, keep it secret |
| Virtual key | Generated keys you give to apps/users |
| DATABASE_URL | Your Supabase Postgres connection string |
| OPENAI_API_KEY | Your real OpenAI key — stays inside LiteLLM |