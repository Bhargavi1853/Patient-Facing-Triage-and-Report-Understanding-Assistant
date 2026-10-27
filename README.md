# Patient-Facing-Triage-and-Report-Understanding-Assistant
## Agentic AI workflow for patient triage and report understanding

A system that helps patients make sense of their symptoms and medical reports, gets urgent cases to a clinician quickly, and makes sure routine cases get followed up.

One trained machine learning classifier decides how urgent a case is. Six AI agents support it: they prepare its input, explain the results in plain English, check those explanations for mistakes, and act on the decision. The system never gives a diagnosis or treatment advice, and it hands the case to a human whenever it isn't sure.

Coursework for Intelligent Systems, MSc Artificial Intelligence, University of East London. In progress since October 2026.

<img width="2760" height="1995" alt="triage_workflow_detailed" src="https://github.com/user-attachments/assets/2f2fca4e-6e95-414d-8559-734cd17e2ab1" />


## What goes in and what comes out

Input: symptoms the patient types in, vital signs, and the text of lab or radiology reports.

Output:

- an urgency level: emergency, urgent, routine or self-care
- a plain-language explanation of the report that has been checked against the source
- one of two next steps: a handover to a clinician for urgent cases, or scheduled reminders for routine ones

## Features

- Structured intake. Pulls symptoms, vital signs, lab values, units and conditions out of free text and report documents.
- Urgency classification. A trained and evaluated ML model predicts one of four urgency levels.
- Red-flag rules. Critical signs such as chest pain with breathlessness, stroke signs or very low oxygen saturation force an emergency result, whatever the model predicts.
- Plain-language explanations. Medical terms and results are rewritten so a patient can follow them.
- Fact-checking. Every number and claim in the explanation is checked against the source report, and unsafe wording is blocked.
- Clinician handover. Urgent cases go to a human with a short, structured summary.
- Follow-up reminders. Routine cases get reminders so follow-ups aren't missed.
- Human oversight. Low model confidence or a failed fact-check sends the case to a clinician.

## Components

| # | Component | What it does | Method |
| --- | --- | --- | --- |
| Main | Triage classifier | Predicts urgency | XGBoost on structured features, or fine-tuned BERT on text |
| 1 | Intake and parser agent | Turns input into structured data | Biomedical NER (scispaCy) plus rules |
| 2 | Terminology simplifier agent | Explains findings in plain language | LLM with RAG over a medical glossary |
| 3 | Fact-checker and safety auditor agent | Checks explanations against the source | Rule checks plus LLM verification |
| 4 | Router agent | Chooses escalation or follow-up | Decision logic on urgency, confidence and audit result |
| 5 | Handover summariser agent | Writes the clinician summary for urgent cases | LLM with a fixed summary template |
| 6 | Scheduling and reminder agent | Books follow-ups for routine cases | Calendar or notification API |

### Triage classifier

The main decision-maker. It takes the structured features from the intake agent (vital signs, age, symptoms and key lab values) and predicts emergency, urgent, routine or self-care, with a calibrated confidence score. Red-flag rules run first, so a critical sign forces an emergency result whatever the model says.

The planned model is XGBoost on tabular triage data, with probabilities calibrated by isotonic regression. It's judged on accuracy, macro F1 and above all recall on the emergency and urgent classes, because under-triage is the dangerous error.

### 1. Intake and parser agent

Reads typed symptoms, vital signs and uploaded lab or radiology text, and extracts conditions, test names, values and units. It standardises units, flags values outside normal ranges, and asks the patient for missing essentials (such as age or main symptom) before triage runs.

### 2. Terminology simplifier agent

Looks up lay definitions of medical terms in a glossary (RAG) and rewrites each finding at a reading age of about 11 to 12. Every number and unit stays exactly as it was in the source. It says what a result means without diagnosing or advising treatment.

### 3. Fact-checker and safety auditor agent

Checks every number, unit and finding in the explanation against the source report, and flags anything added, missing or changed. It blocks wording that diagnoses, prescribes or falsely reassures. A failed explanation is regenerated once; if it fails again, the case goes to a clinician.

### 4. Router agent

Sends emergency and urgent cases to the handover summariser and routine or self-care cases to the scheduling agent. If confidence is below the threshold or the audit fails, it escalates to a human. Emergency cases see advice to contact emergency services straight away, without waiting for the explanation.

### 5. Handover summariser agent

Writes a summary the clinician can scan in seconds: main complaint, vital signs, abnormal results, red flags, predicted urgency, confidence and why the case was escalated. It uses a fixed structure and sends the summary to the clinician dashboard or queue.

### 6. Scheduling and reminder agent

Creates follow-up reminders (a GP appointment or a repeat test, for example), sends them by email or in the app, and records whether the patient confirmed. If the patient reports new or worse symptoms, the case goes back through triage.

## Data

| Dataset | Use |
| --- | --- |
| [Hospital Triage and Patient History Data](https://www.kaggle.com/datasets/maalona/hospital-triage-and-patient-history-data/data) (Kaggle) | Training and testing the triage classifier |
| [Emergency Service - Triage Application](https://www.kaggle.com/datasets/ilkeryildiz/emergency-service-triage-application) (Kaggle) | Training and testing the triage classifier |
| [MIMIC-IV](https://mimic.mit.edu/docs/iv/) | Clinical records and report text (credentialed access through PhysioNet) |

[PyHealth](https://pyhealth.readthedocs.io/en/latest/api/datasets/pyhealth.datasets.BaseDataset.html) is used to load and process the clinical datasets.


## Tech stack

Python, XGBoost, scikit-learn, scispaCy, PyHealth.

## Evaluation plan

- Triage classifier: accuracy, macro F1, and recall on the emergency and urgent classes, plus a confusion matrix showing where under-triage happens.
- Calibration: reliability curve before and after isotonic regression.
- Red-flag rules: a set of test cases that must always come out as emergency.
- Explanations: how often the fact-checker catches changed numbers or unsafe wording, and a readability score for the plain-language output.
- End to end: sample patient cases run through the full workflow, checking each one takes the right path.


## Progress

- [x] Workflow designed and agents specified
- [ ] Triage datasets cleaned and labels mapped to four urgency levels
- [ ] Triage classifier trained, calibrated and evaluated
- [ ] Red-flag rules and tests
- [ ] Intake and parser agent
- [ ] Terminology simplifier with RAG
- [ ] Fact-checker and safety auditor
- [ ] Router, handover and scheduling agents
- [ ] End-to-end demo and write-up

## References

- NHS England (2022). Guidance for emergency departments: initial assessment. With the Royal College of Emergency Medicine. https://www.england.nhs.uk/guidance-for-emergency-departments-initial-assessment/
- Siira, E., Johansson, H. and Nygren, J. (2025). Mapping and summarizing the research on AI systems for automating medical history taking and triage: scoping review. *Journal of Medical Internet Research*, 27, e53741. https://doi.org/10.2196/53741
- Johnson, A. et al. MIMIC-IV. PhysioNet. https://mimic.mit.edu/docs/iv/

## Author

Bala Bhargavi Godari · [LinkedIn](https://www.linkedin.com/in/godaribalabhargavi) · [GitHub](https://github.com/Bhargavi1853)
