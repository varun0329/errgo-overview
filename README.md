# Errgo — AI Recruiting Platform

> Replacing traditional coding interviews with real-world, project-based technical assessments — graded by AI, reviewed by humans.

[![Status](https://img.shields.io/badge/status-live-brightgreen)]()
[![Candidates](https://img.shields.io/badge/candidates-1%2C000%2B-blue)]()
[![Graded](https://img.shields.io/badge/assessments_graded-180%2B-purple)]()
[![Stack](https://img.shields.io/badge/stack-TypeScript%20%7C%20Python%20%7C%20React%20%7C%20Firebase%20%7C%20GCP-orange)]()

---

## The Problem

Traditional technical hiring is broken:
- **LeetCode doesn't reflect real work.** Engineers solve algorithmic puzzles that rarely map to day-to-day responsibilities.
- **Manual screening doesn't scale.** Reviewing 500+ applicants per role is a bottleneck for growing teams.
- **Candidates get no feedback.** Most applicants never hear back, let alone understand where they fell short.

## What Errgo Does

Errgo is a full-stack platform that sends candidates **real-world project assignments** (not puzzles), **automatically grades their submissions** using AI-powered code analysis, and surfaces the best candidates to human reviewers — complete with code breakdowns, scores, and talking points for interviews.

**The pipeline:**

```
Candidate applies → ATS scoring → Project assignment sent →
GitHub repo provisioned → Candidate builds → Submission detected →
AI grading pipeline runs → Human reviewer notified → Interview or feedback
```

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         ERRGO PLATFORM                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────────┐  │
│  │  Lovable      │    │  Firebase     │    │  GitHub API          │  │
│  │  Dashboard    │◄──►│  Firestore    │◄──►│  Repo Provisioning   │  │
│  │  (React)      │    │  (Database)   │    │  Submission Detection│  │
│  └──────────────┘    └──────┬───────┘    └──────────────────────┘  │
│                             │                                       │
│                    ┌────────┴────────┐                              │
│                    │  Cloud Functions │                              │
│                    │  (Event Triggers)│                              │
│                    └────────┬────────┘                              │
│                             │                                       │
│              ┌──────────────┼──────────────┐                       │
│              ▼              ▼              ▼                        │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐               │
│  │ Submission    │ │ Node.js      │ │ Python ML/DS │               │
│  │ Collector     │ │ Project      │ │ Project      │               │
│  │ (Detection)   │ │ Grader       │ │ Grader       │               │
│  └──────────────┘ └──────────────┘ └──────────────┘               │
│                             │                                       │
│              ┌──────────────┼──────────────┐                       │
│              ▼              ▼              ▼                        │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐               │
│  │ SendGrid     │ │ GCP Cloud    │ │ GitHub        │               │
│  │ (Emails)     │ │ Tasks        │ │ Actions CI/CD │               │
│  └──────────────┘ └──────────────┘ └──────────────┘               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. ATS Scoring Engine
Automated application scoring runs every 6 hours, evaluating candidates on resume relevance, experience fit, and skill match. Scores are normalized and combined with project grades using a weighted priority system (30% ATS + 70% project grade).

### 2. Submission Collector
A scheduled GitHub Actions workflow that monitors candidate repositories for completed assessments. Detects "Assessment Completed" commits, verifies admin collaborator access, and triggers the grading pipeline. Handles edge cases like Git LFS files by falling back to the GitHub Contents API.

### 3. Automated Grading Pipeline

**Two specialized graders run independently:**

| Grader | Language | Projects Covered | Techniques |
|--------|----------|-----------------|------------|
| **Node.js Grader** | TypeScript/JS | Ticket Support, CloudSync, Task Management | AST parsing, dependency analysis, test execution |
| **Python ML/DS Grader** | Python | Echo Loop, SmartAssist, TechFlow Solutions | Notebook parsing, model evaluation, code quality scoring |

Both graders use:
- **AST parsing** for code structure analysis
- **Subprocess sandboxing** for safe test execution
- **Custom rubric engines** with weighted scoring across code quality, architecture, testing, and functionality
- **Key file detection** to locate and evaluate the most important source files

### 4. Universal Grader
A baseline grading pass that evaluates every submission regardless of project type — analyzing repository structure, commit history, documentation quality, and code organization.

### 5. Reviewer Assignment System
Firebase Cloud Functions automatically assign graded candidates to human reviewers using round-robin distribution:

| Track | Reviewers |
|-------|-----------|
| ML / Data Science | Daman, Ayush |
| Backend / Full-Stack | Brandon, Mason |

Reviewers are notified via SendGrid email when 3+ candidates are ready for review.

### 6. Review Dashboard (React)
A custom-built dashboard for human reviewers featuring:
- **Real-time Firestore sync** — candidate data updates live
- **Syntax-highlighted code viewer** — review submissions without cloning repos
- **Score override capabilities** — reviewers can adjust AI-generated scores
- **Pipeline management** — move candidates through stages (graded → in review → interview → rejected)
- **Interview prep tab** — auto-generated questions, talking points, and red flags per candidate
- **Cluely Context ZIP download** — packaged candidate data for interview prep

### 7. Candidate Communication
Three automated email flows triggered by Firestore state changes:
- **Reviewer notification** — batched alerts when candidates are ready for review
- **Interview invitation** — sent when a candidate is pushed to interview stage, includes Cal.com booking link
- **Rejection with feedback** — personalized feedback generated via Groq LLM explaining strengths and areas for improvement

---

## Priority Classification

Candidates are ranked using a combined scoring system:

```
Combined Score = (ATS Score / ATS Max × 30%) + (Project Score / Project Max × 70%)
```

| Priority | Score Range | Action |
|----------|------------|--------|
| **P1** | ≥ 75% | Fast-track to interview |
| **P2** | ≥ 55% | Human review recommended |
| **P3** | ≥ 35% | Optional deeper review |
| **P4** | < 35% | Auto-rejection with feedback |

---

## Tech Stack

| Layer | Technologies |
|-------|-------------|
| **Frontend** | React, Lovable, Tailwind CSS |
| **Backend** | Node.js, Python, Firebase Cloud Functions |
| **Database** | Firebase Firestore |
| **Cloud** | GCP (Cloud Functions, Cloud Tasks), GitHub Actions |
| **APIs** | GitHub REST API, SendGrid, OpenAI, Groq, Cal.com |
| **Grading** | Python AST, subprocess, custom rubric engines |
| **CI/CD** | GitHub Actions (scheduled workflows for each pipeline stage) |

---

## Pipeline Automation Schedule

| Pipeline | Schedule | Repository |
|----------|----------|------------|
| ATS Scoring | Every 6 hours | User-Pipeline |
| Submission Collector + Universal Grader | Daily 7:00 AM UTC | errgo-submission-collector |
| Node.js Project Grader | Daily 8:00 AM UTC | errgo-project-grader |
| Python ML/DS Project Grader | Daily 9:00 AM UTC | project-grader-pipeline |

All pipelines also support manual triggering via the dashboard's "Grade This Candidate" button.

---

## Metrics

| Metric | Value |
|--------|-------|
| Candidates onboarded | 1,000+ |
| Assessments fully graded | 180+ |
| Project types supported | 6 (backend, full-stack, ML/DS) |
| Avg. grading time per candidate | < 3 minutes |
| Automated email flows | 3 (reviewer, interview, rejection) |

---

## Repository Structure

> Source code is in private repositories.

```
errgo/
├── errgo-submission-collector    # Detects completed assessments, runs universal grading
├── errgo-project-grader          # Node.js/TypeScript project grading + ATS sync
├── project-grader-pipeline       # Python ML/DS project grading
├── errgo-assessment-sender       # Cloud Functions (assignment, review, interview, rejection)
├── User-Pipeline                 # ATS scoring automation
└── errgo-dashboard               # React review dashboard (Lovable-built)
```

---

## What I Learned

- **Designing evaluation rubrics is harder than writing graders.** The rubric engineering — deciding what "good code" means across different project types — was the most iterative part of the project.
- **GitHub API has sharp edges.** Git LFS bandwidth limits, tree API returning empty for certain repo structures, and rate limiting all required creative workarounds.
- **Serverless event-driven architecture scales well but debugging is painful.** Chaining Cloud Functions with Firestore triggers creates powerful automation, but tracing failures across multiple async triggers requires careful logging.
- **AI grading needs human calibration.** The automated scores are a strong signal, but human reviewers catching false positives/negatives is what makes the system trustworthy.

---

## Status

**Live and actively processing candidates.** The platform runs daily automated pipelines and has processed 180+ full assessment cycles end-to-end.

---

*Built by [Varun Agarwal](https://www.linkedin.com/in/vagarwal2903/) — Northeastern University MS CS '26*
