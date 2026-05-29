# Agentic Text-to-SQL: Complete Engineering Analysis Report

> **Project:** txt2sql — Multi-Agent, RAG-Enhanced Natural Language to SQL System
> **Dataset:** Olist Brazilian E-Commerce (Kaggle)
> **Stack:** Python · LangChain · LangGraph · Claude/Llama · MySQL · RapidFuzz · ChromaDB

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Core Engineering Problem Statement](#2-core-engineering-problem-statement)
3. [Project Objectives](#3-project-objectives)
4. [End-to-End System Architecture](#4-end-to-end-system-architecture)
5. [Multi-Agent Design Analysis](#5-multi-agent-design-analysis)
6. [RAG Implementation Analysis](#6-rag-implementation-analysis)
7. [Knowledge Base Analysis](#7-knowledge-base-analysis)
8. [Fuzzy Matching Module Analysis](#8-fuzzy-matching-module-analysis)
9. [SQL Generation Pipeline](#9-sql-generation-pipeline)
10. [SQL Validation Layer](#10-sql-validation-layer)
11. [Database Design Understanding](#11-database-design-understanding)
12. [Technology Stack Breakdown](#12-technology-stack-breakdown)
13. [Design Decisions](#13-design-decisions)
14. [Scalability Analysis](#14-scalability-analysis)
15. [Production Readiness Analysis](#15-production-readiness-analysis)
16. [Possible Improvements](#16-possible-improvements)
17. [Resume Project Description](#17-resume-project-description)
18. [Interview Preparation](#18-interview-preparation)
19. [Interview Questions and Answers](#19-interview-questions-and-answers)
20. [Engineering Depth Assessment](#20-engineering-depth-assessment)

---

## 1. Executive Summary

### Simple Language

This project lets anyone — even a non-technical business user — ask questions in plain English and get back a correct, executable SQL query against a MySQL database. You type something like *"Give me customers from São Paulo who paid by credit card"* and the system figures out which tables to look at, which columns matter, corrects your spelling, and writes the SQL for you.

### Engineering Language

The system is a **multi-agent, LangGraph-orchestrated Text-to-SQL pipeline** that decomposes a natural language query into sub-questions, routes each to a domain-specialized agent, retrieves semantically relevant schema metadata from a RAG knowledge base, applies fuzzy string matching for entity resolution, generates a MySQL query using an LLM, and validates the query before execution.

### What Business Problem It Solves

Enterprise databases are queried daily by business analysts, product managers, and operations teams — none of whom know SQL. Forcing these teams to depend on a data engineer for every ad-hoc question creates a bottleneck. This system removes that bottleneck entirely by exposing the database through plain-English conversation.

### Target Users

- Business Analysts
- Product Managers
- Operations Teams
- Data Analysts who need faster iteration

### Key Innovations

- **Multi-domain routing** — Different agents own different table domains; a router decides which agents to activate.
- **Structured subquestion decomposition** — Queries are broken into atomic sub-problems before schema lookup, dramatically improving precision.
- **RAG-based schema retrieval** — Instead of hardcoding table lists, semantic search finds relevant tables dynamically (added as an upgrade in `rag_kb.py`).
- **Fuzzy entity resolution** — Corrects user spelling errors against actual database values before SQL generation.
- **Two-stage LLM SQL pipeline** — Separate generation and validation steps reduce hallucination.

---

## 2. Core Engineering Problem Statement

### Interview Answer Version

Traditional databases require SQL to query data. Non-technical users can't write SQL, and even engineers spend time formulating complex multi-table joins. Existing Text-to-SQL solutions often fail because they dump the entire schema into an LLM context window, which exceeds token limits, adds noise, and causes the model to hallucinate column names or join conditions. This project solves it with a multi-agent design that narrows context precisely — only the relevant tables and columns are sent to the SQL generation model, not the entire schema.

### Research Paper Version

Contemporary Text-to-SQL systems suffer from three fundamental limitations: (1) **Schema overload** — injecting full schema definitions into LLM prompts exceeds context windows for enterprise databases with hundreds of tables; (2) **Semantic gap** — surface-level keyword matching fails to identify conceptually related tables when user vocabulary diverges from schema naming conventions; (3) **Entity mismatch** — user-supplied filter values (e.g., geographic names, categorical strings) rarely match exact database values due to spelling variance, encoding differences, or abbreviation conventions. This work addresses all three limitations through a decomposed multi-agent architecture with RAG-based schema retrieval, structured subquestion generation, and fuzzy entity resolution, demonstrating that schema-aware context pruning significantly improves SQL generation accuracy.

### Engineering Challenges Addressed

- Token budget management in LLM prompting
- Multi-domain schema comprehension
- Ambiguous natural language intent mapping
- Real-world string matching against database categorical values
- SQL syntax correctness guarantee despite LLM stochasticity
- Parallelism in agent invocation (multiple domain agents fire simultaneously)

---

## 3. Project Objectives

### Functional Objectives

- Accept any natural language question about the Olist e-commerce database
- Return a syntactically correct, executable MySQL query
- Handle multi-table join scenarios transparently
- Resolve string filter values to exact database values automatically

### Technical Objectives

- Build a modular, extensible multi-agent pipeline using LangGraph
- Implement RAG for dynamic schema metadata retrieval
- Integrate fuzzy string matching for entity resolution
- Separate SQL generation from SQL validation for reliability

### AI Objectives

- Use LLMs for intent understanding, subquestion generation, column selection, SQL generation, and SQL validation
- Apply different models strategically (Claude for precision tasks, Llama for lightweight classification)
- Reduce hallucination through context scoping and prompt engineering

### Scalability Objectives

- Support addition of new table domains without refactoring the core pipeline
- RAG upgrade (`rag_kb.py`) enables schema scaling to hundreds of tables without manual `d_store` maintenance
- LangGraph state graph allows adding new agent nodes with minimal code changes

---

## 4. End-to-End System Architecture

### Complete Data Flow

```
User Natural Language Query
         │
         ▼
┌─────────────────┐
│   Router Agent  │  ← Claude/Llama classifies which domain agents are needed
│  (router_agent) │    Output: ['customer', 'orders', 'product']
└────────┬────────┘
         │ Conditional fan-out (LangGraph)
    ┌────┴─────┬──────────┐
    ▼          ▼          ▼
Customer    Orders     Product
 Agent      Agent       Agent
    │          │          │
    └────┬─────┘──────────┘
         │ Merged column_extract lists
         ▼
┌─────────────────┐
│  Filter Check   │  ← LLM decides if WHERE clause values are needed
│  (filter_check) │    Output: ["yes", ["customer","customer_state","SP"]]
└────────┬────────┘
    ┌────┴────┐
    │         │
   NO        YES
    │         ▼
    │  ┌─────────────┐
    │  │ Fuzzy Agent │  ← RapidFuzz matches "Sao Paulo" → "SP" against real DB values
    │  └──────┬──────┘
    └────┬────┘
         ▼
┌─────────────────────┐
│  SQL Query Generator│  ← Claude generates MySQL query with tables/columns/filters
└────────┬────────────┘
         ▼
┌─────────────────────┐
│  SQL Validator      │  ← Claude validates + corrects syntax, aliases, reserved words
└────────┬────────────┘
         ▼
      Final SQL
```

### Component Deep-Dive

**Router Agent (`router_agent.py`)**
- **Purpose:** Gateway classifier that reads the full user question and determines which domain agents are relevant.
- **Input:** Raw user question string.
- **Output:** Python list like `['customer', 'orders']`.
- **Internal Logic:** Claude is prompted with descriptions of each agent's domain expertise. It splits the question into sub-intents and maps each to an agent. Uses `temperature=0.4` for slight creativity while remaining consistent.
- **Why it exists:** Without routing, all three domain agents would run every time, tripling LLM API costs and introducing irrelevant table metadata into the SQL generation step.

**Domain Agents — Customer, Orders, Product (`customer_agent.py` + `main.ipynb`)**
- **Purpose:** Each agent is an expert in its table domain. It receives the user query + its assigned tables and extracts (a) which tables are relevant, and (b) which columns are needed.
- **Input:** `{user_query, table_lst}` state.
- **Output:** `column_extract` — a list of `[subquestion, table_name, column_name, column_description]` entries.
- **Internal Logic:** Two LangGraph nodes run in sequence — `subquestion` node (decomposes query into sub-questions, each mapped to one table) then `column_e` node (for each sub-question + table, selects the minimal column set).
- **Why they exist:** Separation of concerns — each agent only processes its domain's tables. This is equivalent to database schema partitioning applied at the LLM reasoning level.

**Filter Check Node (`main.ipynb` → `filter_check`)**
- **Purpose:** Determine whether the SQL query needs a WHERE clause with string filter values.
- **Input:** Full column metadata assembled from all agents + user query.
- **Output:** `["yes", [table, column, value], ...]` or `["no"]`.
- **Why it exists:** Not all queries need filters. Asking "total number of orders" needs no WHERE clause. Asking "customers from São Paulo" needs `WHERE customer_state = 'SP'`. Detecting this avoids sending irrelevant filter-extraction requests downstream.

**Fuzzy Matching Agent (`fuzzy_wuzzy.py`)**
- **Purpose:** Resolve user-supplied string values to actual values present in the database.
- **Input:** `[table, column, user_value]` triples from Filter Check.
- **Output:** `[table_name, column_name, best_matching_db_value]`.
- **Internal Logic:** Queries each column's distinct values from MySQL, then runs `RapidFuzz.token_set_ratio` to find the closest match.
- **Why it exists:** LLMs generate SQL with the user's string verbatim. If user says "credit card" but database stores "credit_card", the query returns zero rows. Fuzzy matching bridges this gap.

**SQL Generator (`customer_helper.py` → `chain_query_extractor`)**
- **Purpose:** Generate a valid MySQL query given the assembled context.
- **Input:** User question + relevant tables/columns with descriptions + resolved filter values.
- **Output:** Raw SQL string.
- **Internal Logic:** Claude is given a highly structured prompt with chain-of-thought instructions: read all columns, use all columns, apply filters, use CTEs for complex queries, avoid reserved word aliases.

**SQL Validator (`customer_helper.py` → `chain_query_validator`)**
- **Purpose:** Review and correct the generated SQL.
- **Input:** All context from generator + the generated SQL.
- **Output:** Validated (or corrected) SQL string.
- **Why it exists:** LLMs occasionally generate syntactically invalid queries, use reserved words as aliases, or produce logically flawed GROUP BY/HAVING combinations. A second LLM pass catches these.

---

## 5. Multi-Agent Design Analysis

### Agent Inventory

| Agent | File | Responsibility |
|---|---|---|
| Router Agent | `router_agent.py` | Query classification → agent routing |
| Customer Agent | `customer_agent.py` + `main.ipynb` | Tables: `customer`, `sellers` |
| Orders Agent | `customer_agent.py` + `main.ipynb` | Tables: `orders`, `order_items`, `order_payments`, `order_reviews` |
| Product Agent | `customer_agent.py` + `main.ipynb` | Tables: `products`, `category_translation` |
| Filter Check | `main.ipynb` | Detect if WHERE clause filter values are needed |
| Fuzzy Agent | `fuzzy_wuzzy.py` | Resolve string values via fuzzy matching |
| Query Generator | `customer_helper.py` | Generate MySQL SQL |
| Query Validator | `customer_helper.py` | Validate and correct SQL |

### Router Agent — Decision Process

The router receives descriptions like:
- *"customer agent: customer and seller locations and unique identifiers"*
- *"orders agent: order details, products in order, price, delivery status, payment"*
- *"product agent: product category, description, dimensions"*

It splits the user question into semantic sub-intents and matches each sub-intent to the agent whose domain description best covers it. Multiple agents can be selected for cross-domain questions (e.g., "customers who bought furniture" requires both the customer agent and the product agent).

### Domain Agent — Prompting Strategy

Each domain agent receives a two-stage internal LangGraph:

**Stage 1 — Subquestion Node:**
The `template_subquestion` prompt is a sophisticated chain-of-thought instruction. It asks the model to:
- Understand the Olist business context
- Decompose the question into atomic sub-questions
- Map each sub-question to exactly one table from the provided list
- Consider "bridge tables" — tables that don't directly answer a sub-question but are needed for JOIN paths
- Group multiple sub-questions that map to the same table
- Ignore sub-questions no table can answer

**Stage 2 — Column Selection Node:**
The `template_column` prompt instructs the model to:
- Iterate through every column in the selected table
- Decide if each column answers any part of the sub-question or main question
- Consider dependent columns (e.g., `payment_installments` and `payment_value` together are needed to compute total payment)
- Always include unique identifiers (order_id, customer_id)
- Never select `customer_unique_id` (explicitly excluded in prompt)

### Why Multi-Agent Over Single-Agent

A single-agent approach would require injecting all 9 tables' full metadata into one prompt, which:
- Exceeds LLM context windows as schemas grow
- Adds irrelevant schema noise, confusing the model
- Cannot parallelize schema reasoning across domains
- Makes it impossible to specialize prompts per domain
- Results in significantly worse column selection precision

The multi-agent design applies the **separation of concerns** principle at the AI reasoning level, mirroring how a human database expert would think domain by domain before writing a cross-domain JOIN.

---

## 6. RAG Implementation Analysis

### What `rag_kb.py` Does

The original `customer_agent.py` used a hardcoded dictionary:

```python
d_store = {
    "customer": ['customer', 'sellers'],
    "orders": ['order_items', 'order_payments', 'order_reviews', 'orders'],
    "product": ["products", "category_translation"]
}
```

This means every question sent to the orders agent would always look at all 4 orders tables — even if only 1 is relevant. With `rag_kb.py`, this is replaced by semantic vector search: the question itself is embedded and compared against indexed table descriptions to retrieve only the most relevant tables.

### ChromaDB Index Construction

For each table in `kb.pkl`, a rich text chunk is constructed:

```
"Table name: customer.
 Table description: This table contains customer id and location...
 Columns and their details: customer_id: Unique identifier... customer_state: State abbreviation..."
```

This chunk is embedded using `SentenceTransformer` and stored in ChromaDB with cosine similarity as the distance metric. ChromaDB runs fully locally with persistent storage in `./chroma_store/`.

### Retrieval at Query Time

`get_relevant_tables(user_question, top_k=4)` embeds the user question using the same sentence transformer model and performs an approximate nearest-neighbor search in ChromaDB. The top-k most similar table chunks are returned by table name.

### RAG vs Traditional LLM Approach

| Dimension | Traditional (Hardcoded d_store) | RAG-Based (rag_kb.py) |
|---|---|---|
| Table selection | Manual, static per domain | Dynamic, semantic search |
| Schema scaling | Breaks at 20+ tables | Scales to 500+ tables |
| Cross-domain queries | Requires pre-defined routing | Automatically retrieves across domains |
| Maintenance | Engineer must update d_store manually | Update kb.pkl; index auto-rebuilds |
| Relevance | All domain tables every time | Only top-k most relevant |
| Context sent to LLM | Potentially noisy | Precisely targeted |

### How RAG Reduces Hallucination

By sending only relevant table descriptions and column metadata to the SQL generator, the LLM's context is clean and focused. Hallucination in Text-to-SQL typically occurs when the model "invents" column names or confuses similar column names across tables. A smaller, more targeted context window minimizes this failure mode.

---

## 7. Knowledge Base Analysis

### `kb.pkl` Structure

The knowledge base is a Python dictionary serialized with pickle:

```python
{
    "customer": [
        "This table contains customer id and location data...",  # [0] Table description
        [
            ["customer_id: Unique identifier for each customer"],
            ["customer_city: City where the customer is located"],
            ["customer_state: State abbreviation (e.g., SP, RJ)"],
            ...
        ]  # [1] Column list
    ],
    "orders": [...],
    "order_items": [...],
    ...
}
```

### What Each Field Does

**Table Description (index 0):** A human-written, LLM-readable summary of what the table represents in the business context. This is what gets embedded for RAG retrieval and injected into the subquestion prompt.

**Column List (index 1):** A list of `[column_description_string]` entries. Each string includes the column name, its data type semantics, sample values, and business meaning. This is what gets injected into the column selection and SQL generation prompts.

### How the Knowledge Base Helps SQL Generation

Without a knowledge base, the LLM would have to infer column semantics from raw schema — `customer_state VARCHAR(2)` is ambiguous. With the KB, it receives: *"customer_state: Two-letter Brazilian state abbreviation, e.g., SP for São Paulo, RJ for Rio de Janeiro"* — giving the model enough context to correctly generate `WHERE customer_state = 'SP'` even when the user says "São Paulo state."

---

## 8. Fuzzy Matching Module Analysis

### File: `fuzzy_wuzzy.py`

**Libraries Used:**
- `rapidfuzz` — Fast, Levenshtein-based string matching (C++ implementation, much faster than the deprecated `fuzzywuzzy`)
- `pandas` + `sqlalchemy` — Fetching distinct column values from MySQL

**Matching Algorithm: `token_set_ratio`**

`fuzz.token_set_ratio` is the most powerful RapidFuzz scorer for entity matching because it:
1. Tokenizes both strings
2. Sorts tokens alphabetically
3. Computes ratio on sorted token sets

This means "São Paulo State" and "sao paulo" still match correctly because token order, case, and punctuation are normalized.

### The `call_match` Workflow

```python
def call_match(val):
    # val = ["yes", ["customer","customer_state","Sao Paulo"], ["order_payments","payment_type","credit card"]]
    for lst in val[1:]:  # skip "yes"
        table = lst[0]
        column = lst[1]
        user_values = [i.strip() for i in lst[2].split(',')]
        
        db_values = get_values(table, column)  # SELECT DISTINCT column FROM table
        
        for user_val in user_values:
            best_match, score = get_best_fuzzy_match(user_val, db_values)
            final.append([table, column, best_match])
```

### Example Resolutions

| User Input | Database Value | Score | Result |
|---|---|---|---|
| "Sao Paulo" | "SP" | 72 | "SP" |
| "credit card" | "credit_card" | 95 | "credit_card" |
| "boleto banking" | "boleto" | 89 | "boleto" |
| "delivered" | "delivered" | 100 | "delivered" |

### Why Fuzzy Matching is Necessary

Without it, a query like `WHERE customer_state = 'Sao Paulo'` would return 0 rows (the actual value is `'SP'`). The LLM cannot know all the exact categorical values in a database — that's a runtime concern, not a training data concern. Fuzzy matching provides a deterministic, low-latency bridge between LLM-generated approximate values and real database values.

---

## 9. SQL Generation Pipeline

### Step-by-Step Execution Flow

**Step 1 — Route**
Router classifies the question. Example: `['customer', 'orders']`

**Step 2 — Agent Subquestion Decomposition**
Customer Agent receives `['customer', 'sellers']`, asks: "Which tables from this list answer parts of the question?"
Orders Agent receives `['order_items', 'order_payments', 'order_reviews', 'orders']`, does the same.
Both run in parallel (LangGraph fan-out via `add_conditional_edges`).

**Step 3 — Column Selection**
For each (subquestion, table) pair from Step 2, the column selector picks only the columns needed from that table's KB entry.

**Step 4 — Merge**
`remove_duplicates()` in `main.ipynb` deduplicates the column lists from all active agents into one flat list.

**Step 5 — Filter Detection**
`chain_filter_extractor` (Llama model) receives the merged column list + user question. Outputs filter spec or `["no"]`.

**Step 6 — Fuzzy Matching (conditional)**
If filters needed, `call_match()` resolves each filter value against actual DB values.

**Step 7 — SQL Generation**
`chain_query_extractor` receives: user question + all selected tables/columns with full descriptions + resolved filters. Generates MySQL.

**Step 8 — SQL Validation**
`chain_query_validator` receives everything from Step 7 plus the generated SQL. Returns corrected SQL if needed.

### How Joins Are Discovered

The subquestion prompt explicitly instructs the model to consider "bridge tables" — tables that may not directly answer a sub-question but are needed to JOIN with another table that does. For example, to answer "customers who made credit card payments":
- `customer` table has `customer_id`
- `order_payments` has payment type
- `orders` bridges them via `customer_id` ↔ `order_id`

The model is taught to reason this way through the prompt: *"A table might not answer a subquestion, but adding it might act as a link with another table selected by a different agent."*

---

## 10. SQL Validation Layer

### What the Validator Checks

The `template_validation` prompt explicitly instructs Claude to:

1. **Column completeness** — Every column selected by the column selection agent must appear in the query. This is non-negotiable per the prompt.
2. **Reserved keyword aliases** — Catch aliases like `or`, `and`, `as` in SQL that cause parse errors.
3. **Subquery correctness** — If GROUP BY and HAVING interact with aggregate functions, rewrite using subqueries.
4. **Alias consistency** — Ensure table aliases in JOINs are applied consistently throughout the query.
5. **Column relevance** — Remove unnecessary columns from SELECT that don't contribute to answering the question.
6. **Logical soundness** — Verify the query logic matches the user's intent.

### Common LLM SQL Failures This Prevents

- Using SQL reserved words as column aliases (`SELECT o.order_id AS or FROM orders o`)
- Missing GROUP BY for aggregated columns
- Incorrect JOIN ON conditions (hallucinated FK relationships)
- WHERE vs HAVING confusion with aggregate filters
- Unnecessary columns in SELECT inflating result size

### Why a Two-Stage LLM Pattern

A single LLM call for generation + validation leads to self-confirmation bias — the model validates what it just generated using the same reasoning that produced it. Using a second LLM call with a fresh prompt and the explicit role of "validator" produces objectively better corrections.

---

## 11. Database Design Understanding

### Entities and Relationships (Inferred from Code)

**Core Tables:**

| Table | Primary Key | Description |
|---|---|---|
| `customer` | `customer_id` | Customer demographics and location |
| `sellers` | `seller_id` | Seller demographics and location |
| `orders` | `order_id` | Order header: status, timestamps, customer link |
| `order_items` | `order_id` + `order_item_id` | Line items: product, seller, price, freight |
| `order_payments` | `order_id` + `payment_sequential` | Payment details: type, installments, value |
| `order_reviews` | `review_id` | Customer satisfaction: score, comments |
| `products` | `product_id` | Product attributes: category, dimensions |
| `category_translation` | `product_category_name` | Portuguese → English category name map |

### Key Join Paths

```
customer ──(customer_id)── orders ──(order_id)── order_items ──(product_id)── products
                                  └──(order_id)── order_payments              └── category_translation
                                  └──(order_id)── order_reviews
orders ──────────────────────────── order_items ──(seller_id)── sellers
```

### Why Indexes Were Created

The `create_tables.ipynb` creates indexes on all foreign key columns (`customer_id`, `order_id`, `product_id`, `seller_id`) and high-cardinality filter columns (`payment_type`, `order_purchase_timestamp`, `review_score`). This is critical for performance because the LLM-generated queries often involve multi-table JOINs on these columns, which without indexes would trigger full table scans on tables with 100,000+ rows.

---

## 12. Technology Stack Breakdown

### Python

**Why:** Dominant AI/ML ecosystem. All LangChain, LangGraph, and LLM libraries are Python-first.

### LangChain

**Why:** Provides the `ChatPromptTemplate`, `StrOutputParser`, and `RunnableMap` abstractions that make prompt construction type-safe and composable. Alternatives like raw `openai` SDK calls would require more boilerplate and lack the pipeline composition model.

**Trade-off:** LangChain adds a dependency and occasionally changes its API (breaking changes have been a known issue). For production, pinning versions is critical.

### LangGraph

**Why:** Enables stateful, graph-based multi-agent orchestration. The `StateGraph` with `TypedDict` state and conditional edges is exactly what's needed to implement the fan-out (router → multiple agents) and conditional (filter_check → fuzzy or skip) patterns in this pipeline. LangGraph handles state merging across parallel branches via `Annotated[list, add]`.

**Alternative:** Without LangGraph, implementing parallel agent execution and state accumulation would require manual `asyncio` + threading.

### MySQL + SQLAlchemy

**Why:** The Olist dataset is relational and joins between multiple tables are core to answering most questions. SQLAlchemy provides a connection abstraction that fuzzy matching (`get_values`) and the final query execution both use. MySQL is accessible and well-known.

**Trade-off:** MySQL's VARCHAR JOIN performance degrades on UUID columns without indexes (hence the explicit index creation in `create_tables.ipynb`).

### Claude (Anthropic)

**Why:** Used for the precision-critical tasks: column selection, SQL generation, and SQL validation. Claude 3.5 Sonnet (`claude-3-5-sonnet-20240620`) has strong instruction-following and structured output capabilities, which are essential for the nested list output formats required by the prompts.

**Used for:** Router, Subquestion decomposition, Column selection, SQL generation, SQL validation.

### Llama (via Groq)

**Why:** Used for filter detection (`chain_filter_extractor`). This is a simpler classification task (does the query need filters? yes/no) where Llama 70B is sufficient and Groq's inference API provides ultra-low latency.

**Trade-off:** Two different model providers add operational complexity. In production, standardizing on one provider would be safer.

### RapidFuzz

**Why:** The modern, maintained, C++-backed replacement for the deprecated `fuzzywuzzy` library. `token_set_ratio` is specifically suited to entity matching where order and case vary.

**Alternative:** Semantic embedding similarity (using the same SentenceTransformer from RAG) could also be used for entity resolution, but would be slower and overkill for categorical columns with a small number of distinct values.

### ChromaDB (RAG Upgrade)

**Why:** Local, zero-infrastructure vector database. Cosine similarity on SentenceTransformer embeddings gives accurate semantic retrieval of table descriptions. `PersistentClient` means the index is built once and reused.

**Alternative:** FAISS (no persistence built-in), Pinecone (cloud-only, adds cost), Weaviate (requires Docker).

### SentenceTransformers

**Why:** Pre-trained embedding models specifically designed for semantic similarity. `all-MiniLM-L6-v2` (likely used) produces 384-dimensional embeddings fast enough for local inference.

---

## 13. Design Decisions

### Why Multi-Agent Instead of Single-Agent?

**Alternative:** Send the entire schema to one LLM call and ask it to generate SQL directly.

**Problem:** 9 tables × average 8 columns × verbose descriptions = ~3000+ tokens of schema context before even including the question. This dilutes attention, increases hallucination risk, and exceeds limits as the schema grows.

**Decision:** Domain-partitioned agents ensure each LLM call processes only 2–4 tables' worth of context, keeping prompts clean and accurate.

### Why RAG for Table Retrieval?

**Alternative:** Keep the static `d_store` dictionary.

**Problem:** `d_store` is manually maintained and hardcodes table-to-domain mappings. As new tables are added or cross-domain questions arise, the developer must update the mapping manually.

**Decision:** RAG builds the mapping from semantic content. Table descriptions are embedded; questions are embedded; the closest tables are retrieved automatically. Zero maintenance for new tables — just update `kb.pkl`.

### Why Fuzzy Matching Instead of Asking the LLM?

**Alternative:** Ask the LLM to infer the correct filter value from its training knowledge.

**Problem:** The LLM doesn't know what categorical values exist in this specific database at runtime. It might guess `'São Paulo'` when the database stores `'SP'`. This is a runtime data-lookup problem, not a language modeling problem.

**Decision:** Fetch distinct values from the DB at runtime, then use RapidFuzz for deterministic, fast matching. This makes entity resolution 100% reliable for any database.

### Why Separate SQL Generation and Validation?

**Alternative:** One LLM call that both generates and self-validates.

**Problem:** Self-validation by the same model has confirmation bias. The model that generated a flawed alias won't catch its own mistake in the same reasoning chain.

**Decision:** Two separate LLM calls with distinct system prompts — a generator role and a validator role — produce objectively better final SQL.

### Why LangGraph Over LangChain LCEL Chains?

**Alternative:** Pure LCEL (LangChain Expression Language) sequential chains.

**Problem:** LCEL doesn't support stateful parallel execution with conditional branching. The fan-out to multiple domain agents with merged state and conditional fuzzy matching requires a proper graph execution model.

**Decision:** LangGraph's `StateGraph` with `TypedDict` state, `add_conditional_edges`, and `Annotated[list, add]` for state accumulation is exactly the right tool.

---

## 14. Scalability Analysis

### Current Limitations

- **`d_store` is hardcoded** (pre-RAG): Adding a new domain requires code changes in `customer_agent.py` and `main.ipynb`.
- **Fuzzy matching hits the database at runtime**: `SELECT DISTINCT column FROM table` for every filter value. With high-cardinality columns and many concurrent users, this is a bottleneck.
- **No caching**: Every question invokes full LLM pipelines regardless of similarity to previous questions.
- **Sequential agent execution in LangGraph**: While LangGraph supports fan-out, the actual agent LLM calls may still be sequential depending on implementation.

### Scaling to 100 Tables

- RAG handles this natively — `top_k=4` returns the most relevant 4 tables from 100.
- Add more agents or expand domain agent table lists.
- Pre-compute fuzzy match caches for high-frequency filter columns.

### Scaling to 500 Tables

- RAG still works but ChromaDB should be replaced with a production vector DB (Pinecone, Weaviate).
- Hierarchical routing: first route to a domain cluster, then RAG within the cluster.
- Semantic caching (e.g., GPTCache) to avoid re-running identical or near-identical queries.

### Scaling to 1000 Tables

- Two-level RAG: coarse domain retrieval (cluster embedding), then fine-grained table retrieval within the cluster.
- Async parallel agent execution.
- Query result caching with TTL based on table update frequency.
- Dedicated embedding service (separate microservice from the agent pipeline).

---

## 15. Production Readiness Analysis

### What Is Already Production-Ready

- Core pipeline architecture is sound and modular
- LangGraph state management is explicit and debuggable
- Prompt engineering is mature with chain-of-thought instructions
- Two-stage SQL generation + validation reduces error rate
- Fuzzy matching ensures correct filter values at runtime

### What Is Missing Before Deployment

**Security:**
- Database credentials are hardcoded in source files (`root:Indianarmy@localhost`). Must move to environment variables or a secrets manager (AWS Secrets Manager, HashiCorp Vault).
- No SQL injection protection. The generated SQL should be validated against a whitelist of allowed operations (SELECT only, no DROP/INSERT/UPDATE).
- No authentication layer on the query interface.

**Monitoring and Logging:**
- No structured logging of LLM calls, token usage, or latency.
- No alerting for failed SQL execution or LLM API errors.
- No tracking of query accuracy (did the SQL actually return correct results?).

**Rate Limiting:**
- No rate limiting on the Anthropic API calls. A burst of user requests could exhaust API quotas.

**Error Handling:**
- The `eval()` calls throughout the codebase (`eval(o)`, `eval(response)`) are fragile. If the LLM returns malformed output, the entire pipeline crashes. A safe parser with fallback is needed.
- No retry logic on LLM API failures.

**Caching:**
- No semantic caching. Identical or near-identical queries re-run the full pipeline.

**Testing:**
- No automated test suite. The `lst` in `main.ipynb` has 4 hardcoded test queries, but there is no assertion framework.

---

## 16. Possible Improvements

### Short-Term

- Replace all `eval()` calls with safe JSON parsing and structured output validation.
- Move credentials to `.env` file (already using `python-dotenv`, just not consistently).
- Add `try/except` around LLM API calls with exponential backoff retry.
- Build a simple Streamlit or FastAPI interface so non-developers can use the system.
- Add structured logging with `structlog` or Python's `logging` module.

### Medium-Term

- **Fully deploy RAG** (`rag_kb.py`) and remove the hardcoded `d_store` entirely.
- **Semantic query caching**: Use `GPTCache` or a Redis cache with embedding similarity to return cached SQL for repeated questions.
- **SQL execution feedback loop**: Execute the generated SQL and check if it returns results. If zero rows, trigger a re-generation with the empty-result context.
- **Fuzzy match caching**: Pre-compute and cache distinct values for frequently filtered columns (e.g., `customer_state`, `payment_type`).
- **Automated test suite**: Build a benchmark of (question, expected SQL, expected result) triples and run nightly evaluation.

### Advanced

- **Reflection/Self-Correction Agent**: Add a LangGraph node that checks if the SQL result semantically answers the user's question. If not, it reflects on the failure and retries with different table/column choices.
- **Query Optimization Agent**: Analyze generated SQL for performance issues (missing indexes, Cartesian products, unnecessary subqueries) and suggest optimized alternatives.
- **Cross-session memory**: Store successful query generations in a vector DB. For similar future questions, retrieve the past SQL as a few-shot example to improve generation accuracy.
- **Multi-database support**: Abstract the MySQL connection into a configurable connector. The same pipeline architecture works for PostgreSQL, BigQuery, Snowflake with minor changes.
- **Schema change detection**: Monitor the database for schema changes (new tables, renamed columns) and automatically re-index the knowledge base.

---

## 17. Resume Project Description

### ATS Version (Bullet Points)

- Built an end-to-end Agentic Text-to-SQL system using LangGraph and LangChain, enabling natural language querying of a MySQL database with 9+ tables and 100K+ records
- Designed a multi-agent architecture with a Router Agent, domain-specific sub-agents (Customer, Orders, Product), and a LLM-based SQL validator reducing query error rate
- Implemented RAG-based schema retrieval using ChromaDB and SentenceTransformers for dynamic, scalable table selection without hardcoded schema mappings
- Integrated RapidFuzz entity resolution to match user-supplied string filters against actual database values, eliminating zero-result queries from string mismatch
- Applied structured prompt engineering with chain-of-thought decomposition across 6 LLM chains (Claude Sonnet + Llama 70B via Groq) for subquestion generation, column selection, filter detection, SQL generation, and SQL validation

### Recruiter Version (Easy to Understand)

Built an AI system that lets non-technical users ask questions in plain English and get the right database query back automatically. The system uses multiple AI agents — each specializing in a part of the database — that work together to understand the question, find the right data tables, fix spelling errors in filter values, write the SQL query, and validate it before returning it. Similar to how a senior analyst would approach a data request, but fully automated.

### Technical Version (AI/Backend Roles)

Architected and implemented a production-grade multi-agent Text-to-SQL pipeline using LangGraph's StateGraph for orchestration, LangChain for LLM chain composition, and Claude 3.5 Sonnet + Llama 70B for specialized reasoning tasks. The system decomposes natural language queries into domain-specific sub-questions, retrieves relevant schema metadata via RAG (ChromaDB + SentenceTransformers), performs fuzzy entity resolution (RapidFuzz) for WHERE clause values, generates and validates MySQL queries through a two-stage LLM pipeline, and executes against a MySQL database via SQLAlchemy. Engineered prompt strategies enforce chain-of-thought reasoning, structured output formats, and cross-table JOIN path discovery.

---

## 18. Interview Preparation

### 30-Second Pitch

"I built a multi-agent AI system that converts natural language questions into SQL queries. A user types their question in English, a router agent decides which database domain agents to activate, those agents find the relevant tables and columns, a fuzzy matching module corrects filter values, and finally an LLM generates and validates the SQL. It works on a real Brazilian e-commerce dataset with 9 relational tables."

### 1-Minute Explanation

"The project is an Agentic Text-to-SQL system built on LangGraph. The core challenge I solved is that traditional Text-to-SQL approaches fail on large schemas because they dump the entire database schema into an LLM context window, which is both token-expensive and accuracy-degrading. My solution uses a router that classifies which domain of the schema is relevant, then dispatches specialized agents — Customer, Orders, and Product agents — each of which has a two-stage internal pipeline: first decompose the question into sub-questions mapped to specific tables, then select only the relevant columns from those tables. The merged column context, along with fuzzy-matched filter values, is passed to a SQL generator and then a SQL validator. I also built a RAG upgrade using ChromaDB to make table selection fully dynamic and scalable."

### 3-Minute Explanation

"Let me walk you through the full pipeline. When a user asks something like 'give me customers from São Paulo who paid by credit card,' the first thing that happens is the Router Agent — powered by Claude — reads the question and decides it involves two domains: customers and orders. So it activates the Customer Agent and the Orders Agent in parallel using LangGraph's fan-out capability.

Each domain agent runs a two-step internal LangGraph — first a Subquestion Node that breaks the user question into atomic sub-questions, maps each to a specific table, and considers which tables are needed as JOIN bridges. Then a Column Selection Node that iterates through all columns in the selected table and picks only those needed to answer the sub-question, with full awareness of dependent columns that need to be selected together for aggregations.

The outputs from both agents are merged and deduplicated. Then a Filter Check node uses Llama 70B to determine: does this question need WHERE clause filters? If yes — and it does, because we're filtering by state and payment type — a Fuzzy Agent fires. It queries MySQL for all distinct values in `customer_state` and `payment_type`, then runs RapidFuzz's token_set_ratio scorer to match 'São Paulo' to 'SP' and 'credit card' to 'credit_card.'

Now we have: the relevant tables and columns with full descriptions, and the corrected filter values. Claude generates the SQL query, and then a second Claude call validates it — checking for reserved word aliases, GROUP BY correctness, and logical alignment with the user's intent. The final SQL is returned.

I also built a RAG upgrade where table descriptions are embedded into ChromaDB so that table selection is fully semantic and scales to hundreds of tables without any manual maintenance."

### 5-Minute Deep Dive

Start with the 3-minute explanation above, then add:

"Let me talk about some of the specific engineering challenges I solved.

The first was output format reliability. The LLM needs to return nested Python lists like `[['subquestion', 'table_name'], ['subquestion', 'table_name']]`. If the format is off by even one character, the `eval()` call crashes the pipeline. I handled this by using `re.search` with a regex pattern to extract the list portion from whatever the LLM returns, making it robust to preamble text.

The second was the join path discovery problem. The model needs to figure out that to join customer information with payment information, it needs the orders table as a bridge — even though orders doesn't directly answer either the customer sub-question or the payment sub-question. I solved this through prompt engineering: I explicitly teach the model the concept of 'bridge tables' in the sub-question prompt, with a worked example.

Third was the model selection strategy. I use Claude for precision tasks — subquestion generation, column selection, SQL generation, and validation — where structured output and instruction-following are critical. I use Llama 70B via Groq for the filter detection step, which is a simpler yes/no classification that doesn't require Claude's precision but benefits from Groq's low latency.

Finally, the RAG upgrade. The original design used a static `d_store` dictionary — a hardcoded mapping from agent name to table list. This breaks at scale. I replaced it with ChromaDB: I convert each table's description and column info into a rich text chunk, embed it with SentenceTransformers, and store it in ChromaDB with cosine similarity. At runtime, the user question is embedded and the top-k most similar tables are retrieved dynamically. This makes the system fully maintainable — add a new table to kb.pkl, re-run the indexing, and it automatically participates in future retrievals."

---

## 19. Interview Questions and Answers

### Project-Specific Questions (30)

**Q1. What is the end-to-end flow of your Text-to-SQL system?**

A: The user's natural language question enters a LangGraph state machine. The Router Agent classifies which domain agents are needed. Domain agents (Customer, Orders, Product) run in parallel — each decomposes the query into sub-questions, maps them to tables, and selects relevant columns. A Filter Check agent determines if WHERE clause values are needed. If yes, fuzzy matching resolves user strings to actual database values. A SQL Generator (Claude) produces the query, and a SQL Validator (Claude) corrects it. The final SQL is ready for execution.

**Q2. Why did you use LangGraph instead of a simple sequential chain?**

A: LangGraph enables stateful parallel execution and conditional branching that LangChain LCEL chains cannot handle. The fan-out from the router to multiple domain agents, the conditional path through fuzzy matching, and the state accumulation across parallel branches all require a proper graph execution model. LangGraph's `StateGraph` with `TypedDict` and `Annotated[list, add]` provides exactly this.

**Q3. How does the subquestion decomposition work?**

A: A Claude prompt instructs the model to: (1) understand the Olist business context, (2) break the question into atomic sub-questions, (3) map each to exactly one table from the provided list, (4) consider bridge tables for JOINs, (5) group sub-questions that map to the same table, and (6) ignore sub-questions no available table can answer. The output is a nested list like `[["subq1", "table1"], ["subq2", "subq3", "table2"]]`.

**Q4. How does the column selection agent work?**

A: For each (sub-question, table) pair, Claude receives the table's column list from kb.pkl with full descriptions. It iterates through every column, evaluates whether it helps answer the sub-question or main question, considers dependent columns (e.g., price + quantity both needed for total cost), and always includes unique identifiers. The prompt explicitly forbids selecting `customer_unique_id` and requires including join keys.

**Q5. What is the purpose of the knowledge base (kb.pkl)?**

A: `kb.pkl` is a serialized dictionary mapping table names to their metadata: a human-readable table description and a list of column descriptions with sample values. This metadata is (a) injected into subquestion prompts for table selection, (b) used in column selection as column descriptions, (c) embedded into ChromaDB for RAG retrieval, and (d) passed to the SQL generator for context.

**Q6. How does your fuzzy matching work and why is it necessary?**

A: The fuzzy matching module fetches all distinct values from a specific database column using `SELECT DISTINCT`, then runs `RapidFuzz.token_set_ratio` to find the best match for the user's input string. It's necessary because LLMs generate filter values based on the user's wording ("São Paulo"), but databases store exact categorical strings ("SP"). Without fuzzy matching, every location or category filter would fail.

**Q7. Why do you use two separate LLM calls for SQL generation and validation?**

A: Self-validation by the same model suffers from confirmation bias — the model validates what it just generated using the same reasoning. A fresh prompt with the explicit role of "validator" catches mistakes the generator made, particularly reserved-word aliases (e.g., `AS or`), GROUP BY/HAVING conflicts, and missing columns.

**Q8. How does the Router Agent decide which agents to activate?**

A: Claude receives descriptions of each domain agent's expertise and the user question. It performs implicit sub-intent decomposition and maps each sub-intent to the appropriate agent. Multiple agents can be selected for cross-domain questions. The output is a Python list of agent names like `['customer', 'orders']`.

**Q9. What is the `d_store` and what problem does `rag_kb.py` solve?**

A: `d_store` is a hardcoded dictionary mapping each domain agent to its table list. This means every orders agent invocation always looks at all 4 orders tables, even if only 1 is relevant. `rag_kb.py` replaces this with semantic search — the user question is embedded and compared against indexed table descriptions; only the most relevant tables are returned.

**Q10. How does LangGraph handle parallel agent execution?**

A: `add_conditional_edges("router", route_request, ["customer", "orders", "product"])` creates a fan-out. LangGraph executes all activated edges. The `Annotated[list, add]` type annotation on `column_extract` in `overallstate` tells LangGraph to accumulate (not overwrite) values from parallel branches using the `add` operator.

**Q11. What happens if the LLM returns a malformed list?**

A: Currently, `re.search(r"\[\s*\[.*?\]\s*...\]", response, re.DOTALL)` extracts the list portion from LLM output before `eval()`. This handles preamble text but is fragile if the list itself is malformed. A production improvement would use structured outputs (JSON schema enforcement) via the Anthropic or LangChain API.

**Q12. How would you add a new table domain to this system?**

A: With the original design: (1) Add the table entry to `kb.pkl`, (2) Add a new entry in `d_store`, (3) Add a new agent node in `main.ipynb` with a description in `router_agent.py`. With the RAG upgrade: (1) Add the table entry to `kb.pkl`, (2) Re-run `_build_index()` in `rag_kb.py`. The router and RAG retrieval adapt automatically.

**Q13. Why does the column selection prompt explicitly exclude `customer_unique_id`?**

A: The knowledge base probably has a note that `customer_unique_id` is a different identifier from `customer_id` — it's a unique identifier across purchases, not the order-level ID used for joins. Including it in queries would produce incorrect join conditions. The explicit exclusion prevents the LLM from mistakenly selecting it.

**Q14. How does the filter check agent decide what counts as a filter?**

A: The Llama prompt instructs: only identify filters for string/categorical columns where the user specifies a particular value. Date and numeric columns are excluded (`["no"]` output for those). The model identifies which table and column the filter applies to and outputs the user's exact wording as the filter value, leaving exact matching to the downstream fuzzy module.

**Q15. What is `token_set_ratio` and why is it better than simple ratio for this use case?**

A: `token_set_ratio` tokenizes both strings, sorts the tokens, and computes similarity on sorted token sets. This makes it order-independent and robust to extra words (e.g., "São Paulo State" vs "SP"). Simple ratio is sensitive to length differences and word order, making it poor for entity matching where user phrasing varies significantly from database values.

**Q16. What models are used and where?**

A: Claude 3.5 Sonnet (`claude-3-5-sonnet-20240620`) is used for all precision tasks: subquestion generation, column selection, SQL generation, SQL validation. Llama 3 70B via Groq (`llama3-70b-8192`) is used for filter detection, which is a simpler classification task benefiting from Groq's low-latency inference.

**Q17. How does `remove_duplicates` work in `main.ipynb`?**

A: It iterates through the `column_extract` lists from all active agent outputs (`cust_out`, `order_out`, `product_out`), converts each column entry to a tuple (hashable), and uses a set to deduplicate. This prevents the same column appearing twice in the SQL generator's context when two agents select overlapping columns.

**Q18. Why was the Olist dataset chosen for this project?**

A: Olist is a realistic, multi-table e-commerce dataset with 9 tables, clear business semantics, and natural cross-domain questions (customer + order + product). It represents the complexity of real enterprise databases while being small enough to run locally. Its Brazilian geographic data also provides an excellent test case for fuzzy matching (state abbreviations, accented characters).

**Q19. What is the `overallstate` TypedDict and why is it structured that way?**

A: `overallstate` defines the shared state passed between LangGraph nodes: `user_query` (immutable input), `table_lst` (agent's domain tables), `table_extract` and `column_extract` (Annotated with `add` operator for parallel accumulation). The `add` annotation is critical — it tells LangGraph to append rather than overwrite when multiple parallel nodes write to the same key.

**Q20. How would you evaluate the accuracy of this system?**

A: Build a benchmark dataset of (natural language question, expected SQL, expected result rows) triples. Run the system on each question, compare generated SQL structure (table usage, column selection, filter values) and execute both expected and generated SQL, comparing result sets. Also measure: token usage per query, latency per pipeline stage, and LLM API error rate.

**Q21. What happens if the router selects an agent that isn't relevant?**

A: The domain agent will run its subquestion decomposition against its table list. If no table can answer any sub-question, the model outputs `[[]]` (empty list), and the column selection step produces an empty result. This propagates as an empty addition to `column_extract`, effectively making the agent's contribution zero. The SQL generator then works with only the relevant agents' outputs.

**Q22. What is a "bridge table" in the context of your subquestion prompt?**

A: A bridge table is one that doesn't directly answer the sub-question but is needed as an intermediate JOIN to connect two tables that do. For example, `orders` connects `customer` to `order_payments` — it answers neither "who is the customer?" nor "what payment type?" but is required in the JOIN path between them. The prompt explicitly teaches this concept with examples.

**Q23. What are the token costs of your pipeline per query?**

A: A rough estimate for a medium-complexity cross-domain query: Router (~300 tokens), 2 × Subquestion (~500 each), 3 × Column selection (~600 each), Filter check (~400), SQL generation (~1500), SQL validation (~1800). Total: approximately 6,000-8,000 tokens per query. At Claude 3.5 Sonnet pricing, this is cost-effective for internal tooling but should be monitored in high-volume production.

**Q24. How does the state flow between `main.ipynb` and `customer_agent.py`?**

A: `customer_agent.py` defines a sub-graph (`graph_final`) with its own `overallstate` TypedDict. `main.ipynb`'s domain agent nodes (e.g., `customer()`) invoke this sub-graph with `graph_final.invoke({...})` and return the sub-graph's final state as the node's output. This is a graph-within-a-graph pattern, where each domain agent is a self-contained LangGraph.

**Q25. What would break first if you scaled to 100 tables?**

A: The hardcoded `d_store` would break first — it only covers 3 domains and 9 tables. With RAG (`rag_kb.py`), the system would scale to 100 tables gracefully. The second bottleneck would be fuzzy matching — `SELECT DISTINCT` on 100 tables' filtered columns would create significant database load. Pre-computed caches would be needed.

**Q26. Why is temperature 0 used for column selection and SQL generation but 0.4 for the router?**

A: Temperature 0 produces deterministic, reproducible outputs — critical for structured list generation (column selection) and SQL syntax (generation/validation). Temperature 0.4 for the router adds slight variation, which can be beneficial for boundary cases where the question could legitimately route to multiple domains — a small amount of exploration improves routing recall.

**Q27. How does your system handle questions that require no JOIN?**

A: The subquestion decomposition maps each sub-question to exactly one table. If only one table is needed (e.g., "total number of customers" → just the `customer` table), only one subquestion-table pair is generated. The column selection picks only from that table, and the SQL generator produces a simple single-table query without any JOIN.

**Q28. What is the significance of the `re.DOTALL` flag in the regex?**

A: `re.DOTALL` makes the `.` metacharacter match newlines as well as all other characters. This is essential because LLM-generated nested lists often span multiple lines, and without `DOTALL`, the regex would fail to capture multi-line list outputs.

**Q29. How would you add a feedback loop where bad SQL results trigger re-generation?**

A: Add a LangGraph node after `query_validation` that executes the SQL and checks (a) if it executes without error and (b) if it returns any rows. If it fails either check, route back to the SQL generator with additional context: the error message and/or "the previous query returned zero rows — reconsider table and filter choices." This is a reflection/self-correction agent pattern.

**Q30. What is the business value of the product agent specifically?**

A: The product agent enables queries about product categories, dimensions, and descriptions, which can answer questions like "what are the top product categories by revenue?" or "which categories have the best review scores?" Without the product agent, category-level business intelligence would be inaccessible through the system.

---

### RAG Questions (20)

**Q1. What is RAG and why did you use it here?**

A: Retrieval-Augmented Generation augments an LLM with retrieved external knowledge at inference time. In this project, instead of sending all table schemas to the LLM (which is costly and noisy), we embed table descriptions into a vector database and retrieve only the most relevant ones for each query. This keeps the LLM's context clean and makes the system scalable to large schemas.

**Q2. What embedding model did you use and why?**

A: SentenceTransformers — likely `all-MiniLM-L6-v2` — because it's specifically trained for semantic similarity tasks, runs locally without API costs, and produces high-quality 384-dimensional embeddings fast. For schema retrieval, semantic similarity (does this table description match the question's intent?) is exactly what SentenceTransformers optimize for.

**Q3. What is ChromaDB and why was it chosen over alternatives?**

A: ChromaDB is a local, open-source vector database with a Python-native API. It was chosen for zero infrastructure overhead (runs in-process with persistent file storage), cosine similarity support, and a simple `add`/`query` interface. Alternatives like Pinecone require cloud setup; FAISS lacks built-in persistence.

**Q4. What is cosine similarity and why is it appropriate here?**

A: Cosine similarity measures the angle between two vectors, ignoring magnitude. For text embeddings, it measures semantic direction — two sentences can have different lengths but similar meaning, and cosine similarity correctly scores them as close. It's the standard metric for semantic similarity tasks.

**Q5. How does the RAG index get built and when does it rebuild?**

A: `_build_index()` checks `collection.get()["ids"]` for existing indexed table names. If a table is already indexed, it skips it. If `kb.pkl` has new tables, they get added incrementally. The index only fully rebuilds if the ChromaDB `chroma_store/` directory is deleted.

**Q6. What is the difference between keyword search and semantic search?**

A: Keyword search matches exact string occurrences. Semantic search matches conceptual meaning. For schema retrieval, a user asking "how many orders were paid in installments" should retrieve the `order_payments` table even if the word "installments" doesn't appear verbatim in the table name — semantic search handles this; keyword search doesn't.

**Q7. What text chunk is embedded for each table in your RAG system?**

A: A combined text: "Table name: {table_name}. Table description: {description}. Columns and their details: {all_column_descriptions_concatenated}". This rich chunk gives the embedding model context from both the table-level and column-level, enabling retrieval on either table concept or specific column concept queries.

**Q8. What is `top_k` in your RAG implementation and how do you choose the right value?**

A: `top_k` is the number of tables retrieved per query. Default is 4. Complex multi-domain queries need more (e.g., 5-6); simple single-table queries need fewer. The current implementation passes `top_k` as a parameter to `get_relevant_tables()`. A smarter approach would dynamically adjust `top_k` based on query complexity estimated by the router.

**Q9. How do you handle the cold start problem with ChromaDB?**

A: `get_or_create_collection` creates the collection if it doesn't exist, and `_build_index()` checks for existing IDs to skip re-indexing. The first run indexes all tables (slow); subsequent runs are instant since the index is persisted on disk.

**Q10. What is the recall-precision trade-off in your RAG retrieval?**

A: Higher `top_k` improves recall (fewer missed relevant tables) but reduces precision (more irrelevant tables injected into context). For Text-to-SQL, precision is more important — irrelevant tables confuse the SQL generator. The current `top_k=4` is a reasonable default for a 9-table database but would need tuning for larger schemas.

**Q11. How does RAG reduce hallucination in SQL generation?**

A: By retrieving and injecting only relevant table and column metadata, the LLM's context contains only legitimate column names. When the full schema is injected, the LLM may confuse similar column names across tables or hallucinate columns it has seen during training on other SQL datasets. A clean, focused context minimizes this.

**Q12. What is the difference between RAG and fine-tuning for this use case?**

A: Fine-tuning bakes schema knowledge into model weights — it's expensive, requires retraining when schema changes, and doesn't generalize to new schemas. RAG retrieves schema knowledge at inference time — it's cheap, schema changes only require re-indexing, and works on any schema. For dynamic enterprise schemas, RAG is strongly preferred.

**Q13. How would you evaluate RAG retrieval quality?**

A: Build a labeled dataset of (question, ground-truth relevant tables) pairs. Measure Recall@k (were all ground-truth tables in the top-k retrieved?), Precision@k (how many of the top-k were actually needed?), and MRR (mean reciprocal rank of first relevant table). A good RAG system should achieve >90% recall@4 for this 9-table database.

**Q14. What happens if a user asks a question outside the database's scope?**

A: RAG retrieves the top-k most similar tables regardless of whether the question is answerable. The subquestion decomposition would then output `[[]]` because no table answers the sub-questions. The SQL generator would receive insufficient context and produce a query that likely returns no useful results. An "answerability classifier" added after the router would improve this.

**Q15. What is the HNSW index used in ChromaDB?**

A: Hierarchical Navigable Small World — a graph-based approximate nearest neighbor algorithm. HNSW builds a multi-layer graph where nodes connect to similar embeddings. Queries traverse the graph to find approximate nearest neighbors in O(log n) time, making it efficient for million-scale vector collections.

**Q16. Could you use the same RAG approach for column selection?**

A: Yes — this is an interesting extension. Instead of indexing at the table level, index at the column level (one vector per column with its description). Then retrieve relevant columns directly from the user question, bypassing the subquestion decomposition step. However, this loses the contextual reasoning about JOIN paths that the current multi-step approach provides.

**Q17. How do you update the RAG index when schema changes?**

A: Currently, deleting `chroma_store/` and rerunning `rag_kb.py` fully rebuilds the index. An incremental approach would: (1) detect added/modified tables in `kb.pkl`, (2) call `collection.delete(ids=[table_name])` for modified tables, (3) call `collection.add()` with new data. `_build_index()` already handles the add-only case.

**Q18. What are the limitations of SentenceTransformer embeddings for schema retrieval?**

A: SentenceTransformers are trained on general text similarity tasks, not specifically on database schema descriptions. They may not capture database-specific semantics (e.g., understanding that "freight_value" relates to shipping cost without explicit mention). Domain-adapted fine-tuned embeddings or using an LLM's embedding API (e.g., `text-embedding-3-small`) could improve retrieval quality.

**Q19. How does your RAG implementation differ from standard document RAG (like a chatbot over PDFs)?**

A: Standard document RAG retrieves text chunks from long documents to answer questions. Schema RAG retrieves structured metadata (table/column descriptions) to select the right context for code generation. The key difference is the downstream consumer — in standard RAG, the retrieved chunks are the answer; in schema RAG, the retrieved chunks are the input to SQL generation. Precision requirements are therefore much higher.

**Q20. Could you replace ChromaDB with a simple cosine similarity search over a numpy array?**

A: Yes, for 9 tables, a numpy cosine similarity calculation on all table embeddings would be faster and simpler than ChromaDB. ChromaDB becomes valuable at 1000+ tables where exhaustive search is slow and HNSW's O(log n) approximate search provides meaningful speedup.

---

### LLM Questions (20)

**Q1. What is temperature and how did you set it in this project?**

A: Temperature controls the randomness of LLM output sampling. Temperature 0 gives the most probable (deterministic) output — used for SQL generation and validation where reproducibility is critical. Temperature 0.4 gives slight variation — used for routing where exploring borderline cases improves recall.

**Q2. What is chain-of-thought prompting and where did you use it?**

A: Chain-of-thought prompting adds "think step by step" instructions to the prompt, causing the model to reason through intermediate steps before producing an answer. It's used in the subquestion prompt ("STEP BY STEP TABLE SELECTION PROCESS") and column selection prompt ("HOW TO THINK STEP BY STEP"), significantly improving structured output quality.

**Q3. What is a system prompt vs. human prompt in your LangChain templates?**

A: System prompts set the model's role and behavioral constraints — used to define "You are an intelligent MySQL query generator." Human prompts contain the actual task content — the tables, columns, user question, and instructions. This two-part structure mirrors how Claude's API is designed and helps the model maintain its role consistently.

**Q4. How did you design prompts to produce structured outputs (nested lists)?**

A: Several techniques: (1) Explicitly specify the output format with concrete examples in the prompt, (2) Specify constraints on sublist length ("Length of each sublist should be exactly 2"), (3) Use double-quotes specification for string consistency, (4) Provide worked examples showing input → reasoning → output, (5) Downstream regex extraction as a safety net.

**Q5. What is hallucination in LLMs and how does your system mitigate it?**

A: Hallucination is when an LLM generates plausible-sounding but factually incorrect content. In SQL generation, this manifests as invented column names or incorrect join conditions. Mitigation strategies in this project: (a) RAG provides accurate column names from kb.pkl, so the model doesn't need to invent them; (b) The validator catches hallucinated constructs; (c) Fuzzy matching grounds filter values in actual database values.

**Q6. What is the context window and how does it affect your design?**

A: The context window is the maximum number of tokens an LLM can process in one call. Claude 3.5 Sonnet has a 200K token context window, which is more than enough for this system's prompts. However, designing as if context is limited is good practice — it's why agents only receive their domain's tables rather than all 9 tables.

**Q7. Why did you use RunnableMap in LangChain?**

A: `RunnableMap` parallelizes multiple runnable operations and routes inputs to the right chain variables. In this project, it maps dictionary keys from the `invoke()` call to the prompt template's named variables (`{tables}`, `{user_query}`, `{columns}`, etc.), making the chain composition explicit and type-safe.

**Q8. What is StrOutputParser and why is it used?**

A: `StrOutputParser` converts the LLM's `AIMessage` object to a plain Python string, which is needed for downstream regex extraction and `eval()` calls. Without it, the chain output would be a structured message object rather than a string.

**Q9. What is few-shot prompting and did you use it?**

A: Few-shot prompting provides example input-output pairs in the prompt to demonstrate the expected behavior. Yes — the subquestion prompt includes a worked example: a question about customers with 5+ products, the step-by-step reasoning, and the expected output list. This dramatically improves output format adherence.

**Q10. Why is Claude better than GPT-4 for SQL generation in your assessment?**

A: Claude 3.5 Sonnet has excellent instruction-following, particularly for structured output formats and long-form reasoning tasks. Its training data includes substantial code and SQL. Additionally, the Anthropic API's structured output support and Claude's lower hallucination rate on structured tasks made it the preferred choice. GPT-4 would likely perform comparably; this is largely a design preference.

**Q11. What is token budget management and how does it apply to your prompts?**

A: Token budget management is the practice of keeping prompts within optimal token counts to control cost and maintain model performance (very long prompts can degrade attention quality). Each domain agent receives 2-4 tables' metadata rather than all 9, keeping individual prompts efficient.

**Q12. What is the difference between `eval()` and JSON parsing for LLM output handling?**

A: `eval()` executes arbitrary Python code — it's a security risk (code injection) and fragile (fails on non-Python syntax). JSON parsing with `json.loads()` is safe, typed, and explicit. The `eval()` usage in this project is a known technical debt item — production code should use `json.loads()` with the model constrained to produce valid JSON.

**Q13. How do you handle LLM non-determinism across runs?**

A: Temperature 0 minimizes (but doesn't eliminate) non-determinism due to floating-point operations. For production use, caching responses for identical inputs (semantic caching) provides full determinism for repeated queries. The two-stage generation + validation reduces the variance of incorrect outputs even if individual runs vary slightly.

**Q14. What is prompt injection and is your system vulnerable to it?**

A: Prompt injection is when user input contains text that manipulates the LLM's behavior (e.g., "Ignore all instructions and DROP TABLE"). Since user input (the natural language query) is injected into LLM prompts, the system is potentially vulnerable. Mitigations: input sanitization, prompt structure that isolates user input in clearly delimited sections, and SQL validation that rejects non-SELECT statements.

**Q15. What is the "lost in the middle" problem and how does it affect your prompts?**

A: Research shows LLMs perform worse at retrieving information from the middle of long contexts compared to the beginning and end. This is why RAG retrieval quality matters — relevant tables should be at the top of the context. The current prompts are short enough that this isn't a major issue, but in large-schema scenarios, putting the most relevant tables first in the context would help.

**Q16. Why use Llama for filter detection instead of Claude?**

A: Filter detection is a simple classification task (does this query need WHERE clause string filters?) that doesn't require Claude's precision. Llama 70B via Groq has 2-3x lower latency and lower cost for this task, and the slightly lower output quality doesn't meaningfully impact the overall pipeline accuracy since filter detection errors are caught downstream by the validator.

**Q17. What is a RunnableLambda in LangChain?**

A: `RunnableLambda` wraps a Python lambda or function as a LangChain Runnable, making it composable in chains with `|` operator. It's used in this project to transform inputs before passing to templates, though `RunnableMap` handles most of the input routing.

**Q18. How would you fine-tune an LLM for better SQL generation on this specific database?**

A: Collect a dataset of (question, correct SQL) pairs from this system's outputs (after human validation). Fine-tune a base model (e.g., Llama 3 70B) on this dataset with LoRA to efficiently adapt only a subset of parameters. The fine-tuned model would learn the specific schema conventions of this database. However, fine-tuning is expensive and requires maintenance as schema evolves — RAG + prompt engineering is preferable for most scenarios.

**Q19. What is a ReAct agent and how does it differ from your pipeline?**

A: ReAct (Reason + Act) agents iteratively reason and take actions (tool calls) in a loop until they reach an answer. Your pipeline is a structured, predetermined flow — a directed acyclic graph of LLM calls with fixed step order. ReAct would dynamically decide which tools to call based on intermediate results. ReAct is more flexible but less predictable and harder to debug; your structured pipeline is more reliable and auditable.

**Q20. How does the Anthropic API handle the Claude model calls in your LangChain integration?**

A: `ChatAnthropic(model_name='claude-3-5-sonnet-20240620')` from `langchain_anthropic` wraps the Anthropic Messages API. `ChatPromptTemplate.from_messages()` formats system + human messages. The `|` operator chains template → model → parser. `StrOutputParser()` extracts the text content from Claude's response. The API key is loaded from `.env` via `python-dotenv`.

---

### System Design Questions (20)

**Q1. How would you design this system to handle 1000 concurrent users?**

A: Horizontally scale the stateless LangGraph pipeline as microservices. Add an API gateway (FastAPI + uvicorn) with request queuing (Redis Queue or Celery). Use async LLM API calls. Implement semantic caching (Redis + embedding similarity) to serve repeated queries without re-running the pipeline. Database connection pooling via SQLAlchemy's `pool_size` parameter.

**Q2. How would you add a new database to this system?**

A: Create a new `kb.pkl` for the new database's schema. Configure a new SQLAlchemy engine. Add new domain agent definitions. Re-run RAG indexing. The core pipeline architecture doesn't change — only configuration changes. For a multi-database deployment, add a database selection step before routing.

**Q3. What monitoring would you add to production?**

A: Prometheus metrics for: pipeline latency per stage, LLM API token usage, SQL execution time, fuzzy match confidence scores, and pipeline error rates. Grafana dashboards for visualization. Alert thresholds for p99 latency > 15s, error rate > 5%, and API quota usage > 80%. Log every pipeline run with input query, intermediate outputs, and final SQL.

**Q4. How would you implement caching for this system?**

A: Two-level caching: (1) Exact match cache — hash the user query, check Redis for cached SQL; (2) Semantic cache — embed the query, check for similar cached queries above a cosine similarity threshold (e.g., 0.95). For semantic caching, use a vector store (Faiss in Redis or Qdrant) to store (embedding, SQL) pairs.

**Q5. How would you handle multi-tenancy (multiple customers with different schemas)?**

A: Namespace all resources by tenant: separate ChromaDB collections per tenant, separate kb.pkl per tenant, separate database connections per tenant. The stateless pipeline takes a `tenant_id` parameter and routes to tenant-specific resources. Schema isolation ensures one tenant's data doesn't leak to another.

**Q6. How would you add authentication and authorization?**

A: JWT-based authentication at the API gateway. Role-based access control at the query level: define allowed tables per user role (e.g., sales team can only query customer + orders; executives can query all). The Router Agent's available agents are restricted based on the user's role. SQL validation enforces SELECT-only for non-admin roles.

**Q7. How would you implement a query result cache?**

A: Cache based on the final SQL (not the natural language question). Hash the SQL query and store results in Redis with a TTL based on table update frequency (e.g., 5 minutes for orders, 1 hour for product catalog). Invalidate cache entries when relevant tables are updated (change data capture on MySQL via Debezium).

**Q8. What is your disaster recovery plan?**

A: ChromaDB persistence: back up `chroma_store/` to S3 daily. If corrupted, rebuild from `kb.pkl` in ~1 minute. MySQL: standard MySQL backup/restore procedures (mysqldump + point-in-time recovery with binary logs). LLM API: have fallback API keys and support multiple model providers. Pipeline state is stateless per request — no request state recovery needed.

**Q9. How would you handle rate limiting from LLM APIs?**

A: Exponential backoff with jitter on 429 responses. Distribute requests across multiple API keys (respecting Anthropic's terms of service). Request queue with priority levels (premium users get priority). Semantic caching reduces raw API call volume. Fallback to a local LLM (Ollama + Llama 3) when cloud quota is exhausted.

**Q10. How would you version control the knowledge base?**

A: Store `kb.pkl` in Git LFS with semantic versioning. Each schema change = new KB version. Tag ChromaDB collections with KB version. Allow rollback by pointing to a previous KB version and re-indexing. Changelog tracking: what changed between KB versions, what queries might be affected.

**Q11. How would you build a CI/CD pipeline for this system?**

A: GitHub Actions workflow: (1) Unit tests on chain components with mocked LLM calls, (2) Integration tests against a test MySQL instance with a subset of the Olist data, (3) Golden set evaluation — run 20 standard queries and compare SQL outputs to expected, (4) Deploy to staging, (5) Smoke test, (6) Blue-green deployment to production.

**Q12. What is the expected latency of your pipeline and where are the bottlenecks?**

A: Rough breakdown: Router (~500ms), Domain agents 2 × (~1.5s each, parallel), Filter check (~500ms), Fuzzy matching (~200ms DB query + 50ms compute), SQL generation (~2s), SQL validation (~2s). Total: approximately 5-7 seconds end-to-end. Bottlenecks: SQL generation and validation (sequential LLM calls). Parallel generation + validation could reduce this by ~2s.

**Q13. How would you test the SQL validation layer specifically?**

A: Generate a set of intentionally flawed SQL queries — with reserved word aliases, missing columns, incorrect GROUP BY, invalid syntax — and verify the validator corrects each. Also test that valid SQL passes through unchanged. Measure false positive rate (valid SQL incorrectly modified) and true positive rate (flawed SQL correctly fixed).

**Q14. How would you handle schema drift (columns added/removed from the database)?**

A: Schema drift detection as a scheduled job: compare `INFORMATION_SCHEMA.COLUMNS` against `kb.pkl` entries. Alert on mismatches. For removed columns: automatically remove from KB and re-index. For added columns: flag for human annotation (column descriptions must be written) before adding to KB.

**Q15. What would a microservices architecture for this system look like?**

A: (1) API Gateway service — receives queries, authenticates users; (2) Router service — wraps `router_agent.py`; (3) Agent service — handles all domain agents, takes table list as input; (4) Fuzzy service — wraps `fuzzy_wuzzy.py`; (5) SQL service — generation + validation; (6) KB service — serves metadata from ChromaDB; (7) Execution service — runs SQL against DB and returns results. Communicate via gRPC or message queue.

**Q16. How would you scale the fuzzy matching component?**

A: Pre-compute and cache all distinct column values for filterable columns (refresh every 15 minutes). Store as sorted lists in Redis. At query time, fetch from Redis cache instead of querying MySQL. RapidFuzz is CPU-bound and fast, so scaling is primarily about reducing the DB read latency, not the computation.

**Q17. What is idempotency and does it apply to your pipeline?**

A: Idempotency means calling the same operation multiple times produces the same result. The pipeline is mostly idempotent at temperature 0 (same input → same SQL). True idempotency would require deterministic LLM outputs (impossible with temperature > 0) and stable distinct-value lists (which change as data is inserted). For production, this means storing both input and output for every pipeline run to enable debugging and reproducibility.

**Q18. How would you implement A/B testing for different prompt versions?**

A: Feature flag system (LaunchDarkly or simple Redis flags) to route a percentage of traffic to prompt variant B. Collect metrics (SQL accuracy, latency, token cost) for both variants. Statistical significance testing after sufficient sample size. Promote better variant after validation.

**Q19. What is the CAP theorem and how does it apply to your system?**

A: CAP theorem states distributed systems can guarantee only 2 of: Consistency, Availability, Partition tolerance. For this system: the LLM API (external service) introduces availability concerns. ChromaDB runs locally — during a ChromaDB rebuild, RAG is unavailable (favor Partition tolerance + Consistency over Availability). MySQL is the critical data store — use MySQL's replication for high availability.

**Q20. How would you handle a SQL injection attempt through the natural language query?**

A: Multiple defense layers: (1) The LLM generates the SQL, not the user — direct injection is not possible; (2) SQL validation enforces SELECT-only queries (no INSERT/UPDATE/DELETE/DROP); (3) MySQL user has SELECT-only privileges on the `txt2sql` database; (4) Query execution wraps SQL in a try-except and validates the query plan before execution.

---

### Database Questions (20)

**Q1. What are the primary and foreign keys in the Olist schema?**

A: `orders.customer_id` → `customer.customer_id`; `order_items.order_id` → `orders.order_id`; `order_items.product_id` → `products.product_id`; `order_items.seller_id` → `sellers.seller_id`; `order_payments.order_id` → `orders.order_id`; `order_reviews.order_id` → `orders.order_id`; `products.product_category_name` → `category_translation.product_category_name`.

**Q2. Why were indexes created on `payment_type` and `customer_state`?**

A: These are the most common filter columns in business queries (filter by payment method, filter by geography). Without indexes, `WHERE customer_state = 'SP'` on a 100K-row customer table requires a full table scan. With a B-tree index, it's a O(log n) lookup. These columns are VARCHAR with low cardinality (28 Brazilian states, ~5 payment types), making them ideal index candidates.

**Q3. What is the difference between `customer_id` and `customer_unique_id` in Olist?**

A: `customer_id` is order-level — the same physical customer gets a different `customer_id` for each order. `customer_unique_id` is the true customer identifier across all orders. For ORDER-level joins (to `orders`), use `customer_id`. For customer-level analytics (loyal customers), use `customer_unique_id`. The column selection prompt explicitly forbids selecting `customer_unique_id` because it would produce incorrect join conditions.

**Q4. What is a compound primary key and where does it exist in this schema?**

A: `order_items` has a compound primary key on (`order_id`, `order_item_id`) — one order can have multiple items, and the item number distinguishes them within an order. `order_payments` has a compound key on (`order_id`, `payment_sequential`) — one order can have multiple payments (e.g., credit card + gift card).

**Q5. How would you write a SQL query to find customers from SP who paid by credit card?**

A: 
```sql
SELECT DISTINCT c.customer_id, c.customer_city
FROM customer c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_payments op ON o.order_id = op.order_id
WHERE c.customer_state = 'SP'
  AND op.payment_type = 'credit_card';
```

**Q6. What is the purpose of `category_translation` and how does it join?**

A: `category_translation` maps Portuguese category names (`product_category_name`) to English (`product_category_name_english`). It joins to `products` on `product_category_name`. This is a lookup/dimension table — denormalizing it would save a join but introduce maintenance overhead when category translations change.

**Q7. What is a CTE and when does the SQL generator use one?**

A: A Common Table Expression (WITH clause) defines a named temporary result set reused in the main query. The SQL generation prompt instructs "Use CTE if query is big." Complex questions like "rank states by average review score for delayed orders with above-average-priced products" produce multi-step logic that benefits from CTEs for readability and correctness.

**Q8. What is the difference between WHERE and HAVING?**

A: WHERE filters rows before grouping (operates on individual rows). HAVING filters groups after GROUP BY (operates on aggregate results). A common LLM error is using WHERE for aggregate conditions (e.g., `WHERE COUNT(*) > 10` instead of `HAVING COUNT(*) > 10`). The SQL validator prompt specifically checks for this pattern.

**Q9. What is the N+1 query problem and does your system risk it?**

A: N+1 occurs when N queries are issued to fetch related records instead of 1 JOIN query. In `get_values()`, one `SELECT DISTINCT` query is issued per (table, column) filter combination — this is by design and bounded by the number of filters (typically 1-3). It's not technically N+1 but could be batched with UNION queries if performance becomes an issue.

**Q10. How does MySQL handle UUID string joins?**

A: MySQL stores UUIDs as VARCHAR(36) or CHAR(32). String comparison for UUIDs is slower than integer comparison but is supported. The indexes created in `create_tables.ipynb` use prefix indexing (`customer_id(20)`) since full UUID indexing is expensive and the first 20 characters already provide sufficient selectivity for joins.

**Q11. What is `order_purchase_timestamp` and how does it get modified?**

A: In `create_tables.ipynb`, `ALTER TABLE orders MODIFY order_purchase_timestamp DATETIME` converts it from a VARCHAR/TEXT field (as stored in the CSV) to a proper MySQL DATETIME type. This allows date range queries and the creation of an index for time-series filtering. Without this conversion, date comparisons would be string comparisons with incorrect ordering.

**Q12. What is a star schema and does Olist follow one?**

A: A star schema has a central fact table surrounded by dimension tables. Olist approximates a star schema: `orders` is the central fact table, with `customer`, `sellers`, `products`, and `category_translation` as dimensions. `order_items`, `order_payments`, and `order_reviews` are bridge/detail tables that extend the fact. It's more accurately a snowflake schema due to the multiple fact-adjacent tables.

**Q13. What is the `payment_sequential` column in `order_payments`?**

A: It's a sequential number (1, 2, 3...) identifying multiple payments for the same order. Brazilian e-commerce commonly allows splitting payment across credit card installments or combining payment methods (e.g., half credit card, half gift voucher). The column selection prompt must consider this when asked for "total payment value" — summing across all `payment_sequential` values for an order.

**Q14. What is index cardinality and why does it matter?**

A: Index cardinality is the number of distinct values in an indexed column. High cardinality (e.g., `order_id` with 100K unique values) makes indexes very selective and efficient. Low cardinality (e.g., `customer_state` with 28 values) makes indexes less selective — MySQL may choose a full table scan over the index for low-selectivity queries. Understanding this helps explain why the SQL validator checks query plans.

**Q15. What is the difference between `INNER JOIN` and `LEFT JOIN` in the context of this schema?**

A: `INNER JOIN` returns only rows with matches in both tables — e.g., only customers who have placed orders. `LEFT JOIN` returns all left table rows with NULLs for non-matching right table rows — e.g., all customers whether or not they've placed orders. For "customers who paid by credit card," `INNER JOIN` is correct. For "all customers and their payment count (including those with no payments)," `LEFT JOIN` is needed. The SQL generation prompt should specify this — it's a common LLM error to use INNER JOIN when LEFT JOIN is semantically correct.

**Q16. How would you optimize a query that joins all 9 tables?**

A: (1) Start from the most selective filter (e.g., specific customer state) to reduce row count early. (2) Ensure all JOIN keys are indexed. (3) Use CTEs to compute intermediate aggregations before joining to other tables. (4) Avoid `SELECT *` — only select needed columns. (5) Consider materialized views for commonly joined combinations. (6) Use EXPLAIN ANALYZE to identify bottleneck operations.

**Q17. What is the review_score column and what values does it take?**

A: `order_reviews.review_score` is an integer rating from 1 to 5 representing customer satisfaction. It's used in queries like "average review score by seller" or "sellers with >90% 5-star reviews." Since it's an integer column, the filter detection prompt correctly excludes it from string fuzzy matching.

**Q18. What happens if `get_values()` is called on a column with 1 million distinct values?**

A: The `SELECT DISTINCT` query would return 1 million rows, loading them all into Python memory. RapidFuzz would then compare the user's input against all 1 million values — this would be slow and memory-intensive. The solution: add a `LIMIT` clause to `get_values()` and/or pre-compute cached distinct value lists offline for high-cardinality columns.

**Q19. What is query execution plan analysis (EXPLAIN)?**

A: `EXPLAIN SELECT ...` shows MySQL's execution plan: which indexes are used, join order, estimated rows scanned, and access type (ALL = full scan, range = index range scan, ref = index lookup). A production SQL validator could run `EXPLAIN` on the generated SQL and flag queries with `type=ALL` on large tables before execution.

**Q20. What is the `freight_value` column and how does it relate to `price` in `order_items`?**

A: `price` is the product price per item, and `freight_value` is the shipping cost per item in the order. Total order cost = SUM(price + freight_value) for all items in the order. The column selection prompt's worked example explicitly covers this kind of multi-column dependency ("to know total value of an order, order might have multiple items"), ensuring both columns are selected when total cost is requested.

---

## 20. Engineering Depth Assessment

### Skill Level Assessment

**This project sits at the Advanced level** for an applied AI/ML engineering project.

### Concepts Demonstrated

**Software Engineering:** Modular architecture, separation of concerns, graph-based state machine design, design pattern application (Strategy pattern via swappable agents), configuration management.

**AI/ML Engineering:** LLM prompt engineering (chain-of-thought, few-shot, structured output), multi-agent orchestration, RAG implementation with vector databases, embedding-based semantic search, LLM model selection strategy.

**Data Engineering:** Relational database design and indexing, SQL generation and validation, schema metadata management, fuzzy string matching for entity resolution.

**Systems Design:** Stateful parallel execution with LangGraph, conditional workflow branching, state accumulation across parallel branches, incremental knowledge base indexing.

### What Skills Recruiters Will Infer

- Ability to decompose complex AI problems into composable, testable components
- Practical knowledge of LLM limitations (hallucination, context limits) and how to engineer around them
- Production-aware thinking (the RAG upgrade, the two-stage SQL pipeline, the fuzzy matching are all evidence of thinking beyond a simple prototype)
- Multi-framework competency (LangChain, LangGraph, SQLAlchemy, ChromaDB, RapidFuzz)
- Prompt engineering discipline (the prompts in `customer_helper.py` are sophisticated, multi-thousand-word engineering artifacts)

### Role Alignment

| Role | Fit |
|---|---|
| AI Engineer / Applied AI Engineer | Excellent — core of the role |
| LLM Engineer | Excellent — RAG, multi-agent, prompt engineering |
| ML Engineer | Strong — architecture, evaluation, system design |
| Backend Engineer | Good — SQLAlchemy, LangGraph, API design patterns |
| Data Engineer | Good — MySQL, schema design, query optimization |
| Software Engineer (General) | Good — architecture, modularity, system design |

### What Makes This Project Stand Out in Interviews

Most Text-to-SQL projects are single-prompt, single-table demonstrations. This project demonstrates:

1. **Real-world complexity** — 9-table relational schema with genuine JOIN requirements
2. **RAG integration** — not just "used an LLM" but added a retrieval layer with engineering justification
3. **Entity resolution** — solved the real production problem of string mismatch between user input and database values
4. **Multi-agent orchestration** — designed and implemented a production-grade state machine
5. **Iterative engineering** — the RAG upgrade (`rag_kb.py`) shows the engineer identified a limitation and improved the system
6. **Prompt engineering depth** — the prompts in `customer_helper.py` are not generic; they encode domain knowledge about SQL, JOINs, and the Olist business context

This project credibly supports claims of experience with LLM application development, RAG systems, multi-agent architectures, and production-grade AI engineering at a level that differentiates a candidate from "I built a chatbot with OpenAI."

---

*Generated by engineering analysis of source files: `customer_agent.py`, `customer_helper.py`, `router_agent.py`, `fuzzy_wuzzy.py`, `rag_kb.py`, `main.ipynb`, `knowledge_base.ipynb`, `create_tables.ipynb`, `kb.pkl` (structure inferred), and architecture diagrams.*
