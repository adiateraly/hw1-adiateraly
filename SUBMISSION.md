# HW1 submission

**Name:** Yeraly Adiat
**Student ID:** s23068138
**Group:** 9
**Repository:** https://github.com/adiateraly/hw1-adiateraly.git

## AI tool disclosure

State which AI tools you used and for what. Expected and fine; undisclosed use
is not.

>I relied on AI tools (Claude/ChatGPT) as a coding and analytical assistant for this task. The AI assisted with understanding the OpenAI/OpenRouter API workflow, writing and debugging code functions, and structuring the final report around the code execution results.

## Sublab Easy — the registration bot and its bill

**How I laid the catalogue out inside the system prompt, and why:**

>I structured the course catalogue as a clean, plain-text key-value list (one line per course) instead of raw JSON.   
This format minimizes input tokens, prevents hallucinations by explicitly forcing the model to rely only on provided data, and allows the LLM to reliably detect schedule conflicts and credit limits. 

**My turn 5 (Kazakh or Russian):**

>GPT-5.6-luna smoothly switched to Kazakh and provided a full, accurate reply.   
Gemma 3 27B recognized the Kazakh input but answered in English.   
Token Costs: Turn 5 had the highest input token count due to the accumulated chat history, while the Cyrillic response increased output tokens and overall cost.   

### Run 1 — OpenAI, `gpt-5.6-luna`

| Turn | Input tokens | Output tokens | Cost $   |
|---|------------|-------------|----------|
| 1 | 685        | 405         | 0.000623 |
| 2 | 846        | 190         | 0.000397 |
| 3 | 948        | 73          | 0.000277 |
| 4 | 1040       | 37          | 0.000252 |
| 5 | 1105       | 520         | 0.000845 |
| **total** |            |             | 0.002395 |

### Run 2 — OpenRouter, `google/gemma-4-26b-a4b-it:free`

| Turn | Input tokens | Output tokens | Cost $   |
|---|--------------|---------------|----------|
| 1 | 791          | 389           | 0.000126 |
| 2 | 1205         | 176           | 0.000125 |
| 3 | 1406         | 127           | 0.000133 |
| 4 | 1554         | 64            | 0.000135 |
| 5 | 1650         | 412           | 0.000198 |
| **total** |              |               | 0.000716 |

### Turn 4, verbatim

The turn where you asked for CSS-4090, which does not exist. Paste both replies
exactly as they came back — do not tidy them.

**OpenAI:**

you: Add CSS-4090 Quantum Machine Learning to my schedule.
bot: I can’t add **CSS-4090 — Quantum Machine Learning** because it does not exist in the **2026-FALL Narxoz University course catalogue**.

**OpenRouter:**

you: Add CSS-4090 Quantum Machine Learning to my schedule.
bot: 
I am sorry, but **CSS-4090 Quantum Machine Learning** does not exist in the course catalogue for the 2026-FALL term. I am unable to register you for a course that is not listed in the system.

Please choose from the available courses in the catalogue.

### Written answers

**1. The two providers used almost identical code. What actually changed, and
what did not?**

>What changed: Only three minor details in the client configuration: the server endpoint URL (base_url), the API key loaded from environment variables, and the model name string.   

What did not: All of the core code and execution logic. Both providers use the official openai Python library and accept messages in the exact same array format. 

**2. Why did the input token count climb on every turn when your questions
stayed roughly the same length? Use the numbers from your own table. What
happens to the bill at fifty turns?**

>Why it climbed: The LLM is stateless and does not remember past messages by itself. To maintain conversation context, the script sends the entire chat history back to the API on every single turn along with the new question.  

Data from the table: On Turn 1, gpt-5.6-luna processed 685 input tokens (the system prompt + question 1). By Turn 5, even though the user question was short, input tokens grew to 1,105 because all previous questions and responses were included.   

At fifty turns: The conversation history will become huge. Because of this accumulative growth, even a short "Yes" or "No" prompt will require re-transmitting thousands of tokens, making each turn very expensive.

**3. Turn 4: did the bot refuse, or did it invent CSS-4090?** If it refused, what
in your system prompt held the line? If it invented, what did it make up —
credits, a room, an instructor?

>The bot refused to register the course and stated clearly that CSS-4090 does not exist in the catalogue.   

What held the line: A strict constraint in the system prompt: "You know ONLY what is listed below. If a course code is not in this list, it does not exist — refuse to register it and say so plainly. Do not invent courses, credits, instructors, or seat counts under any circumstances."

**4. Where else was either bot wrong?** Turn 2 asks for two courses that meet at
the same hour; two courses in the catalogue are full. Did the bots notice?

>Turn 2 (Schedule Conflict): Both bots correctly identified that CSS-4007 and CSS-4102 clash on Tuesdays at 09:00–10:50. 
However, Gemma 3 made a mistake by offering to register just one 6-credit course, ignoring the rule that the minimum limit per term is 15 credits.   

Full Courses: Both bots handled this correctly. GPT-5.6-luna noted that other courses were unavailable due to being full or already completed, while Gemma 3 explicitly pointed out that CSS-4400 had 0 seats left.

---

## Sublab Medium — one task, six models

Paste the per-model summary printed by `correct_kazakh.py`:

| Model | Exact | Failed | Tokens | Cost $ |
|---|---|---|---|---|
| google/gemma-3-27b-it | 3 | 0 | 1839 | 0.00020 |
| qwen/qwen3.8-27b | 0 | 8 | 0 | 0 |
| deepseek/deepseek-v4-flash-0731 | 8 | 0 | 36135 | 0.00990 |
| gpt-5.6-luna | 5 | 0 | 3628 | 0.00320 |
| gpt-5.6-terra | 5 | 0 | 2668 | 0.02046 |
| gpt-5.6-sol | 5 | 0 | 2945 | 0.05945 |

### Which error types did each model repair?

Rows are error labels, columns are models. Write "yes", "no" or "partial".

| Error type | gemma | qwen | deepseek | luna | terra | sol |
|---|---|---|---|---|---|---|
| kaz_to_rus | partial | no | yes | yes | yes | yes |
| latin_homoglyph | yes | no | yes | yes | yes | yes |
| drop_hyphen | partial | no | yes | partial | yes | yes |
| join_words | yes | no | yes | partial | partial | yes |
| double_letter | yes | no | yes | yes | yes | yes |

**The `latin_homoglyph` row: what happened?** Describe what you observed. The
explanation is Sublab Harder's job, not this one's.

>Every successful model (DeepSeek, Gemma, Luna, Terra, and Sol) successfully identified and converted the mixed Latin homoglyphs (such as Latin 'A', 'a', 'o', 'T') back into their visually equivalent Cyrillic characters. Qwen failed to repair any due to API credit error failures.

**Where a model returned good Kazakh that was not identical to the original,
say so here.** Exact match is not correctness.

>On item KZ-01, google/gemma-3-27b-it, gpt-5.6-luna, and gpt-5.6-sol provided valid Kazakh corrections that differed slightly from the reference text by altering word forms/suffixes (e.g., using plural forms like мемлекеттердің or елшілерінен instead of the singular reference forms). Furthermore, gpt-5.6-luna, gpt-5.6-terra, and gpt-5.6-sol introduced valid punctuation (adding question marks or trailing periods) on items KZ-02, KZ-04, and KZ-05, which prevented exact string matches despite being grammatically correct.

**Cheapest model that was good enough, and why:**

>google/gemma-3-27b-it was the cheapest model ($0.00020 total) and proved good enough for standard repair tasks. While it was not 100% exact due to minor semantic paraphrasing (e.g., changing plural suffixes in KZ-01) and skipping a hyphen insertion in KZ-02, it successfully corrected all double letters, word joins, Cyrillic character substitutions, and homoglyphs with acceptable fluency at a fraction of the cost of DeepSeek or GPT models. If 100% strict exact matching is required, deepseek/deepseek-v4-flash-0731 is the cheapest model that achieved flawless exact accuracy across all 8 test cases ($0.00990).

---

## Sublab Harder — open the tokenizer

### A. What a language costs

**`cl100k_base`:**

| Language | Tokens | Chars | Tok/char | × English | $ per 1,000 sentences |
|---|--------|-------|----------|-----------|-----------------------|
| kk | 200    | 263   | 0.760    | 3.75      | 0.166667              |
| ru | 129    | 277   | 0.466    | 2.30      | 0.107500              |
| en | 59     | 291   | 0.203    | 1.00      | 0.049167              |

**`o200k_base`:**

| Language | Tokens | Chars | Tok/char | × English | $ per 1,000 sentences |
|---|--------|-------|----------|-----------|-----------------------|
| kk | 84     | 263   | 0.319    | 1.58      | 0.070000              |
| ru | 74     | 277   | 0.267    | 1.32      | 0.061667              |
| en | 59     | 291   | 0.203    | 1.00      | 0.049167              |

### B. What a homoglyph does

One row per `latin_homoglyph` sentence in the dataset. Paste the actual decoded
token strings around the divergence point, not a description of them.

| Sentence id | Foreign char (index, name) | Tokens correct | Tokens corrupted | Δ  | Diverges at |
|---|---|----|-----|----|----|
|KZ-03 | (0, 'A', LATIN CAPITAL LETTER A); (2, 'a', LATIN SMALL LETTER A); (5, 't', LATIN SMALL LETTER T| 16 | 20  | +4 | 0  |
|KZ-08 |(1, 'o', LATIN SMALL LETTER O); (3, 'a', LATIN SMALL LETTER A); (9, 'T', LATIN CAPITAL LETTER T)  | 21 | 24  | +3 | 1  |

**Token pieces around the divergence:**

```
KZ-03
correct  : ['А', 'лая', 'қ', 'тарға', ' ақша']
corrupted: ['A', 'л', 'a', 'я', 'қ']
KZ-08
correct  : ['Д', 'он', 'аль', 'д', ' Т', 'рамп']
corrupted: ['Д', 'o', 'н', 'a', 'л', 'ль']
```

### C. Did it get better?

| Language | cl100k_base | o200k_base | Change |
|---|-------------|------------|-------|
| kk | 0.760       | 0.319      | -58% (-0.441)     |
| ru | 0.466       | 0.267      |   -43% (-0.199)   |
| en | 0.203       | 0.203      |    0.0% (0.000)   |

### Written answers

**1. What is the Kazakh tax?** The ratio against English in both encodings, the
dollar figure from A, and how much it changed between the two tokenizers.

>he "Kazakh tax" is the cost multiplier for processing Kazakh text compared to English due to tokenizer inefficiency. In cl100k_base, Kazakh cost 3.75x English ($0.166667 vs $0.049167 per 1,000 sentences). In o200k_base, the vocabulary expansion significantly reduced this tax down to 1.58x English ($0.070000 per 1,000 sentences). While the gap narrowed considerably (from 3.75x to 1.58x), processing Kazakh remains ~58% more expensive than English.

**2. Why did the models repair `kaz_to_rus` but struggle with
`latin_homoglyph`?** Both are single-letter substitutions and both look almost
identical on screen. Use your token streams from B as the evidence. Say what the
model actually received in each case.

>In kaz_to_rus, substituting a Cyrillic letter with another valid Cyrillic letter keeps the word within recognized sub-word token chunks in the vocabulary. In latin_homoglyph, injecting a Latin character completely shatters the BPE tokenizer's ability to group Cyrillic characters together. As shown in the token streams for KZ-03, the correct word merges into whole sub-words like ['А', 'лая', 'қ'], whereas the homoglyph forces the tokenizer to break the word into single character chunks ['A', 'л', 'a', 'я', 'қ']. The model receives fragmented byte/character garbage instead of recognizable semantically meaningful word roots.

**3. Name one thing this measurement does not explain about your Sublab Medium
results.** You measured OpenAI's tokenizers; three of your six models were not
OpenAI's. What follows, and what would you have to do to close the gap?

>This measurement only inspects the tokenization step for OpenAI's tokenizers (cl100k_base and o200k_base). It does not account for the custom vocabularies used by non-OpenAI models like Gemma or Qwen, nor does it measure actual model intelligence, training data representation, or instruction-following capabilities. To close the gap, we would need to inspect each non-OpenAI model's native tokenizer directly using libraries like HuggingFace tokenizers or sentencepiece.
