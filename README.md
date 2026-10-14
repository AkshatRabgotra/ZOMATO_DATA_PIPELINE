# Zomato Data Pipeline

This repository contains an end-to-end modern data engineering and AI pipeline built for processing, analyzing, and interacting with Zomato dataset records. It leverages a Medallion architecture on Snowflake, transforms data via dbt, orchestrates workflows with Apache Airflow, and integrates OpenAI capabilities for advanced data enrichment and querying.

## Architecture Overview

![Zomato Data Pipeline Architecture](./image_305664.jpg)

The pipeline processes data through a structured Medallion architecture, enhancing it with a dedicated AI lane before serving it to end-users.

### Core Data Flow
* **Source:** The raw Zomato Dataset consists of 7 CSV folders containing data on country, users, restaurants, orders, order items, menus, and reviews.
* **Lake (S3):** Raw CSVs are uploaded to an Amazon S3 bucket.
* **Bronze (Snowflake RAW):** Data is ingested into Snowflake using a `COPY` command via storage integration.
* **Silver (STAGING):** Uses dbt to clean, type-cast, and join data into views. The staging models include `stg_food.sql`, `stg_menu.sql`, `stg_orders.sql`, `stg_restaurants.sql`, `stg_reviews.sql`, and `stg_users.sql`.
* **Gold (MARTS):** Uses dbt to build dimensions, incremental models, and facts. Key models include `fact_orders.sql`, `dim_customer.sql`, and aggregate performance marts like `mart_daily_city_revenue.sql` and `mart_delivery_sla.sql`.
* **Serve:** The final Gold layer is served via Snowsight and Streamlit for BI dashboards and self-serve applications.

### AI Capabilities (The AI Lane)
The pipeline integrates OpenAI models (GPT-4o mini, text-embedding-3-small) to provide three distinct AI capabilities:
1. **LLM Enrichment:** Uses an LLM as a transform step (`enrich_reviews.py`) to parse `stg_reviews` and generate JSON summaries, outputting to `AI.REVIEW_ENRICHED` and subsequently `mart_review_insights`.
2. **RAG (Retrieval-Augmented Generation):** Embeds review vectors into a vector store, allowing users to chat with their reviews through grounded answers and sources via `rag_chat.py`.
3. **Text-to-SQL:** Translates natural language to SQL queries against the MARTS schema using `text_sql.py`. This is protected by a SELECT-only guard and executes under a restricted `DBT_ROLE`.

### Orchestration
Apache Airflow orchestrates the workflow utilizing a DAG named `zomato_batch.py`. The execution sequence is:
1. `upload_raw`: Moves data from S3 to RAW.
2. `dbt_build_core`: Transforms RAW to STAGING.
3. `enrich_reviews`: Executes the AI enrichment step.
4. `dbt_build_all`: Builds the final AI and MARTS models.

## Repository Structure

* **/ai**: Contains Python scripts for the AI lane, including `enrich_reviews.py`, `rag_chat.py`, and `text_sql.py`.
* **/airflow**: Houses orchestration configurations, including the `Dockerfile`, `docker-compose.yaml`, and the `dags/zomato_batch.py` DAG definition.
* **/zomato**: The core dbt project directory containing `dbt_project.yml` and the transformation models.
  * `/models/staging`: Silver layer models (e.g., `stg_orders.sql`, `stg_reviews.sql`).
  * `/models/marts`: Gold layer facts and dimensions (e.g., `fact_order_items.sql`, `dim_restaurants.sql`, `mart_restaurant_performance.sql`).

## Tech Stack
* **Languages & Libraries:** Python, Pandas
* **Data Warehouse:** Snowflake
* **Transformation:** dbt
* **Orchestration:** Apache Airflow
* **AI / LLM:** OpenAI
* **Serving / UI:** Streamlit
