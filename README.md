# Warranty Agent for n8n

A personal Telegram bot built with n8n that keeps track of warranties on everything you buy.

- **Send a receipt.** Snap a photo of a paper receipt or forward a PDF invoice to the bot.
- **Claude reads it.** It picks the items that come with a warranty (and skips food, bags and services), checks whether the receipt states a shorter or longer warranty, and asks you to confirm.
- **Everything is stored in Airtable**, including the receipt itself, so you can throw the paper away. In many countries a photo of the receipt is enough proof of purchase.
- **Clothing and shoes:** the bot asks for a photo of the product so you can recognise it later.
- **Morning reminders:** 6 months, 1 month and 1 week before a warranty ends, the bot messages you with the known defects of that model, found on the web. That way you can still file a claim in time.
- **Ask anything:** "Until when is my coffee machine under warranty?", "What expires this year?"

The bot replies in the language you write in.

## How it works

| Part | What it does |
|---|---|
| **n8n Agent "Warranty Agent"** | Talks to you in Telegram, sends proposals for confirmation and remembers pending purchases. It has web search enabled and a scheduled task, "Morning warranty check", that runs every day at 9:00. |
| **Workflow "Warranty Agent - Tools"** ([workflow/warranty-agent-tools.json](workflow/warranty-agent-tools.json)) | One workflow that the agent calls as 7 tools: `read_receipt`, `save_purchase`, `save_product_photo`, `search_products`, `warranties_due`, `mark_notified` and `get_settings`. |
| **Airtable base** | The tables `Settings`, `Receipts` and `Products`. See [airtable/schema.md](airtable/schema.md). |

```
Telegram ──► n8n Agent (Claude) ──► "Warranty Agent - Tools" workflow ──► Airtable
                 │                          │
                 └── web search             └── Claude (reads receipts, describes product photos)
```

Each person runs their **own copy**: their own n8n, their own Telegram bot, Airtable base and Anthropic API key. No data is shared between users.

## What you need

- **n8n** with the Agents feature (n8n Cloud or self-hosted).
- **A Telegram bot.** Create one with `@BotFather` (`/newbot`) and copy its token.
- **An Anthropic API key** from [console.anthropic.com](https://console.anthropic.com). A claude.ai subscription does not include API access; the API is billed separately, per use.
- **An Airtable account.** The free plan is enough for one person.

## Setup

### 1. Airtable
Create the base and the three tables exactly as described in [airtable/schema.md](airtable/schema.md). Fill in the one row in `Settings`, including your Telegram user ID.

### 2. Credentials in n8n
In **Credentials → Add credential** create:
1. **Telegram API**: your bot token.
2. **Anthropic API**: your API key.
3. **Airtable Personal Access Token**: create the token at [airtable.com/create/tokens](https://airtable.com/create/tokens) with the scopes `data.records:read`, `data.records:write` and `schema.bases:read`, limited to your warranty base.

### 3. Import the tools workflow
1. In n8n choose **Import from file** and import [workflow/warranty-agent-tools.json](workflow/warranty-agent-tools.json).
2. Open the **Config** node and set `airtable_base_id`. You can also change:
   - `model`: the Claude model, `claude-sonnet-5-5` by default,
   - `reminder_days`: when to send reminders, in days before the warranty ends (`183,31,7`),
   - `default_warranty_months`: the warranty length used when the receipt says nothing (24 months is the EU statutory minimum; change it for your country),
   - `timezone`.
3. Assign credentials:
   - **Download file from Telegram**: your Telegram credential.
   - **Claude reads PDF**, **Claude reads photo**, **Claude describes product**: your Anthropic credential. If n8n assigns "Gateway credits" automatically, switch them to your own key.
   - All HTTP Request nodes that call Airtable: *Authentication → Predefined credential type → Airtable Personal Access Token API*, then pick your Airtable credential.
4. **Save and publish** the workflow. The agent can only call published workflows.

### 4. Create the agent
In n8n create a new **Agent** named "Warranty Agent":
1. **Model:** Anthropic, `claude-sonnet-5-5` (or another Claude model), with your Anthropic credential.
2. **Instructions:** paste the contents of [agent/instructions.md](agent/instructions.md).
3. **Memory:** on. **Web search:** on (native provider).
4. **Tools:** add the workflow "Warranty Agent - Tools" **seven times**, once per tool, with the tool names, descriptions and input bindings from [agent/agent-config.json](agent/agent-config.json). In each tool the `action` input is fixed to the tool's action. Inputs the tool does not use are fixed to an empty value. Only the inputs marked `ai` are filled in by the model.
5. **Scheduled task:** add "Morning warranty check" as described in [agent/morning-task.md](agent/morning-task.md).
6. **Channel:** connect **Telegram** with your bot credential. Set the access mode to **private** and add your Telegram user ID, so the bot ignores everyone else.
7. **Publish** the agent.

> **Tip:** if you use Claude (or another AI assistant) connected to n8n through MCP, you can ask it to create the agent from [agent/agent-config.json](agent/agent-config.json). Replace `<YOUR_ANTHROPIC_CREDENTIAL_ID>` and connect Telegram afterwards.

### 5. First test
1. Send your bot "Hi". It should introduce itself.
2. Send a photo of a receipt. You should get a proposal listing the items with a warranty and their end dates.
3. Correct something in free text ("don't save the cable"), then confirm.
4. Check Airtable: the receipt with its file, and new rows in `Products`.
5. Ask: "What expires this year?"
6. To test the morning reminder, set a test product's purchase date about 18 months back and run the "Morning warranty check" task manually.

## Costs and limits

- **Airtable Free** allows 1,000 records per base, 1 GB of attachments and **1,000 API calls a month**. One receipt uses about 4–5 calls and the daily check 1–2, which is plenty for one person.
- **Anthropic API** is billed per use. Reading one receipt costs a few cents at most. The morning check runs every day, but it is a short run when nothing is due.
- Airtable accepts files up to 5 MB. Photos are shrunk to 1,600 px (JPEG 80 %) before they are saved.

## Testing without Telegram

Besides a Telegram `file_id`, the tools workflow accepts the inputs `file_base64`, `file_name` and `file_mime`, so you can send a file directly. This is useful for testing and for future imports, for example from e-mail. The agent never uses these inputs; they are fixed to empty values.

## Known limitations

- **PDFs:** the Telegram channel of n8n Agents is documented for photos. If PDF documents do not reach the agent, send a screenshot or a photo instead.
- **Not yet built:**
  - recognising a product from a photo (for example torn shoes) and finding its record,
  - a warranty claim assistant that looks up the store's claim terms and pre-fills the claim form,
  - importing receipts from e-mail.

## Privacy

Everything stays in your own n8n, Airtable and Anthropic accounts. Receipts are sent to the Anthropic API to be read. Nothing personal is hard-coded in this repository: the only value you fill in is your own Airtable base ID.

## License

MIT, see [LICENSE](LICENSE).
