# ASSIGNMENT: Sampling and Reproducibility in Python

Read the blog post [Contact tracing can give a biased sample of COVID-19 cases](https://andrewwhitby.com/2020/11/24/contact-tracing-biased/) by Andrew Whitby to understand the context and motivation behind the simulation model we will be examining.

Examine the code in `whitby_covid_tracing.py`. Identify all stages at which sampling is occurring in the model. Describe in words the sampling procedure, referencing the functions used, sample size, sampling frame, any underlying distributions involved, and how these relate to the procedure outlined in the blog post.

Run the Python script file called whitby_covid_tracing.py as is and compare the results to the graphs in the original blog post. Does this code appear to reproduce the graphs from the original blog post?

Modify the number of repetitions in the simulation to 100 (from the original 1000). Run the script multiple times and observe the outputted graphs. Comment on the reproducibility of the results.

Alter the code so that it is reproducible. Describe the changes you made to the code and how they affected the reproducibility of the script file. The output does not need to match Whitby’s original blogpost/graphs, it just needs to produce the same output when run multiple times

# Author: Bitewulign Mekonnen

## Analysis of Sampling in the Model
The given Python script `whitby_covid_tracing.py` simulates a contact-tracing model for COVID-19 infections at two types of events: weddings and brunches. The process involves multiple stages of sampling, each influencing the final proportions of infections and traced cases. Below, I describe each stage:

## 1. Initial Population Sampling
**Sampling procedure:** The simulation creates a population of `1,000` individuals, split between two event types: `200` attending weddings and `800` attending brunches.\
**Sampling frame:** The entire population consists of people attending these two event types.\
**Underlying distribution:** A fixed categorical distribution where `20%` attend weddings and `80%` attend brunches.\
**Relevance to blog post:** This step represents the real-world attendance of people at different social events.

## 2. Sampling of Individuals for Infection
**Function used:**
```python
infected_indices = np.random.choice(ppl.index, size=int(len(ppl) * ATTACK_RATE), replace=False)
ppl.loc[infected_indices, 'infected'] = True
```

**Sampling procedure:** Randomly selects a subset of individuals to be infected.\
**Sample size:** `int(len(ppl) * ATTACK_RATE)` (`10%` of the total population, which is `100` individuals out of `1000` or `10` individuals out of `100`).\
**Sampling frame:** The indices of the DataFrame `ppl`, which represents all individuals attending events (200 weddings and 800 brunches).\
**Underlying distribution:** Uniform distribution (each individual has an equal probability of being selected for infection).\
**Description:** A random subset of individuals is selected to be infected based on the `ATTACK_RATE`. The `np.random.choice` function is used to sample without replacement, ensuring no individual is infected more than once.\
**Relevance to blog post:** Reflects how infections occur at events and how randomly they spread.

## 3. Primary Contact Tracing Sampling
**Function used:**
```python
ppl.loc[ppl['infected'], 'traced'] = np.random.rand(sum(ppl['infected'])) < TRACE_SUCCESS
```
**Sampling procedure:** Among infected individuals, a subset is randomly marked as `"traced"` with a probability of `20%`.\
**Sample size:** Approximately `20% of 100 = 20` individuals.\
**Sampling frame:** The subset of individuals marked as infected in the previous step.\
**Underlying distribution:** Bernoulli distribution (each infected individual has a `20%` chance of being traced).\
**Description:** For each infected individual, a random number is generated using np.random.rand. If the number is less than TRACE_SUCCESS (`0.20`), the individual is marked as traced. This simulates the probabilistic nature of primary contact tracing.\
**Relevance to blog post:** Demonstrates that tracing does not detect all infections, only a random subset.

## 4. Secondary Contact Tracing Sampling
**Function used:**
```python
event_trace_counts = ppl[ppl['traced'] == True]['event'].value_counts()
events_traced = event_trace_counts[event_trace_counts >= SECONDARY_TRACE_THRESHOLD].index
ppl.loc[ppl['event'].isin(events_traced) & ppl['infected'], 'traced'] = True
```
**Sampling procedure:** If at least two traced individuals were at a given event, all infected people at that event are also traced.\
**Sample size:** Varies depending on how many traced individuals attended the same event.\
**Sampling frame:** The set of infected individuals at events with at least 2 traced cases.\
**Underlying distribution:** Conditional probability (depends on the previous step).\
**Description:** Secondary tracing is performed by counting the number of traced individuals per event type (wedding or brunch). If an event has at least `SECONDARY_TRACE_THRESHOLD` traced cases, all infected individuals from that event are marked as traced. This step does not involve random sampling but rather a deterministic rule based on the results of primary tracing.\
**Relevance to blog post:** This step introduces bias because events with more attendees are more likely to have secondary tracing, even if they weren’t the original source.

## 5. Sampling for Simulation Repetition
**Function Used:** List comprehension and `pd.DataFrame`\
**Sample Size:** `1000 (or 100)` iterations of the simulation.\
**Sampling Frame:** Each iteration generates a new set of infections and tracing outcomes.\
**Underlying Distribution:** The distributions of infections and tracing outcomes are determined by the previous sampling steps.\
**Description:** The simulation is run 1000 (or 100) times to generate a distribution of outcomes. Each iteration involves the sampling procedures described `whitby_covid_tracing.py` before line `66`, and the results are aggregated into a DataFrame (props_df) for analysis.

## Comparison of Results with Whitby’s Blog Post
After running the original script `(whitby_covid_tracing.py)`, we can compare its histograms to those in Andrew Whitby’s blog post.
The histograms generated should show that traced cases disproportionately come from weddings, even though infections are more evenly distributed.
If the output closely matches Whitby’s, then the code correctly reproduces the biases described in his blog.
If there are differences, potential reasons could be:
Differences in random seed settings (i.e., lack of reproducibility).
Slight variations in how tracing is modeled.
Minor changes in implementation compared to Whitby’s original model.
Effect of Reducing Repetitions to `100`
Reducing the number of repetitions from `1,000 to 100` and running the script multiple times shows variability in results.

##  Observations:
With `100` repetitions, histograms fluctuate more across runs, showing inconsistent distributions.
The variability arises because the sample size is too small to approximate the true distribution.
Some runs might overestimate or underestimate the proportion of traced infections coming from weddings due to randomness.
The results suggest that `100` repetitions are insufficient for stable conclusions.
Making the Code Reproducible
To make the simulation fully reproducible, we need to set a random seed before any random sampling occurs. This ensures that every time the script runs, it generates the same results.

## Summary of Sampling Stages:
**1. Infection Sampling:** Randomly select individuals to be infected.\
**2. Primary Tracing Sampling:** Randomly determine which infected individuals are traced.\
**3. Secondary Tracing Sampling:** Deterministically trace individuals based on event-level thresholds.\
**4. Simulation Repetition Sampling:** Repeat the entire process 1000 times to generate a distribution of outcomes.

Each stage involves different sampling techniques, ranging from random sampling with uniform distributions to deterministic rules based on thresholds. The results are then visualized using histograms to analyze the impact of contact tracing on perceived infection sources.

## Criteria

|Criteria|Complete|Incomplete|
|--------|----|----|
|Altercation of the code|The code changes made, made it reproducible.|The code is still not reproducible.|
|Description of changes|The author explained the reasonings for the changes made well.|The author did not explain the reasonings for the changes made well.|

## Submission Information

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

### Submission Parameters:
* Submission Due Date: `23:59 - 16/02/2025`
* The branch name for your repo should be: `assignment-1`
* What to submit for this assignment:
    * This markdown file (a1_sampling_and_reproducibility.md) should be populated.
    * The `whitby_covid_tracing.py` should be changed.
* What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sampling/pull/<pr_id>`
    * Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:
- [ ] Create a branch called `assignment-1`.
- [ ] Ensure that the repository is public.
- [ ] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [ ] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via the help channel in Slack. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.
