# OpenAI vs Gemini vs Claude APIs: Returning Structured Data with TypeScript

In the first post in this series, we created a clean TypeScript workspace and used the OpenAI, Gemini, and Claude APIs to run a very simple first script.

**Note: if you have not set up your TypeScript environment, including possibly signing up for OpenAI, Gemini, and Claude developer accounts, and obtaining API keys, refer to this blog post for that setup:**

[Quick Guide to AI APIs](https://gravizot.com/quick-guide-to-ai-apis/)

The goal of that first script was intentionally basic:

```text
Send a prompt → receive a text response → print the result
```

That is the right place to start because it proves that your local TypeScript setup works, your API keys are configured correctly, and each provider SDK can successfully communicate with its remote AI service.

In this second post, we will move one level deeper.

Instead of asking the model to return a plain sentence or paragraph, we will ask it to return **structured output**.

Structured output means the model response should follow a specific data shape, such as a JSON object with known field names and value types.

In this post, the response should not be random prose. It should be an object our TypeScript code can inspect, validate, and use.

The new goal looks more like this:

```text
Send source text → request a specific JSON shape → validate the result → use the data safely
```

This is an important step because real applications usually need more than a paragraph of generated text. A program often needs data it can trust and use directly.

For example, instead of asking an AI model to explain photosynthesis in a paragraph, we might ask it to create a small lesson card with:

* a title
* a short summary
* a list of key terms
* a quiz question
* a suggested next step

That kind of response is much easier for an application to display, save, search, transform, or send to another part of a system.

## Why Structured Output Matters

Plain text is flexible, but it can also be unpredictable.

A model might return a beautiful explanation, but your program may not know where the title ends, where the summary begins, or how many key terms were included.

Structured output helps solve that problem.

Instead of saying:

```text
Please explain this topic.
```

We can say something closer to:

```text
Return an object with this exact shape:
title
summary
keyTerms
quizQuestion
nextStep
```

This does not make the model perfect, but it makes the output much easier to work with in code.

## Adding Runtime Validation with Zod

In this post, we will also use **Zod**, a TypeScript-friendly validation library.

TypeScript types are helpful while we write code, but they do not automatically protect us from unexpected data at runtime.

That matters here because API responses arrive while the program is running.

In this post, the schema has two related jobs.

First, it describes the shape we want the AI model to return.

Second, it gives our TypeScript code a way to check the returned data before we trust it.

Those two ideas are related, but they are not exactly the same thing.

The API uses the schema to guide or constrain the model response. Our application uses the schema to validate the final result at runtime.

In other words:

```text
TypeScript helps while writing the code.
Zod helps while the code is running.
```

For this post, we will define a `LessonCardSchema` and use it to describe the object we want back from each API.

## Same Goal, Different API Styles

OpenAI, Gemini, and Claude can all be used for this kind of structured output workflow, but they do not all express the idea in exactly the same way.

That is one of the main learning goals of this post.

We are not just trying to make the scripts work. We are trying to understand how each provider thinks about the same problem.

At a high level:

| Provider | What the script sends | What the script receives | Where validation/parsing happens |
|---|---|---|---|
| **OpenAI** | A Zod schema converted by `zodTextFormat(...)` | A parsed object on `response.output_parsed` | Mostly through the OpenAI SDK helper |
| **Gemini** | A JSON Schema generated from the Zod schema | JSON text on `response.text` | Explicitly in our code with `JSON.parse(...)` and `LessonCardSchema.parse(...)` |
| **Claude** | A Zod schema converted by `zodOutputFormat(...)` | A parsed object on `message.parsed_output` | Mostly through the Anthropic SDK helper |

The syntax is different, but the mental model is similar:

```text
Define the shape → ask the model for that shape → validate the result
```

This is a small step beyond the first post, but it is a meaningful one.

Once you understand this pattern, you can start building scripts that do more useful work:

* extract fields from text
* generate quiz questions
* summarize articles into reusable objects
* create flashcards
* classify user input
* prepare data for a database or frontend application

We will still keep the scripts beginner-friendly.

Each script will use the same source text, the same basic schema, and the same goal: create a simple educational lesson card from a short explanation of photosynthesis.

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

## 📁 Verify the Project Structure

If you followed the steps in the post on setting up your TypeScript environment, you should have this folder structure:

`ai-quickstarts/src`

**Note: if you have not set up your TypeScript environment, including possibly signing up for OpenAI, Gemini, and Claude developer accounts, and obtaining API keys, refer to this blog post for that setup:**

[Quick Guide to AI APIs](https://gravizot.com/quick-guide-to-ai-apis/)

## 📦 Install the zod Package

Zod is used to define and validate the expected response shape. TypeScript helps us while writing code, but Zod helps us check real data while the program is running.

Make sure you are in the directory containing the package.json file and execute this command:

```bash
npm install zod@latest
```

## Create Files for Our New API Scripts

We want to have these script files in place:

```text
ai-quickstarts/
├── src/
│   ├── openai-lesson-card.ts
│   ├── gemini-lesson-card.ts
│   └── claude-lesson-card.ts
```

You may have scripts created in the previous post and that is fine.

Create those three files in the src directory and paste the following provider-specific quickstart code into these files:

- `src/openai-lesson-card.ts`
- `src/gemini-lesson-card.ts`
- `src/claude-lesson-card.ts`

**Note: if you are on the Mac terminal or on a Linux machine, you can probably create these files after CDing into the `src` directory with this command: **

`touch openai-lesson-card.ts`
`touch gemini-lesson-card.ts`
`touch claude-lesson-card.ts`

## The Shared Lesson Card Schema

All three scripts use the same basic response shape.

That shape is called `LessonCardSchema`.

The schema says that a valid lesson card should include:

* `title`: a string
* `summary`: a string
* `keyTerms`: an array of term/definition objects
* `quizQuestion`: an object with a question and answer
* `nextStep`: a string

Using the same schema for all three providers makes the comparison easier.

The API syntax changes from provider to provider, but the goal stays the same:

```
Return one lesson card object that matches this schema.
```

## Add the Following Content to Each File

## 📄 File 1: `src/openai-lesson-card.ts`

```ts
import OpenAI from "openai";
import { zodTextFormat } from "openai/helpers/zod";
import { z } from "zod";

const client = new OpenAI();

const LessonCardSchema = z.object({
  title: z.string(),
  summary: z.string(),
  keyTerms: z.array(
    z.object({
      term: z.string(),
      definition: z.string(),
    })
  ),
  quizQuestion: z.object({
    question: z.string(),
    answer: z.string(),
  }),
  nextStep: z.string(),
});

type LessonCard = z.infer<typeof LessonCardSchema>;

const sourceText = `
Photosynthesis is the process plants use to make food.
Plants take in carbon dioxide from the air and water from the soil.
Using energy from sunlight, they turn those ingredients into sugar and oxygen.
The sugar helps the plant grow, and the oxygen is released into the air.
`;

async function main() {
  try {
    const response = await client.responses.parse({
      model: "gpt-5.5",
      instructions:
        "You are a beginner-friendly science teacher. Create a short lesson card for an elementary school student.",
      input: sourceText,
      text: {
        format: zodTextFormat(LessonCardSchema, "lesson_card"),
      },
    });

    const lessonCard: LessonCard | null = response.output_parsed;

    if (!lessonCard) {
      throw new Error("OpenAI did not return a parsed lesson card.");
    }

    console.log("OpenAI Lesson Card:");
    console.dir(lessonCard, { depth: null });
  } catch (error) {
    console.error("OpenAI Execution Error:", error);
  }
}

main();
```

In the OpenAI script, we use the Responses API with `responses.parse(...)`.

The important detail is that we do not manually call `JSON.parse(...)`.

Instead, we pass the Zod schema into OpenAI’s helper function:

```ts
zodTextFormat(LessonCardSchema, "lesson_card")
```

That helper gives the OpenAI SDK the structured-output format it needs.

If the model returns a response that matches the schema, the SDK gives us the parsed result here:

response.output_parsed

So in the OpenAI version, the response feels very direct:

Zod schema → OpenAI structured output format → parsed TypeScript object

## 📄 File 2: `src/gemini-lesson-card.ts`
```ts
import { GoogleGenAI } from "@google/genai";
import * as z from "zod";

const ai = new GoogleGenAI({});

const LessonCardSchema = z.object({
  title: z.string(),
  summary: z.string(),
  keyTerms: z.array(
    z.object({
      term: z.string(),
      definition: z.string(),
    })
  ),
  quizQuestion: z.object({
    question: z.string(),
    answer: z.string(),
  }),
  nextStep: z.string(),
});

type LessonCard = z.infer<typeof LessonCardSchema>;

const sourceText = `
Photosynthesis is the process plants use to make food.
Plants take in carbon dioxide from the air and water from the soil.
Using energy from sunlight, they turn those ingredients into sugar and oxygen.
The sugar helps the plant grow, and the oxygen is released into the air.
`;

async function main() {
  try {
    const prompt = `
You are a beginner-friendly science teacher.
Create a short lesson card for an elementary school student.

Source text:
${sourceText}
`;

    const response = await ai.models.generateContent({
      model: "gemini-3.5-flash",
      contents: prompt,
      config: {
        responseMimeType: "application/json",
        responseJsonSchema: z.toJSONSchema(LessonCardSchema),
      },
    });

    if (!response.text) {
      throw new Error("Gemini did not return text.");
    }

    const lessonCard: LessonCard = LessonCardSchema.parse(
      JSON.parse(response.text)
    );

    console.log("Gemini Lesson Card:");
    console.dir(lessonCard, { depth: null });
  } catch (error) {
    console.error("Gemini Execution Error:", error);
  }
}

main();
```
## Gemini Structured Output Approach

The Gemini script is slightly different.

Gemini receives a JSON Schema-style response format, so we convert our Zod schema into JSON Schema:

```ts
z.toJSONSchema(LessonCardSchema)
```

Then we tell Gemini that we want JSON output:

responseMimeType: "application/json"

After Gemini responds, the response is still text from our program’s point of view:

response.text

So we do two explicit steps:

JSON.parse(response.text)

This converts the JSON string into a JavaScript object.

Then we validate the object with Zod:

LessonCardSchema.parse(...)

So the Gemini version looks like this:

Zod schema → JSON Schema for Gemini → JSON text response → JSON.parse → Zod validation

Note: this example uses Gemini’s `generateContent` API because it matches the style used in the first post in this series. Google’s current documentation also recommends the newer Interactions API for access to the latest Gemini features and models. If this script stops working in the future, check the current Gemini structured-output documentation first.

## 📄 File 3: `src/claude-lesson-card.ts`
```ts
import Anthropic from "@anthropic-ai/sdk";
import { zodOutputFormat } from "@anthropic-ai/sdk/helpers/zod";
import { z } from "zod/v4";

const client = new Anthropic();

const LessonCardSchema = z.object({
  title: z.string(),
  summary: z.string(),
  keyTerms: z.array(
    z.object({
      term: z.string(),
      definition: z.string(),
    })
  ),
  quizQuestion: z.object({
    question: z.string(),
    answer: z.string(),
  }),
  nextStep: z.string(),
});

type LessonCard = z.infer<typeof LessonCardSchema>;

const sourceText = `
Photosynthesis is the process plants use to make food.
Plants take in carbon dioxide from the air and water from the soil.
Using energy from sunlight, they turn those ingredients into sugar and oxygen.
The sugar helps the plant grow, and the oxygen is released into the air.
`;

async function main() {
  try {
    const message = await client.messages.parse({
      model: "claude-sonnet-4-6",
      max_tokens: 1000,
      system:
        "You are a beginner-friendly science teacher. Create a short lesson card for an elementary school student.",
      messages: [
        {
          role: "user",
          content: sourceText,
        },
      ],
      output_config: {
        format: zodOutputFormat(LessonCardSchema),
      },
    });

    const lessonCard: LessonCard | null | undefined = message.parsed_output;

    if (!lessonCard) {
      throw new Error("Claude did not return a parsed lesson card.");
    }

    console.log("Claude Lesson Card:");
    console.dir(lessonCard, { depth: null });
  } catch (error) {
    console.error("Claude Execution Error:", error);
  }
}

main();

```

## Claude Structured Output Approach

The Claude script uses Anthropic’s Messages API with structured output support.

Like the OpenAI script, the Claude version does not manually call `JSON.parse(...)`.

Instead, we pass the Zod schema into Anthropic’s helper:

```ts
zodOutputFormat(LessonCardSchema)
```

That format is sent through:

output_config: {
  format: zodOutputFormat(LessonCardSchema),
}

If Claude returns a valid structured response, the SDK gives us the parsed object here:

message.parsed_output

So the Claude version looks like this:

Zod schema → Claude output format → parsed TypeScript object

Claude also requires an explicit max_tokens value, which controls the maximum size of the response.

## Open package.json and Add Lines to Execute These Scripts

  "scripts": {
    "start:openai-lesson-card": "tsx src/openai-lesson-card.ts",
    "start:gemini-lesson-card": "tsx src/gemini-lesson-card.ts",
    "start:claude-lesson-card": "tsx src/claude-lesson-card.ts"
  }

**Note: you may have script entries from the previous post and that is fine. **

## 🚀 Step 7: Execute the Scripts

With your API key environment variables loaded and your scripts mapped in `package.json`, run each provider test independently.

```bash
# Execute the OpenAI Responses API script
npm run start:openai-lesson-card

# Execute the Google Gemini Gen AI script
npm run start:gemini-lesson-card

# Execute the Claude Anthropic Messages script
npm run start:claude-lesson-card
```
**Note: the Gemini script took significantly longer to execute compared with the OpenAI and Claude scripts. Your results may differ. **

**Note:** AI APIs and SDKs change quickly. Model names, helper imports, and structured-output syntax may change after this post is published.

If a script fails, first check:

* the current model name in the provider documentation
* the installed SDK version
* the provider’s current structured-output examples
* the exact error message in your terminal

You can also paste the script and error into an AI assistant for debugging help, but always compare the answer with the current official documentation.

The following are the response I received, yours will likely differ:

**OpenAI Response:**

`
{
  title: 'Photosynthesis: How Plants Make Food',
  summary: 'Photosynthesis is how plants make their own food. Plants take in carbon dioxide from the air and water from the soil. With energy from sunlight, they turn these into sugar for food and oxygen, which goes back into the air.',
  keyTerms: [
    {
      term: 'Photosynthesis',
      definition: 'The process plants use to make their own food using sunlight.'
    },
    {
      term: 'Carbon dioxide',
      definition: 'A gas in the air that plants take in through their leaves.'
    },
    {
      term: 'Sugar',
      definition: 'Food made by the plant that helps it grow.'
    },
    {
      term: 'Oxygen',
      definition: 'A gas plants release into the air that people and animals breathe.'
    }
  ],
  quizQuestion: {
    question: 'What do plants need for photosynthesis?',
    answer: 'Plants need sunlight, water, and carbon dioxide to make sugar and oxygen.'
  },
  nextStep: 'Look at a plant leaf outside or in your home and remember: it is like a tiny food-making factory!'
}
`

**Gemini Response:**

`
{
  title: 'How Plants Make Their Own Food!',
  summary: "Did you know plants cook their own food using sunlight? Through a process called photosynthesis, plants take carbon dioxide from the air and water from the soil. They use the sun's energy to turn them into yummy sugar to grow, and they release fresh oxygen for us to breathe!",
  keyTerms: [
    {
      term: 'Photosynthesis',
      definition: 'The super cool process that plants use to make their own food using sunlight.'
    },
    {
      term: 'Carbon Dioxide',
      definition: 'A gas in the air that plants breathe in and use to make food.'
    },
    {
      term: 'Oxygen',
      definition: 'A clean gas that plants release into the air, which helps us breathe.'
    }
  ],
  quizQuestion: {
    question: 'What two ingredients does a plant need from the air and soil to make its food?',
    answer: 'Carbon dioxide from the air and water from the soil!'
  },
  nextStep: 'Go outside and find a green leaf, then think about how it is busy making food right now using the sunshine!'
}
`

**Claude Response:**

`
{
  title: 'How Plants Make Their Own Food',
  summary: 'Plants are amazing because they can make their own food! They use sunlight, water from the soil, and air (carbon dioxide) to create sugar that helps them grow. As a bonus, they release oxygen into the air, which is what we breathe!',
  keyTerms: [
    {
      term: 'Photosynthesis',
      definition: 'The process plants use to turn sunlight, water, and carbon dioxide into food (sugar) and oxygen.'
    },
    {
      term: 'Carbon Dioxide',
      definition: 'A gas found in the air that plants breathe in to make their food.'
    },
    {
      term: 'Oxygen',
      definition: "A gas that plants release into the air during photosynthesis — it's what humans and animals breathe!"
    },
    {
      term: 'Chlorophyll',
      definition: 'The green stuff in plant leaves that captures energy from sunlight to power photosynthesis.'
    }
  ],
  quizQuestion: {
    question: 'What three things does a plant need to make its own food through photosynthesis?',
    answer: 'A plant needs sunlight, water (from the soil), and carbon dioxide (from the air) to make its food through photosynthesis.'
  },
  nextStep: 'Try this! Place a plant in a sunny spot and another in a dark closet for a few days. What do you notice? This shows how important sunlight is for photosynthesis!'
}
`

Each API provided a different response, but all three responses followed the structured output shape we requested.

This is the main lesson of the post.

Structured output helps control the **shape** of the response.

It does not automatically guarantee that every generated detail is true, complete, or perfectly based on the source text.

For example, a model may include an extra term, choose a different quiz question, or phrase a summary differently.

That is why structured output is useful for application data, but you still need normal application judgment, testing, and validation.

## JSON Mode vs Structured Output

It is also worth separating two ideas:

```text
JSON mode means: return valid JSON.
Structured output means: return JSON that follows a specific schema.
```

Valid JSON is not enough by itself.

This could be valid JSON:

{
  "message": "Plants use sunlight to make food."
}

But that is not the shape our program expects.

Our program expects fields like:

title
summary
keyTerms
quizQuestion
nextStep

Structured output is stronger because it tries to make the response match the specific object shape our application needs.

What Is Different Across the Three APIs?

The main difference is not the goal.

The goal is the same for all three APIs:

```text
Create a lesson card object that matches the schema.
```

The difference is how each API gets there.

OpenAI

OpenAI gives us a very integrated structured-output flow.

We define a Zod schema, pass it through an OpenAI helper, and read the parsed object from response.output_parsed.

Gemini

Gemini makes the JSON boundary more visible.

We ask Gemini for JSON, read the generated JSON as text, manually parse it, and then validate it with Zod.

This is slightly more code, but it is useful for learning because the parsing and validation steps are easy to see.

Claude

Claude’s approach is closer to OpenAI in this example.

We pass a schema-based output format through output_config, and the SDK gives us the parsed object on message.parsed_output.

Claude also requires max_tokens, so the request has one extra required setting compared with the OpenAI and Gemini examples.

## Final Notes

This workspace is intended for simple API learning and experimentation.

Before running these scripts:

- Confirm your API keys are configured.
- Set hard billing limits.
- Keep test loops small.
- Avoid hardcoding secrets.
- Use lower-cost models for sandbox work.