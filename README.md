# Veritas-AI-Hackathon-Project
# FirstShift: spot fake job postings before you apply

**Track:** Finance
**Team:** Mason Moghadam

**This project is a prototype and is not financial advice.**

[Open the notebook in Google Colab](https://colab.research.google.com/github/mm-tab/FirstShift/blob/main/FirstShift.ipynb)

## The problem
Job scams are growing fast. Reported U.S. losses rose from $90 million in 2020 to $501 million in 2024 (FTC). For 18–24-year-olds, employment scams are the riskiest scam type: almost 30% of their reports to the BBB, with a median loss of $1,819. Teens and young adults applying for their first jobs often can't tell a real posting from a fake one, and the jobs they qualify for (part-time, entry level, remote) are the kind scammers imitate.

## What we built
FirstShift is a machine learning tool for a teen applying for their first job online. They paste in a job posting, and FirstShift returns how likely it is to be fake, plus the "red flag" words in that posting that suggest a scam, so they learn what to watch for themselves.

**How to run it:** open the Colab link above, click **Runtime → Run all**, and use the demo link printed by the last cell. The training data downloads automatically, and the 2026 test set loads from this repo.

## What we added
Our own idea (not a suggested project). Beyond building a scam classifier, I:
1. **Designed it around one user,** a teen looking for a first job, and tested it specifically on postings a teen could apply for (part-time, temporary, entry level, or no degree required).
2. **Audited it for fairness to teen jobs:** checked whether it wrongly flags real entry-level, internship and remote jobs more often than other jobs, since a tool that scares teens away from real jobs would do harm.
3. **Tested it on 2026-style scams:** built a separate test set of 40 synthetic postings (20 modern scams, 20 real teen jobs) to see if a model trained on 2012–14 data still works today.
4. **Checked for leakage:** found that 21% of test postings were repeats of training postings, and report results with them removed.

## How it works
**Data:** [Real / Fake Job Posting Prediction](https://www.kaggle.com/datasets/shivamb/real-or-fake-fake-jobposting-prediction) (Kaggle; originally EMSCAD, University of the Aegean): 17,880 postings from 2012–14, 866 (4.8%) fake. I combined the title, company profile, description, requirements and benefits into one text field, filled blanks, and split the data 80/20, keeping the same share of fakes in both halves.

**Model:** TF-IDF (word and two-word-phrase scores) with logistic regression, weighted so the rare fakes count more. I chose this model because it handles text well and gives each word a weight, which powers the red-flag explanations. I tuned its settings (C = 10) with cross-validation on training data only, and chose the decision threshold (0.405) by maximizing F2 (which counts catching scams twice as much as avoiding false alarms) on cross-validated training predictions. The test set wasn't used for any of these choices.

## Results
Test set: 3,576 postings, 173 fake.

| Model | Precision | Recall | False alarms |
|---|---|---|---|
| Baseline: always "real" | 0.000 | 0.000 | 0 |
| Baseline: "no company logo = fake" | 0.149 | 0.601 | – |
| FirstShift, all test postings | 0.822 | 0.908 | 34 |
| **FirstShift, repeats removed (honest result)** | **0.736** | **0.864** | **34** |

*Precision: of postings flagged as scams, how many really were. Recall: of the real scams, how many were caught.*

Our baseline (the logo rule) caught 60% of scams; FirstShift caught 86% on held-out postings it had never seen, with far fewer false alarms.

**Teen-focused results:**
- **Fair to teen-type jobs:** real entry-level, internship and remote jobs were wrongly flagged 0.7% of the time, about the same as other real jobs (1.1%).
- **More scams in teen jobs:** 6.2% of postings a teen could apply for were fake, vs. 4.8% overall.
- **Teen-eligible postings (repeats removed):** FirstShift caught 79.6% of scams and wrongly flagged 1.7% of real jobs.

**Where it fails, and why:**
- **Newer scams:** on the 2026 test set it caught only 9 of 20 scams (45% recall, 75% precision). It caught fake-check, reshipping, data-entry and MLM scams, but missed app-based "task" jobs, gift-card and crypto schemes, upfront-fee jobs, and recruiters who message on chat apps. These scam types barely exist in its 2012–14 training data.
- **Missed scam example, "Vemma Brand Partner":** a multi-level marketing pitch that uses professional words like "team" and "skills" and a formal age requirement, so it reads like a real sales job.
- **False alarm example, "Director of Quality":** a real job posted by a recruiting firm for an unnamed client. Hiding the employer's name and using recruiter phrases are also common in scams, so the model flagged it because of how it was posted, not what the job was.
- **Memorization:** some of the strongest "scam" words are company names from reposted scam ads, which won't help with new scams.

**Limitations:** the training data is from 2012–14; the 2026 test set is small (40) and AI-written; "teen-eligible" is my own definition; FirstShift only reads the posting's text and doesn't check the company or recruiter.

## What we'd do next
1. Retrain on modern scam data to close the gap on 2026-style scams, and remove duplicate postings before splitting.
2. Add the posting's details (company logo, remote, screening questions) as features, and test the demo with more teens applying for real jobs.

## Sources
- Dataset: [Real / Fake Job Posting Prediction (Kaggle)](https://www.kaggle.com/datasets/shivamb/real-or-fake-fake-jobposting-prediction), from EMSCAD, University of the Aegean
- 2026 test set (`test_2026.csv`): 40 synthetic postings written with help from Claude (Anthropic), based on scam types described by the FTC and BBB
- Problem statistics: [FTC data via Bitdefender](https://www.bitdefender.com/en-au/blog/hotforsecurity/ftc-americans-lost-12-5-billion-to-scams-last-year), [BBB Institute youth scams report](https://bbbmarketplacetrust.org/bbb-institute-releases-new-report-on-youth-scams/), [Knox Pages coverage of the BBB report](https://www.knoxpages.com/?p=184041)
- Libraries: pandas, NumPy, scikit-learn, Gradio, kagglehub
- AI tools: Claude (Anthropic) was used as a coding and learning assistant (explaining concepts, debugging, suggesting code) and to help write the synthetic 2026 test set and this README.
