# QnA-with-LLM
This repository contains a simple implementation of a question-answering system using a large language model (LLM), LangChain and aisuite. The system is designed to answer questions based on a given context, which can be a text file or any other source of information.

## Workflow
Our approach involves the following steps:
1. **Summarize a corpus of documents**: This is a preliminary step where we summarize the content of the documents to create a concise representation of the information. This summary will serve later in the process to generate questions and answers.
2. **Create Multiple Choice Questions (MCQs)**: We generate multiple-choice questions based on the summarized content. This helps for the next step, where we will use these questions to differentiate the subtopics or themes within the documents.
3. **Answer Questions**: Finally, we use the LLM to answer questions based on the context provided. The LLM can be prompted to answer questions based on the content of the documents, and it can also generate answers in a specific format.

### Summarize a Corpus of Documents
The first step is to summarize the content of the documents.The goal is to create a concise representation of the information that can be used for generating questions and answers. It can be achived by sending the entire corpus or just a subset. 

- In one experiment, we use the LLM chat to summarize the content of the documents by sending it the entire documents. Here is the response returned by the LLM chat:
"I’ve taken a look at your uploaded document (documents_masked.json). It contains a diverse set of text entries from what appears to be sports discussions, possibly from forums or mailing lists, mostly centered around hockey and baseball, with occasional player statistics, commentary, and historical data."
- In another experiment - [20ng_gen](20ng_gen.ipynb) - we use the LLM API to summarize the documents by sending the subset due to the input token limit. Here is the response returned by the LLM API: "Based on the analysis of the provided corpus of documents, several key subtopics related to sports, particularly hockey and baseball, emerge. Below is a detailed summary of the key subtopics, including the identification of specific sports games, contrasts, and overlaps in themes across the documents..."

### Create Multiple Choice Questions (MCQs)
The next step is to generate multiple-choice questions based on the summarized content. This can be done using various techniques, 
- NIPS paper example: [automatic_questions_generation_v2](automatic_questions_generation_v2.ipynb). It send LLM the subsets of abstracts of the NIPS papers and asks LLM to generate multiple-choice questions (MCQs) that highlight key differences among those papers. 
Here is the prompt:
```
You are an AI assistant analyzing a corpus of NIPS papers spanning multiple years. 
Your task is to design multiple-choice questions (MCQs) that highlight key differences among those papers. 
I will provide abstracts of NIPS papers, and you will generate unique, comprehensive MCQs based on them. 

These questions should:
- Help to differentiate between the papers.
- Are common to all articles, not specific to any one paper.
- Cover various aspects, including but not limited to research focus, methodology, theoretical advancements, and applications.
- Are unique and not repeated in any other batch.
- Include "None of the above" as one of the options for each question.

Below is the format and example I need:

1. What is the primary focus of the paper?
   - A. Reinforcement learning
   - B. Deep learning
   - C. Bayesian methods
   - D. Kernel methods
   - E. None of the above

2. What is the primary focus of the paper?
   - A. Faster convergence compared to existing methods
   - B. Higher accuracy
   - C. Better scalability to large datasets
   - D. Improved interpretability of results
   - E. None of the above

Generate as many unique and comprehensive questions as possible based on the given paper abstracts below. 
Each question should be designed to highlight key differences between the papers.

Abstracts: {chunk}
```
- 20 News Groups example: [20ng_langchain_v2](20ng_langchain_v2.ipynb). It send LLM directly the summary of the subsets of the 20 News Groups and asks LLM to generate multiple-choice questions (MCQs) to distinguish subtopics. Here is the prompt:
```
Given the following sports-related forum posts:
{joined_text}

It contains a diverse set of text entries from what appears to be sports discussions, possibly from forums or mailing lists, mostly centered around hockey and baseball.

Generate 3 multiple-choice questions that help distinguish subtopics, such as hockey or baseball of discussion.

Each question should have 4 options (A, B, C, D), including “None of the above” as one of the answer choices for every question.

Format your response as a valid JSON array like this:

[
    {{"question": "What is the main topic of the post?", "options": ["A. Player analysis", "B. Media complaints", "C. Statistics discussion", "D. None of the above"]}},
    ...
]
Only return the JSON — no prose or commentary.
```
It is clear that the LLM is capable of generating multiple-choice questions (MCQs) that help distinguish subtopics. It can also follow your instructions to generate the questions in a specific format.

### Answer Questions
The final step is to use the LLM to answer questions based on the context provided. This can be done using various techniques.
- 20 News Group example: [20ng_generate_answers](20ng_generate_answers.ipynb). It sends LLM the subsets of abstracts of the NIPS papers and asks LLM answer the MCQs generated in the previous step. Here is the prompt:
```
pre = "You are an AI trained to understand documents and generate concise answers to multiple-choice questions based on the content. \
        Please read the following document carefully. After reading, answer ALL the questions listed below. \
            Your answers must be in capital letters and formatted as a single string, where each question number is followed by its corresponding answer letter. \
                Separate each question-answer pair with a semicolon. \
                    Example format: 1A;2B;3C;4D;... \n\n"

prompt = pre + f"Document Content:\n{article_content}\n\n Questions: {questions}\n"
```
- UN General Debate Corpus: [QnA_ZH](QnA_ZH.ipynb). It sends LLM the speeches of the UN General Debate Corpus and asks LLM to answer the MCQs generated in the previous step. Here is the prompt:
```
{
            "role": "system",
            "content": "You are expert analyzing UN General Assembly speeches and answer MCQs by country.",
        },
        {
            "role": "user",
            "content": f"""
I will provide speech delivered by representatives from various countries at the United Nations General Assembly.

Your task is to:
1. Read the entire provided speech document. 
2. Answer the following multiple-choice questions based solely on the text content identified as belonging to that specific country's speech section within the document.
3. Format your response as a valid JSON array like this:
{{"Q1": "A","Q2": "A","Q3": "C","Q4": "A","Q5": "A","Q6": "A"}}

Only return the JSON — no prose or commentary.

--- Questions ---
{json.dumps(mcqs, indent=2)}

--- Speech ---
{text_data}

""",
        }
```

It is obvious that the LLM is capable of answering questions based on the context provided. It can also follow your instructions to generate the answers in a specific format.

## Lessons Learned
- The LLM can effectively summarize the content of documents, either by sending the entire document or a subset of it. If you are sending the sample document, try randomly sampling the document to get a better representation of the content, and try multiple times and compare the results.
- Token limits are important to consider when using LLMs. If the size of the corpus is large, you may need to break it down into smaller chunks or subsets to fit within the token limits of the LLM. The token limit for different LLMs varies, so be sure to check the documentation for the specific model you are using. You can also leverage the file upload feature of the LLM to upload the document and ask LLM to summarize the content.
- The LLM can generate multiple-choice questions (MCQs). Here the prompt engineering is very important to get the desired results. The questions generated by the LLM is driven by the prompt and the context provided. Hence the prompt need to articulated in the direction or space that you want the LLM to go. For example, in the NIPS paper example, we ask LLM to generate questions that highlight key differences among the papers. In the 20 News Groups example, we ask LLM to generate questions that help distinguish subtopics. Providing "None of the above" as one of the options for each question is also important to get the desired results.
- The LLM can answer questions based on the context provided, and it can also generate answers in a specific format. If your corpus is big, you can save the response from LLM at batch, for example - every 500 documents - so that even if the LLM got interrupted, you can continue from where it left off. 
- Different LLM models have different capabilities and performance. Just be aware of the limitations of the LLM you are using. Not one single prompt will work for all LLMs. You need to experiment with different prompts and models to get the desired results.
- Different LLM models have different pricing. The cost of using LLMs can add up quickly, especially if you are processing large amounts of data. Be sure to monitor your usage and costs carefully. Be smart to use different LLM models for different tasks. For example, you can use the cheaper LLM model to answer the MCQs and but use the more expensive LLM model to generate MCQ questions, as the quality of the questions is more critical than the tasks of answering questions.

## Summary
This repository provides a simple implementation of a question-answering system using a large language model (LLM). The system is designed to answer questions based on a given context, which can be a text file or any other source of information. The workflow involves summarizing a corpus of documents, creating multiple-choice questions (MCQs), and answering questions using the LLM. The LLM models evolve rapidly, the limits of the context are increasing, the performance is improving, and the prompt engineering is becoming more and more important and sophisticated to specific tasks.
The examples provided in this repository demonstrate the capabilities of LLMs in generating questions and answers based on the context. This repository serves as a starting point for building more advanced question-answering systems using LLMs, LangChain and aisuite.


## Security
### Secure the API key
- The API key is stored in the `.env` file. This file is not included in the repository. This can be done by adding the file to the `.gitignore` file.

```python
# Environments
.env
```
- Example of the `.env` file:
```python
OPENAI_API_KEY=********************
```
- The API key is loaded using the `os` and `load_dotenv` library in Python.
```python
import os
from dotenv import load_dotenv

load_dotenv()  # This loads the environment variables from .env file

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
```