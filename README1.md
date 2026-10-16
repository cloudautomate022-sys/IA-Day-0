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
5. Select **Save connection**, then **Test connection** to send a short request and verify the endpoint, model, key, network, and CORS setup.
6. Open **Chat**, choose an agent if you have one, and send a message.

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
6. Select **Save connection** to keep the settings for this browser session, then select **Test connection** to verify them.

For a private or local model, make sure the browser can reach the host and that the endpoint accepts requests from the app's origin. If the service uses a different API format, has no compatible chat completions endpoint, or blocks CORS, it needs an adapter or a backend proxy; changing the URL alone will not make it compatible.

### OmniRoute

1. Install and start [OmniRoute](https://github.com/diegosouzapw/OmniRoute) on this computer.
2. In **Models & settings**, choose **OmniRoute**. The endpoint defaults to `http://localhost:20128/v1/chat/completions`; both the URL and model ID are editable.
3. Leave **API key** blank for a keyless local OmniRoute instance, or enter the gateway key if authentication is enabled. Select **Load OmniRoute models** to fetch `/v1/models`, then choose a listed model instead of `auto` to control routing. OpenCode-only `oc/` models are excluded from the list. Or select **Find first working model** to sequentially test direct chat models and save the first success. The scan asks for confirmation, can be cancelled, and may incur usage charges.
4. Select **Test connection** to send a short chat request. On success, the app saves the settings and shows the connected status. The browser must reach the gateway, and OmniRoute must allow requests from the app's origin.

### Microsoft Foundry

1. In **Models & settings**, choose **Microsoft Foundry**.
2. Enter the full v1 chat completions URL, for example `https://YOUR-RESOURCE-NAME.openai.azure.com/openai/v1/chat/completions` (the `services.ai.azure.com` host is also supported).
3. Enter the model deployment name and a Foundry resource API key. The app sends this key in the `api-key` header; API keys are held in memory and are not saved in browser storage.
4. Select **Test connection**. On success, the app saves the settings and shows the connected status. Browser access requires the Foundry endpoint to allow CORS from the app's origin.

## 4. Create an agent

1. In **Agent studio**, enter an **Agent name**.
2. Describe the agent's role in **What should this agent do?**.
3. Write its behavior, tone, boundaries, and response format in **Instructions**.
4. Choose **Use connected model** from **Model** to use the provider and model configured in **Models & settings**. The preset options instead select their named model IDs; use one supported by your configured provider.
5. Check the knowledge-source options the agent should be associated with. Select **Web pages** to enter http(s) URLs, one per line.
6. Select **Save agent**. Use **Open** to chat with the agent, **Edit** to change it, or **Delete** to remove it.

Web page URLs are saved with the agent and included in chat as reference links when source selections are enabled. The prototype does not fetch page contents, so the model may not be able to answer questions about a linked page unless it independently has web access. Agents are stored in the browser's local storage. They are not synced between browsers or users.

## 5. Select sources and upload files

1. Open **Data sources**.
2. Select the source options you want associated with agents. Current options include SharePoint, OneDrive, Teams, web pages, and workspace notes.
3. Select **Add files** to list local files in the current page session. Use **Remove** to remove a listed file.

**Prototype limitation:** the connector options are selection placeholders, not authenticated integrations. The app does not retrieve content from SharePoint, OneDrive, Teams, or web pages. Files uploaded under **Data sources** record only the file name and size. In **Chat**, use **Attach files** to attach text-readable files (`.txt`, `.md`, `.csv`, `.json`, common code files, and similar formats) to the next message; their contents are read in the browser and included in that model request. Chat attachments are limited to 100 KB per file and 200,000 characters total, remain in page memory, and are removed from the attachment tray after a successful send. Binary formats such as PDF and Word are not supported. Do not attach sensitive files when using an untrusted model provider. Selected connector names may be included as context hints, but they do not give the model access to underlying data. Real retrieval requires connector authorization, ingestion, and a secure backend.

## 6. Chat with an agent

1. Open **Chat** and select an agent from the menu at the top.
2. Select **Attach files** to add supported text files. Remove a file from the attachment tray with **×** before sending if needed.
3. Enter a question about the attached files and select the send button, or press Enter. Use Shift+Enter for a new line. File contents are included with that message, and the conversation history retains them in page memory.
4. Use **Clear chat** to start a fresh conversation.

The chat request includes the agent instructions and description, selected model ID, temperature, response-length setting, and (when enabled) source names. Chat history remains in page memory and is cleared when the page is refreshed.

## 7. Review token usage and estimated cost

Open **Models & settings** and review **Token usage and estimated cost**. Successful chat requests and connection tests are counted when the provider returns token usage in its response. The totals are saved in this browser. If an API does not report usage, the app marks that request as unreported instead of estimating tokens from text.

To estimate cost, enter the connected model's input and output prices in USD per million tokens. The app applies those rates to API-reported prompt and completion tokens for requests that use the connected model ID. Requests with no configured rates or a different model ID still count toward token totals, but are excluded from cost estimates. This is an estimate, not provider billing; verify prices and final charges with your provider. Select **Reset usage totals** to clear the locally saved totals.

## API key and deployment safety

This prototype stores the API key only in JavaScript memory for the current page session; it is not written to local storage and is lost on refresh. However, a key used directly in a browser can still be exposed to the person using or inspecting the page. Do not use production credentials in this prototype. For a shared or production deployment, send requests through a backend you control, keep provider keys on that server, authenticate users, and implement the required source connectors there.
