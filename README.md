# LLM rubric evaluation pipeline

The response-generation and rubric-grading pipeline built for **"Emotional Framing as a Control
Channel: Effects of Prompt Valence on LLM Performance"** (GenProCC Workshop, NeurIPS 2025) —
[paper on OpenReview](https://openreview.net/pdf?id=l3YyW4JEgQ).

Two notebooks, one job each: send a prompt set to any of three model providers and collect the
answers, then score those answers against a fixed rubric with an LLM judge.

```
notebooks/
  generate_responses.ipynb   prompt set -> model answers   (OpenAI / Anthropic / Gemini)
  grade_responses.ipynb      model answers -> rubric scores (7 dimensions, two rubric variants)
examples/
  sample_prompts.json        6 synthetic prompts, input to the generator
  sample_answered.json       3 synthetic answered records, input to the grader
```

**Generation** (`generate_responses.ipynb`) wraps the three provider SDKs behind one
`BaseProvider` interface with shared retry and exponential backoff, caches by prompt text so an
interrupted run resumes without repaying for completed calls, checkpoints every 25 records, and
writes a run manifest recording library versions, seeds, and the input file.

**Grading** (`grade_responses.ipynb`) scores each answer on seven dimensions — relevance,
factual accuracy, coherence, depth, linguistic quality, instruction sensitivity, and creativity
— under either of two rubric variants: `original`, which scores each dimension independently
from 0–5, or `anchored`, which starts every dimension at 2.50 and adjusts. Judge output is
parsed defensively: direct JSON parse first, regex extraction of the outermost object as a
fallback, then coercion into a fixed `{scores, total, comments, rationales}` shape so a
malformed judge response degrades to a recoverable record rather than killing the run.

## Setup

```bash
pip install -r requirements.txt
cp .env.example .env     # fill in only the providers you plan to call
```

Keys are read from the environment. No key is ever written into a notebook.

Run `generate_responses.ipynb` first, pointing `PROMPTS_PATH` at your prompt set, then
`grade_responses.ipynb` with `ANSWERS_PATH` set to the generator's output. Both notebooks ship
pointed at the files in `examples/` so they run end to end before you supply real data.

## Scope

This repository contains only the pipeline code I wrote. The paper is joint work with
Ethan Hin, Shu Ze (Wayne) Chen, Tiki Li, Ayo Akinkugbe, and Kevin Zhu.

Deliberately not included, because they belong to the joint project rather than to this
pipeline: the valence prompt sets and the script that generates them, the experimental results
and per-response records, the figure and table notebooks, and the statistical analysis. The
findings themselves are in the paper. The `examples/` data here is synthetic, written for this
repository, and is not the prompt set used in the study.

## Known limitations

The two notebooks each carry their own copy of the provider classes and JSON helpers — same
retry constants, near-identical structure, differing only in whether the call returns free text
or JSON. They evolved in parallel rather than from a shared import. Extracting one provider
module both notebooks import is the first change I would make.

## Citation

```bibtex
@inproceedings{felixpena2025emotional,
  title     = {Emotional Framing as a Control Channel: Effects of Prompt Valence on LLM Performance},
  author    = {Felix-Pena, Enmanuel and Hin, Ethan and Chen, Shu Ze and Li, Tiki and
               Akinkugbe, Ayo and Zhu, Kevin},
  booktitle = {GenProCC Workshop, NeurIPS},
  year      = {2025},
  url       = {https://openreview.net/pdf?id=l3YyW4JEgQ}
}
```

## License

MIT — see [LICENSE](LICENSE).
