# Gemini 2.5 Flash Setup Guide

This guide documents the steps taken to successfully integrate and verify the `gemini-2.5-flash` model within the Agyn platform using LiteLLM.

## 1. LiteLLM Version Update
The default LiteLLM version in the platform's `docker-compose.yml` (`v1.80.5-stable`) encountered a `TypeError` when processing Gemini requests. Switching to the latest `main-latest` tag resolved this issue.

**Update `platform/docker-compose.yml`:**
```yaml
  litellm:
    image: ghcr.io/berriai/litellm:main-latest
```

## 2. Environment Preparation
If previous attempts have left the LiteLLM database in a corrupted state (e.g., "Unsupported provider" errors), wipe the persistent volume and restart.

**Linux/Bash:**
```bash
cd platform
docker compose stop litellm litellm-db
docker compose rm -f litellm litellm-db
docker volume rm platform_litellm_pgdata
docker compose up -d litellm-db litellm
```

**Windows/PowerShell:**
```powershell
cd platform
docker compose stop litellm litellm-db
docker compose rm -f litellm litellm-db
docker volume rm platform_litellm_pgdata
docker compose up -d litellm-db litellm
```

## 3. Registering Credentials
Add your Google AI Studio API key to the platform.
- **Provider:** `gemini`
- **Credential Name:** `gemini-studio-fresh`
- **Field:** `api_key`

**Linux/Bash (curl):**
```bash
curl -X POST http://localhost:3010/api/settings/llm/credentials \
     -H "Content-Type: application/json" \
     -d '{
       "name": "gemini-studio-fresh",
       "provider": "gemini",
       "values": { "api_key": "YOUR_API_KEY" }
     }'
```

**Windows/PowerShell:**
```powershell
$body = @{
    name = "gemini-studio-fresh"
    provider = "gemini"
    values = @{ api_key = "YOUR_API_KEY" }
} | ConvertTo-Json
Invoke-RestMethod -Uri "http://localhost:3010/api/settings/llm/credentials" -Method Post -Body $body -ContentType "application/json"
```

## 4. Adding the Model
The model string must include the `gemini/` prefix for LiteLLM to route it correctly to Google.
- **Model Name:** `gemini-2.5-flash`
- **Model String:** `gemini/gemini-2.5-flash`

**Linux/Bash (curl):**
```bash
curl -X POST http://localhost:3010/api/settings/llm/models \
     -H "Content-Type: application/json" \
     -d '{
       "name": "gemini-2.5-flash",
       "provider": "gemini",
       "model": "gemini/gemini-2.5-flash",
       "credentialName": "gemini-studio-fresh"
     }'
```

**Windows/PowerShell:**
```powershell
$body = @{
    name = "gemini-2.5-flash"
    provider = "gemini"
    model = "gemini/gemini-2.5-flash"
    credentialName = "gemini-studio-fresh"
} | ConvertTo-Json
Invoke-RestMethod -Uri "http://localhost:3010/api/settings/llm/models" -Method Post -Body $body -ContentType "application/json"
```

## 5. Verification
Verify the model by calling the platform's test endpoint or calling LiteLLM directly.

**Direct LiteLLM Test (Linux/Bash):**
```bash
curl -X POST http://localhost:4000/v1/chat/completions \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer sk-dev-master-1234" \
     -d '{
       "model": "gemini/gemini-2.5-flash",
       "messages": [{ "role": "user", "content": "Hello" }]
     }'
```

**Direct LiteLLM Test (Windows/PowerShell):**
```powershell
$body = @{
    model = "gemini/gemini-2.5-flash"
    messages = @( @{ role = "user"; content = "Hello" } )
} | ConvertTo-Json
Invoke-RestMethod -Uri "http://localhost:4000/v1/chat/completions" -Method Post -Body $body -ContentType "application/json" -Headers @{ Authorization = "Bearer sk-dev-master-1234" }
```
