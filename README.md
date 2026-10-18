# Logistics Management Agent

An AI-powered logistics and fleet dispatch assistant that combines operational fleet data, logistics safety procedures, and contextual tools to help dispatchers make informed decisions.

## Overview

Managing logistics requires continuous monitoring of shipments, fleet operations, and temperature-sensitive transportation procedures. Finding the right operational information quickly can be challenging when data is spread across databases and policy documents.

The **Logistics Management Agent** addresses this challenge by combining a conversational AI agent with structured logistics data and unstructured Standard Operating Procedures (SOPs).

The application allows users to ask natural-language questions about fleet operations and cold-chain incident procedures. The AI agent can use tools to retrieve operational information from SQL Server, look up weather information, and search relevant SOP documents through a vector database.

The project is built using Python 3.12, Streamlit, LangGraph, SQL Server, and Pinecone.

## Key Features

- **Conversational Logistics Assistant:** Ask operational questions using natural language.
- **Fleet Data Access:** Retrieve relevant fleet and shipment information from a SQL Server database.
- **Controlled Database Access:** Use a dedicated read-only database account and restricted SQL views for agent queries.
- **SOP Retrieval:** Search cold-chain policies and incident procedures using Pinecone vector search.
- **Weather Lookup:** Retrieve weather information through the Open-Meteo service.
- **Tool-Calling AI Agent:** Use LangGraph to coordinate the language model and available tools.
- **Interactive Dispatch Console:** Access the assistant through a Streamlit web interface.
- **Audit Logging Support:** Support logging of assistant activity for operational review, depending on the configured database schema.
- **Deployment Workflow:** Include a GitHub Actions workflow for an EC2 deployment process.

## System Architecture

The application connects structured operational data with policy documents and external information sources.

```text
                 USER
                   |
                   v
        Streamlit Dispatch Console
                 src/ui.py
                   |
                   v
          LangGraph Orchestrator
             src/orchestrator.py
                   |
                   v
             AI Agent Tools
             src/agent_tools.py
                   |
        +----------+----------+
        |          |          |
        v          v          v
     SQL Server  Pinecone   Open-Meteo
     Fleet Data  SOP Search Weather API
        |          |          |
        +----------+----------+
                   |
                   v
       Contextual AI Response
                   |
                   v
          Dispatch Console
```

### Architecture Components

| Component | Responsibility |
|---|---|
| Streamlit | Provides the user interface for dispatchers. |
| LangGraph | Coordinates the agent workflow and tool execution. |
| Language model | Interprets user questions and generates responses. |
| SQL Server | Stores structured logistics and fleet data. |
| Pinecone | Provides vector search over indexed SOP documents. |
| Open-Meteo | Supplies weather information through an external API. |
| GitHub Actions | Automates the configured deployment workflow. |

## Technology Stack

| Category | Technologies |
|---|---|
| Language | Python |
| User interface | Streamlit |
| AI orchestration | LangGraph |
| Language models | Configurable provider, depending on implementation |
| Relational database | Microsoft SQL Server |
| Vector database | Pinecone |
| Weather service | Open-Meteo |
| Data ingestion | Python, Pandas and SQL tooling as configured |
| Deployment | AWS EC2, systemd and GitHub Actions |
| Database container | Docker (optional, for SQL Server) |
| Database security | Restricted SQL views and read-only database access |

## Repository Structure

```text
logistic_management_assistant/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── data/
│   ├── cache/
│   │   └── ingestion_hash_cache.json
│   ├── policy/
│   │   └── Cold_Chain_Incident_SOP_v2.md
│   ├── raw/
│   │   └── dynamic_supply_chain_logistics_dataset.csv
│   └── source/
│       └── data.txt
├── scripts/
│   ├── ingest_legacy_data.py
│   ├── ingest_sop_pinecone.py
│   └── setup_security_and_view.sql
├── src/
│   ├── agent_tools.py
│   ├── orchestrator.py
│   ├── prompts/
│   │   └── system_prompt.txt
│   └── ui.py
├── docs/
│   └── instrutions.md
├── .gitignore
├── requirements.txt
└── README.md
```

### Important Files

- `src/ui.py` — Streamlit interface for interacting with the assistant.
- `src/orchestrator.py` — Coordinates the AI agent and its tools.
- `src/agent_tools.py` — Contains integrations for operational queries, weather lookup, and SOP retrieval.
- `src/prompts/system_prompt.txt` — Defines the agent's instructions and behavioral guidance.
- `scripts/ingest_legacy_data.py` — Loads the logistics dataset into the expected legacy SQL schema.
- `scripts/ingest_sop_pinecone.py` — Processes SOP content for vector search.
- `scripts/setup_security_and_view.sql` — Sets up restricted database access and the fleet view.
- `data/raw/` — Contains the supplied logistics dataset.
- `data/policy/` — Contains cold-chain policy and incident-procedure documentation.
- `docs/instrutions.md` — Detailed operational notes for local database setup and EC2 deployment.
- `.github/workflows/deploy.yml` — Contains the configured EC2 deployment workflow.

## How It Works

### 1. Fleet Data Ingestion

The logistics dataset is loaded into SQL Server using the data ingestion script. The script maps dataset columns to the database schema expected by the application.

### 2. Database Security Configuration

A SQL setup script creates a restricted view for fleet queries and configures a read-only agent account. This helps limit the data the AI agent can access.

### 3. SOP Indexing

The SOP ingestion script processes cold-chain policy documents and indexes their content in Pinecone for semantic retrieval.

### 4. AI Agent Orchestration

The LangGraph-based orchestrator coordinates the language model and the available tools. Depending on the question, the agent can retrieve fleet information, consult SOP documents, or look up weather information.

### 5. Response Generation

The language model uses the retrieved information to formulate a response relevant to the dispatcher's question.

### 6. User Interface

The Streamlit application provides a conversational interface through which users can ask questions and review operational information.

## Prerequisites

Before running the project, prepare the following:

- Python 3.12 (the pinned dependency set was prepared for Python 3.12.13).
- Git.
- ODBC Driver 18 for SQL Server and the `unixODBC` development package on Linux.
- Docker, if you intend to run SQL Server in a container.
- A Microsoft SQL Server instance.
- A Pinecone account and API key for SOP retrieval.
- Either Ollama running locally, or credentials for OpenAI or DeepSeek.
- An internet connection for the configured external services.

AWS credentials and deployment configuration are only necessary if you intend to deploy the application to EC2. The application also requires a Pinecone API key at startup, even when the SQL or language-model integration is being tested.

## Installation and Setup

### Step 1: Clone the Repository

```bash
git clone https://github.com/JetJerry/logistic_management_assistant.git
cd logistic_management_assistant
```

### Step 2: Create a Virtual Environment

On Windows:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

On Linux or macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### Step 3: Install Dependencies

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Confirm that installation completes successfully before proceeding.

### Step 4: Configure Environment Variables

Create a local `.env` file in the repository root. The application loads these names directly from that file:

```dotenv
# Language model configuration
Agent_llm=OLLAMA
OPENAI_API_KEY=
DEEPSEEK_API_KEY=

# Embedding configuration: LOCAL or OPENAI
Embeddings_model=LOCAL
Local_Embedding_Model=BAAI/bge-m3

# Pinecone configuration (required)
PINECONE_API_KEY=

# SQL Server configuration
SQL_SERVER_HOST=localhost
SQL_SERVER_PORT=1433
SQL_ADMIN_USER=sa
SQL_ADMIN_PASSWORD=
SQL_AGENT_USER=USR_FDE_RO
SQL_AGENT_PASSWORD=
```

Use `Agent_llm=OLLAMA` for the local default, `Agent_llm=OPENAI` for OpenAI, or `Agent_llm=DEEPSEEK` for DeepSeek. If `Embeddings_model=OPENAI`, configure the OpenAI key and use the OpenAI Pinecone index; otherwise the default local `BAAI/bge-m3` model and local Pinecone index are used.

The ingestion script uses the administrator variables. The application tools and audit logging use the agent variables. `SQL_AGENT_USER` must be able to read `FDE_VIEWS.VW_ACTIVE_FLEET` and insert into `FDE_VIEWS.AgentAuditLog`.

**Security requirements:**

- Never commit `.env` or real API keys to GitHub.
- Never publish production database credentials.
- Use placeholders in documentation and examples.
- Confirm that `.gitignore` excludes secrets and local environment files.

When using Ollama, install and start Ollama separately and make the configured `qwen2.5:7b` model available locally:

```bash
ollama pull qwen2.5:7b
```

## Database Setup

### Step 1: Start SQL Server

Run SQL Server locally or connect to an existing SQL Server instance.

If you use Docker, configure the SQL Server image, persistent storage, environment variables, and port mappings according to Microsoft's documentation.

Do not expose database port `1433` publicly. Restrict access to trusted networks and authorized application hosts.

### Step 2: Ingest the Logistics Dataset

After configuring database connectivity and setting `SQL_ADMIN_USER` and `SQL_ADMIN_PASSWORD`, run:

```bash
python scripts/ingest_legacy_data.py
```

This script is intended to load the logistics dataset into the schema expected by the application.

Before executing it, confirm that:

- The dataset is present in `data/raw/`.
- The database is running and reachable.
- The required database and table configuration is correct.
- The configured database user has the necessary permissions.

### Step 3: Configure Restricted Database Access

Execute the SQL script in an appropriately privileged SQL Server session:

```text
scripts/setup_security_and_view.sql
```

Review the script before running it. Confirm the database names, login configuration, permissions, and view definitions are suitable for your environment.

The script creates `FDE_VIEWS.VW_ACTIVE_FLEET` and the `USR_FDE_RO` login. Review the hard-coded login password in the SQL script before running it and replace it with a securely managed value for non-development environments.

### Step 4: Create the audit table

The Streamlit audit view expects `FDE_VIEWS.AgentAuditLog` to exist. Run this statement in an appropriately privileged SQL Server session after the security script:

```sql
CREATE TABLE FDE_VIEWS.AgentAuditLog (
    LogID INT IDENTITY(1,1) PRIMARY KEY,
    Timestamp DATETIME DEFAULT GETDATE(),
    SessionID VARCHAR(50),
    NodeExecuted VARCHAR(50),
    ToolName VARCHAR(100),
    Content NVARCHAR(MAX)
);

GRANT INSERT ON FDE_VIEWS.AgentAuditLog TO USR_FDE_RO;
```

## Configure SOP Retrieval

The project uses Pinecone to search indexed cold-chain policy documents.

1. Create or select a Pinecone index that matches the application's configuration.
2. Configure the Pinecone API key securely.
3. Confirm that the embedding model and vector dimensions match the index.
4. Run the ingestion script:

```bash
python scripts/ingest_sop_pinecone.py
```

5. Check the script output and confirm that the documents have been indexed successfully.

The ingestion process depends on the Pinecone configuration, embedding provider, and any additional settings required by the implementation.

## Run the Application

Once the dependencies, database, environment variables, language model, and SOP index are configured, launch the Streamlit application:

```bash
streamlit run src/ui.py
```

Open the local URL printed in the terminal by Streamlit, usually:

```text
http://localhost:8501
```

Keep the terminal session running while using the application.

## Example Questions

The following are illustrative questions to try after the corresponding data sources and tools have been configured:

**Fleet operations**
- Which active shipments are near Los Angeles?
- What fleet information is available for a particular shipment?
- Which operational records match a given location or status?

**Cold-chain policies**
- What does the incident SOP recommend when a temperature excursion occurs?
- What procedures should be followed when a shipment is delayed?

**Weather**
- What is the current weather for a specified location?
- How might the retrieved weather conditions affect a planned shipment?

The accuracy and availability of answers depend on the underlying dataset, language model, tool implementation, and external service responses.

## Deployment

The repository includes a manually triggered GitHub Actions workflow at:

```text
.github/workflows/deploy.yml
```

The workflow is intended for EC2 deployment. It expects the GitHub environment secrets `EC2_HOST`, `EC2_USER`, and `EC2_SSH_KEY`, and restarts an existing `streamlit` systemd service. It does not provision EC2, install dependencies, create SQL Server, or create the systemd service automatically.

A typical deployment process involves:

1. Provisioning and securing an EC2 instance.
2. Installing Python, the ODBC driver, and the system dependencies.
3. Cloning the repository and creating the virtual environment.
4. Configuring SQL Server connectivity and external service credentials.
5. Creating and enabling a `streamlit` systemd service.
6. Configuring the GitHub environment secrets.
7. Manually dispatching the workflow and checking application logs, connectivity, and health.

The precise steps depend on the workflow implementation and the target EC2 environment. Avoid committing private SSH keys, cloud credentials, or production secrets.

## Security Considerations

Because the assistant interacts with operational data and external services, security should be part of the deployment process.

- **Least privilege:** Use restricted database accounts for AI-agent queries.
- **Read-only access:** Prefer read-only database permissions for operational retrieval.
- **Query safety:** Validate or constrain SQL operations in accordance with the application's design.
- **Secret management:** Keep API keys, database passwords, and deployment credentials outside source control.
- **Network security:** Restrict database and application access using appropriate firewall and cloud security-group rules.
- **Auditability:** Review the configured audit logging and ensure sensitive data is handled appropriately.
- **Input validation:** Treat user questions and retrieved documents as untrusted input.
- **Policy verification:** Do not rely on an AI-generated answer alone for critical safety or compliance decisions; consult the applicable approved SOP.

## Troubleshooting

| Issue | What to check |
|---|---|
| SQL Server connection fails | Verify the server address, port, credentials, database availability, and ODBC driver. |
| Dataset ingestion fails | Check the CSV path, schema mapping, database permissions, and required tables. |
| Pinecone connection fails | Verify the API key, index name, embedding configuration, and network connectivity. |
| SOP search returns no results | Confirm that ingestion succeeded and that the query uses the expected index and namespace. |
| Language model requests fail | Check the configured provider, credentials, model availability, and usage limits. |
| Streamlit fails to start | Confirm that dependencies are installed and the Python environment is active. |
| Deployment fails | Inspect GitHub Actions logs, EC2 access, the configured remote path in `deploy.yml`, environment variables, and the existing `streamlit` systemd service. |

## Future Improvements

Potential improvements include:

- Shipment temperature anomaly detection.
- Real-time fleet and sensor integrations.
- Predictive shipment-delay analysis.
- Automated alerts for critical cold-chain incidents.
- More comprehensive agent evaluation and tool-use tests.
- Role-based access controls in the user interface.
- Improved audit trails and observability.
- Automated testing and deployment checks.

These are possible future enhancements and should not be interpreted as features already implemented.

