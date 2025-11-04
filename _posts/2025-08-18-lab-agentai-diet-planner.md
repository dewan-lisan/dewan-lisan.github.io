---
title: "LAB: Agentic AI Diet Planner (Work in Progress)"
date: 2025-08-18 12:00:00 +0200
categories: [Generative AI, Agentic AI]
tags: [genai, lab, agentic-ai]     # TAG names should always be lowercase
---

## Goal
- Build an Agentic AI Diet Planner that can:
  - Create a personalized diet plan based on user preferences and health goals
  - Integrate with external data sources (e.g., nutrition databases, fitness trackers)
- Deployed the application to the GCP (Cloud Run)

## Tools Used
- Streamlit
- LLM model using Gemini Pro 2.5
- Agentic AI orchestrator using Vertex AI
- BigQuery
- Cloud Run
- Cloud build and Artifact Registry

## Environment Setups
- **GCP Project**: Create a new GCP project and enable the necessary APIs (Vertex AI, BigQuery, Cloud Run)  
  ```bash
  gcloud config set project <GCP_PROJECT_ID>
  
  # Enable APIs
  gcloud services enable aiplatform.googleapis.com \
                           run.googleapis.com \
                           cloudbuild.googleapis.com \
                           cloudresourcemanager.googleapis.com \
                    bigquery.googleapis.com
  ```
- **Open Editor**: I'll be using cloud shell terminal and Editor for the development. Make sure the Cloud Code project is set in the bottom left corner of the Cloud Editor.
  ![Code Project](../assets/img/Cloud_shell_editor_code_project.png)
- **Python Environment and Dependencies**: 
  - Create a virtual environment and install the required dependencies:
    ```bash
    mkdir agent_diet_planner
    cd agent_diet_planner
    python3 -m venv .env
    source .env/bin/activate
    pip install -r requirements.txt
    pip list # check installed packages
    ```
    
  - Content of the requirements.txt file
    ```bash
    streamlit==1.33.0
    google-cloud-aiplatform
    google-cloud-bigquery
    pandas==2.2.2
    db-dtypes==1.2.0
    pyarrow==16.1.0
    ```
## Create Configuration File
The config file will be used to store all the config parameters of the project.

### Create a service account with the following roles
agentic-ai@PROJECT_ID.iam.gserviceaccount.com  

- BigQuery Admin
- Cloud Run Admin
- Cloud Run Invoker
- Vertex AI Service Agent
- Vertex AI User

### Create a new JSON type key

```bash
# secrets.toml (for Streamlit sharing)
# Store in .streamlit/secrets.toml

[gcp]
project_id = "graphical-elf-469607-p4"
location = "europe-north1"

[gcp_service_account]
type = "service_account"
project_id = "graphical-elf-469607-p4"
private_key_id = "1066220c307af47f8fd4c7f13c1f8e30ece72e9d"
private_key = '''-----BEGIN PRIVATE KEY-----
your private key here
-----END PRIVATE KEY-----'''
client_email = "agentic-ai@graphical-elf-469607-p4.iam.gserviceaccount.com"
client_id = "111747009679046921953"
auth_uri = "https://accounts.google.com/o/oauth2/auth"
token_uri = "https://oauth2.googleapis.com/token"
auth_provider_x509_cert_url = "https://www.googleapis.com/oauth2/v1/certs"
client_x509_cert_url = "https://www.googleapis.com/robot/v1/metadata/x509/agentic-ai%40graphical-elf-469607-p4.iam.gserviceaccount.com"
```

### Create BigQuery Dataset
Create a bigquery dataset called "diet_planner_data" 
```bash
bq --project=graphical-elf-469607-p4 mk diet_planner_data
```

gcloud run services add-iam-policy-binding diet-planner-service \
--region=europe-north1 \
--member="user:lisanul.dewan@seb.se" \
--role="roles/run.invoker"

### Create artifact registry repository

```bash
gcloud artifacts repositories create diet-planner-deploy \
--repository-format=docker \
--location=europe-north1 \
--description="Docker repository for diet planner deployment"
```
