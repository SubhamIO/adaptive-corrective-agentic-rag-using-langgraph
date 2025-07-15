# This Repo Explains The Implementation Of Adaptive RAG

## What is Adaptive RAG (in short)?

**Adaptive RAG** is a smart RAG system that adjusts its behavior based on the **quality of the results**. If the retrieved docs are bad or irrelevant, it doesn't just generate an answer blindly. Instead, it rewrites the question and tries again — adapting to the situation.

## Code = Complete Adaptive RAG System

Here’s how your code implements it step by step:

### **Start with a User Question**

* Example: `"What is Machine Learning?"`
* Initial state only has the question.

### **Route the Question**

* A smart LLM decides:
   Use **Vectorstore** (your saved documents)?
   Or go to **Web Search**?

✅ **If it’s about Agents, Prompt Engineering, etc. → Vectorstore**
❌ Else → Web Search

### **Retrieve Documents**

* From **Vectorstore** (if applicable) or Web.
* These are the “facts” to answer your question.

### **Grade Document Relevance**

* Each document is checked:

  * Is it **relevant** to the question? ✅ or ❌
  * Only **relevant docs** are passed forward.

### **Decision Point** – Are docs useful?

* If **some documents are good** → go to answer generation.
* If **all documents are irrelevant** → adapt by:

* Rewriting the question for clarity (Query Rewriter).
* Go back and **search again** with the new question.

This is the **adaptive loop**.

### **Generate Answer**

* Using relevant docs + question → LLM generates a response.

### **Check for Hallucination**

* Ask: “Is the answer **based on the documents**?”
* If yes, go to next step.
* If not, go back and generate again (retry with same question/docs).

### **Check if Answer Actually Helps**

* Ask: “Does this answer **actually solve** the question?”

  * ✅ If yes → Done.
  * ❌ If no → Re-write the question and try again.

This loop continues until:

* The answer is **grounded** (not hallucinated), and
* The answer is **useful**.

## Summary of Adaptive Behavior in Your Code

| Step               | Adaptation Behavior                     |
| ------------------ | --------------------------------------- |
| No good docs?      | Rewrites question and retries retrieval |
| Hallucination?     | Regenerates answer again                |
| Not useful answer? | Rewrites question and starts again      |
| Works fine?        | Stops and returns final answer ✅       |


##  One-Liner Definition for This Code:

> This code implementation is a **feedback-driven RAG pipeline** that adapts at every stage — retrieval, question formulation, and generation — using smart grading and decision routing via LangGraph



# Optional But Intresting  

## 🔍 What is a **hallucination** in AI?

In your code's context:

> A **hallucination** is when the AI makes up information that **is not found in the retrieved documents** (facts).
> It's like confidently saying something that's **not backed by the source**.

---

## ✅ What does **grounded** mean?

> An answer is **grounded** if it is **supported by the retrieved documents** — either directly (exact words) or semantically (same meaning).
> It means the LLM didn't invent facts — it stayed true to what it read.

---

## 🧠 Where is this checked in your code?

Look at this section in your code:

```python
class GradeHallucinations(BaseModel):
    """ Binary score for hallucination present in generation answer ."""
    binary_score:str =Field(
        description="Answer is grounded in the Facts, 'yes' or 'no'"
    )
```

And this prompt:

```python
system=""" 
You are a grader assessing whether an LLM generation is grounded in / supported by a set of retrieved facts.\n
Give a binary score 'yes' or 'no'. 'Yes' means that the answer is grounded in / supported by the set of facts. 
"""
```

Then later in the LangGraph node:

```python
score = hallucination_grader.invoke(
    {"documents": documents, "generation": generation}
)
```

Here’s what it does:

* It sends both the **generated answer** and the **retrieved documents** to the LLM grader.
* The grader checks:
  👉 *"Is this answer supported by the documents?"*
* Returns `"yes"` if **grounded**
* Returns `"no"` if it’s a **hallucination**

---

## 🧪 Real Example:

### 📥 Question:

> What are types of Agent Memory?

### 📚 Documents (retrieved):

> "There are 3 types of agent memory: Episodic, Semantic, and Procedural..."

### 🧠 AI Answer:

> "Agent memory includes episodic, semantic, and long-term memory."

### ✅ Result:

* The answer matches the document = **Grounded** = ✅

---

### 📥 Question:

> Who is the CEO of OpenAI?

### 📚 Documents (retrieved):

> (Nothing about the CEO is present)

### 🧠 AI Answer:

> "The CEO of OpenAI is Elon Musk."

### ❌ Result:

* Not found in documents + incorrect = **Hallucination** = ❌

## 🔁 Why this check matters?

Because in **Adaptive RAG**, we don’t trust the AI blindly.
We **check** whether it’s hallucinating.
If it is — we **retry or improve the question**.

That’s how you keep your RAG system **factual and reliable** ✅
'''