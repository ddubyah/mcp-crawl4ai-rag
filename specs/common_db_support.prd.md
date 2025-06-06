# Supporting Optional Supabase or Standard Postgres

## Introduction/Overview
This feature enables the MCP server to connect either to Supabase or to a standard Postgres database, such as an AWS RDS instance, while keeping backward compatibility with the existing Supabase setup. The goal is to make database choice configurable and ensure all crawling and retrieval features work with both options.

## Goals
- Allow deployment with either Supabase or a plain Postgres server.
- Maintain existing functionality when Supabase is used.
- Provide clear setup instructions for connecting to an RDS‑hosted Postgres database with pgvector.

## User Stories
- **As a maintainer**, I want to specify Postgres credentials in the `.env` file so the server can run without Supabase.
- **As a developer**, I can switch between Supabase and Postgres via a configuration flag, keeping the rest of the code the same.

## Functional Requirements
1. **Environment Configuration**
   - Add a flag (e.g., `USE_SUPABASE=true/false`) or detect a `DATABASE_URL` to choose the database backend.
   - Extend `.env.example` with variables for Postgres (`DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`).
2. **Database Client Layer**
   - Implement `get_postgres_client()` using the `psycopg[binary]` driver.
   - Update `crawl4ai_lifespan` to initialize either a Supabase client or a Postgres connection based on configuration.
3. **Data Access Helpers**
   - Refactor functions in `src/utils.py` (`add_documents_to_supabase`, `search_documents`, `add_code_examples_to_supabase`, etc.) so they execute SQL directly when a Postgres connection is used.
   - Reuse existing Supabase logic when `USE_SUPABASE` is true.
4. **SQL Setup**
   - Ensure `crawled_pages.sql` can be executed on a standard Postgres server. Document using `psql` to run the script.
   - Include instructions for enabling the `pgvector` extension on RDS before running the script.
5. **Documentation Updates**
   - Update README to describe both database options and how to configure each one.
   - Provide example `.env` values for connecting to an RDS instance.
6. **Dependencies**
   - Add `psycopg[binary]` to `pyproject.toml` and install it in the Dockerfile/setup steps.

## Non-Goals
- Converting the project to an asynchronous Postgres driver.
- Supporting databases other than Supabase or Postgres with pgvector.

## Design Considerations
- Keep existing Supabase RPC functions for backward compatibility.
- Expose a small wrapper layer so future extensions (e.g., async drivers) can be added with minimal changes.

## Technical Considerations
- `pgvector` must be installed on the target Postgres server. AWS RDS allows this via the `pgvector` extension.
- Ensure SQL queries replicate the behavior of current Supabase RPC functions (`match_crawled_pages` and `match_code_examples`).

## Success Metrics
- The server starts and performs crawling, document insertion, and search operations when connected to an RDS Postgres instance.
- Existing Supabase workflow continues to operate without changes.

## Open Questions
- Should the SQL functions (`match_crawled_pages`, `match_code_examples`) remain in the database or be implemented in Python for the Postgres path?
- Any additional security or network configuration needed for RDS connectivity?
