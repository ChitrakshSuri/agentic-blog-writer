# Agentic Blog Writer

A blog post generation system that orchestrates research, content planning, parallel section writing, and image generation using LangGraph and LangChain.

## Overview

The system implements a multi-stage pipeline:

1. **Router**: Analyzes the topic and selects research mode (closed-book, hybrid, open-book)
2. **Research**: Executes web search via Tavily with deduplication
3. **Planning**: Generates structured task breakdown with Pydantic schemas
4. **Orchestration**: Coordinates parallel section writers with evidence grounding
5. **Reduction**: Merges sections, decides image placement, generates diagrams via Gemini

All components are orchestrated via LangGraph with conditional routing and parallel fanout.

## Architecture

```
┌─────────────┐
│   Router    │  Analyzes topic → decides research mode
└──────┬──────┘
       │
       ├──→ Closed Book    (no research)
       ├──→ Hybrid         (targeted research)
       └──→ Open Book      (comprehensive research)
            │
            ↓
       ┌──────────────┐
       │  Research    │  Tavily search + deduplication
       └──────┬───────┘
              │
              ↓
       ┌──────────────────┐
       │  Orchestrator    │  Plan section structure
       └──────┬───────────┘
              │
              ↓
       ┌──────────────────────────────┐
       │  Workers (parallel)          │  Write each section
       │  - Evidence grounding        │  - Citation support
       │  - Section coherence         │  - Target word counts
       └──────┬───────────────────────┘
              │
              ↓
       ┌────────────────────────────────────┐
       │  Reducer Subgraph (3-node flow)    │
       ├────────────────────────────────────┤
       │ 1. merge_content                   │  Combine sections
       │ 2. decide_images                   │  Add [[IMAGE_N]] placeholders
       │ 3. generate_and_place_images       │  Gemini image generation
       └────────────────────────────────────┘
              │
              ↓
       ┌──────────────────┐
       │  Final Output    │  Markdown + images/
       └──────────────────┘
```

## Data Models

**Content Schemas**
- `Task`: Blog section specification (goal, bullets, word count targets)
- `Plan`: Blog outline (title, audience, tone, task list)
- `EvidenceItem`: Research result (title, URL, snippet)

**Image Generation**
- `ImageSpec`: Individual image metadata (placeholder, filename, prompt, size)
- `GlobalImagePlan`: Complete image strategy (markdown + image list)

**State**
- `State`: TypedDict tracking topic, plan, research results, section outputs, and image pipeline

## Getting Started

### Setup

```bash
python -m venv myvenv
source myvenv/bin/activate
pip install -r requirements.txt
```

### Environment Configuration

Create `.env` with required API keys:
```
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
GOOGLE_API_KEY=AIzaSy...
```

Optional (LangSmith tracing):
```
LANGCHAIN_API_KEY=ls-...
LANGCHAIN_TRACING_V2=true
LANGCHAIN_ENDPOINT=https://api.smith.langchain.com
LANGCHAIN_PROJECT=agentic-blog-writer
```

### Usage

**Python API**
```python
from bwa_backend import app

result = app.invoke({"topic": "Self-Attention in Neural Networks"})
print(result["final"])
```

**Web UI**
```bash
streamlit run bwa_frontend.py
```

Open http://localhost:8501

## Dependencies

Core libraries:
- `langgraph` - Graph-based agent orchestration
- `langchain` - LLM abstraction and utilities
- `langchain-openai` - OpenAI integration
- `pydantic` - Schema validation
- `langchain-community` - Tavily search integration
- `google-genai` - Gemini image generation
- `streamlit` - Web UI framework
- `python-dotenv` - Environment variable management

Full dependency list in `requirements.txt`.

## Project Layout

```
agentic-blog-writer/
├── README.md
├── requirements.txt
├── .env                           # API keys (not committed)
├── 1_bwa_basic.ipynb              # Notebook implementation
├── blog_writing_agent.ipynb       # Alternative notebook version
├── bwa_backend.py                 # LangGraph pipeline (pure Python)
├── bwa_frontend.py                # Streamlit interface
├── myvenv/                        # Virtual environment
└── images/                        # Generated output (runtime)
```

## 🔄 Workflow Example

```python
from bwa_backend import app

# Single invocation
output = app.invoke({
    "topic": "Transformers and Attention Mechanisms",
})

# Streaming (if using LangGraph with invoke_with_stream)
for chunk in app.stream({
    "topic": "Self-Attention Architecture",
}):
    print(chunk)
```

## Image Generation

The system evaluates whether images improve understanding and:

1. Analyzes content to identify diagram opportunities
2. Inserts `[[IMAGE_N]]` placeholders into markdown
3. Generates image prompts for Gemini (technical diagrams, flows, etc.)
4. Creates images at specified resolution (1024x1024, 1024x1536, 1536x1024)
5. Saves to `images/` directory
6. Falls back gracefully if generation fails

## Configuration

### Router Modes

- `closed_book`: No web research, relies on model knowledge
- `hybrid`: Targeted search for key topics
- `open_book`: Comprehensive research on all sections

### Customization

Edit `bwa_backend.py` to customize:

- Model selection: `ChatOpenAI(model="gpt-4.1-mini")`
- Image parameters: resolution, quality, max count
- System prompts: tone, style, audience for each stage
- Output paths and format

## 📊 Output Format

The system produces:

1. **Markdown file**: `{blog_title}.md`
   ```markdown
   # {Title}
   
   ## Section 1
   ... content with [[IMAGE_1]] placeholder ...
   
   ## Section 2
   ... more content ...
   ```

2. **Images directory**: `images/`
   ```
   images/
   ├── qkv_flow.png
   ├── attention_mechanism.png
   └── transformer_arch.png
   ```

3. **Return state**: Full execution state with metadata

## 🐛 Troubleshooting

### Issue: `GOOGLE_API_KEY not found`
**Solution**: Add `GOOGLE_API_KEY` to `.env` and run `python -m dotenv load`

### Issue: Tavily search returns empty results
**Solution**: Verify `TAVILY_API_KEY` in `.env`; use more specific search queries

### Issue: Image generation fails
**Solution**: Check Gemini API quota; system will fall back to text-only (graceful degradation)

### Issue: LangSmith tracing not showing
**Solution**: Verify `LANGCHAIN_TRACING_V2=true` and valid `LANGCHAIN_API_KEY`

## 📝 Roadmap

- [ ] Multi-language support
- [ ] Custom LLM backends (Anthropic, LLaMA)
- [ ] Persistent blog library storage
- [ ] Advanced SEO optimization
- [ ] Image style customization
- [ ] Bulk blog generation

## 🤝 Contributing

Suggestions, bug reports, and PRs welcome!

## 📄 License

MIT

---

**Built with ❤️ using LangGraph, LangChain, and Gemini**
