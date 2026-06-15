# FitFindr — Starter Kit

This starter kit contains everything you need to begin Project 2.

## What's Included

```
ai201-project2-fitfindr-starter/
├── data/
│   ├── listings.json          # 40 mock secondhand listings
│   └── wardrobe_schema.json   # Wardrobe format + example wardrobe
├── utils/
│   └── data_loader.py         # Helper functions for loading the data
├── planning.md                # Your planning template — fill this out first
└── requirements.txt           # Python dependencies
```

## Setup

```bash
pip install -r requirements.txt
```

Set your Groq API key in a `.env` file (get a free key at [console.groq.com](https://console.groq.com)):
```
GROQ_API_KEY=your_key_here
```

## The Mock Listings Dataset

`data/listings.json` contains 40 mock secondhand listings across categories (tops, bottoms, outerwear, shoes, accessories) and styles (vintage, y2k, grunge, cottagecore, streetwear, and more).

Each listing has: `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`.

Load it with:
```python
from utils.data_loader import load_listings
listings = load_listings()
```

## The Wardrobe Schema

`data/wardrobe_schema.json` defines the format your agent uses to represent a user's existing wardrobe. It includes:

- `schema`: field definitions for a wardrobe item
- `example_wardrobe`: a sample wardrobe with 10 items you can use for testing
- `empty_wardrobe`: a starting template for a new user

Load an example wardrobe with:
```python
from utils.data_loader import get_example_wardrobe
wardrobe = get_example_wardrobe()
```

## Where to Start

1. **Read `planning.md` and fill it out before writing any code.**
2. Verify the data loads correctly by running `python utils/data_loader.py`.
3. Build and test each tool individually before connecting them through your planning loop.

## Tools

List every tool that this agent will use.

### Tool 1: search_listings

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
Search the mock listings (loaded from load_listing()) dataset for items matching the description,
optional size, and optional price ceiling.

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `description` (str): ...  Keywords describing what the user is looking for
                     (e.g., "vintage graphic tee").
- `size` (str): ...  The desired size.  Case-insensitive.  The check is skipped if it is None.
- `max_price` (float): ... The maximum price (inclusive). The filter is skipped if it is None.

**What it returns:**
<!-- Describe the return value — what fields does a result contain? -->
A list of matching listing dicts, sorted by relevance (best match first).
        

**What happens if it fails or returns nothing:**
<!-- What should the agent do if no listings match? -->
Returns an empty list if nothing matches (returns nothing) — does NOT raise an exception.  If it fails, then it also should return an empty list.

---

### Tool 2: suggest_outfit

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
Given a thrifted item and the user's wardrobe, suggest 1–2 complete outfits by asking the LLM to give advice.

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `new_item` (dict): ...   A listing dict (the item the user is considering buying).
- `wardrobe` (dict): ...   A wardrobe dict with an 'items' key containing a list of
                  wardrobe item dicts.  It may or may not be empty.

**What it returns:**
<!-- Describe the return value -->
A non-empty string with outfit suggestions given by the LLM.

**What happens if it fails or returns nothing:**
<!-- What should the agent do if the wardrobe is empty or no outfit can be suggested? -->
If the wardrobe is empty, offer general styling advice for the item.  Tell the LLM (with a string) to provide general styling advice for the given item instead.

---

### Tool 3: create_fit_card

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
Generate a short, shareable outfit caption for the thrifted find.

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `outfit` (str):  The outfit suggestion given by suggest_outfit().
- `new_item` (dict):  The listing dict for the thrifted item.

**What it returns:**
<!-- Describe the return value -->
 A 2–4 sentence caption (a string) to be used as a social media caption.

**What happens if it fails or returns nothing:**
<!-- What should the agent do if the outfit data is incomplete? -->
Return a descriptive error message, not an exception or crash.

---

## Planning Loop

**How does your agent decide which tool to call next?**
<!-- Describe the logic your planning loop uses. What does it look at? What conditions change its behavior? How does it know when it's done? -->
It will use tools 1, 2, then 3 sequentially, only moving on to the next tool once the previous tool has finished.  There should also be a system prompt that says to do something different if an "error" has been returned as specified in the spec (e.g., if tool 1 returns with an empty string, tell the agent to say "There are no matches for this item :(".

---

## State Management

**How does information from one tool get passed to the next?**
<!-- Describe how your agent stores and accesses state within a session. What data is tracked? How is it passed between tool calls? -->
The agent will be asked to remember it and will be asked to use remembered state as input into the next tool.

---

## A Complete Interaction (Step by Step)

Write out what a full user interaction looks like from start to finish — tool call by tool call. Use a specific example query.

**Example user query:** "I'm looking for a vintage graphic tee under $30. I mostly wear baggy jeans and chunky sneakers. What's out there and how would I style it?"

**Step 1:**
<!-- What does the agent do first? Which tool is called? With what input? -->
Tool 1 is called.  With parameters extracted from the text.

**Step 2:**
<!-- What happens next? What was returned from step 1? What tool is called now? -->
Tool 2 will be called after Tool 1 returns, following the guideline in the spec.  EXCEPT if Tool 1 returns an empty string, do not continue; instead, return the specified error message.

**Step 3:**
<!-- Continue until the full interaction is complete -->
If Tool 2 returns an empty list (failure case), stop and tell the agent to provide general styling advice for the item (as seen in the spec).  Else, continue and use Tool 3.

**Final output to user:**
<!-- What does the user actually see at the end? -->
If Tool 3 fails, return that the LLM outfit input is missing/incomplete; else, return the generated caption from Tool 3 and the outfit suggestions from Tool 2.

Your implementation files go in this same directory. There's no required file structure for your agent code — organize it however makes sense for your design.
