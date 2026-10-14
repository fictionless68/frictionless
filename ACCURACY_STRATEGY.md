# Frictionless Second Brain: Data Accuracy Strategy

## Core Principles

### 1. Source-of-Truth Hierarchy
- **Primary**: Archive extraction (Archive_extracted/) — frozen snapshot at point of export
- **Secondary**: ChromaDB vector store (chroma/) — semantic search index
- **Ternary**: Mirror directory (mirror/) — claude.ai artifact sync state
- **Quaternary**: Live data (frictionless/) — runtime working copies

### 2. Update Cadence
- **Mirror sync**: Runs when Chris exports new Claude.ai data
- **Chroma re-ingest**: Triggered by `ingest_chromadb.py` or `mirror-ledger-init`
- **Daily digest**: `frictionless.py mirror-plan` + `mirror-mark-synced`
- **Weekly audit**: Compare mirror vs. ledger for discrepancies

### 3. Data Change Detection
- **Ledger tracking**: `.ledger.json` records every write with version numbers
- **Batch diff**: Compare new ArtifactData batches against `.ledger.json`
- **Fail-closed**: If earlier batches unsynced, mirror refuses writes (unless `--replace`)
- **Leak scan**: `LEAK_BULLETIN_RE` + `NEVER_TERMS` redact sensitive/proprietary content

### 4. Verification Gates
- **Schema validation**: `SCHEMA.md` defines entity structure (frontmatter + content)
- **Decision log**: `decisions.json` tracks who approved what release/version
- **People safety**: `people[].sensitive` never appears in mirror/output
- **Bulletin redaction**: Mastercard AP/MEA 13552.1 details stripped to `BULLETIN_NOTE`

### 5. RAG-Powered Accuracy
- Semantic search retrieves most relevant entities before LLM response
- `fact_check_hook.py` validates responses against verified sources
- `research_guardrail.py` requires 3+ sources from 2+ domains
- `skillspector_gate.py` scans any skill/plugin install for security risks

### 6. Change Management Workflow
```bash
# 1. Pull new Claude.ai export
# 2. Extract Archive.zip -> Archive_extracted
# 3. Run: python3 scripts/ingest_chromadb.py  # rebuild vector index
# 4. Run: python3 scripts/mirror-ledger-init --listing <file>  # init ledger
# 5. Run: python3 scripts/mirror-plan  # generate ArtifactData batches
# 6. Send batches to claude.ai artifact
# 7. Run: python3 scripts/mirror-mark-synced --batch <file>  # confirm synced
# 8. Verify: fact_check_hook + research_guardrail on any generated content
```

### 7. Fallback When Data Stages
- If vector DB empty -> fall back to `read_file` on data JSON files
- If mirror out of date -> `mirror-ledger-set` to correct versions
- If schema mismatch -> `verify.py` validates JSON against SCHEMA.md
- If leak detected -> auto-redact via `LEAK_BULLETIN_RE` pattern

## Accuracy-Enforced Commands

### Daily Accuracy Check
```bash
hermes config set security.redact_secrets true
# Ensure no secrets leak into responses
```

### Semantic Search Query
```bash
# Query the vector DB for most relevant facts
python3 -c "
import chromadb
client = chromadb.PersistentClient(path='/opt/data/frictionless_brain/chroma')
collection = client.get_collection('decisions')
results = collection.query(query_texts=['release management'], n_results=3)
for i, doc in enumerate(results['documents'][0]):
    print(f'{i+1}. {doc[:200]}')
"
```

### Guardrail-Powered Response
```bash
# Any generated content must pass:
# 1. fact_check_hook: no hallucination markers
# 2. research_guardrail: 3+ sources from 2+ domains  
# 3. skillspector_gate: no security risks in plugins
```