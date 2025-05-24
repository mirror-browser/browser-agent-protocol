# 🍚 BAP — Browser Agent Protocol

**BAP** is an open protocol for building a reliable, fast, and transparent communication interface between browser-based AI agents and real websites.
With BAP, you can simple "ask" your browser to do repetitive task reliablly, fastly, transparently.

⚠️ **Warning: This project is still under active development.**
The BAP spec is not finalized, and components may be unstable or non-functional.
We plan to release BAP Spec 1.0 along with a working Agent demo in **Q3 of 2025**.

## Demo
Working on it...


## 📚 Terminology

| Term       | Description |
|------------|-------------|
| **Agent**  | The orchestrator that receives a user prompts and system prompts and calls available Tool to accomplish the task |
| **WebTool**   | A single callable function, e.g. `sendEmail`, `listInbox` |
| **WebToolbox**| A set of tools for a specific website (e.g. Gmail, Twitter, Zillow) |
| **NativeTool** | A single callable function that interacts with native environment, browser (getCurrentTab, writeFile, searchWebTool, callWebTool)  |


## Workflow Overview
<img src="https://github.com/mirror-browser/browser-agent-protocol/blob/main/assets/workflow_overview.jpg?raw=true" />

## How to define WebTool?
There are two primary ways to expose agent-controllable tools on a website:
### 1. First-Party (Provided by the Website Itself)

```js
window.bapTools = [
  {
    name: "addToCart",
    description: "Add current product to cart",
    parameters: { quantity: "number" },
    run: async ({ quantity }) => { /* ... */ }
  }
];

// Websites can also explicitly opt out of agent usage:
window.bapTools = [];
```

### 2. Third-Party (Injected through adapters repository)
```js
const adapters = [
  {
    domain: "twitter.com",
    actions: {
      postTweet: { /* ... */ }
    }
  }
];
```

## Why BAP's approach?

Most current AI agents try to guess how websites work by visually interpreting the UI — which is slow, fragile, and unreliable.

Take a simple task:
`
“Monitor this Zillow page, and when a new 2-bedroom listing under $250k appears, write it to a Google Sheet and let me know.”
`

With existing browser-agent tools, this could take 3–5 minutes and still break easily.
With BAP, it runs around in 10 seconds — reliably and visibly.

### ⚡ Fast
- Most AI agents today rely on screenshot-based interpretation to simulate clicks, extract text, and perform tasks like “Add this item to cart.”
- That’s slow and error-prone — especially for workflows like “Summarize this page and send to Slack,” which can take several minutes.
- BAP executes real JavaScript in the page, skipping all the guesswork.

### 🔍 Transparent & Inspectable
-	You can see everything the agent is doing.
- Every action is declared, documented, and inspectable — not opaque LLM behavior.

### 🔐 Secure & Local
- It’s just your browser — smarter.
- No API keys, no auth flows, no data leakage.
- Sensitive info (like cookies, credentials, passwords) never leaves your computer.


### 🧭 Respectful of Websites
-	Websites are where value is created — shopping, searching, booking.
-	But current AI agents give zero agency to websites in how they are used.
- BAP changes that: like robots.txt or SEO metadata, sites can declare how they want to be automated (or not).
- Sites can choose to expose tools — or explicitly opt out.
- Yet individual developers can still create and share their own Adapters to support sites that haven’t explicitly defined how they want to interact with agents.

---

### 🆚 Why Not Other Solutions?

#### ❌ OpenAI Operator / Anthropic Computer Use

These tools rely on LLMs visually navigating the browser — interpreting screenshots to decide how to click/type/extract.

That means:
	•	❌ Slow execution — each step requires screenshot + model inference
	•	❌ No composability — even if the LLM figures out what to do once, it can’t reuse or persist that logic

BAP flips the model: no vision, no guessing — just structured JavaScript.

#### ❌ Zapier / n8n

Great when APIs exist. But:
- ❌ Can't automate websites without integrations
- ❌ Require messy auth tokens and setup
- ❌ No visibility into browser state

**BAP works with real websites in your real browser** — just like a human, but automated:
- No API needed
- You’re already logged in
- You can see everything the agent is doing

---

## Available Agent
- Originally this protocol has been developed to be used on [Mirror Browser]()
- But nothing stops you from making new browser or new browser extension that is Agent
- We have plans to make Chrome Extension BAP Agent in 2025 Q3

## Agent Spec
```typescript
interface Agent {
  systemPrompt: string
  userPrompt: string
  nativeTools: Array<{
    name: string
    description: string // Agent readable description of tool
    parameters?: JSONSchema, // undefined means tool without parameters
    response?: JSONSchema, // undefined means tool without response
  }>,
  // 3rd party web tools
  webTools: [
    { type: "repository" url: "https://github.com/mirror-browser/browser-agent-protocol" },
    // TODO: Syntax
  ]
}
```

```typescript
// Agent loop pseudocode
let messages = [{
  role: "system"
  content: agent.systemPrompt
}, {
  role: "user",
  content: agent.userPrompt
}]

while (true) {
  let nextMessage = llmProvider.chat({
    messages,
    tools: agent.nativeTools
  });
  messages.push(nextMessage)

  if nextMessage.role == "toolCall" {
    const response = agent.callNativeTool({ name: nextMessage.name, parameters: nextMessage.parameters })
    messages.push({ role: "toolReply", content: response })
    // Continue to next loop
  } else {
    // Otherwise, wait for further instruction from user
    message.push({ role: "user", content: await getUserChatInput() })
  }
}
```

## Available Adapters
- Mirror will maintains sets of public adapters, which to be used as default adapters for Mirror Browser

<!-- ---

## 🧰 Using BAP in Your Agentic Browser

1. Fork this repo
2. Add or modify Adapters under `/adapters`
3. Point your agent/browser to your custom toolbox:

   ```ts
   toolbox: "https://github.com/yourname/bap"
   ```

4. Test locally
5. Open a PR — once merged, everyone can use it!

---

## 🛠 Contributing

1. Pick a website you want to automate
2. Create a new Adapter under `adapters/<domain>/`
3. Define your actions using `run()`
4. Submit a PR with real test coverage

We review contributions regularly and prioritize sites based on demand. -->