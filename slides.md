---
theme: frankfurt
title: "How to use LLMs to extract Topics from Text"
author: Andres L. Marin
date: 2026/05/19
infoLine: true
addons:
  - slidev-addon-typst
transition: slide-left
mdc: true
fonts:
  sans: 'Inter'
  serif: 'Inter'
  mono: 'JetBrains Mono'
  weights: '300,400,500,600,700'
  italic: false
---

# LLMs as a reader

How to use them to extract Topics from Text

<div class="pt-8 opacity-70 text-sm">
2nd Workshop on Frontiers in Measurement and Survey Methods<br>
MeToD Project · University of Calabria · 19 May 2026
</div>

<!--
Workshop, not a research talk. Spine: two ways to bring LLMs into topic
extraction — as a component, then as the whole pipeline. Open with the
methods-transfer line (physics / CSS / ML, not economics).
-->

---
section: Motivation
---

# What this talk covers

<div class="grid grid-cols-2 gap-4 pt-4">
  <div class="agenda-card">
    <div class="agenda-num">1</div>
    <div class="agenda-title">Text → topics</div>
    <div class="agenda-sub">the measurement problem</div>
  </div>
  <div class="agenda-card">
    <div class="agenda-num">2</div>
    <div class="agenda-title">BERTopic</div>
    <div class="agenda-sub">traditional topic modelling</div>
  </div>
</div>

<div class="phase-bracket phase-traditional mt-2">Traditional</div>

<div class="grid grid-cols-3 gap-4 pt-4">
  <div class="agenda-card phase-llm">
    <div class="agenda-num">3</div>
    <div class="agenda-title">Approach 1</div>
    <div class="agenda-sub">LLM as embedding space</div>
  </div>
  <div class="agenda-card phase-llm">
    <div class="agenda-num">4</div>
    <div class="agenda-title">Approach 2</div>
    <div class="agenda-sub">LLM reads end-to-end</div>
  </div>
  <div class="agenda-card phase-llm">
    <div class="agenda-num">5</div>
    <div class="agenda-title">Validation</div>
    <div class="agenda-sub">when to use what</div>
  </div>
</div>

<div class="phase-bracket phase-llm mt-2">LLM-powered</div>

<div class="pt-6 text-center text-lg">

From **LLM as a component** to **LLM as the whole reader**.

</div>

---

# Topic modeling: from text to data

<div class="pb-3">

Raw text at scale is essentially opaque, humans can't read a million documents, and standard statistics can't summarize meaning.

</div>

```typst
#import "@preview/fletcher:0.5.7" as fletcher: diagram, node, edge

#let primary = rgb("#3333B3")
#let bright  = rgb("#6366f1")
#let soft    = rgb("#eef2ff")
#let ink     = rgb("#0f172a")
#let muted   = rgb("#64748b")
#let border  = rgb("#cbd5e1")

#html.frame(diagram(
  spacing: (32mm, 0mm),
  node-stroke: 0.9pt + border,
  node-inset: 14pt,
  node-fill: white,
  edge-stroke: 1.3pt + muted,
  mark-scale: 90%,

  node((0, 0),
    stack(spacing: 9pt,
      text(fill: bright, weight: 600, size: 0.9em, tracking: 0.08em)[CORPUS],
      text(fill: ink, size: 1.15em)[Survey responses],
      text(fill: ink, size: 1.15em)[Focus group transcript],
      text(fill: ink, size: 1.15em)[News articles],
      text(fill: ink, size: 1.15em)[Social media],
    ),
    stroke: 1.2pt + bright,
    fill: soft,
    corner-radius: 10pt,
    inset: 20pt,
  ),

  node((1, 0), text(fill: white, weight: 600, size: 1.15em)[Topic model],
    fill: bright, stroke: none, corner-radius: 8pt, inset: 18pt),

  node((2, 0), text(fill: white, weight: 600, size: 1.15em)[Topics & categories],
    fill: primary, stroke: none, corner-radius: 8pt, inset: 18pt),

  edge((0, 0), (1, 0), "->"),
  edge((1, 0), (2, 0), "->",
    label: text(fill: muted, size: 0.9em)[extract]),
))
```

<div class="pt-2 text-center max-w-4xl mx-auto">

**Topic modeling** is the task of discovering or assigning coherent, interpretable themes (*topics*) that organize a collection of documents. 

Topics may be word distributions (traditional) or named labels with descriptions (LLM-based). Either way, you get categories you can **count, compare, and reason about**.

</div>

---
hide: true
---

# A measurement problem, not a tech problem

This workshop is about *Measurement Tools Design* — so think of an LLM as an **instrument**.

<div class="grid grid-cols-3 gap-4 pt-4">
<div class="p-3 rounded bg-gray-100">

**An instrument…**

has a reading, a calibration, and an error budget.

</div>
<div class="p-3 rounded bg-gray-100">

**…we will calibrate**

validate against human coders; measure its agreement and bias.

</div>
<div class="p-3 rounded bg-gray-100">

**…with known limits**

reproducibility, prompt sensitivity, cost — stated up front.

</div>
</div>

<div class="pt-6 opacity-70">

Coming from physics / computational social science / ML — not economics. A methods transfer.

</div>

---
section: Traditional
---

# Topic modelling, two decades on

```typst
#import "@preview/fletcher:0.5.7" as fletcher: diagram, node, edge

#let primary = rgb("#0f766e")
#let bright  = rgb("#14b8a6")
#let soft    = rgb("#f0fdfa")
#let ink     = rgb("#0f172a")
#let muted   = rgb("#64748b")
#let border  = rgb("#cbd5e1")

#html.frame(diagram(
  spacing: (30mm, 6mm),
  node-stroke: none,
  node-fill: none,

  node((0, -1.3), stack(spacing: 3pt,
    text(fill: bright, weight: 600, size: 0.72em, tracking: 0.08em)[1999],
    text(fill: ink, weight: 700, size: 1.05em)[NMF],
    text(fill: muted, size: 0.72em)[matrix factorisation of \ the document–term matrix],
  )),
  node((1, -1.3), stack(spacing: 3pt,
    text(fill: bright, weight: 600, size: 0.72em, tracking: 0.08em)[2003],
    text(fill: ink, weight: 700, size: 1.05em)[LDA],
    text(fill: muted, size: 0.72em)[documents as mixtures \ of latent topics],
  )),
  node((2, -1.3), stack(spacing: 3pt,
    text(fill: bright, weight: 600, size: 0.72em, tracking: 0.08em)[2014],
    text(fill: ink, weight: 700, size: 1.05em)[STM],
    text(fill: muted, size: 0.72em)[LDA + document \ covariates],
  )),
  node((3, -1.3), stack(spacing: 3pt,
    text(fill: bright, weight: 600, size: 0.72em, tracking: 0.08em)[2022],
    text(fill: ink, weight: 700, size: 1.05em)[BERTopic],
    text(fill: muted, size: 0.72em)[embed → reduce → \ cluster → keywords],
  )),

  node((0, 0), box(width: 10pt, height: 10pt, radius: 100%, fill: primary)),
  node((1, 0), box(width: 10pt, height: 10pt, radius: 100%, fill: primary)),
  node((2, 0), box(width: 10pt, height: 10pt, radius: 100%, fill: primary)),
  node((3, 0), box(width: 10pt, height: 10pt, radius: 100%, fill: bright)),

  edge((0, 0), (1, 0), stroke: 2pt + border),
  edge((1, 0), (2, 0), stroke: 2pt + border),
  edge((2, 0), (3, 0), stroke: 2pt + border),

  edge((0, 0), (0, 2.2), "->", stroke: 1pt + bright),
  edge((1, 0), (1, 2.2), "->", stroke: 1pt + bright),
  edge((2, 0), (2, 2.2), "->", stroke: 1pt + bright),
  edge((3, 0), (3, 2.2), "->", stroke: 1pt + bright),

  node((0, 2.2),
    text(fill: primary, size: 0.8em, weight: 600)[word lists],
    stroke: 0.8pt + bright, fill: soft, corner-radius: 5pt, inset: 6pt),
  node((1, 2.2),
    text(fill: primary, size: 0.8em, weight: 600)[topic-word \ distributions],
    stroke: 0.8pt + bright, fill: soft, corner-radius: 5pt, inset: 6pt),
  node((2, 2.2),
    text(fill: primary, size: 0.8em, weight: 600)[topics + \ covariate effects],
    stroke: 0.8pt + bright, fill: soft, corner-radius: 5pt, inset: 6pt),
  node((3, 2.2),
    text(fill: primary, size: 0.8em, weight: 600)[clusters + \ keywords],
    stroke: 0.8pt + bright, fill: soft, corner-radius: 5pt, inset: 6pt),

  edge((-0.4, 3.5), (2.4, 3.5), stroke: 2pt + primary),
  edge((2.6, 3.5), (3.4, 3.5), stroke: 2pt + bright),
  node((1, 4.1), text(fill: primary, weight: 600, size: 0.72em, tracking: 0.1em)[WORD-COUNT STATISTICS]),
  node((3, 4.1), text(fill: bright, weight: 600, size: 0.72em, tracking: 0.1em)[SEMANTIC EMBEDDINGS]),
))
```

<div class="pt-6 text-center opacity-80">

Two decades of evolution: from **word-count statistics** (NMF, LDA, STM) to **semantic embeddings** (BERTopic). The representation of text itself keeps getting richer.

</div>

<div class="absolute bottom-6 left-0 right-0 text-center text-xs opacity-55 px-8">

Lee & Seung 1999 (*Nature*) · Blei, Ng & Jordan 2003 (*JMLR*) · Roberts et al. 2014 (*AJPS*) · Grootendorst 2022 (*arXiv*)

</div>

---

# How BERTopic works

BERTopic isn't one model, it's a **modular pipeline**.

```typst
#import "@preview/fletcher:0.5.7" as fletcher: diagram, node, edge

#let slate    = rgb("#e2e8f0")
#let slateInk = rgb("#334155")
#let indigo   = rgb("#4f46e5")
#let sky      = rgb("#0ea5e9")
#let teal     = rgb("#14b8a6")
#let violet   = rgb("#8b5cf6")
#let emerald  = rgb("#10b981")
#let muted    = rgb("#64748b")

#html.frame(diagram(
  spacing: (22mm, 6mm),
  node-stroke: none,
  node-fill: white,
  edge-stroke: 1.4pt + muted,
  mark-scale: 80%,

  node((0, 0), text(fill: white, weight: 700, size: 0.9em)[Embeddings],
    fill: indigo, corner-radius: 7pt, inset: 10pt),
  node((1, 0), text(fill: white, weight: 700, size: 0.9em)[Dim. \ reduction],
    fill: sky, corner-radius: 7pt, inset: 10pt),
  node((2, 0), text(fill: white, weight: 700, size: 0.9em)[Clustering],
    fill: teal, corner-radius: 7pt, inset: 10pt),
  node((3, 0), text(fill: white, weight: 700, size: 0.9em)[Tokenizer],
    fill: violet, corner-radius: 7pt, inset: 10pt),
  node((4, 0), text(fill: white, weight: 700, size: 0.9em)[Weighting \ scheme],
    fill: emerald, corner-radius: 7pt, inset: 10pt),

  edge((0, 0), (1, 0), "->"),
  edge((1, 0), (2, 0), "->"),
  edge((2, 0), (3, 0), "->"),
  edge((3, 0), (4, 0), "->"),
))
```

<div class="text-sm pt-4 space-y-1">

- **Embeddings:** Turn documents into vector representations -- [docs](https://maartengr.github.io/BERTopic/getting_started/embeddings/embeddings.html)
- **Dimensionality reduction:** Compress the embedding space (UMAP, PCA, t-SNE) -- [docs](https://maartengr.github.io/BERTopic/getting_started/dim_reduction/dim_reduction.html)
- **Clustering:** Group similar documents (HDBSCAN, k-means, BIRCH) -- [docs](https://maartengr.github.io/BERTopic/getting_started/clustering/clustering.html)
- **Tokenizer:** Split each cluster back into tokens for the weighting step -- [docs](https://maartengr.github.io/BERTopic/getting_started/vectorizers/vectorizers.html)
- **Weighting scheme:** Surface the most distinctive words per cluster (c-TF-IDF, BM25) -- [docs](https://maartengr.github.io/BERTopic/getting_started/ctfidf/ctfidf.html)

</div>

<div class="pt-3 text-center">

Every stage is swappable. Today we focus on **one stage: the embeddings**.

</div>

<div class="absolute bottom-6 left-0 right-0 text-center text-xs opacity-55 px-8">

Grootendorst 2022 (*arXiv:2203.05794*) · `maartengr.github.io/BERTopic`

</div>

---
hide: true
---
# Where traditional methods hurt

<div class="grid grid-cols-2 gap-8 pt-2">
<div>

- **You pick *K*** — the number of topics is an input, not a result
- **Bag-of-words** (LDA/NMF) — word order and context thrown away
- **Topics ≠ labels** — output is word lists you must interpret
- **Instability** — re-runs and seeds shift the topics

</div>
<div>

- **Heavy preprocessing** — stop-words, stemming, n-grams, tuning
- **Short text struggles** — tweets, one-line survey answers
- **Embeddings are only as good as the embedding model**

</div>
</div>

<div class="pt-6 text-lg">

That last point is the opening: **what if the embedding model were an LLM?**

</div>

---
section: Approach 1
---

# Two ways to bring LLMs in

```typst
#import "@preview/fletcher:0.5.7" as fletcher: diagram, node, edge

#html.frame(diagram(
  spacing: (10mm, 12mm),
  node-stroke: 0.5pt,
  node((-1.3,0), text(weight: "bold")[Approach 1], stroke: none),
  node((0,0), [Documents]),
  node((1.7,0), [LLM \ embeddings], fill: blue.lighten(80%)),
  node((3.6,0), [BERTopic \ UMAP + HDBSCAN]),
  node((5.4,0), [Topics]),
  edge((0,0), (1.7,0), "->"),
  edge((1.7,0), (3.6,0), "->"),
  edge((3.6,0), (5.4,0), "->"),
  node((-1.3,1), text(weight: "bold")[Approach 2], stroke: none),
  node((0,1), [Documents]),
  node((2.6,1), [LLM reads \ with a codebook], fill: blue.lighten(80%)),
  node((5.4,1), [Topics]),
  edge((0,1), (2.6,1), "->"),
  edge((2.6,1), (5.4,1), "->"),
))
```

- **Approach 1** — replace *one box*: the embedding model. BERTopic does the rest.
- **Approach 2** — replace *the whole pipeline*: the LLM reads and names topics itself.

We do both — starting with Approach 1.

---

# Approach 1: LLM as the embedding space

Keep BERTopic. Swap its embedding model for **LLM embeddings**.

```typst
#import "@preview/fletcher:0.5.7" as fletcher: diagram, node, edge

#html.frame(diagram(
  spacing: (8mm, 8mm),
  node-stroke: 0.5pt,
  node((0,0), [Documents]),
  node((1,0), [LLM \ embeddings], fill: blue.lighten(75%), stroke: 1pt),
  node((2,0), [UMAP \ reduce dims]),
  node((3,0), [HDBSCAN \ cluster]),
  node((4,0), [c-TF-IDF \ topic words]),
  node((5,0), [Manual \ labelling], fill: red.lighten(85%)),
  edge((0,0), (1,0), "->"),
  edge((1,0), (2,0), "->"),
  edge((2,0), (3,0), "->"),
  edge((3,0), (4,0), "->"),
  edge((4,0), (5,0), "->"),
))
```

Swap one box. Everything downstream is **unchanged, traditional BERTopic**.

---

# Approach 1: what changes, what stays

<div class="grid grid-cols-2 gap-8 pt-2">
<div>

**Better embeddings help with**

- short, noisy text (tweets, one-liners)
- multilingual corpora
- domain-specific meaning
- semantic clusters, not keyword clusters

</div>
<div>

**Still your job — still traditional**

- tune UMAP / HDBSCAN, handle outliers
- topics still come out as **word lists**
- you still **read and label** them
- the LLM improved the *representation*, not the *reading*

</div>
</div>

<div class="pt-4 opacity-80">

That last point is exactly what **Approach 2** changes.

</div>

---

# Approach 1 in practice

Precompute LLM embeddings, hand them to a standard BERTopic pipeline:

```python
from bertopic import BERTopic
from openai import OpenAI
client = OpenAI()

# 1 — embed documents with an LLM embedding model
def embed(docs):
    resp = client.embeddings.create(
        model="text-embedding-3-large", input=docs)
    return [d.embedding for d in resp.data]
embeddings = embed(documents)

# 2 — hand them to traditional BERTopic (UMAP + HDBSCAN + c-TF-IDF)
topic_model = BERTopic()
topics, probs = topic_model.fit_transform(documents, embeddings=embeddings)
```

**One argument changes.** The pipeline is the same.

<!--
Show on screen: the BEV-news corpus, the topics this produced, and a
default-embedding BERTopic run side by side.
-->

---
section: Approach 2
---

# Approach 2: skip BERTopic — the LLM reads

No embedding space, no clustering. The LLM **reads each document** and names topics directly.

```typst
#import "@preview/fletcher:0.5.7" as fletcher: diagram, node, edge

#html.frame(diagram(
  spacing: (12mm, 8mm),
  node-stroke: 0.5pt,
  node((0,0), [Documents]),
  node((0,1), [Codebook / \ instructions]),
  node((2,0.5), [LLM \ reads & labels], fill: blue.lighten(75%), stroke: 1pt),
  node((4,0.5), [Topics \ named + described], fill: green.lighten(80%)),
  edge((0,0), (2,0.5), "->"),
  edge((0,1), (2,0.5), "->"),
  edge((2,0.5), (4,0.5), "->"),
))
```

Output: topics with **names and descriptions** — not word lists to decode.

---

# Two modes of LLM topic extraction

```typst
#import "@preview/fletcher:0.5.7" as fletcher: diagram, node, edge

#html.frame(diagram(
  spacing: (11mm, 9mm),
  node-stroke: 0.5pt,
  node((-1,0), text(weight: "bold")[Inductive], stroke: none),
  node((0,0), [Documents]),
  node((2,0), [LLM], fill: blue.lighten(80%)),
  node((4,0), [Discovered \ topic list]),
  edge((0,0), (2,0), "->"),
  edge((2,0), (4,0), "->"),
  node((-1,1.6), text(weight: "bold")[Deductive], stroke: none),
  node((0,1.1), [Documents]),
  node((0,2.1), [Taxonomy / \ codebook]),
  node((2,1.6), [LLM], fill: blue.lighten(80%)),
  node((4,1.6), [Assigned \ labels]),
  edge((0,1.1), (2,1.6), "->"),
  edge((0,2.1), (2,1.6), "->"),
  edge((2,1.6), (4,1.6), "->"),
))
```

**Inductive** — discover what topics exist. **Deductive** — classify into a scheme you already have.

---

# The prompt *is* your codebook

Write it the way you would brief a research assistant.

```text
ROLE: You are coding open-ended responses about <domain>.

TASK: Assign each response to one or more topics.

TOPICS (with definitions and examples):
  - price_concerns: cost, affordability, value for money
      e.g. "too expensive for what you get"
  - reliability: breakdowns, durability, trust in the product
  - ...

RULES: use only the topics above; if none fit, return "other";
       return a short supporting quote for each assignment.
```

A good codebook prompt is **explicit, exemplified, and bounded**.

---

# Make the output structured

Free text is unparseable and irreproducible. Ask for a fixed schema.

```json
{
  "response_id": "R0427",
  "topics": [
    { "label": "price_concerns", "confidence": 0.9,
      "quote": "way too expensive for a small battery" },
    { "label": "reliability",    "confidence": 0.6,
      "quote": "friend's one broke after a year" }
  ]
}
```

Use the model's **JSON / structured-output mode** — pin a schema, validate every record.

---

# Long documents: chunk, then merge

LLMs have a context limit — split, process, recombine.

```typst
#import "@preview/fletcher:0.5.7" as fletcher: diagram, node, edge

#html.frame(diagram(
  spacing: (11mm, 6mm),
  node-stroke: 0.5pt,
  node((0,1), [Long \ document]),
  node((2,0), [Chunk 1]),
  node((2,1), [Chunk 2]),
  node((2,2), [Chunk 3]),
  node((4,0), [LLM], fill: blue.lighten(80%)),
  node((4,1), [LLM], fill: blue.lighten(80%)),
  node((4,2), [LLM], fill: blue.lighten(80%)),
  node((6,1), [Merge & \ de-duplicate], fill: green.lighten(80%)),
  edge((0,1), (2,0), "->"),
  edge((0,1), (2,1), "->"),
  edge((0,1), (2,2), "->"),
  edge((2,0), (4,0), "->"),
  edge((2,1), (4,1), "->"),
  edge((2,2), (4,2), "->"),
  edge((4,0), (6,1), "->"),
  edge((4,1), (6,1), "->"),
  edge((4,2), (6,1), "->"),
))
```

Mind chunk boundaries — don't split mid-argument.

---

# Settings that matter

<div class="grid grid-cols-2 gap-8 pt-2">
<div>

**Few-shot examples**
A handful of labelled cases in the prompt anchors the codebook far better than definitions alone.

**Temperature**
Use **0** (or near-0) for coding tasks — you want consistency, not creativity.

</div>
<div>

**Model choice**
Bigger ≠ always better; test a cheap model first, escalate only if agreement is poor.

**Pin everything**
Model name, version/date, prompt, temperature — these *are* your method.

</div>
</div>

---

# A ready framework: TopicGPT

<div class="pt-2">

`TopicGPT` (Pham et al., NAACL 2024) — a prompt-based pipeline that does both modes:

</div>

```typst
#import "@preview/fletcher:0.5.7" as fletcher: diagram, node, edge

#html.frame(diagram(
  spacing: (9mm, 7mm),
  node-stroke: 0.5pt,
  node((0,0), [Generate \ topics]),
  node((1.4,0), [Refine \ merge / drop]),
  node((2.8,0), [Assign topics \ + quotes]),
  edge((0,0), (1.4,0), "->"),
  edge((1.4,0), (2.8,0), "->"),
))
```

Topics come out as **natural-language labels with descriptions**, hierarchically — and you can edit the topic list without retraining. Good baseline for Approach 2. <span class="opacity-60 text-sm">(code: github.com/chtmp223/topicGPT)</span>

---
section: Validation
---

# Does it work? Treat the LLM as a coder

Validate it exactly as you would validate a human research assistant.

```typst
#import "@preview/fletcher:0.5.7" as fletcher: diagram, node, edge

#html.frame(diagram(
  spacing: (14mm, 9mm),
  node-stroke: 0.5pt,
  node((0,1), [Sample of \ documents]),
  node((2,0), [Human coder \ gold standard], fill: orange.lighten(80%)),
  node((2,2), [LLM coder], fill: blue.lighten(80%)),
  node((4,1), [Agreement \ κ / F1 / confusion], fill: green.lighten(80%)),
  edge((0,1), (2,0), "->"),
  edge((0,1), (2,2), "->"),
  edge((2,0), (4,1), "->"),
  edge((2,2), (4,1), "->"),
))
```

Hand-code a subset, compare, report agreement — same logic as inter-rater reliability.

---

# The honest caveats

<div class="grid grid-cols-2 gap-8 pt-2">
<div>

- **Reproducibility** — Approach 1 inherits BERTopic's; Approach 2 needs care: pin model, version, prompt, temperature
- **Prompt sensitivity** — wording shifts results; run a robustness check
- **Hallucinated topics** — constrain with a taxonomy; flag low confidence

</div>
<div>

- **Systematic bias** — does coding differ across respondent subgroups?
- **Measurement error** — topic labels used as variables → attenuation / bias downstream
- **Cost & access** — embedding vs. per-token cost; open vs. closed models

</div>
</div>

<div class="pt-5 opacity-80">

Stating the error budget is what makes it a *measurement* method.

</div>

---
section: Wrap-up
---

# When to use what

<div class="text-sm leading-tight">

| | Traditional BERTopic | Approach 1 — + LLM embeddings | Approach 2 — LLM only |
|---|---|---|---|
| Short / noisy / multilingual text | weak | strong | strong |
| Output | word lists | word lists | named + described |
| You still label by hand | yes | yes | no |
| Reproducibility | high | high-ish | needs care |
| Cost | low | embedding API | per-token API |
| Discovery vs. codebook | discovery | discovery | both |

</div>

<div class="pt-3 text-lg">

Often the answer: **Approach 1 to explore, Approach 2 to read.**

</div>

---

# Resources to take home

<div class="grid grid-cols-2 gap-6 pt-2 text-sm">
<div>

**Methods & frameworks**
- Grootendorst (2022), *BERTopic* — `maartengr.github.io/BERTopic` (see *embedding models* docs)
- Pham et al. (2024), *TopicGPT*, NAACL — `github.com/chtmp223/topicGPT`
- Roberts, Stewart, Tingley et al. (2014), *Structural Topic Models for Open-Ended Survey Responses*, AJPS
- Blei, Ng & Jordan (2003), *LDA*, JMLR

**LLMs as annotators**
- Gilardi et al. (2023), *ChatGPT outperforms crowd workers…*, PNAS
- Ziems et al. (2024), *Can LLMs Transform Computational Social Science?*, Comp. Linguistics

</div>
<div>

**Rigour & reproducibility**
- Egami et al., *Design-Based Semi-Supervised Learning* — valid inference from LLM labels
- *Open-source LLMs for text annotation: a practical guide*, J. Computational Social Science (2024)
- Spirling (2023), *Open-source generative AI for science*, Nature
- Chen, Zaharia & Zou (2024), *How is ChatGPT's behavior changing over time?*, HDSR

**Companion notebook**
- `TODO` — link to the runnable Colab / Jupyter (both approaches on the BEV-news corpus)

</div>
</div>

---
layout: center
class: text-center
---

# Thank you

From **LLM as a component** to **LLM as the whole reader** — a calibratable instrument for turning text into topics.

<div class="pt-6 opacity-70">
Andres L. Marin · andres.laverde-marin@ec.europa.eu<br>
Slides & notebook: <span class="opacity-60">github.com/AndresLaverdeMarin/llm-topic-extraction</span>
</div>
