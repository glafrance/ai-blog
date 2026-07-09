# Function Calling - OpenAI API and TypeScript

In the first post in this series, we created a clean TypeScript workspace for experimenting with the OpenAI, Google Gemini, and Anthropic Claude APIs.

That first post focused on the most basic API workflow:

```text
Send a prompt → receive a text response → print the result
```

That was the right place to start because before building anything more advanced, we need to prove a few basic things:

* Node.js and TypeScript are working
* the SDK packages are installed
* API keys are available as environment variables
* each provider can successfully receive a request and return a response
* we understand the basic request and response shape for each API

If you have not yet set up the TypeScript environment for this blog post series, start with this post:

TODO: ADD LINK TO TYPESCRIPT ENVIRONMENT POST

In the second post, we used the same TypeScript environment but moved one step deeper. Instead of asking the model for ordinary text, we asked it to return structured output.

That changed the workflow from this:

```text
Send a prompt → receive a paragraph
```

to something closer to this:

```text
Send source text → 
    request a specific object shape → 
        validate the result → 
            use the data safely
```

Structured output matters because real applications usually need more than a nice paragraph. A program often needs predictable data.

For example, if we are building an educational application, we may not want the model to simply explain photosynthesis in free-form prose. We may want it to return a lesson card with specific fields:

* `title`
* `summary`
* `keyTerms`
* `quizQuestion`
* `nextStep`

That kind of response is much easier to display in a frontend, save in a database, validate with code, or pass to another part of an application.

Structured output teaches an important lesson:

```text
The model can return controlled data that our program can inspect.
```

If you need a refresher on structured output, see this blog post:

TODO: ADD LINK TO STRUCTURED OUTPUT POST

## 💰 Account Requirements & Cost Advisory

This post uses the OpenAI API.

Before running the code, make sure you have:

* an OpenAI API account
* an OpenAI API key
* billing or credits configured for API usage
* the `OPENAI_API_KEY` environment variable available in your terminal

The first post in this series explains the environment variable setup in more detail.

Every programmatic API request sends tokens to a remote AI service. Those requests can create real costs billed to your API account.

Before running automation, protect yourself with these safeguards.

### Set Hard Usage Caps

Log into your provider billing dashboard and set aggressive daily or monthly budget limits, such as **$5.00**, while learning.

This can help prevent runaway scripts from draining credits or creating unexpected bills.

### Avoid Expensive Models for Sandbox Loops

For simple tests, debugging, or CI loops, use lower-cost models when possible.

Save larger or more expensive models for targeted production use.

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

## Function Calling / Tool Use with OpenAI API

In this post, we move to the next major concept:

```text
The model can request that our program execute a function.
```

This is usually called **function calling** or **tool use**.

In this post, we focus specifically on function calling with the OpenAI API.

Subsequent posts will use a similar example with the Gemini and Claude APIs.

## What Is Function Calling?

Function calling does not mean the AI model secretly runs code by itself.

Instead, we describe one or more functions that our application knows how to run. The model can then decide that one of those functions would help answer the user's request.

The model returns a structured function call request.

Our TypeScript code reads that request, executes the real function, and sends the result back to the model.

A simplified version of the flow looks like this:

```text
User asks a question
        ↓
OpenAI receives the request and sees the available tool
        ↓
OpenAI requests a tool call
        ↓
Our TypeScript code executes the function
        ↓
The function result is sent back to OpenAI
        ↓
OpenAI writes the final response
```

The important boundary is this:

```text
The model requests the function call.
Your application executes the function.
```

That boundary matters because it keeps your application in control.

The model does not directly access your database, your files, your email, or your internal systems. It can only request the tools that your application has explicitly made available.

## Why Function Calling Matters

A plain AI response is useful when we want the model to write text, summarize text, explain something, classify something, or transform text.

Structured output is useful when we want the model to return predictable data.

Function calling becomes useful when the model needs help from our application.

For example, a model might need to:

* look up a customer record
* check appointment availability
* retrieve an order status
* search a product catalog
* create a support ticket
* calculate a price
* get a classroom activity from application code

The model is good at understanding the user's request.

Your application is good at safely executing real code.

Together, they create a much more useful workflow:

```text
Model understands the request.
Application performs the action.
Model explains the result.
```

## A More Realistic Example

Imagine a small business email assistant.

A customer sends this message:

```text
Hi, I need to reschedule my plumbing appointment. I was supposed to have someone come Friday morning, but I need something next week instead.
```

A basic AI script could write a polite reply.

That is useful, but limited.

A function-calling workflow could do more:

```text
Read the customer email
        ↓
Call a function to find the customer
        ↓
Call a function to look up the current appointment
        ↓
Call a function to check next week's availability
        ↓
Draft a reply with real appointment options
        ↓
Send the draft to a human for approval
```

In that kind of system, the model is not replacing the business application.

The model is helping decide which application functions need to be used.

The actual work is still controlled by your code.

We will not build that full system in this post.

For now, we will use one very small local TypeScript function so we can focus on the function-calling pattern itself.

## The Example Function for This Post

For this post, we will create a small function that returns a classroom science activity.

The function will accept two arguments:

* `topic`
* `gradeLevel`

The `topic` argument tells the function what the activity should be about.

The `gradeLevel` argument changes how simple or detailed the activity should be.

This keeps the example small, but still shows the main idea:

```text
The model passes arguments.
Our function uses those arguments.
The result changes based on those arguments.
```

## Create the Shared Function File

Create this file:

```text
src/science-activity.ts
```

Add the following code to the file:

```ts
export type GradeLevel = "elementary" | "middle-school";

export type ScienceActivityInput = {
  topic: string;
  gradeLevel: GradeLevel;
};

export type ScienceActivity = {
  topic: string;
  gradeLevel: GradeLevel;
  title: string;
  materials: string[];
  instructions: string;
};

export function getScienceActivity(
  input: ScienceActivityInput
): ScienceActivity {
  const isElementary = input.gradeLevel === "elementary";

  return {
    topic: input.topic,
    gradeLevel: input.gradeLevel,
    title: isElementary
      ? `${input.topic} Observation Activity`
      : `${input.topic} Investigation Activity`,
    materials: isElementary
      ? ["paper", "pencil", "colored pencils"]
      : ["science notebook", "pencil", "timer"],
    instructions: isElementary
      ? `Draw what you notice about ${input.topic} and write one sentence about it.`
      : `Observe ${input.topic}, record evidence, and explain what pattern you notice.`,
  };
}
```

This function is intentionally simple.

It does not call a database.

It does not call another API.

It does not use any AI by itself.

It is just normal TypeScript code.

That is important because function calling is not about letting the model magically run code.

Function calling is about letting the model request that our application run a specific function.

## How This Function Uses Its Arguments

This function uses both arguments passed into it.

The `topic` value appears in the activity title and instructions.

For example, if the topic is `photosynthesis`, the function can return text about a photosynthesis activity.

The `gradeLevel` value changes the activity.

For `elementary`, the function returns a simpler observation activity.

For `middle-school`, the function returns a slightly more detailed investigation activity.

So if our code calls:

```ts
getScienceActivity({
  topic: "photosynthesis",
  gradeLevel: "elementary",
});
```

the function returns a simpler activity.

But if our code calls:

```ts
getScienceActivity({
  topic: "photosynthesis",
  gradeLevel: "middle-school",
});
```

the function returns a slightly more detailed version.

This is enough to demonstrate the core idea.

The model will decide what arguments to send, and our TypeScript function will use those arguments.

## Why Put This Function in a Separate File?

We are putting this function in its own file so the API-calling code stays focused on OpenAI.

Later, if we create similar posts for Gemini and Claude, those examples can use the same function.

The project will look like this:

```text
src/
├── science-activity.ts
└── openai-tool-call.ts
```

The OpenAI script will import the function from `science-activity.ts`.

That keeps the example cleaner because the OpenAI file can focus on:

* defining the tool for OpenAI
* sending the request
* reading the function call
* executing the local function
* sending the function result back
* printing the final response

## Why the Import Uses `.js`

When we import this function from another TypeScript file, the import will look like this:

```ts
import {
  getScienceActivity,
  type ScienceActivityInput,
} from "./science-activity.js";
```

This may look strange because the file we created is named:

```text
science-activity.ts
```

However, this project uses modern Node.js ECMAScript Modules.

In this setup, TypeScript source files use `.ts`, but the import path is written the way Node.js expects it after TypeScript becomes JavaScript.

So the source file is:

```text
science-activity.ts
```

but the import path uses:

```text
./science-activity.js
```

This is a normal pattern when using `"module": "nodenext"` and `"type": "module"`.

## OpenAI Function Calling Example

Now that we have created the shared function in `src/science-activity.ts`, we can write the OpenAI-specific script.

This script will:

* describe our local `getScienceActivity` function as an OpenAI tool
* send a user request to the OpenAI API
* receive a function call request from the model
* execute our local TypeScript function
* send the function result back to OpenAI
* print the final user-facing response

Create this file:

```text
src/openai-tool-call.ts
```

Add the following code to the file:

```ts
import OpenAI from "openai";

import {
  getScienceActivity,
  type ScienceActivityInput,
} from "./science-activity.js";

const client = new OpenAI();

const tools = [
  {
    type: "function" as const,
    name: "getScienceActivity",
    description:
      "Get a classroom science activity for a requested science topic and grade level.",
    parameters: {
      type: "object",
      properties: {
        topic: {
          type: "string",
          description:
            "The science topic for the classroom activity, such as photosynthesis or gravity.",
        },
        gradeLevel: {
          type: "string",
          enum: ["elementary", "middle-school"],
          description:
            "The student grade level. Use elementary for a simpler activity and middle-school for a slightly more detailed activity.",
        },
      },
      required: ["topic", "gradeLevel"],
      additionalProperties: false,
    },
    strict: true,
  },
];

function parseScienceActivityInput(argumentsJson: string): ScienceActivityInput {
  const parsed = JSON.parse(argumentsJson) as Partial<ScienceActivityInput>;

  if (typeof parsed.topic !== "string") {
    throw new Error("Tool argument 'topic' must be a string.");
  }

  if (
    parsed.gradeLevel !== "elementary" &&
    parsed.gradeLevel !== "middle-school"
  ) {
    throw new Error(
      "Tool argument 'gradeLevel' must be 'elementary' or 'middle-school'."
    );
  }

  return {
    topic: parsed.topic,
    gradeLevel: parsed.gradeLevel,
  };
}

async function main() {
  try {
    const firstResponse = await client.responses.create({
      model: "gpt-5.5",
      instructions:
        "You are a helpful teaching assistant. Use the available tool when the user asks for a classroom science activity.",
      tools,
      tool_choice: {
        type: "function",
        name: "getScienceActivity",
      },
      input:
        "I teach elementary school science. Give me a classroom activity about photosynthesis.",
    });

    const functionCall = firstResponse.output.find(
      (item) => item.type === "function_call"
    );

    if (!functionCall) {
      console.log("OpenAI Response:");
      console.log(firstResponse.output_text);
      return;
    }

    if (functionCall.name !== "getScienceActivity") {
      throw new Error(`Unknown tool requested: ${functionCall.name}`);
    }

    const activityInput = parseScienceActivityInput(functionCall.arguments);
    const activity = getScienceActivity(activityInput);

    const finalResponse = await client.responses.create({
      model: "gpt-5.5",
      previous_response_id: firstResponse.id,
      instructions:
        "Write a friendly final answer for a teacher. Use the tool result, but do not mention JSON, function calls, or internal tool execution.",
      input: [
        {
          type: "function_call_output",
          call_id: functionCall.call_id,
          output: JSON.stringify(activity),
        },
      ],
    });

    console.log("OpenAI Final Response:");
    console.log(finalResponse.output_text);
  } catch (error) {
    console.error("OpenAI Execution Error:", error);
  }
}

main();
```

## Add the OpenAI Script to `package.json`

Open your `package.json` file and find the `"scripts"` block.

Add this entry:

```json
"start:openai-tool-call": "tsx src/openai-tool-call.ts"
```

Your `"scripts"` block may already contain entries from the earlier posts. That is fine.

Each script line needs a comma after it except the final script line.

## Execute the OpenAI Tool-Calling Script

Run the script from the project root:

```bash
npm run start:openai-tool-call
```

If everything is working, the script should print a final response that looks like a normal answer for a teacher.

The model should use the result from our local `getScienceActivity` function, but the final response should not mention JSON, internal function calls, or tool execution details.

## My Response

Your result will probably be different, but here is one example response I received:

```text
OpenAI Final Response:
Here’s a simple elementary-friendly activity you can use:

## Photosynthesis Observation Activity

**Objective:**  
Students will observe and represent what plants need for photosynthesis, then explain their thinking in one sentence.

**Materials:**
- Paper  
- Pencil  
- Colored pencils  

**Instructions:**
1. Begin with a short discussion: “What do plants need to grow?” Guide students toward sunlight, water, air, and soil/nutrients.
2. Explain that **photosynthesis** is how plants use sunlight, water, and air to make their own food.
3. Have students draw a plant showing the main parts involved:
   - Sunlight shining on the leaves
   - Water going into the roots
   - Air going into the leaves
   - Food/sugar being made inside the plant
4. Students color their drawings and label at least three things the plant needs.
5. At the bottom of the page, students write one sentence about photosynthesis, such as:  
   **“Plants use sunlight, water, and air to make food.”**

**Optional Sharing:**  
Invite students to share their drawings with a partner or the class and explain one part of photosynthesis they included.
```

## How OpenAI Handles Function Calling

In the OpenAI Responses API, a function is provided to the model as a **tool**.

The tool definition tells the model:

* the function name
* what the function does
* what arguments it accepts
* which arguments are required

In our script, the tool definition starts like this:

```ts
const tools = [
  {
    type: "function" as const,
    name: "getScienceActivity",
    description:
      "Get a classroom science activity for a requested science topic and grade level.",
```

The important detail is that this does not run the function.

It only tells OpenAI that a function named `getScienceActivity` is available.

Then we describe the arguments the function accepts:

```ts
parameters: {
  type: "object",
  properties: {
    topic: {
      type: "string",
      description:
        "The science topic for the classroom activity, such as photosynthesis or gravity.",
    },
    gradeLevel: {
      type: "string",
      enum: ["elementary", "middle-school"],
      description:
        "The student grade level. Use elementary for a simpler activity and middle-school for a slightly more detailed activity.",
    },
  },
  required: ["topic", "gradeLevel"],
  additionalProperties: false,
},
strict: true,
```

This is similar to the structured-output idea from the previous post.

The difference is that this schema describes the arguments for a tool call.

In other words, we are not asking the model to return a lesson card directly.

We are telling the model:

```text
If you need a classroom science activity, you may request a call to getScienceActivity.
When you request that call, provide a topic and gradeLevel.
```

For this learning example, we also use `tool_choice`:

```ts
tool_choice: {
  type: "function",
  name: "getScienceActivity",
},
```

By default, the model can decide whether to call zero, one, or multiple tools.

For this post, we want the tool call to happen so the example clearly demonstrates function calling.

Next, we send the first request:

```ts
const firstResponse = await client.responses.create({
  model: "gpt-5.5",
  instructions:
    "You are a helpful teaching assistant. Use the available tool when the user asks for a classroom science activity.",
  tools,
  tool_choice: {
    type: "function",
    name: "getScienceActivity",
  },
  input:
    "I teach elementary school science. Give me a classroom activity about photosynthesis.",
});
```

This request gives the model:

* the user's request
* the tool definition
* instructions telling it when the tool should be used
* a `tool_choice` setting that forces this specific tool call for the demo

If the model uses the tool, OpenAI returns an output item with this type:

```ts
"function_call"
```

In our script, we search the response output for that function call:

```ts
const functionCall = firstResponse.output.find(
  (item) => item.type === "function_call"
);
```

A function call includes the name of the function and the arguments the model wants to pass.

Before trusting those arguments, our code parses and checks them:

```ts
const activityInput = parseScienceActivityInput(functionCall.arguments);
```

Then our TypeScript code executes the real function:

```ts
const activity = getScienceActivity(activityInput);
```

This is the key moment.

OpenAI did not execute `getScienceActivity`.

The model requested the function call.

Our application executed the function.

After the function runs, we send the result back to OpenAI:

```ts
const finalResponse = await client.responses.create({
  model: "gpt-5.5",
  previous_response_id: firstResponse.id,
  instructions:
    "Write a friendly final answer for a teacher. Use the tool result, but do not mention JSON, function calls, or internal tool execution.",
  input: [
    {
      type: "function_call_output",
      call_id: functionCall.call_id,
      output: JSON.stringify(activity),
    },
  ],
});
```

The `function_call_output` item contains the result from our local TypeScript function.

The `call_id` connects that result to the specific tool call OpenAI requested.

The `previous_response_id` tells OpenAI that this second request continues from the first response.

At this point, the model has:

* the original user request
* the function call it requested
* the function result returned by our code

Now the model can write the final answer:

```ts
console.log(finalResponse.output_text);
```

So the OpenAI function-calling flow looks like this:

```text
define tool
        ↓
send user request with tool available
        ↓
receive function_call from OpenAI
        ↓
parse and validate arguments
        ↓
execute local TypeScript function
        ↓
send function_call_output back to OpenAI
        ↓
print final model response
```

The main lesson is:

```text
OpenAI helps the model request a function call.
Your application still controls the actual function execution.
```

That is why function calling is powerful.

The model can decide that a tool is needed, but your code controls what tools exist, what they can do, and how their results are handled.

## What We Learned

In this post, we built a minimal OpenAI function-calling example.

The important lesson is not the specific classroom activity.

The important lesson is the pattern:

```text
Describe a function to the model.
Let the model request the function.
Run the function in your own code.
Return the result to the model.
Use the model to produce the final answer.
```

This is a major step beyond a basic prompt-response script.

A basic script asks the model to answer from its own training and current prompt context.

A function-calling script lets the model ask your application for help.

## Why We Kept the Example Small

The example in this post used a local TypeScript function instead of a database, API, or external service.

That was intentional.

The goal was not to build a full production system yet.

The goal was to clearly understand the function-calling pattern without extra distractions.

Once this pattern makes sense, the local function could later be replaced with something more realistic, such as:

```text
getCustomerByEmail(...)
getAvailableAppointments(...)
getOrderStatus(...)
searchKnowledgeBase(...)
createEmailDraft(...)
```

The OpenAI API flow would be similar.

The main difference would be what your application function does behind the scenes.

## Function Calling Is a Building Block for Agents

Function calling is also one of the building blocks behind AI agents.

An agent usually does more than call one function one time.

A more advanced agent might:

```text
read the user request
        ↓
choose a tool
        ↓
inspect the result
        ↓
choose another tool
        ↓
inspect that result
        ↓
continue until it can answer
```

We are not building that full loop in this post.

But this post introduces the core mechanism that makes that kind of workflow possible.

Before building agents, it is important to understand the simpler version:

```text
one request
one tool call
one function result
one final answer
```

## Important Safety Reminder

Function calling gives the model a way to request actions from your application.

That means you should be careful about what tools you expose.

For beginner examples, calling a small local function is safe.

For real applications, you should think carefully about:

* validating arguments
* limiting what each tool can do
* logging tool calls
* requiring approval before sensitive actions
* preventing destructive actions from running automatically
* handling unexpected model output
* protecting private data

Even when the model provides structured arguments, your code should still validate them before executing anything important.

## Final Notes

In this post, we focused only on the OpenAI API.

We used the same TypeScript environment from the earlier posts and added:

```text
src/science-activity.ts
src/openai-tool-call.ts
```

This workspace is intended for simple API learning and experimentation.

Before running these scripts:

* confirm your API keys are configured
* set hard billing limits
* keep test loops small
* avoid hardcoding secrets
* use lower-cost models for sandbox work when possible

In the next posts, we can apply the same idea to the Gemini and Claude APIs and compare how each provider represents function calling in its own SDK.
