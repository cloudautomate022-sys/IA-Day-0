# Fieldnote AI Agent Studio

A browser-based prototype for defining agents, choosing a model endpoint, selecting knowledge-source options, and sending chat requests to an OpenAI-compatible chat completions API.

See [ARCHITECTURE.md](ARCHITECTURE.md) for a graphical view of the current prototype and a proposed production design.

## 1. Open the app

1. Open `AI/index.html` in a browser.
2. The Agent studio page is the starting screen. Agents and source selections are saved in this browser's local storage.
3. For API calls, serve the page from a local web server or your application host if the provider blocks requests from `file://`. The API must allow browser CORS from the page's origin.

For a quick local preview, open PowerShell in the `AI` folder and run:

```powershell
py -m http.server 8000
```

Then browse to `http://localhost:8000`. Stop the server with `Ctrl+C` when finished.

## 2. Connect an OpenAI model

1. Open **Models & settings** from the left navigation.
2. Set **Provider** to **OpenAI**.
3. Enter a model ID that your OpenAI API account can use, such as `gpt-4o-mini`.
4. Paste your OpenAI API key into **API key**.
5. Select **Save & test**. This saves the endpoint and model; it does not send a test request yet.
6. Open **Chat**, choose an agent if you have one, and send a message. That first message makes the API request and verifies that the endpoint, model, key, network, and CORS setup work.

API usage is billed separately from ChatGPT subscriptions and may incur usage charges. Check OpenAI's [API pricing](https://openai.com/api/pricing/) and the model's availability in your API account before use.

## 3. Connect another or private model API

The app sends requests using the OpenAI-compatible Chat Completions format. Your provider or private model gateway must accept a request with:

- `POST` to a chat completions endpoint
- JSON containing `model`, `messages`, `temperature`, and `max_tokens`
- A bearer token in the `Authorization` header
- A response containing `choices[0].message.content`

To configure it:

1. Open **Models & settings**.
2. Set **Provider** to **Custom OpenAI-compatible API**.
3. Enter the full chat completions URL, for example `https://your-host.example/v1/chat/completions` or the endpoint supplied by your model host.
4. Enter the exact model ID expected by that endpoint.
5. Enter the API key or token expected by the endpoint. The current form requires a non-empty value.
6. Select **Save & test**, then send a message in **Chat** to test it.

For a private or local model, make sure the browser can reach the host and that the endpoint accepts requests from the app's origin. If the service uses a different API format, has no compatible chat completions endpoint, or blocks CORS, it needs an adapter or a backend proxy; changing the URL alone will not make it compatible.

## 4. Create an agent

1. In **Agent studio**, enter an **Agent name**.
2. Describe the agent's role in **What should this agent do?**.
3. Write its behavior, tone, boundaries, and response format in **Instructions**.
4. Choose a model from **Model**. The workspace model uses the configured API model. The preset options select their named model IDs; use a model your configured provider actually supports. **Custom API** uses the model ID configured under Models & settings.
5. Check the knowledge-source options the agent should be associated with.
6. Select **Save agent**. Use **Open** to chat with the agent, **Edit** to change it, or **Delete** to remove it.

Agents are stored in the browser's local storage. They are not synced between browsers or users.

## 5. Select sources and upload files

1. Open **Data sources**.
2. Select the source options you want associated with agents. Current options include SharePoint, OneDrive, Teams, web pages, and workspace notes.
3. Select **Add files** to list local files in the current page session. Use **Remove** to remove a listed file.

**Prototype limitation:** the connector options are selection placeholders, not authenticated integrations. The app does not retrieve content from SharePoint, OneDrive, Teams, or web pages. File uploads currently record only the file name and size; file contents are not read, indexed, or sent to the model. For now, selected source names may be included in the prompt as context hints, but they do not give the model access to the underlying data. Real retrieval requires connector authorization, ingestion, and a secure backend.

## 6. Chat with an agent

1. Open **Chat** and select an agent from the menu at the top.
2. Enter a question and select the send button, or press Enter. Use Shift+Enter for a new line.
3. Use **Clear chat** to start a fresh conversation.

The chat request includes the agent instructions and description, selected model ID, temperature, response-length setting, and (when enabled) source names. Chat history remains in page memory and is cleared when the page is refreshed.

## API key and deployment safety

This prototype stores the API key only in JavaScript memory for the current page session; it is not written to local storage and is lost on refresh. However, a key used directly in a browser can still be exposed to the person using or inspecting the page. Do not use production credentials in this prototype. For a shared or production deployment, send requests through a backend you control, keep provider keys on that server, authenticate users, and implement the required source connectors there.
