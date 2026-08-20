# Tokens
LLMs don't understand text — they only understand numbers. So before anything reaches the model, your text gets converted into numbers through a process called tokenization.

Here's how it works:

### Step 1 — Split into subwords
A tokenizer algorithm breaks your text into smaller units called tokens. Tokens are not always full words — they are subword units. A common word like "cat" gets one token, but a rare or complex word like "unbelievable" might get split into multiple tokens like "un", "believ", "able" — each getting its own token.

### Step 2 — Convert to numbers
Each token gets mapped to a unique number from a fixed lookup table called a vocabulary. Think of it like a dictionary where every known subword has a fixed index assigned to it — not random, always the same number for the same token.

```text
"Hi, how are you?"  →  ["Hi", ",", " how", " are", " you", "?"]  →  [2, 15, 284, 389, 345, 30]
```

### Step 3
Model processes numbers, outputs numbers
The LLM receives the token IDs, processes them, and outputs a new sequence of token IDs as the response.

### Step 4
Convert back to text
The output token IDs get passed back through the vocabulary lookup table and converted into readable text — which is the response you see.

```text
[2, 15, 284, 389]  →  "Hi, I am fine"
```
### Why this matters practically:
- You pay per token, not per word or character
- 1 token ≈ 4 characters ≈ 0.75 words in English
- Rare words, technical terms, and non-English text cost more tokens because they get split into more pieces
- The context window limit is measured in tokens, not words

# Context Window
The context window is the total amount of text — measured in tokens — that the LLM can see and process at one time, including both your input and the model's output. Think of it as a fixed-length tape with a shared budget. Everything you send in a single API call — system prompt, user message, and previous assistant responses — plus the model's generated response, all share that same budget.

```text
[system tokens] [user tokens] [assistant tokens] [new user tokens] [output tokens]
|_________________________________________________________________________|
                              context window
                             (e.g. 200,000 tokens)
```

### Why it matters:

- Every API call must fit within the limit — system + user + all assistant history combined
- Longer conversations consume more of the window on every call
- Once the limit is hit, you need to summarize or drop older messages to make room for new ones

# Temperature
Temperature controls how random or predictable the model's output is. It works by adjusting the probability distribution over the next possible token before the model picks one.

- Low temperature (0.0 – 0.3) — the model almost always picks the highest probability token. Output is consistent, predictable, and focused. Run the same prompt twice and you get nearly the same answer.
- High temperature (0.7 – 1.0+) — the probability gets flattened across more tokens, so less likely options get a real chance of being picked. Output is more varied and creative, but less reliable.

```text
Prompt: "The capital of France is"

Temperature 0.0  →  "Paris"         (always, deterministic)
Temperature 0.7  →  "Paris"         (usually, occasionally varies in phrasing)
Temperature 1.5  →  "a city of..."  (unpredictable, creative)
```
When to use what:

| Task | Temperature |
| --- | --- |
| Data extraction, JSON output, classification | 0.0 |
| Q&A, summarization, reasoning | 0.1 - 0.3 |
| Conversational chat | 0.5 - 0.7 |
| Creative writing, brainstorming | 0.8 - 1.0+ |

# top_p (Nucleus Sampling)
Instead of considering all tokens in the vocabulary, top_p tells the model to only consider the smallest group of top tokens whose combined probability adds up to the top_p value — then picks from that group only.
Example:

If top_p = 0.9, the model looks at the vocabulary, ranks tokens by probability, and keeps adding them until the cumulative probability hits 90% — then it only samples from that shortlist.

```text
Token      Probability
"Paris"    0.60  ← include
"Lyon"     0.20  ← include (cumulative = 0.80)
"France"   0.10  ← include (cumulative = 0.90) ← stop here
"London"   0.05  ← excluded
"Berlin"   0.03  ← excluded
...rest    ...   ← excluded

top_p = 0.90 → model picks from ["Paris", "Lyon", "France"] only
```
- Low top_p (0.1 – 0.3) — very small shortlist, only the most probable tokens, more focused and consistent output
- High top_p (0.9 – 1.0) — larger shortlist, more tokens in the pool, more varied output

# top_k
top_k filters the vocabulary down to the top K highest probability tokens and the model only picks from that fixed shortlist — everything else gets excluded regardless of its probability.

Example:

If top_k = 3, the model ranks all tokens by probability and keeps only the top 3:
```text
Token      Probability
"Paris"    0.60  ← include
"Lyon"     0.20  ← include
"France"   0.10  ← include
"London"   0.05  ← excluded
"Berlin"   0.03  ← excluded
...rest    ...   ← excluded

top_k = 3 → model picks from ["Paris", "Lyon", "France"] only
```

- Low top_k (1 – 10) — very tight shortlist, more deterministic output
- High top_k (50 – 100+) — larger shortlist, more varied output

# max_tokens
max_tokens sets the maximum number of tokens the model can generate in its response. It only applies to the output — your input (system + user + assistant messages) is not counted against it.

Once the model hits that limit it stops generating and returns whatever it has produced so far — even if the response is incomplete.

```python
client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=100,   # response will be cut off after 100 output tokens
    messages=[...]
)
```

# Message Roles
When you make an LLM API call, you don't just send a single string — you send a structured list of messages, where each message has a role and content. The role tells the model who is speaking, and the content is what they said.

There are three roles:

## System Role
A system role is a special instruction block given to the LLM at the very top of every conversation, before any user message. It's used to define the model's persona, tone, behavioral rules (guardrails), output format, and — in agentic systems — the tools it has access to and when to use them.

### Example
You are an expert clinical documentation specialist. Your job is to extract structured clinical information — HPI, ROS, Physical Exam, etc. — from a patient-provider conversation transcript. Maintain a professional, clinical tone throughout.

#### Guardrails
Only extract information directly from the transcript. Do not infer, assume, or add anything not explicitly stated. If asked to do anything other than extraction, respond with: "Sorry, I can't help with that." Always provide evidence (character positions) for every extracted value.

#### Output Format
Return a JSON array only — no extra text:
```json
[
  {
    "section": "HPI",
    "value": "Patient reports chest pain for 3 days...",
    "evidence": [{ "start": 42, "end": 97 }]
  }
]
```
In API call you pass this under the system role
```json
{
  "role": "system",
  "content": "You are an expert clinical ....]"
}
```


## User Role
The user role sends the human's input to the model. This is where you pass the actual data or question you want the model to work with. In an agentic system, this isn't always a person typing — it can be programmatically constructed, like injecting a transcript, a document, or tool results.

### Example
Using our clinical documentation example — the user role passes the patient-provider transcript to the model in a single API call:

```json
{
  "role": "user",
  "content": """
    ## Transcript

    Provider: Good morning, Mr. Davis...
    Patient: Yeah, it started yesterday afternoon...
    ...
  """
}
```
The model reads this alongside the system prompt and extracts the structured clinical information accordingly.

**Response:**
```json
[
  {
    "section": "HPI",
    "value": "Patient reports chest tightness that started yesterday afternoon during light weeding, described as a heavy pressure in the middle of the chest (severity 5-6/10), lasting 15-20 minutes, with radiation to the left side of the neck, accompanied by mild shortness of breath and cold sweat, resolving with rest. Mild recurrence noted the following morning while climbing stairs.",
    "evidence": [
      { "start": 42, "end": 546 }
    ]
  }
]
```

## Assistent Role
The assistant role contains the model's previous responses. You pass them back to the LLM on each API call so it has context of what it already said — this is how conversation history works. The model has no memory of its own, so your code is responsible for maintaining that list and sending it back every time.

**Example — multi-turn chat:**
```json
messages = [
    {
        "role": "user",
        "content": "## Transcript\n\nProvider: Good morning Mr. Davis..."
    },
    {
        "role": "assistant", # previous response passed back
        "content": '[{"section": "HPI", "value": "Patient reports chest tightness..."}]'
    },
    {
        "role": "user", # new follow-up question
        "content": "Now extract the ROS section from the same transcript."
    }
]
```

The model reads the full list top to bottom — it sees what the user asked, what it previously answered, and what the user is asking now. It responds accordingly.

**Note:**
The assistant role can also be used to prefill the model's response — you pass a partial assistant message and the model continues from where you left off. Useful for forcing a specific format like making it start with [ to guarantee JSON output.

```json
{"role": "assistant", "content": "["}  # model continues from here
```

### How It Works Under the Hood
All three roles get merged into a single token sequence and fed to the model in order — system first, then user, then assistant. The model reads it top to bottom like a single document and generates the next token from there.

```text
{system_start} ... {system_end}
{user_start} ... {user_end}
{assistant_start} ... {assistant_end}
```

Position matters because of how attention works. Earlier content sets the context for everything that follows. When the model reads the user message, it's already been shaped by the system prompt — so the system prompt doesn't need special authority, it just goes first. Same reason a contract's opening clause influences how you interpret every clause after it — not because it's bold or highlighted, but because you read it first.