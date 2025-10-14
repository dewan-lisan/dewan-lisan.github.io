---
title: "LAB: Agentic AI Diet Planner"
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
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQDZ68wr8S5DQEKO\nNxZekphYsdz5WUo/bhYoxlb/SI1WjInU4e1pOdmBaP1W6RtY29ZoYDmuVnMJL/3N\nC62hfUUdScoYy/5B4qNh78dxloc70+XqtBUjGC77yXWF95Kz3DEm7qpiR3bLZg7N\nXPafNvWE0vUeIxOUo8WavY7TZpJq+IFS3lB12nDbR/WZDif9uaybClshe9rX1Ymg\nt1+nWZ79hZLbDirqskUe38yzeIsfg0o+ZHyIaCe49AjOlLN4u96KGSlMvtVkBbwa\n3FfP5jtS2l1HbyRvjo9CjAO8u9mZJZReufHE9sVgn4sJ7EhD35T3cU1NKNX5yiOy\nL1fRP8pDAgMBAAECggEACDAue0Y0LU24UnyqaAJNHCQOwAXFXu3FgmG1eiEhQmvE\ncA3PLGGClTS7NC1NHEEiZMic0jqoVuOJP0+dhhBdbPTNVbIwiww3hGIMle2Ihkx3\neqKqmqd5eHeA5XhAeGahKCvWlhvGUG00yC2ijKf1gLimgiviO/cNYTuXJsVXYhKD\nkhN6y6Fmv3B+g3X0DOBUn0KApMb2S+A9WFW37oppxiGkW07A+S5fgZTB6V8suAtw\nU+MiHXRA0CU8PVXG3MEEJouNlk3XVt9bvw1Bpu2v8MOzVSJXFKSh+fk+ZxxO0hj1\nmr4AKcJ2Ox8Zgtzu6CaXZE/diBLXITVLWh4pwH3EWQKBgQD1iR6FRv8njYGheH7J\nWnrnw0w5ss3Jl9ObkRgDPDh34DgXQfTed/ZsZUud15W6P0wbZ6tRkLdSECV1M0z1\nr5sv9oORjJWfobu4DA8yY/b9Q0Qt67SyMCDtEnUwT6nW36/P3c66A/Kd2j0AuFsN\nmABBoi0HVYwvf29eryzoO2EmmQKBgQDjNWTXst1TgiUk/H1U4hndDk70wLFi7U3Z\natcbhVfoHEWVR/+/VH9f7cqAqVpOzMI97lN+zTyvJyof8cXS1ZA9g6hKrfm81Psz\nyNYeEsELzTGla+BD4jTb+mxYZ78cNE+X75/ot8A3yTJKE1p3bUn7weCk/hIacInO\nELsHgYstOwKBgGQZeKXhIdigKf8IPrgb+QtPZV4IdTkAerZrWpzHCkZk1Lk2nHut\n8HqUeVVqNJJvh7mMdB2WoAYGqx6ywWdQJjZRi6Xk6ILhzsPjtrZWZrUtnTgTZeFX\nGbVM1xXRBG6jVuupg8P2JA0SkdgfUI+kLkaTtUPOLo6Wp3K0e9xZiOvRAoGBAIq8\nxdD4RTGC3M+S5az5SzWyUQAe0bJImSrTlHoXmDABY3PePQpFvGmFOAwMXTqUyV8r\nsgxRomaJka1j4pn1ElidlhvT1BU8MA/U6PoAFaTxLQmHr6+D5ycT6SiqYQYF4zwx\ndAGUgmkOEAkvfCREtdJm9peJFODUKzGLAcl5jtSVAoGAd9ZduJ6n63FB5bKEOdRC\nK+20scZE47wJxvNzU5YpYELdPkrAQ7WIUR8bkbi5vPc++09yoZarQuCDVZuFaLY2\nM+DxRK+lgNd2hwfM+YiVseeNePHstM2kZpYNP/sKVwHNQWHZW6qUbNXoVTOyGgtr\n4fLBocfXDXoEfZ7NSk3DVCc=
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
