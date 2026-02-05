# verse-embeddings

Generate vector embeddings for semantic search and RAG (Retrieval Augmented Generation).

## Synopsis

```bash
# Single collection mode
verse-embeddings --verses-dir PATH --output PATH [OPTIONS]

# Multi-collection mode
verse-embeddings --multi-collection --collections-file PATH [OPTIONS]
```

## Description

The `verse-embeddings` command generates vector embeddings for verses, enabling semantic search and AI-powered guidance features. It supports:
- **Single collection mode**: Process one collection at a time
- **Multi-collection mode**: Process multiple collections at once
- **OpenAI provider** (recommended): Higher quality, requires API key
- **HuggingFace provider** (free): Local models, no API key needed

## Options

### Single Collection Mode

- `--verses-dir PATH` - Path to verses directory (e.g., `_verses/`)
- `--output PATH` - Output JSON file path (e.g., `data/embeddings.json`)

### Multi-Collection Mode

- `--multi-collection` - Enable multi-collection processing
- `--collections-file PATH` - Path to collections.yml file (default: `_data/collections.yml`)
- `--output PATH` - Output JSON file path (default: `data/embeddings.json`)

### Common Options

- `--provider PROVIDER` - Embedding provider: `openai` or `huggingface` (default: `openai`)
- `--model MODEL` - Model to use (provider-specific)

## Examples

### Multi-Collection Mode (Recommended)

Process all enabled collections at once:

```bash
# Using OpenAI (recommended for quality)
verse-embeddings --multi-collection --collections-file _data/collections.yml

# Using HuggingFace (free, local)
verse-embeddings --multi-collection --collections-file _data/collections.yml --provider huggingface
```

This processes all collections with `enabled: true` in `collections.yml` and creates a unified embeddings file at `data/embeddings.json`.

### Single Collection Mode

Process one collection at a time:

```bash
# Using OpenAI (recommended)
verse-embeddings --verses-dir _verses/hanuman-chalisa --output data/embeddings.json

# Using HuggingFace (free)
verse-embeddings --verses-dir _verses/hanuman-chalisa --output data/embeddings.json --provider huggingface
```

### Provider Comparison

**OpenAI** (`text-embedding-3-small`):
- 1536 dimensions
- High quality semantic understanding
- Cost: ~$0.10 for 700 verses (one-time)

**HuggingFace** (`all-MiniLM-L6-v2`):
- 384 dimensions
- Good quality, runs locally
- No API key or cost
- First run downloads model (~80MB)

## Generated Output

Creates a JSON file with embeddings for all verses:

### Multi-Collection Output

```json
{
  "embeddings": [
    {
      "collection": "hanuman-chalisa",
      "verse_id": "verse_01",
      "text": "Combined verse text...",
      "embedding": [0.123, -0.456, ...]
    },
    {
      "collection": "sundar-kaand",
      "verse_id": "chaupai_01",
      "text": "Combined verse text...",
      "embedding": [0.123, -0.456, ...]
    },
    ...
  ],
  "metadata": {
    "total_verses": 150,
    "collections": ["hanuman-chalisa", "sundar-kaand"],
    "provider": "openai",
    "model": "text-embedding-3-small",
    "dimensions": 1536,
    "generated_at": "2024-01-15T10:30:00Z"
  }
}
```

### Single Collection Output

```json
{
  "embeddings": [
    {
      "verse_id": "verse_01",
      "text": "Combined verse text...",
      "embedding": [0.123, -0.456, ...]
    },
    ...
  ],
  "metadata": {
    "total_verses": 40,
    "provider": "openai",
    "model": "text-embedding-3-small",
    "dimensions": 1536,
    "generated_at": "2024-01-15T10:30:00Z"
  }
}
```

## Embedding Content

The command combines multiple fields from each verse:

- Sanskrit (devanagari)
- Transliteration
- Translations (English & Hindi)
- Word meanings
- Interpretive meaning
- Story/context
- Practical applications

This creates rich semantic embeddings that capture the full meaning of each verse.

## Workflow

### Multi-Collection Workflow

```bash
# 1. Ensure collections are configured
cat _data/collections.yml

# 2. Generate embeddings (one-time setup)
verse-embeddings --multi-collection --collections-file _data/collections.yml

# 3. Verify output
ls -lh data/embeddings.json
cat data/embeddings.json | jq '.metadata'

# 4. Use in your application
# The embeddings file is loaded client-side for semantic search

# 5. Regenerate after adding new verses or collections
verse-embeddings --multi-collection --collections-file _data/collections.yml
```

### Single Collection Workflow

```bash
# 1. Generate embeddings for specific collection
verse-embeddings --verses-dir _verses/hanuman-chalisa --output data/hanuman-chalisa-embeddings.json

# 2. Verify output
cat data/hanuman-chalisa-embeddings.json | jq '.metadata'
```

## Integration Example

Client-side semantic search with multi-collection support (JavaScript):

```javascript
// Load embeddings
const response = await fetch('/data/embeddings.json');
const { embeddings, metadata } = await response.json();

console.log(`Loaded ${metadata.total_verses} verses from ${metadata.collections?.length || 1} collection(s)`);

// Search for relevant verses (works with both single and multi-collection)
function findRelevantVerses(query, topK = 3, filterCollection = null) {
  const queryEmbedding = await generateEmbedding(query);

  let filtered = embeddings;
  if (filterCollection) {
    filtered = embeddings.filter(v => v.collection === filterCollection);
  }

  const scores = filtered.map(verse => ({
    ...verse,
    score: cosineSimilarity(queryEmbedding, verse.embedding)
  }));

  return scores
    .sort((a, b) => b.score - a.score)
    .slice(0, topK);
}

// Search across all collections
const results = await findRelevantVerses("devotion and strength");

// Search within specific collection
const hanumanResults = await findRelevantVerses("devotion", 5, "hanuman-chalisa");
```

## Cost & Performance

### OpenAI

- **Model**: text-embedding-3-small
- **Cost**: $0.00002 per 1,000 tokens
- **For Bhagavad Gita** (700 verses, ~500 tokens each):
  - Total tokens: ~350,000
  - Cost: ~$0.10 (one-time)
- **Generation time**: ~2-3 minutes

### HuggingFace

- **Model**: all-MiniLM-L6-v2
- **Cost**: Free
- **Download size**: ~80MB (first run only)
- **Generation time**: ~10-15 minutes (depends on CPU)
- **Memory**: ~500MB RAM

## Provider Comparison

| Feature | OpenAI | HuggingFace |
|---------|--------|-------------|
| Quality | Excellent | Good |
| Dimensions | 1536 | 384 |
| Cost | ~$0.10 | Free |
| Speed | Fast | Moderate |
| Setup | API key | Model download |
| Best for | Production | Development/testing |

## Requirements

### OpenAI Provider

- `OPENAI_API_KEY` environment variable
- Internet connection

### HuggingFace Provider

- Python packages: `torch`, `sentence-transformers`
- ~80MB disk space for model
- No API key needed

## Use Cases

1. **Semantic Search**: Find verses by meaning, not just keywords
2. **RAG Systems**: Provide context for AI spiritual guidance
3. **Similarity**: Find verses with similar themes
4. **Recommendations**: Suggest related verses to readers

## Notes

- Embeddings are generated once and reused (no need to regenerate on every query)
- Regenerate embeddings when adding new verses or collections
- The output JSON file can be loaded client-side (semantic search runs in browser)
- Multi-collection mode processes all enabled collections in `collections.yml`
- OpenAI embeddings are recommended for production (better quality)
- HuggingFace is great for development and testing (free, no API needed)
- Multi-collection output includes `collection` field for filtering

## See Also

- [Multi-Collection Guide](../multi-collection.md) - Detailed guide on working with multiple collections
- [OpenAI Embeddings Documentation](https://platform.openai.com/docs/guides/embeddings)
- [sentence-transformers Documentation](https://www.sbert.net/)
- [Troubleshooting](../troubleshooting.md) - Common issues

## Troubleshooting

### "Error: OPENAI_API_KEY not set"

For OpenAI provider:

```bash
export OPENAI_API_KEY=sk-...
```

Or use HuggingFace instead:

```bash
verse-embeddings --verses-dir _verses --output data/embeddings.json --provider huggingface
```

### "Error: No verse files found"

Check:
- Verse directory path is correct
- Directory contains `.md` files
- Files have YAML frontmatter

### "Out of memory" (HuggingFace)

HuggingFace model needs ~500MB RAM. If hitting limits:
- Close other applications
- Use OpenAI provider instead (no local processing)

## See Also

- [OpenAI Embeddings Documentation](https://platform.openai.com/docs/guides/embeddings)
- [sentence-transformers Documentation](https://www.sbert.net/)
- [Troubleshooting](../troubleshooting.md) - Common issues
