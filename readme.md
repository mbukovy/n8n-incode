# Introducing n8n InCode

InCode is a TypeScript framework for building, organizing, and deploying workflow automations as code.
Design complex workflows using a readable, composable API, manage them in a structured folder hierarchy, and push updates directly to your automation platform.
inCode brings modern developer tools - version control, code reviews, reusability, and CI/CD - to workflow automation.


## 🚀 Getting Started

### 1. **Install inCode**

Install the CLI and framework using npm or yarn:

```bash
npm install -g incode-cli
# or
yarn global add incode-cli
```

Inside your project folder, add the inCode library:

```bash
npm install incode
# or
yarn add incode
```

---

### 2. **Initialize Your Project**

Create a new project directory and initialize inCode:

```bash
incode init
```

This will set up the basic folder structure for your workflows.

---

### 3. **Create Your First Workflow**

Generate a sample workflow:

```bash
incode new myFirstWorkflow
```

This creates `workflows/myFirstWorkflow/workflow.ts` - edit this file to define your workflow using the inCode framework.

---

### 4. **Connect to Your n8n Instance**

Authenticate your CLI with your n8n instance:

```bash
incode login
```

Follow the prompts to set your n8n API endpoint and access token.

---

### 5. **Deploy Your Workflow**

Push your workflow to n8n with a single command:

```bash
incode push
```

Or use `--dry` to preview changes before pushing
```bash
incode push --dry
```

---

### 6. **Next Steps**

* Edit, refactor, and organize workflows in code - use your favorite IDE, git, and all modern developer tools.
* Use `incode pull` to import existing n8n workflows into code.
* Use `incode diff` to see changes between code and n8n.


## ✨ Examples

### 1. **RSS Feed to Slack**

A workflow that pulls the latest items from two RSS feeds, and sends them to a Slack channel.

<img width="864" height="385" alt="image" src="https://github.com/user-attachments/assets/b6a2b095-eaef-4ddb-8d2f-fa1460f4f435" />

```typescript
import { Workflow, multi, RssTriggerNode, SlackNode, Credentials } from "incode";

export default Workflow(
  multi([
    RssTriggerNode("Aktuality RSS", {
      feedUrl: "https://www.aktuality.sk/rss/",
      pollTimes: [{ mode: "everyMinute" }]
    }),
    RssTriggerNode("DennikN RSS", {
      feedUrl: "https://dennikn.sk/feed/",
      pollTimes: [{ mode: "everyMinute" }]
    })
  ])
  .then(
    SlackNode("Send a message", {
      channelId: "C097VHXRJSK",
      text: `={{ $json.title }}\n{{ $json.link }}`,
      credentials: Credentials("HJWdYSbuzsIRUSGR")
    })
  )
);
```

---

### 2. **StarWars API Population Checker**

This workflow checks the total population across the planets from the Star Wars API (SWAPI).
It can be triggered either manually or via a webhook. It fetches the latest planet data, sums all known populations, and then responds with a custom message:

If the total population is over 8 billion, it replies: “Oh, that's a lot of people! I mean aliens...”
Otherwise, it responds: “That's nothing, Earth has more people!”

<img width="1180" height="441" alt="image" src="https://github.com/user-attachments/assets/a0fee001-c0c4-48f8-92d7-80660edc6064" />

```typescript
import {
  Workflow,
  multi,
  ManualTriggerNode,
  WebhookNode,
  HttpNode,
  CodeNode,
  IfNode,
  RespondWebhookNode
} from "incode";

export default Workflow(
  multi([
    ManualTriggerNode(),
    WebhookNode("Webhook", { path: "run-sw-api" })
  ])
  .then(
    HttpNode("HTTP Request", {
      url: "https://swapi.info/api/planets"
    })
  )
  .then(
    CodeNode("Sum Populations", {
      code: function() {
          const totalPopulation = $input.all().reduce(
              (sum, planet) => sum + (planet.json.population === 'unknown' ? 0 : parseInt(planet.json.population)),
              0
          );
          return { totalPopulation };
      }
    })
  )
  .then(
    IfNode("Is Over 8B?", {
      conditions: [
        {
          leftValue: "={{ $json.totalPopulation }}",
          rightValue: 8_000_000_000,
          operator: { type: "number", operation: "gt" }
        }
      ],
    })
    .yes(
      RespondWebhookNode("Many Aliens", {
          respondWith: "text",
          responseBody: "Oh, that's a lot of people! I mean aliens..."
      })
    )
    .no(
      RespondWebhookNode("Not Impressed", {
        respondWith: "text",
        responseBody: "That's nothing, Earth has more people!"
      })
    )
  )
);
```

Notice how `CodeNode` expects an actual function, not a string. This allows you to test, debug, and refactor your business logic just like any other code, making workflows more robust and maintainable.

This approach also makes building and composing nodes much simpler and more reusable. For example, you can define helpers like:

```typescript
const resultResponse = (name, responseBody) =>
  RespondWebhookNode(name, {
    respondWith: "text",
    responseBody
  });
```

And then use them in your workflow to keep things concise and expressive:

```typescript
IfNode("Is Over 8B?", {
  conditions: [
    {
      leftValue: "={{ $json.totalPopulation }}",
      rightValue: 8_000_000_000,
      operator: { type: "number", operation: "gt" }
    }
  ],
})
.yes(
  resultResponse('Many Aliens', `Oh, that's a lot of people! I mean aliens...`)
)
.no(
  resultResponse('Not Impressed', "That's nothing, Earth has more people")
)
```

## 📁 Folder Structure

inCode organizes your workflows using a familiar, developer-friendly folder structure. This makes it easy to manage, version, and scale your automation projects - just like any modern codebase.

A typical project might look like this:

```
project-root/
│
├── workflows/
│   ├── clientA/
│   │   ├── invoices/
│   │   │   └── pullInvoices/
│   │   │       └── workflow.ts
│   │   └── reports/
│   │       └── monthlySummary/
│   │           └── workflow.ts
│   └── shared/
│       └── utils.ts
│
├── node_modules/
├── package.json
└── tsconfig.json
```

**How it works:**

* Each workflow lives in its own folder with a dedicated `workflow.ts` (or `.js`) file.
* Subfolders reflect your business domains, clients, or automation categories.
* inCode uses the folder path to determine the workflow’s name (if not exported explicitly) and tags in n8n, so your organizational structure in code is reflected in your automation platform.

**Best Practices:**

* Organize by client, department, or function—whatever matches your real-world needs.
* Use subfolders to group related workflows and encourage code reuse.
* Version your entire folder structure in git for easy collaboration and rollback.

This structure keeps your automations clean, scalable, and ready for real software development workflows.

>⚠
> The inCode CLI automatically discovers all `workflow.ts` files in your project by **scanning directories recursively**.
> You have **complete flexibility** over your folder structure - simply ensure each workflow resides in a file named `workflow.ts` and follows the export conventions. This way, your workflows are always picked up and managed without any manual configuration.
