<!--
PREPARATION NOTES FOR COREY (delete this block before you share the file)

Still to confirm, marked [CONFIRM] in the text:
  1. Which local models actually ran. Neither the Claude chats nor the ChatGPT notes verify the model names.
     The model names in the appendix are common public examples, not a record of what you used.
  2. The business question and the decision the resume segmentation fed into. Neither source records an outcome.
  3. Whether the "customer sentiment" work you described in the Wells Fargo interview prep was the resume
     pipeline or a separate corpus. Resumes do not carry sentiment, so these may be two different strands.
     The draft treats them as possibly separate. Fix section 4.4 to match what happened.

Where the two sources disagree or differ:
  The first draft said the market analysis classified sentiment in feedback text with a small generative model
  and JSON output. The ChatGPT notes show a resume corpus, a zero-shot classifier, and sentence embeddings with
  clustering. The ChatGPT notes are more specific, so the resume pipeline is now the main example. The generative
  JSON pattern moved to Appendix B as a related technique.

Private source links (from the ChatGPT notes; do not put these on slides):
  Segmentation persona analysis: https://chatgpt.com/g/g-p-68ffd9f48414819188522e17c119c5ed/c/68ffda4f-7490-832f-b449-badf6d0bc7b8
  Resume classification workflow: https://chatgpt.com/g/g-p-68ffd9f48414819188522e17c119c5ed/c/6900f105-a914-8324-8418-90c863755601
  ChatGPT in Work Tasks: https://chatgpt.com/c/698ba50d-5988-8330-b060-500bb41701e6
  Redshift cohort query: https://chatgpt.com/g/g-p-686bd10d1a5c81918ccbea7218318582/c/695bf05b-f960-832c-8c20-6f066de571fe
  SQL query for wages: https://chatgpt.com/g/g-p-686bd10d1a5c81918ccbea7218318582/c/68f6704e-ba30-8333-ae29-3e2ecf24cfcc
  Convert SQL to dbplyr: https://chatgpt.com/g/g-p-686bd10d1a5c81918ccbea7218318582/c/6877f3b6-9c4c-8000-94f1-89216b824120
  Synthetic data creation: https://chatgpt.com/g/g-p-684741c5f78c819180d38fc8a8f71fe2/c/68924ecb-18f8-8329-bc87-ad92005fca8d
  Generate synthetic wage data: https://chatgpt.com/g/g-p-686bd10d1a5c81918ccbea7218318582/c/68f78ce9-8290-8326-9070-4b0b26a606a0
  Synthetic data methods overview: https://chatgpt.com/g/g-p-6914deb6b6d881918250f585845a03fc/c/6914df1e-f248-8328-b8b2-64ece5badcd7
-->

# Small Models, Large Data, and the Unglamorous Work in Between

Notes for a graduate course conversation on AI in applied settings

Corey Sparks, Department of Sociology and Demography, UT San Antonio

## 1. Why I am talking about this

Most of what you hear about AI is about the biggest models, the frontier systems that run in someone else's data center and that you reach through an API. I use those, and I use them a lot. But the work I want to talk about today mostly happened on a laptop, with models small enough to fit in a few gigabytes of memory, or inside a secure environment where the data could not go anywhere else. I think that setting teaches more about applied machine learning than the frontier models do, because the constraints force you to be explicit about what the model is for.

I want to describe three different uses of AI from my time at the Coleridge Initiative, because people tend to lump them together and they should not. The first is small local models that turn a pile of documents into structured features, which I used for audience and market analysis. The second is AI as a technical collaborator, where ChatGPT and Claude Code helped me design and debug data systems I had no formal training to build. The third is AI-assisted synthetic data, which let people learn and test workflows without access to confidential records. Each one needs a different kind of checking, and the value in every case came from matching the tool to the problem and then asking whether the output was good enough for the decision at hand.

## 2. The setting

Coleridge's Applied Data Analytics work brings state agency staff together with linked administrative records to answer policy and operational questions. I was a senior research scientist on a team that built training and research products in the workforce, postsecondary education, and corrections domains, and the data I touched covered program participation, education, corrections, public benefits, and quarterly earnings. Much of that work happens inside the Administrative Data Research Facility (ADRF), and the default posture is simple: sensitive data does not leave the controlled environment.

A typical project starts with a vague question. Someone wants to know whether a program "works." Before anyone fits a model, you have to define the cohort, link records across systems that were never designed to talk to each other, pick a time origin, build pre- and post-program measures, and decide what comparison would actually answer the question. Demographers will recognize this as the same problem we always have: the population at risk, the exposure window, and the denominator. Most of the intellectual work is there, and none of it is glamorous.

The other half of my job was training. We built example data, code, and reproducible materials in R, SQL, and Python for analysts with very different technical backgrounds. So a lot of my AI use was about infrastructure and translation, not about asking a model to write a conclusion for me.

## 3. The local setup

The hardware was modest. I worked across two laptops, one with an RTX 4060 (8 GB of VRAM) and an older one with a GTX 960M (4 GB), the second of which needed new thermal paste before it would run anything for long without cooking itself. This was not a GPU cluster.

It is worth being clear about what "small model" means here, because it is not only small chatbots. There are two families, and they do different jobs. The first is encoder models: sentence-embedding models that turn text into a vector of numbers, and zero-shot classifiers that score how well a piece of text fits a label you supply. These are small, often a few hundred million parameters, they run comfortably on a laptop GPU or even a CPU, and they are deterministic in a way that makes them easy to audit. The second is small generative models, in roughly the 3 to 9 billion parameter range, which I ran through [Ollama](https://ollama.com). Ollama exposes local models through a REST API, and the same client code can talk to a hosted endpoint by changing a base URL, so you can develop on sensitive data locally and keep the option to scale later on infrastructure you control. I reached those models from R through the `ellmer` package and from Python through the `ollama` library.

[CONFIRM: the specific embedding, classification, and generative models you ran.]

## 4. Small local models for audience and market analysis

### 4.1 The question

The market question was about our audience. Who is in the pool of people we train or want to reach? What skills and roles show up again and again? Are there distinct groups that need different examples, different training, or different outreach? The corpus I worked with was a collection of resumes. [CONFIRM: whose resumes, roughly how many, and what decision the analysis was meant to inform.]

### 4.2 The pipeline

I built the pipeline in Python. It read each document, pulled out fields such as degree level, field of study, job titles, skills, sector, and years of experience, and assembled them into a short text summary of each person, which the code called `persona_text`. Then I tried two routes to structure, and the difference between them is the most useful idea in this whole section.

The first route was classification against categories I defined. A zero-shot classifier takes a piece of text and a list of candidate labels, such as sectors or role types, and scores how well the text fits each one. The model does not invent the labels; the analyst supplies them. This works when you already have a taxonomy you trust.

The second route was discovery. A sentence-embedding model turns each persona into a vector, and clustering those vectors groups people with similar backgrounds. Then the real work starts: you look at the common titles and skills in each cluster, read a handful of representative resumes, and give the groups names that make sense for the audience question.

At one point I asked whether the zero-shot classifier could just generate its own labels. It cannot, and the reason is instructive. Classification against a fixed frame and discovery of new categories are different measurement operations. If you want the model to propose labels, you need a different step, either a generative model that names clusters or an analyst who reads them. Getting that distinction straight was worth more than any amount of tuning.

### 4.3 What went wrong

The summary above sounds tidy. The implementation was not. A Word file was silently skipped while the PDFs were processed, so my corpus was smaller than I thought it was. I hit missing `persona_text` and `cluster` columns because one step had not written what the next step expected. And I spent a non-trivial amount of time on Python scope and indentation errors, which is what happens when an R person writes Python. I show you these failures on purpose. In my experience, applied AI pipelines spend far more time on file handling, schema consistency, and validation than on the model call itself, and the model call is the one part of the pipeline that almost never breaks.

### 4.4 Sentiment, and naming clusters

I also used small local models for sentiment and segmentation of free text. [CONFIRM: whether this was the resume work or a separate corpus, and fix this paragraph to match.] When a generative model is in the loop, for sentiment labels or for naming clusters, the prompt has to fit the size of the model. I learned this on a separate project, a voice note app built on `llama3.2:3b`: a 3B model needs short instructions, a small closed set of choices, a temperature of zero so the same input gives the same output, and a structured output format it cannot wander away from. If your prompt reads like something you would send to a frontier model, a small model will ignore half of it. Appendix B shows that pattern.

Cluster naming has its own trap. Model-written segment descriptions drift toward generic marketing language ("experienced professionals seeking growth opportunities") that sounds plausible and says nothing. I prototyped a four-role pipeline to deal with this: an orchestrator that plans and synthesizes, a profiler that describes each segment, a strategist that proposes actions, and a critic whose only job is to flag vague claims. To be clear about what that was, it was a design prototype on synthetic data, built against a hosted model, not part of the local workflow. But the critic role came from a real problem, and I would still build it in.

## 5. Evaluation, honestly

My evaluation of this work was small scale and largely manual. I read the outputs, I compared model labels to my own judgment on samples, and I kept notes on how the model failed.

I used to treat that as a weakness, and in one sense it is: it does not scale, and I cannot hand you an accuracy figure from a large labeled test set. But I have come around to thinking manual evaluation is the right first step. Reading outputs by hand forces you to build a vocabulary for the failure modes before you automate anything. In sentiment work, the usual suspects were sarcasm read as praise, mixed comments forced into one label, and confident labels on text with no real signal in it. Any automated evaluation you build later, whether that is rule checks, agreement across repeated runs, or a second model acting as a judge, is a formalization of those same judgments. If you have never made them by hand, you do not know what the automated metric is measuring.

For the clustering work, the checks I would ask you to run are concrete. Count how many files were read and compare that to how many you meant to read. Look at the extracted text for several documents, because extraction fails quietly. Compare cluster assignments to the actual resumes. Look for duplicate profiles. Change the number of clusters and see whether the groups stay interpretable or dissolve. And ask whether the segments answer a real training or outreach question, or whether they are just a pretty plot.

The demographer's version of all this is that a model-generated label is a measurement, and measurements have error. So treat it like any other imperfect instrument: build a hand-coded validation sample, compute a misclassification matrix, and report precision and recall by category, not one overall accuracy number. A classifier that is 90 percent accurate overall can still be useless for the one category you care about. And a cluster with a convincing name is a hypothesis about a social type, not evidence that the type exists.

One last point on privacy. Running locally means the document text does not go to a hosted service, and that shortens the data governance conversation a lot. But it does not remove the obligation to handle personal information carefully. A resume on your laptop is still a resume.

## 6. AI as a collaborator in data engineering

The most consequential use of AI in my daily work was not the local models. It was ChatGPT and Claude Code helping me cross technical boundaries. My training is in demography and anthropology, not data engineering, and AI let me learn and implement things that would otherwise have taken me much longer: designing tables, translating code between R, SQL, and Python, and debugging queries against large administrative schemas.

The best example I have is a wage window bug. I was writing Redshift queries that joined person, program, and quarterly wage tables, picked the right record when a person had several program episodes, counted quarters with nonmissing wages, and spread wages from different post-program quarters into separate columns. The query ran. The numbers looked reasonable. But people without a particular program key were getting zero wages, because the wage calculation started from the program table. Anyone not in that table was never joined to their wages at all, and a missing value got coerced into a zero somewhere along the way.

The fix was not a syntax trick. It was a data design decision: compute the wage features from the full cohort, and anchor every person's windows to the same quarter. Here is a stripped-down illustration of the two versions, with invented table names:

```sql
-- Version that looks right and is wrong: starts from the program table,
-- so people without a program record silently fall out or become zeros
SELECT p.person_id,
       COALESCE(SUM(w.wages), 0) AS post_wages
FROM program p
LEFT JOIN wages w
  ON w.person_id = p.person_id
 AND w.quarter BETWEEN p.exit_qtr + 1 AND p.exit_qtr + 4
GROUP BY p.person_id;

-- Version that matches the design: start from the cohort, anchor everyone
-- to a common reference quarter, and keep missing as missing
SELECT c.person_id,
       SUM(w.wages)              AS post_wages,
       COUNT(w.wages)            AS n_qtrs_with_wages
FROM cohort c
LEFT JOIN wages w
  ON w.person_id = c.person_id
 AND w.quarter BETWEEN c.ref_qtr + 1 AND c.ref_qtr + 4
GROUP BY c.person_id;
```

I had similar experiences translating SQL to `dbplyr`, where the two versions did not give the same answer at first, and with questions about one row per person, rolling joins, missing industry codes, `NULL` handling, and date conversions. The assistant was very good at proposing patterns and explaining errors. But I still had to specify the grain of every table, decide who belonged in the cohort, look at the counts, and compare implementations. A query that runs is only the first check, and a query that runs and gives plausible numbers is the dangerous one.

So here is how I would describe AI-assisted data engineering: it compressed the learning cycle. I could go from a policy question to a candidate data structure or query, test it, find where it broke, and revise, much faster than I could have alone. The model supplied candidate code and explanations; domain knowledge and validation decided whether the result meant what we thought it meant.

## 7. Synthetic data as infrastructure for learning

Some of the most useful training data I worked with were made up on purpose. I used AI assistance to design synthetic administrative datasets with realistic structure: person tables, program participation, quarterly wages, monthly benefit histories, and related fact tables. For a corrections trends example, I built separate 2021 and 2022 files with prescribed category distributions, seasonal admission patterns, and controlled missingness. For a regression lesson, I generated wage data with a specified difference between participants and nonparticipants, so the example had a known right answer.

The aim was never to make fake records that look like real people. It was to reproduce the analytical problems that learners will face: joining tables at different grains, defining time windows, dealing with missing values, fitting a model, and writing it up. A good synthetic dataset lets students make, and catch, the same mistakes they would make with the real system, without anyone needing access to protected records. Building in a known effect is the part I would stress. If the analysis cannot recover an effect you put there yourself, something is wrong with the analysis, and that is a much easier thing to discover in a classroom than in a report to a state agency.

I also explored more formal synthetic data methods for mixed continuous and categorical variables, including a GAN workflow in R. That was method exploration, not something delivered to a partner, and the distinction matters. A hand-built simulation, a statistical synthesizer, and a generative model can all produce something called "synthetic data," but they have different goals and need different validation.

The checks are the ones you would expect. Are the tables internally consistent, and do the keys join the way they should? Are the time sequences possible, so nobody is released before they are admitted? Are the marginal distributions and missingness close to the design? Does an analysis recover the effect you built in? And is the file clearly labeled so it cannot be mistaken for observed evidence?

## 8. What the constraints taught me

I will name the frustrations plainly, because they are instructive. I did not have the resources to scale the local analysis up, in hardware or in staff time for labeling. The small models made errors that larger models would not have made. And without a large labeled set or a recorded decision, I cannot put a hard number on the value of the market analysis, which is the first thing a manager outside research will ask you about.

But the constraints also bought things. Nothing left the machine. The cost per run was effectively zero, so I could iterate as often as I wanted. And because small models fail visibly, they forced the kind of structure (fixed categories, schemas, validation, review queues) that you should be building around large models anyway, where the failures are rarer and harder to see.

## 9. Related tools I built along the way

A few other projects show the range of what small and local models can do. The voice note app on `llama3.2:3b` with a SQLite backend taught me most of what I know about prompting small models. I implemented a Model Context Protocol (MCP) server in Python on top of a SQLite workforce database, which lets an AI assistant call structured queries as tools instead of guessing at answers. That pattern, giving the model tools rather than asking it to know things, is in my view one of the most useful ideas in applied AI right now. And I set up local code completion in VS Code and Positron through the Continue extension, mixing local models with hosted APIs depending on the task.

## 10. Takeaways

I want to leave you with five points. First, the model is rarely the whole workflow; reading files, defining the unit of analysis, building tables, and checking outputs usually decide whether the analysis is any good. Second, small local models are good at bounded tasks such as embedding, classification, and extraction, and when the corpus and the task are well defined they can be more practical than a large general model. Third, AI can widen who gets to build technical systems; I worked across SQL, R, Python, and database design much faster than my training alone would have allowed, but that did not remove the need to understand the system I was building. Fourth, synthetic data are a teaching and testing tool, especially when the real records are sensitive, but a plausible-looking file still needs deliberate design and validation. Fifth, judge applied machine learning by the decision it supports. A clean cluster plot, a working query, or a fluent explanation is not the endpoint. The endpoint is a defensible answer, or a workflow someone else can reproduce.

And read your outputs. Before any metric and before any dashboard, read a few hundred of them yourself. It is tedious, it is the step people skip, and it is where you will learn the most.

## 11. A possible flow for 30 to 40 minutes

| Time | Segment | Example |
|---|---|---|
| 0 to 5 min | The Coleridge setting | From a broad workforce question to a cohort, a time origin, and outcomes |
| 5 to 15 min | Small local models for audience analysis | Resumes to extracted fields to embeddings and classification to candidate segments |
| 15 to 25 min | AI-assisted data engineering | The wage window bug from starting at the wrong table |
| 25 to 33 min | Synthetic data for training | The two-year corrections files, or a wage panel with a known effect |
| 33 to 40 min | Discussion | Validation, privacy, and what "AI helped" actually means |

## 12. Questions for discussion

If a cluster of resumes has a convincing name, what evidence would make you trust that it is a meaningful audience segment and not an artifact of the embedding model and the number of clusters you chose? Which parts of a data pipeline can an assistant reasonably propose, and which decisions require someone who knows the policy setting and the source data? If a model-derived label goes into a regression as a predictor, what happens to your estimates when that label is misclassified, and how would you correct for it? How would you test whether a synthetic administrative dataset is good enough for teaching without letting it pass as real evidence? And when does running a model locally actually change the privacy story, and what risks are still there when it does?

## Appendix A: embedding, clustering, and zero-shot classification

This is an illustrative sketch of the two routes in section 4.2, not the original project code. The model names are common public examples. [CONFIRM: replace them with the models you used if you show this.]

```python
# pip install sentence-transformers transformers scikit-learn polars
import polars as pl
from sentence_transformers import SentenceTransformer
from sklearn.cluster import KMeans
from transformers import pipeline

profiles = pl.DataFrame({
    "id": [1, 2, 3, 4],
    "persona_text": [
        "MS statistics; data analyst at state workforce agency; SQL, R, Tableau",
        "BA criminal justice; program coordinator in corrections reentry; case management",
        "PhD economics; research scientist; Python, Stata, causal inference",
        "BS information systems; database administrator; Oracle, ETL, Python",
    ],
})

# Route 1: discovery. Embed, cluster, then read the clusters yourself.
embedder = SentenceTransformer("all-MiniLM-L6-v2")
X = embedder.encode(profiles["persona_text"].to_list(), normalize_embeddings=True)
profiles = profiles.with_columns(
    pl.Series("cluster", KMeans(n_clusters=2, n_init=10, random_state=42).fit_predict(X))
)

# Route 2: classification against labels the analyst supplies.
zsc = pipeline("zero-shot-classification", model="facebook/bart-large-mnli")
labels = ["data analysis", "program operations", "research", "data engineering"]
top = [zsc(t, candidate_labels=labels)["labels"][0] for t in profiles["persona_text"]]
profiles = profiles.with_columns(pl.Series("top_label", top))

print(profiles)
# Next steps: check the clusters at several values of k, read representative
# profiles in each cluster, and hand code a sample to compare with top_label.
```

## Appendix B: constrained classification with a small generative model

This sketch shows the classify-and-validate pattern from section 4.4. It assumes Ollama is running locally and that you have pulled a small model.

```python
# pip install ollama pydantic polars
from typing import Literal
import ollama
import polars as pl
from pydantic import BaseModel, ValidationError

class Label(BaseModel):
    sentiment: Literal["positive", "neutral", "negative", "mixed"]
    topic: Literal["content", "access", "support", "cost", "other"]
    confidence: float

SYSTEM = (
    "Classify the comment. Use only the allowed values. "
    "Use 'mixed' if the comment is both positive and negative. "
    "Return JSON only."
)

def classify(text: str, model: str = "llama3.2:3b") -> dict:
    resp = ollama.chat(
        model=model,
        messages=[{"role": "system", "content": SYSTEM},
                  {"role": "user", "content": text}],
        format=Label.model_json_schema(),   # constrain output to the schema
        options={"temperature": 0},         # same input, same output
    )
    try:
        return Label.model_validate_json(resp.message.content).model_dump() | {"valid": True}
    except ValidationError:
        return {"sentiment": None, "topic": None, "confidence": None, "valid": False}

comments = pl.DataFrame({"id": [1, 2, 3], "text": [
    "The SQL module was great but the login process was painful.",
    "Fine.",
    "I could not get support for two weeks.",
]})

out = pl.concat([comments, pl.DataFrame([classify(t) for t in comments["text"]])],
                how="horizontal")

# Invalid rows go to a review queue; a random sample of valid rows goes to
# hand coding for the misclassification matrix.
review_queue = out.filter(~pl.col("valid"))
hand_code_sample = out.filter(pl.col("valid")).sample(fraction=0.2, seed=42)
print(out)
```

The R versions follow the same logic, with `ellmer::chat_ollama()` for the generative step. The category sets in both appendices are invented for the examples; replace them with a frame that fits your own data.
