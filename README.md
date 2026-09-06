# LLM-as-a-Judge Evaluation Framework

A practical study of **LLM-as-a-Judge** using the **MT-Bench** dataset.

In this project, one LLM is used to generate answers and another LLM is used to evaluate those answers. The main focus is to investigate the **reliability, position bias, and verbosity preference** of the Judge LLM.

---

## Overview

The basic evaluation pipeline used in this project is:

```text
MT-Bench Dataset
       │
       ▼
Qwen2.5-0.5B-Instruct
     Target LLM
       │
       ▼
Generated Answers
       │
       ▼
Llama-3.2-1B-Instruct
      Judge LLM
       │
       ▼
Evaluation Results
Models Used
Component	Model	Role
Target LLM	Qwen/Qwen2.5-0.5B-Instruct	Generates answers
Judge LLM	meta-llama/Llama-3.2-1B-Instruct	Evaluates generated answers
Dataset	MT-Bench	Provides evaluation questions

The experiments were implemented in Python using Hugging Face Transformers in Google Colab.

1. MT-Bench Dataset

The project uses the MT-Bench dataset as the source of questions.

For each question, the first turn of the MT-Bench conversation is used as the input question for generating an answer.

The basic process is:

MT-Bench
   │
   ├── Question 1
   ├── Question 2
   ├── Question 3
   ├── ...
   └── Question 80

The questions are then given to the Target LLM.

2. Answer Generation

The Qwen2.5-0.5B-Instruct model is used as the Target LLM.

Its job is to generate answers to the MT-Bench questions.

MT-Bench Question
       │
       ▼
Qwen2.5-0.5B-Instruct
       │
       ▼
Generated Answer

These generated answers are subsequently passed to the Judge LLM for evaluation.

3. Judge LLM

The Llama-3.2-1B-Instruct model is used as the Judge LLM.

The Judge receives the question and the generated answer and evaluates its quality.

For the rating-based experiment, the Judge assigns a score between 1 and 10.

Question
   +
Generated Answer
        │
        ▼
Llama-3.2-1B-Instruct
        │
        ▼
     Score
     1–10

The same Judge model is used throughout the experiments.

4. Reliability Evaluation
Objective

The first experiment investigates the reliability and consistency of the Judge LLM.

The same answer is evaluated multiple times by the same Judge model.

Method

For a given question:

Question
   │
   ▼
Generated Answer
   │
   ├── Judge Evaluation 1 → Score
   ├── Judge Evaluation 2 → Score
   └── Judge Evaluation 3 → Score

Each evaluation produces a rating from 1 to 10.

For example:

Evaluation 1 → 9
Evaluation 2 → 8
Evaluation 3 → 9

The three scores are then compared to see whether the Judge produces consistent evaluations.

Measurements

The reliability analysis considers:

Mean score
Standard deviation
Score range
Exact agreement
Agreement rate

The purpose is to determine how much the Judge's evaluation changes when the same answer is evaluated repeatedly.

5. Position Bias Evaluation
Objective

The second experiment investigates whether the Judge LLM is affected by the position of an answer.

The idea is simple:

If two answers are evaluated, does the Judge choose the better answer based on quality, or does it prefer the answer that appears in a particular position?

Step 1 — Generate Two Answers

For each MT-Bench question, two answers are generated:

Question
   │
   ▼
Qwen2.5-0.5B-Instruct
   │
   ├───────────────┐
   ▼               ▼
Answer A        Answer B

The two answers are kept fixed for the evaluation.

Step 2 — Original Order

The Judge is first given:

Answer A
Answer B

The Judge chooses one of them:

A

or

B
Step 3 — Swap the Positions

The exact same two answers are then presented in reverse order:

Answer B
Answer A

The Judge evaluates them again.

A

or

B

The answers themselves are not changed. Only their positions are swapped.

Step 4 — Compare the Decisions

The two results are mapped back to the original answer identities.

For example:

Original order:

A vs B
Judge → A


Swapped order:

B vs A
Judge → A

In the second evaluation, the Judge selected the answer occupying the first position, which is now the original Answer B.

Therefore:

Original winner → A
After swapping → B

This indicates that the Judge's decision changed because the answer position changed.

Position Bias Pipeline
                 MT-Bench Question
                         │
                         ▼
                Qwen2.5-0.5B-Instruct
                         │
                  ┌──────┴──────┐
                  ▼             ▼
              Answer A       Answer B
                  │             │
                  └──────┬──────┘
                         ▼
                 Llama-3.2-1B
                    Judge
                         │
                      A vs B
                         │
                         ▼
                    Winner AB
                         │
                         ▼
                  Swap Positions
                         │
                         ▼
                      B vs A
                         │
                         ▼
                 Llama-3.2-1B
                    Judge
                         │
                         ▼
                    Winner BA
                         │
                         ▼
               Compare Decisions
                         │
                         ▼
                 Position Bias

This experiment is used to measure whether the Judge is sensitive to answer order.

6. Verbosity Evaluation
Objective

The third experiment investigates whether the Judge LLM prefers a longer answer simply because it contains more information or text.

For the same question, two new answers are generated:

Short Answer
Long Answer
Step 1 — Generate Short and Long Answers

The Qwen2.5-0.5B-Instruct model is used to generate both responses.

                 MT-Bench Question
                         │
                         ▼
                Qwen2.5-0.5B-Instruct
                         │
                  ┌──────┴──────┐
                  ▼             ▼
             Short Answer   Long Answer
Short Answer

The short-answer prompt asks the model to:

Be concise
Include the essential information
Avoid unnecessary details
Long Answer

The long-answer prompt asks the model to:

Provide more detail
Explain important points clearly
Include useful context and reasoning where appropriate
Give substantially more detail than a concise answer
Step 2 — Judge the Two Answers

The two answers are then given to Llama-3.2-1B-Instruct.

Question

SHORT ANSWER:
...

LONG ANSWER:
...

The Judge is asked to select which answer is better:

SHORT

or

LONG
Verbosity Evaluation Pipeline
                 MT-Bench Question
                         │
                         ▼
                Qwen2.5-0.5B-Instruct
                         │
                  ┌──────┴──────┐
                  ▼             ▼
             Short Answer   Long Answer
                  │             │
                  └──────┬──────┘
                         ▼
                 Llama-3.2-1B
                    Judge
                         │
                         ▼
                    SHORT / LONG
                         │
                         ▼
                 Preference Analysis

The resulting preferences are used to investigate whether the Judge tends to prefer longer responses.

7. Complete Experimental Workflow

The work completed so far can be summarized as:

                         MT-Bench
                            │
                            ▼
                  Qwen2.5-0.5B-Instruct
                       Target LLM
                            │
                            ▼
                     Answer Generation
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
        Same Answer      Answer A/B    Short/Long
             │              │              │
             ▼              ▼              ▼
       Reliability     Position Bias   Verbosity
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                  Llama-3.2-1B-Instruct
                         Judge LLM
                            │
                            ▼
                      Evaluation
8. Experiments Summary
Experiment	What was done?	Main Purpose
Reliability	Same answer evaluated 3 times with a 1–10 score	Measure Judge consistency
Position Bias	Two answers evaluated, then their positions were swapped	Detect position-dependent judging
Verbosity	Short and long versions of an answer were compared	Investigate preference for answer length
9. Output and Results

The evaluation outputs are stored in CSV/JSON format for further analysis.

The project uses separate result files for the different experiments.

results/
├── evaluation_results.csv
├── position_bias_results.csv
├── verbosity_bias_results.csv
└── reliability_results.csv

Visualizations are stored separately:

figures/
├── position_bias.png
├── verbosity_bias.png
└── reliability.png
10. Project Structure
LLM-as-a-Judge-Research/
│
├── LLM_as_a_Judge_Framework.ipynb
├── README.md
│
├── results/
│   ├── evaluation_results.csv
│   ├── position_bias_results.csv
│   ├── verbosity_bias_results.csv
│   └── reliability_results.csv
│
└── figures/
    ├── position_bias.png
    ├── verbosity_bias.png
    └── reliability.png
11. Technologies
Python
Google Colab
PyTorch
Hugging Face Transformers
Hugging Face Hub
Pandas
NumPy
Matplotlib
12. Summary

This project implements a practical LLM-as-a-Judge evaluation pipeline using MT-Bench.

The Qwen2.5-0.5B-Instruct model is used as the Target LLM to generate answers, while Llama-3.2-1B-Instruct is used as the Judge LLM.

Three evaluation experiments have been performed:

1. Reliability

The same answer is evaluated three times using a 1–10 rating to investigate whether the Judge provides consistent scores.

2. Position Bias

Two answers are generated and evaluated in their original order. Their positions are then swapped and the same answers are evaluated again to investigate whether the Judge's decision depends on answer position.

3. Verbosity

Short and long versions of answers are generated for the same questions and evaluated to investigate whether the Judge prefers longer responses.
