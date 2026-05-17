# LLMs as a reader

Slides for the **MeToD Workshop** (2nd Workshop on Frontiers in Measurement and Survey Methods, University of Calabria, 19 May 2026).

The talk walks through two ways to use large language models for topic extraction from text: first as a **component** (swapping the embedding stage of BERTopic for a transformer encoder), then as the **whole reader** (asking the model to read each document and assign labels with supporting quotes). The framing is measurement-first — the LLM is treated as an instrument, with a reading, a calibration, and an error budget.

A companion Jupyter notebook (`src/bertopic.ipynb`) runs the BERTopic walkthrough referenced in the deck.

- Deck source: [`slides.md`](slides.md) — built with [Slidev](https://sli.dev)
- Exported deck: [`slides-export.pdf`](slides-export.pdf)
- Speaker script: [`narrative.md`](narrative.md)
