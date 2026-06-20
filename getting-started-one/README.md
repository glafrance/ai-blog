# Setting Up Your Modern TypeScript AI Workspace

A clean, modern TypeScript workspace for testing the **OpenAI**, **Google Gemini**, and **Anthropic Claude** APIs with simple one-file quickstart scripts.

This guide walks through:

- Creating a Node.js + TypeScript project
- Configuring modern ESM modules
- Installing each provider SDK
- Setting API keys as environment variables
- Running separate quickstart scripts for OpenAI, Gemini, and Claude
- Understanding how each API structures requests and responses

---

## 💰 Account Requirements & Cost Advisory

Before running any code in this workspace, create developer accounts and configure billing for each AI provider.

Because platform URLs can change over time, the simplest way to find the correct onboarding portals is to search for these dashboards:

| Provider | What to Search For | Purpose |
|---|---|---|
| **OpenAI** | `OpenAI API Platform Dashboard` | Register a developer account and secure pre-funded API credits. |
| **Google Gemini** | `Google AI Studio` | Initialize a Gemini developer workspace. Gemini may offer a free tier for prototyping, but rate limits apply unless connected to Google Cloud billing. |
| **Anthropic Claude** | `Anthropic Console` | Establish a developer organization and purchase pre-funded API credits. |

---

## ⚠️ Cost Management Best Practices

Every programmatic request sends tokens to a remote AI service. Those requests can create real costs billed to your API accounts.

Before running automation, protect yourself with these safeguards:

### Set Hard Usage Caps

Log into each provider's billing dashboard and set aggressive daily or monthly budget limits, such as **$5.00**.

This can help prevent runaway scripts from draining credits or creating unexpected bills.

### Avoid Production Models for Sandbox Loops

For simple tests, debugging, or CI loops, use cheaper, faster models when possible.

For example:

```text
gemini-3.5-flash
```

Save larger enterprise-grade models for targeted production use.

### Guard Your Loops

When writing custom automation around these scripts, always use strict loop limits.

For example:

```ts
for (let i = 0; i < 5; i++) {
  // Safe bounded loop
}
```

This helps prevent infinite async loops from continuously hitting API endpoints.

---

## ⚖️ Legal Disclaimer

> The source code and implementation instructions in this guide are intended strictly for educational, informational, and prototyping purposes.
>
> The author provides these materials **“as is”** without any express or implied warranties.
>
> By executing the code in this workspace, you acknowledge that you are solely responsible for your developer accounts, API key security, financial liabilities, costs, and data usage charges incurred through your API keys or infrastructure.
>
> The author shall not be held liable for direct, indirect, incidental, special, or consequential damages, including financial losses, unexpected credit usage, account suspension, or billing spikes.
>
> Use responsibly and configure hard billing limits before running automation.

---

## Overview

Before writing code for OpenAI, Gemini, or Claude, set up a clean TypeScript development environment.

TypeScript does not run directly on your computer without a compilation or translation step. To keep this quickstart simple, this workspace uses **tsx**, which lets you execute `.ts` scripts instantly without manually building JavaScript files first.

---

## 🛠️ Step 1: Initialize Your Node.js Workspace

Open your terminal, create a fresh project directory, and move into it:

```bash
mkdir ai-quickstarts && cd ai-quickstarts
npm init -y
```

This creates a default `package.json` file in your project root.

---

## ⚙️ Step 2: Configure Modern ESM Modules

By default, Node.js uses the older **CommonJS** module system.

Modern AI SDKs and TypeScript code typically use clean `import` and `export` statements, so configure the project to use **ECMAScript Modules**, or **ESM**.

Open the generated `package.json` file and replace the entire file contents with this:

```json
{
  "name": "ai-quickstarts",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start:openai": "tsx src/openai-test.ts",
    "start:gemini": "tsx src/gemini-test.ts",
    "start:claude": "tsx src/claude-test.ts"
  }
}
```

---

## 📦 Step 3: Install SDKs and TypeScript Utilities

Install the core AI SDKs:

```bash
npm install openai@latest @google/genai@latest @anthropic-ai/sdk@latest
```

Install the development tools:

```bash
npm install -D typescript@latest tsx@latest @types/node@latest
```

### What Is `tsx`?

**tsx** stands for **TypeScript Execute**.

Instead of manually compiling `.ts` files into a separate JavaScript output folder, `tsx` runs TypeScript files directly by stripping TypeScript-only syntax in memory.

This keeps the workflow fast and simple for quick API experiments.

---

## 📄 Step 4: Create `tsconfig.json`

Generate a TypeScript config file:

```bash
npx tsc --init
```

This creates a `tsconfig.json` file in the project root.

Replace the entire default contents of this file with this clean baseline configuration:

```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "nodenext",
    "moduleResolution": "nodenext",
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"]
}
```

---

## 📁 Step 5: Create the Project Structure

Create a dedicated `src` folder for your executable TypeScript files.

Your project should look like this:

```text
ai-quickstarts/
├── src/
│   ├── openai-test.ts
│   ├── gemini-test.ts
│   └── claude-test.ts
├── package.json
└── tsconfig.json
```

Create those three files in the src directory and paste the following provider-specific quickstart code into these files:

- `src/openai-test.ts`
- `src/gemini-test.ts`
- `src/claude-test.ts`

---

## 📄 File 1: `src/openai-test.ts`

This example uses OpenAI's **Responses API**.

It extracts the model output using the `output_text` helper property.

```ts
import OpenAI from "openai";

// Automatically extracts process.env.OPENAI_API_KEY
const client = new OpenAI();

async function main() {
  try {
    const response = await client.responses.create({
      model: "gpt-5.5",
      input: "Write a one-sentence bedtime story about a unicorn."
    });

    // The modern Responses API uses output_text to aggregate string tokens
    console.log("OpenAI Response:");
    console.log(response.output_text);
  } catch (error) {
    console.error("OpenAI Execution Error:", error);
  }
}

main();
```

---

## 📄 File 2: `src/gemini-test.ts`

This example uses the modern Google Gen AI SDK, `@google/genai`.

It accesses the generated text through `response.text`.

```ts
import { GoogleGenAI } from "@google/genai";

// Automatically extracts process.env.GEMINI_API_KEY
const ai = new GoogleGenAI({});

async function main() {
  try {
    const response = await ai.models.generateContent({
      model: "gemini-3.5-flash",
      contents: "Write a one-sentence bedtime story about a unicorn."
    });

    // An engineered prototype getter dynamically parses the inner JSON array
    console.log("Gemini Response:");
    console.log(response.text);
  } catch (error) {
    console.error("Gemini Execution Error:", error);
  }
}

main();
```

---

## 📄 File 3: `src/claude-test.ts`

This example uses Anthropic's structured **Messages API**.

Claude requests require an explicit `max_tokens` value. The response content is returned as an array of content blocks, so this script checks the first block before printing its text.

```ts
import Anthropic from "@anthropic-ai/sdk";

// Automatically extracts process.env.ANTHROPIC_API_KEY
const client = new Anthropic();

async function main() {
  try {
    const message = await client.messages.create({
      model: "claude-opus-4-8",
      max_tokens: 1000, // Anthropic requires this explicit safety parameter
      messages: [
        {
          role: "user",
          content: "Write a one-sentence bedtime story about a unicorn."
        }
      ]
    });

    // Claude can return alternating text, layout blocks, and tool executions.
    // We drill down into the zero-indexed item array block to grab the text.
    console.log("Claude Response:");
    if (message.content[0].type === "text") {
      console.log(message.content[0].text);
    }
  } catch (error) {
    console.error("Anthropic Execution Error:", error);
  }
}

main();
```

---

## 🔑 Step 6: Configure Environment Variables

All three SDKs can read API credentials from your operating system environment.

That means you do **not** need to hardcode secret keys inside your code.

**If fact, you should not hard code your API keys or any other sensitive information in code files or in any other files that might be committed to source control such as git.**

You create an API key for each of these three APIs. Refer to the developer documentation for each of these APIs for instructions on creating API keys. 

Note that when creating API keys you need to copy them somewhere on your computer, because after they are created you cannot view them again.

Run these commands in your active terminal session before executing the scripts:

```bash
export OPENAI_API_KEY="sk-proj-..."
export GEMINI_API_KEY="AIzaSy..."
export ANTHROPIC_API_KEY="sk-ant-..."
```

---

## Persisting Your Environment Credentials

Running `export KEY="value"` directly in the terminal is temporary.

When you close the terminal tab or restart your computer, those variables disappear.

To avoid re-exporting keys every time, save them in your terminal profile configuration.

---

## Mac and Linux Users

Most modern terminals use either **Bash** or **Zsh**.

Open the appropriate profile file:

```bash
# If your terminal uses Bash:
nano ~/.bash_profile

# If your terminal uses modern macOS default Zsh:
nano ~/.zshrc
```

Scroll to the bottom of the file and paste:

```bash
# AI Provider API Keys
export OPENAI_API_KEY="sk-proj-..."
export GEMINI_API_KEY="AIzaSy..."
export ANTHROPIC_API_KEY="sk-ant-..."
```

Save and exit Nano:

- Press **Ctrl + O**
- Press **Enter**
- Press **Ctrl + X**

Apply the changes immediately:

```bash
source ~/.bash_profile  # or source ~/.zshrc
```

---

## 💻 Other Environment Options

Different shells and operating systems store environment variables differently.

The important point is that these exact variable names must be available globally:

```text
OPENAI_API_KEY
GEMINI_API_KEY
ANTHROPIC_API_KEY
```

| Environment | Example Approach |
|---|---|
| **Windows PowerShell** | Add variables to your persistent profile script using `$PROFILE`. |
| **Windows Command Prompt** | Use `setx` to store variables permanently in the system registry. |
| **VS Code / WebStorm** | Inject these variable names into your IDE Run/Debug Configuration templates. |

### Windows PowerShell Example

```powershell
$env:OPENAI_API_KEY="sk-proj-..."
$env:GEMINI_API_KEY="AIzaSy..."
$env:ANTHROPIC_API_KEY="sk-ant-..."
```

### Windows Command Prompt Example

```cmd
setx OPENAI_API_KEY "sk-proj-..."
setx GEMINI_API_KEY "AIzaSy..."
setx ANTHROPIC_API_KEY "sk-ant-..."
```

> **Note:** These are examples only. Depending on your operating system, shell, editor, and workflow, you may need a different setup.

---

## 🚀 Step 7: Execute the Scripts

With your variables loaded and your scripts mapped in `package.json`, run each provider test independently.

```bash
# Execute the OpenAI Responses API script
npm run start:openai

# Execute the Google Gemini Gen AI script
npm run start:gemini

# Execute the Claude Anthropic Messages script
npm run start:claude
```

Note: these three scripts use the same prompt but generate different results, because they use an LLM (Large Language Model, the AI brain) specific to the API.

**Write a one-sentence bedtime story about a unicorn.**

The following are the response I received, yours will likely differ:

**OpenAI Response:**

Under a moonlit sky, a gentle unicorn sprinkled stardust over the sleepy forest so every creature could dream of magic until morning.

**Gemini Response:**

As the silver moon climbed high, the little unicorn tucked her hooves beneath a blanket of stardust, rested her weary head, and let the magic of her glowing horn fade gently into a quiet night of sweet dreams.

**Claude Response:**

Beneath a blanket of twinkling stars, a gentle unicorn named Luna pranced across meadows of moonlit clover, sprinkling sweet dreams into the hearts of all the sleeping children below.

---

## 🧠 The Architectural Primitives: How Each API Thinks

All three APIs can generate a simple bedtime story, but they use different data shapes under the hood.

Each platform is organized around a different core concept.

| Provider | Core API System | Core Primitive | Mental Model |
|---|---|---|---|
| **OpenAI** | Responses API `/v1/responses` | **Item** | An evolving ledger of stateless event nodes chained together. |
| **Gemini** | Models API `/v1/models` | **Content** | A stateless, multimodal array of parts, such as text, image, or audio, processed as one layout. |
| **Claude** | Messages API `/v1/messages` | **Message** | A strict, stateful dialogue of alternating conversational turns between user and assistant. |

---

## 1. OpenAI: Responses API

OpenAI historically used the widely copied **Chat Completions API**.

The newer **Responses API** shifts the model away from basic text chat arrays and toward an agentic system timeline.

### How It Works

OpenAI wraps interactions into **Items**.

An item can represent:

- A message
- A raw system command
- A function call
- A function call output

### Why This Matters

This structure makes tool use and server-side state tracking easier.

Instead of sending an entire historical message array on every turn, you can pass a `previous_response_id` pointer. OpenAI can then reconstruct the historical chain of items.

---

## 2. Gemini: Google Gen AI SDK

Gemini was designed around native multimodal input.

That means it can process text, images, audio, code, and video-style data through a unified structure.

### How It Works

The primary input parameter is `contents`.

`contents` maps to an array of **Part** objects.

A single user prompt can contain:

- A text part
- An image buffer part
- An audio file part
- Other supported multimodal inputs

### Why This Matters

This gives Gemini a unified processing model for multimodal prompts.

For standard text completions, the SDK provides convenience helpers like:

```ts
response.text
```

This lets you access the final output without manually parsing deeply nested response blocks.

---

## 3. Claude: Anthropic Messages API

Anthropic uses a highly structured, conversation-focused model through the **Messages API**.

### How It Works

Each request is built around an alternating array of `user` and `assistant` message blocks.

Anthropic also enforces clear request boundaries.

For example:

- You must explicitly set `max_tokens`.
- Conversation structure is intentionally strict.
- Responses can include different content block types.

### Why This Matters

Claude's strict structure supports specialized features such as:

- **Extended Thinking**, where the model can return a separate `thinking` block before the final answer.
- **Prompt Caching**, where specific context blocks can be cached to reduce latency and execution cost.

---

## Final Notes

This workspace is intended for simple API learning and experimentation.

Before running these scripts:

- Confirm your API keys are configured.
- Set hard billing limits.
- Keep test loops small.
- Avoid hardcoding secrets.
- Use lower-cost models for sandbox work.
