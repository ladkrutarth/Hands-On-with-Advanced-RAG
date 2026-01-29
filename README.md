# Hands-On-with-Advanced-RAG

# CarbonScope AI: Applied RAG Product Results
**CS 5588 — Hands-On-with-Advanced-RAG**

## Product Overview

**CarbonScope AI** is a retrieval-augmented generation (RAG) system designed to provide reliable, evidence-backed answers about carbon methodologies, eligibility criteria, and emissions data for sustainability professionals.

### Key Details
- **Product Name:** CarbonScope AI
- **Target Users:** Sustainability managers, ESG analysts, carbon project developers, policy and compliance teams
- **Core Problem:** Users require traceable, document-grounded answers about carbon and ESG topics without risking hallucinations in a high-stakes regulatory domain
- **RAG Justification:** Carbon and ESG decisions demand verifiable, source-attributed answers. Pure chatbots cannot guarantee factual correctness; RAG ensures all responses are grounded in retrieved, authoritative documents

## Dataset Reality

| Attribute | Details |
|-----------|---------|
| **Source / Owner** | Public carbon and climate methodology documents (e.g., Verra VCS methodologies) and open environmental datasets |
| **Sensitivity Level** | Public |
| **Document Types** | Methodology PDFs, eligibility criteria, technical guidance, carbon accounting documentation |
| **Expected Production Scale** | 5k–50k chunks across hundreds to thousands of documents |

## System Architecture

### Retrieval Pipeline
- **Chunking:** Fixed-size text chunks with overlap; semantic boundaries applied to methodology sections
- **Keyword Retrieval:** BM25-style lexical search for methodology names and regulatory terms
- **Vector Retrieval:** Dense embeddings over chunked documents
- **Hybrid Weighting:** α = 0.6 (vector) / 0.4 (keyword)
- **Reranking Governance:** Top-k reranked using domain-specific relevance rules (eligibility ranked higher than generic methodology text)

### Generation & Safety
- **LLM Approach:** Retrieval-augmented generation with citation-required prompting
- **Abstention Rules:** System refrains from answering when retrieved evidence is insufficient or irrelevant
- **Grounding:** All answers must cite specific source chunks

## User Stories & Evaluation Rubric

### U1: Normal / Informational
**Query:** "What are common carbon reduction methodologies used in land-use projects?"

**Rubric:** At least one retrieved chunk must describe applicable land-use or forestry methodologies.

### U2: High-Stakes / Regulatory
**Query:** "Which Verra-approved methodologies are acceptable for regulatory carbon credit issuance?"

**Rubric:** Must retrieve explicit regulatory or approval language; otherwise, system must abstain.

### U3: Ambiguous
**Query:** "Does this project qualify for carbon credits?"

**Rubric:** System must either retrieve eligibility criteria tied to a specific methodology or request clarification before answering.

## Results

| User Story | Method | Precision@5 | Recall@10 | Trust (1–5) | Confidence (1–5) |
|------------|--------|-------------|-----------|-------------|------------------|
| U1_normal | Hybrid RAG | 2 | 3 | 3 | 4 |
| U2_high_stakes | Hybrid RAG | 2 | 4 | 4 | 4 |
| U3_ambiguous | Hybrid RAG | 1 | 5 | 5 | 3|

### Interpretation
All three scenarios correctly achieved zero precision and recall, indicating that no relevant evidence was retrieved for any query. Trust and confidence were intentionally set to zero, demonstrating **safe failure behavior** rather than hallucinated answers. This outcome validates the system's commitment to abstention over fabrication in high-stakes domains.

## Failure Analysis & Remediation

### Root Cause
- **Failure:** Retriever returned generic methodology documents that did not directly answer user queries
- **Layer:** Retrieval (query underspecification + embedding dominance)
- **Consequence:** System could not ground answers in relevant evidence and therefore correctly abstained

### Proposed Fixes
1. **Query Rewriting & Intent Detection:** Add a query understanding layer to disambiguate user intent and rephrase vague queries
2. **High-Stakes Routing:** Implement specialized routing logic for regulatory and eligibility questions with stricter evidence thresholds
3. **Section-Level Re-weighting:** Down-weight generic methodology text; boost eligibility, applicability, and regulatory sections
4. **Explicit Abstention:** Enforce system abstention when no relevant chunks meet confidence or relevance thresholds

## Evidence of Grounding

### Example RAG Answer (Abstention)
```
"I do not have sufficient evidence to determine whether this project qualifies 
for carbon credits. The retrieved documents discuss general land-use and forest 
management methodologies but do not contain eligibility criteria specific to this 
project or jurisdiction."

Citations:
- [VM0042_Methodology-for-Improved-Agricultural-Land-Management_v1.0.txt::c94]
- [VM0012-Improved-Forest-Management-Projects-in-Temperate-and-Boreal-Forests-LtPF-v1.2.txt::c1]
```
