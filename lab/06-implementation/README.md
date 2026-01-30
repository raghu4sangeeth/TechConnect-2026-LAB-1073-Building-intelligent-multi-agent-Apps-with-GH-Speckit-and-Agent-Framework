# Module 06 — Implementation

## Implementation Overview

In GitHub Spec Kit, the implementation phase turns the plan and tasks into working code. The goal is to produce a minimal, readable solution that demonstrates the feature while honoring every constitutional guardrail.

## Implementation Contents

- Source files updated or created during **/speckit.implement**.
- Guidance for agents, orchestrators, and FastAPI endpoints.
- Streaming and aggregation patterns, including fallback behavior.
- Run instructions, logging, and manual validation tips.
- Links to reference implementations for further study.

## Step 1: Generate the Implementation with GitHub Copilot

- [ ] In VS Code, goto the Copilot Chat panel. If not open, open it by selecting the sidebar icon or pressing Ctrl+Shift+I.
- [ ] Paste the prompt template below (it also includes **/speckit.implement**).
- [ ] Hit **Enter** in Github Copilot to generate the actual code for the application.

    ![SpecKit](./images/1.png)

### Copilot Prompt Template (Copy/Paste in Copilot)

```text
/speckit.implement
# Implement — HelloWeather Web Application

## Goal
- Produce simple Python code that loads env variables, initializes AzureOpenAIChatClient, and defines WeatherAgent and CityAgent without static data.
- Build a Concurrent workflow with a custom aggregator.
- Expose a FastAPI web UI that collects user introduction that should include city within and processes it using agents to stream the combined result.
- Create all code assets in a single folder for readability and clarity.
- Use the sample code provided below for inspiration to understand using Microsoft Agent Framework for concurrent agent instrumentation - https://github.com/microsoft/Agent-Framework-Samples/blob/main/07.Workflow/code_samples/python/03.python-agent-framework-workflow-ghmodel-concurrent.ipynb.
 
## Files to Update
- app.py — FastAPI routes, input validation, streaming response.
- agents.py — Azure client init, agent definitions, ConcurrentBuilder orchestration, aggregator helpers.
- index.html — simple form for user input plus streaming output area.
- requirements.txt - All the required dependencies such as packages required.
- README.md — Instructions to run the web app and QuickStart notes.
- .env — Consists of Environment Variables required for runtime configuration. 
 
## Dependencies & Auth
- Packages: agent-framework --pre, fastapi, uvicorn, jinja2, python-dotenv. (All the packages should be documented in requirements.txt)
- Default auth via AzureCliCredential with API key fallback.
- Environment vars: 
    AZURE_OPENAI_ENDPOINT (Endpoint url of the OpenAI service eg: https://xxxxxxxfoundry.openai.azure.com/), 
    AZURE_OPENAI_DEPLOYMENT (Model deployment name for chat eg: gpt-4.1-mini),
    AZURE_OPENAI_API_VERSION (Version of the deployed model eg: 2024-12-01-preview), 
    AZURE_OPENAI_API_KEY (optional).
 
## Agent Instructions (System Prompts)
- WeatherAgent: "Given a city name, return a 1-2 sentence approximate weather tip (likely conditions, comfort or packing guidance). Use only Azure OpenAI model knowledge. Do not use static data or external APIs. Friendly and concise."
- CityAgent: "Given a city name, return one short guidance sentence (transit, neighborhood, timing, or safety). Use only Azure OpenAI model knowledge. No static data. Friendly and concise."
 
## Orchestration & Streaming
- Use ConcurrentBuilder with both agents as participants and drive it through run_stream for streaming the orchestration.
- Implement a custom aggregator that merges agent text into a compact paragraph (≤ ~60 words), deduplicates overlap, and appends "Information is approximate—verify locally before planning."
 
## Web Frontend
- GET / renders index.html with input that includes user introduction with city and a disclaimer banner.
- POST / predict validates inputs, launches the concurrent agent workflow, and streams status events and the final user response message (SSE or chunked response).
 
## Resilience & Observability
- Add timeouts and a single retry per agent; if one fails, return the other agent’s output with a polite note.
- Sanitize inputs, cap output length, avoid medical/legal/political advice.
- stream all events status events, per-agent latency, errors, warnings and aggregation outcomes.
 
## Run Instructions
- Implement all the requested user stories without fail including replacing the placeholder code blocks.
- Include inline comments explaining client initialization, agent creation, concurrency wiring, streaming loop, and aggregation.
- Provide steps to run locally: `uvicorn app:app --reload` and other important information required in the readme.md.
- Implement and run unit tests to validate the User Input, client initialization, agent creation, concurrency wiring, streaming loop, and aggregation.
- Mention any quick manual tests (valid intro + city, missing city, simulated agent failure).
 
## Style Notes
- Keep functions short, simple, explicit, and well commented for the users.
- Avoid unused abstractions or premature optimizations; focus on simpicity and clarity.
```

You will need to select "Keep," "Allow," or "Continue" in the chat window for copilot to work.

## Step 2: After Generating the Implementation

### Step 2.1: Output Check

- [ ] Confirm the following files exist and align with the expectations below:
  - [ ] app.py — FastAPI entrypoint with GET/POST routes and streaming response helpers; verify the POST handler validates inputs and streams the aggregator output.
  - [ ] agents.py — Agent-framework bootstrap plus ConcurrentBuilder wiring; inspect aggregation logic and fallback handling for failures.
  - [ ] templates/index.html — Workshop-friendly HTML form with intro and city fields; check the streaming output container renders incremental updates.
  - [ ] .env or .env.example — Contains the Azure OpenAI settings; ensure no secrets are committed and that placeholder values match your deployment names.
  - [ ] README.md (project root) — Updated quickstart mirroring the configuration and run steps in this phase.
  
      ![SpecKit](./images/2.png)

### Step 2.2: Execute the application

*Perform below 3 instructions if only venv is not available.*

- [ ] (optional) In VS Code, choose Terminal > New Terminal or press Ctrl+` to open an integrated shell.
- [ ] (optional) Activate the virtual environment if needed: `.\.venv\Scripts\Activate.ps1` (PowerShell).
- [ ] (optional) Install dependencies: `pip install -r requirements.txt`.

> [!CAUTION]
> Please note that you may see occasionally compilation errors or execution errors as code is generated by AI. Read the [Guidance](#human-in-the-loop-disclaimer).

- [ ] Create or modify the .env file to update using your openai deployment details as needed. To update the **.env** file, follow the below instructions -
  - [ ] In the Azure portal, search for **Foundry**.
  - [ ] Select **Microsoft Foundry**, and navigate to specific resource under **Reources**. See the image below.

    ![SpecKit](./images/5.png)
  
  - [ ] Once the resource page opens - Select **Go to Foundry portal**
  - [ ] In the Foundry portal, Go to **Overview -> Endpoints and Keys -> Libraries -> Azure OpenAI**.
  - [ ] Copy the **Azure OpenAI endpoint** url as displayed below.

    ![SpecKit](./images/6.png)

  - [ ] Update the **.env** file with the endpoint url.
  - [ ] Update the Model name and version with the following.
    - [ ] AZURE_OPENAI_DEPLOYMENT: `gpt-4.1-mini`
    - [ ] AZURE_OPENAI_API_VERSION: `2024-12-01-preview`
  - You may verify these in the foundry portal in **Sidebar ->  My Assets -> Models + Endpoints** section.

    ![SpecKit](./images/3.png)

- [ ] Run the app: `.venv\Scripts\uvicorn app:app --log-level debug`.
- [ ] Open <http://127.0.0.1:8000> in a browser, enter a friendly intro and a city, then submit; confirm streamed content appears and ends with the disclaimer. Example - *I am Dennis from Austin, Texas, USA.*

    ![SpecKit](./images/4.png)

- [ ] The interface you see depends entirely on the choices your agent makes! — so if it looks different, this is expected - proceed forward.
- [ ] You’re expected to encounter a few application errors along the way — just pass them to GitHub Copilot and let it fix them for you. Read the [Guidance](#human-in-the-loop-disclaimer).

### Step 2.3: Validations

*Fully optional, only do it if you are a code freak.*

- [ ] Test validation by submitting an empty city and confirm the form shows a helpful error. Example - *I am Dennis.*
- [ ] Trigger a degraded path by raising an exception inside one agent (temporary change) and verify the other agent’s message plus polite fallback displays. *This requires code change, so we will leave you to your imagination*

### Step 2.4: Exit

- [ ] Return to the terminal and press Ctrl+C to stop Uvicorn when you finish.

## Human-in-the-Loop Disclaimer

- Treat every AI-generated artifact as a draft: read it, edit it, and run tests before trusting it anywhere near production.
- **If something breaks, recruit GitHub Copilot in chat to triage and suggest fixes before paging a proctor or instructor—after all, Copilot wrote the code, so it should fix its own mess faster than it can explain itself to a human.**
- Keep responsibility squarely with you: review security, compliance, and data handling, and document any manual changes you make.

## Congratulations

!!! Congratulations! Labs are now complete !!!

## Appendix - Sample Implementation Reference (Do Not Copy)

- FastAPI entrypoint and streaming route in [hello-weather-lab/07-WorkingCode/app.py](hello-weather-lab/07-WorkingCode/app.py).
- Agent definitions, concurrent workflow, and aggregator helpers in [hello-weather-lab/07-WorkingCode/agents.py](hello-weather-lab/07-WorkingCode/agents.py).
- Workshop-friendly HTML form and streamed output area in [hello-weather-lab/07-WorkingCode/templates/index.html](hello-weather-lab/07-WorkingCode/templates/index.html).
- Environment configuration sample in [hello-weather-lab/07-WorkingCode/.env](hello-weather-lab/07-WorkingCode/.env) (replace with your own secrets).
