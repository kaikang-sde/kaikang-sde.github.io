---
title: Prompt Engineering
date: 2025-03-29
categories: [AI, Prompt Engineering]
tags: [LLM, Prompt Engineering, AI, NLP]
author: kai
---

# 🚀 Prompt Engineering
**Prompt Engineering** is the technique of designing **well-crafted textual inputs** to guide large language models (LLMs) like ChatGPT, Claude, or DeepSeek to generate **accurate and relevant outputs**.

Think of it as writing a **precise instruction manual** for your AI — the better the prompt, the better the result.

## 📝 Definition

> **Prompt Engineering** is the practice of crafting structured natural language inputs that steer a language model to perform a specific task or solve a particular problem.

In simple terms:  
It’s about **telling the AI clearly what you want**, so it delivers the **right results**.


### Think of LLMs as Your Employees

You're the **manager or boss**, and models like OpenAI GPT-4, DeepSeek, or Qwen are your **assistants**.

- If your instructions are **vague**, results will vary or fail.
- If you're **specific and clear**, the model will deliver exactly what you want.

This is **task delegation**, reimagined through AI.


## 🧠 Prompt Engineering vs Programming

| Aspect               | Traditional Coding         | Prompt Engineering               |
|----------------------|----------------------------|----------------------------------|
| Input                | Programming languages       | Natural language                 |
| Output               | Machine-executed behavior   | LLM-generated content/results    |
| Skills needed        | Syntax, logic, API usage    | Clarity, context design, tone    |
| Flexibility          | Deterministic               | Probabilistic, creative          |


## 🧩 The 4 Key Elements of Prompt Design
Prompt engineering isn’t just about typing a question into ChatGPT — it’s about **designing structured instructions** that guide large language models (LLMs) to produce consistent, accurate, and usable results.

A well-crafted prompt contains **four essential elements**:  


### 1. Role Assignment (Role Prompting)

By assigning a **specific identity or role** to the model, you control its **tone, knowledge scope, and output framing**.

| 🔴 Poor Example                  | ✅ Better Example                                                       |
|----------------------------------|------------------------------------------------------------------------|
| "Write a poem about spring"      | "You are a modern poet. Use metaphor to write an 8-line poem about spring." |

> ✅ Clearly telling the model *who it is* improves the relevance and consistency of the response.


### 2. Task Description
Use the **STAR method** to clearly communicate the task:

- **S**ituation: Describe the background or scenario  
- **T**ask: Define the objective  
- **A**ction: Specify steps or instructions  
- **R**esult: Explain the expected outcome  

🔹 Example:

> - **Situation**: A user submitted a technical issue.  
> - **Task**: Provide a clear and accurate solution.  
> - **Action**: Explain the resolution step-by-step.  
> - **Result**: End with a one-sentence summary.


### 3. Output Format
Structure your output for readability and reusability. You can specify:

- Bullet lists or numbered steps  
- Paragraph count  
- Tables (Markdown/HTML)  
- Code blocks  
- JSON/XML/YAML

🔹 Example:

**Output the result in the following JSON format:**

```json
{
  "summary": "A concise summary (under 50 words)",
  "keywords": ["keyword1", "keyword2", "keyword3"]
}
```

> 📐 Structure drives clarity and makes responses easy to parse and integrate into apps.


###  4. Constraints
Set clear **boundaries** to **control output behavior**. Use:

| Type | Example Constraint |
|------|--------------------|
| Length | "Limit response to 200 words" |
| Style | "Use language a middle school student can understand" |
| Content | "Avoid technical jargon or advanced math terms" |
| Logic | "Explain the concept before giving an example" |

🔹 Example:
> “Explain binary trees in less than 200 words, using simple terms and one analogy.”

### Summary Table

| Element | Purpose | Poor Prompt | Improved Prompt |
|------|------------|-------------|-----------------|
| Role Assignment | Define model identity | "Write code" | "You are a senior Java architect..." |
| Task Description | Clarify the action to perform | "Analyze data" | "Use a Markdown table to compare data trends..." |
| Output Format | Specify structure | Freeform answer | Return in JSON with summary and keywords |
| Constraints | Control length, tone, scope | No guidance | "Usder 200 words, no jargon, explain then example" |


## 🏆 The Golden Template: The Standard Three-Part Structure

```text
prompt_template = """
[Role Definition]
You are a {domain} expert with X years of experience, specializing in {skill}.

[Task Description]
Your job is to:
1. {step1}
2. {step2}
3. {step3}

[Output Requirements]
Please format your response as follows:
{output_format_example}
"""
```

## 🛠️ Prompt Debugging

| ❌ **Symptom**               | 🎯 **Likely Cause**                 | ✅ **Recommended Fix**                          |
|-----------------------------|------------------------------------|------------------------------------------------|
| Output drifts off-topic     | Unclear or missing role definition | Add a clear role and include “ignore unrelated content” |
| Output is too generic       | No step-by-step breakdown          | Include "Explain in steps" or use numbered tasks |
| Format is inconsistent      | No example output provided         | Provide a template in XML/JSON/Markdown format  |

### Examples
#### 1. Problem: Output feels vague or unrelated  
**Cause:** Role not defined clearly<br>
**Fix:**

```text
❌ Prompt: Explain spring

✅ Improved Prompt:
You are a nature poet. Ignore unrelated content. Write a modern poem about spring using metaphor.
```


#### 2. Problem: Output is generic or lacks depth
**Cause:** No task decomposition<br>
**Fix:**

```text
❌ Prompt: Compare two frameworks

✅ Improved Prompt:
Explain the differences between Django and Flask by:
1. Describing their architecture
2. Listing pros and cons
3. Recommending use cases
```


#### 3. Problem: Output formatting is inconsistent
**Cause:** No format constraints or examples<br>
**Fix:**

```text
✅ Add this to your prompt:
Respond in the following JSON structure:
{
  "framework": "Flask",
  "description": "Lightweight and flexible",
  "use_case": "Small-to-medium web apps"
}
```


## 🤖 Multi-Scenario Case Studies
Prompt engineering isn't just theory — it’s an **iterative, test-driven practice**.  
Whether you're building with **online APIs (OpenAI, Claude, DeepSeek)** or **local LLMs (LLama, ChatGLM)**, mastering prompt design is key to reliability, relevance, and automation.

Using **4 practical scenarios**, showing how to **analyze, test, and optimize** prompts for better results. Using free version of the ChatGPT for below examples. 

### 1: General Information Request
1️⃣ **Initial Prompt:** 
- **Input:** "Tell me about artificial intelligence."
- **Output:** 
    > Artificial intelligence (AI) refers to the simulation of human intelligence in machines that are programmed to think, learn, and make decisions. It involves creating systems that can perform tasks that typically require human-like cognition, such as understanding language, recognizing patterns, solving problems, and making decisions.
    Here are some key concepts and types of AI: ... (more than 400 words).

2️⃣ **Issues:**
- **Too generic:** No specific topic, angle, or target audience
- **Lacks scope control:** Doesn't define the expected depth or focus of the answer
- **Formatless output:** Leaves structure and tone entirely to the model

3️⃣ **Optimized Prompt:**
- **Input:** 

    ```text
    You are a technology columnist. Please explain the following to a high school student using simple language:
        1. What is artificial intelligence? (Include 1 real-life analogy)
        2. List 3 popular real-world applications of AI today
        3. Limit the entire answer to under 200 words
    Use a clear structure: "First," "Next," and "Finally."
    ```

- **Output:**

    ```text
    First, What is Artificial Intelligence?
    Artificial Intelligence (AI) is when machines or computers are made to think and act like humans. Imagine you have a smart robot that can learn how to clean your room. At first, it might not do the job perfectly, but over time, it learns from its mistakes and gets better. AI works the same way — it learns from data and improves its skills.
    Next, 3 Popular Real-World Applications of AI:
        1. Voice Assistants: AI powers Siri, Alexa, and Google Assistant, which can understand your voice and help with tasks like setting alarms or finding information.
        2. Self-Driving Cars: AI helps cars drive themselves by analyzing the road, making decisions, and learning to avoid obstacles.
        3. Recommendation Systems: AI suggests movies or products on platforms like Netflix or Amazon, based on what you’ve watched or bought before.
    Finally, AI is all around us, making things easier and helping machines get smarter. As it continues to grow, it will change how we live and work!
    ```

4️⃣ **Why This Prompt Works better:**
- **Role Definition:** Adds tone and voice - tech columnist, not a textbook
- **Target Audience:** Makes the language accessible (high school level)
- **Task Breakdown:** Clarifies the sub-goals (definition, analogy, applications)
- **Structure Instruction:** Controls output format with "First...Next...Finally..."
- **Word Limit:** Prevents verbosity and keeps response concise


### 2. Code Generation Assistant
1️⃣ **Initial Prompt:** 
- **Input:** "Write a Python program."
- **Output:** 

    ```text
    Sure! Here's a simple Python program that asks the user for their name and age, then prints a message with that information.

    # Python program to greet the user and calculate their year of birth

    # Ask the user for their name and age
    name = input("What's your name? ")
    age = int(input("How old are you? "))

    # Calculate the year of birth
    current_year = 2025
    year_of_birth = current_year - age

    # Print a greeting message
    print(f"Hello, {name}! You were born in {year_of_birth}.")

    How it works:
    The program asks for the user's name and age using input().
    It calculates the user's year of birth by subtracting their age from the current year (2025 in this case).
    Finally, it prints a message with their name and calculated year of birth.
    You can run this code in any Python environment, and it will greet you and tell you your year of birth based on your age! Let me know if you'd like to modify or extend the program.
    ```

2️⃣ **Issues:**
- **No task detail:** What should the program do?
- **No input/output:** The model doesn't know what the function should receive or return
- **No error handling:** Might ignore edge cases or bad input

3️⃣ **Optimized Prompt:**
- **Input:** 

    ```text
    Write a Python function with the following requirements:

    - Input: A date string in the format "YYYY-MM-DD"
    - Output: The quarter of the year as an integer (1 to 4)

    Requirements:
    - Use the `datetime` module for parsing
    - Raise `ValueError` if the input format is invalid
    - Include unit test cases using `unittest`

    Example:
    Input: "2024-03-15" → Output: 1
    ```

- **Output:**

    ```text
    Here's a Python function that meets your requirements, along with unit tests using the unittest module.

    import datetime
    import unittest

    def get_quarter(date_str):
        try:
            # Parse the date string to a datetime object
            date_obj = datetime.datetime.strptime(date_str, "%Y-%m-%d")
        except ValueError:
            # Raise an error if the date format is invalid
            raise ValueError("Invalid date format. Please use 'YYYY-MM-DD'.")

        # Determine the quarter of the year
        month = date_obj.month
        if 1 <= month <= 3:
            return 1
        elif 4 <= month <= 6:
            return 2
        elif 7 <= month <= 9:
            return 3
        else:
            return 4

    # Unit test cases
    class TestGetQuarter(unittest.TestCase):

        def test_valid_dates(self):
            self.assertEqual(get_quarter("2024-03-15"), 1)  # March is in Q1
            self.assertEqual(get_quarter("2024-06-30"), 2)  # June is in Q2
            self.assertEqual(get_quarter("2024-09-01"), 3)  # September is in Q3
            self.assertEqual(get_quarter("2024-12-25"), 4)  # December is in Q4

        def test_invalid_date_format(self):
            with self.assertRaises(ValueError):
                get_quarter("15-03-2024")  # Invalid format
            with self.assertRaises(ValueError):
                get_quarter("2024/03/15")  # Invalid separator
            with self.assertRaises(ValueError):
                get_quarter("2024-03-32")  # Invalid day (March 32nd)

    if __name__ == "__main__":
        unittest.main()

    Explanation:
    get_quarter(date_str):
    The function takes a date string as input, attempts to parse it using datetime.datetime.strptime() with the format "%Y-%m-%d".
    If the date is in an invalid format, it raises a ValueError.
    The function calculates which quarter the month falls into:
    January to March → Quarter 1
    April to June → Quarter 2
    July to September → Quarter 3
    October to December → Quarter 4
    Unit Tests:
    The TestGetQuarter class contains several test cases:
    test_valid_dates: Tests valid date strings and checks if the quarter returned is correct.
    test_invalid_date_format: Tests invalid date formats and ensures a ValueError is raised.

    Example Execution:
    # Run the unit tests
    if __name__ == "__main__":
        unittest.main()
    This will run the tests and check the function's behavior. If you provide an invalid date format, it will raise an error as expected.
    ```

4️⃣ **Why This Prompt Works better:**
- **Clear function goal:** Makes the task explicit
- **I/O defined:** Prevents confusion about expected usage
- **Implementation detail:** Forces use of datetime (prevents shortcuts)
- **Test coverage:** Ensures quality and correctness
- **Example provided:** Reduces ambiguity and improves accuracy


### 3: Technical Q&A with Structured Output
1️⃣ **Initial Prompt:** 
- **Input:** "How can I improve website performance?"
- **Output:** 
    > Improving website performance is essential for providing a better user experience and enhancing your site's search engine ranking. Here are some effective ways to boost website performance: 1. Optimize Images: Compress images: Use tools like TinyPNG or ImageOptim to compress image sizes without losing quality...(more than 600 words).

2️⃣ **Issues:**
- **Too broad:** No context - frontend or backend? Mobile or desktop?
- **No stack defined:** Model doesn't know which framework is being used
- **No metric requested:** Cannot quantify or evaluate suggestions
- **No exclusions:** Results may include generic suggestions you already know

3️⃣ **Optimized Prompt:**
- **Input:** 

    ```text
    You are a full-stack performance consultant.

    Using a SpringBoot + Vue3 tech stack, provide 5 quantifiable website performance optimization strategies.

    Each strategy must include:
    - Specific implementation steps
    - Estimated performance gain (e.g. LCP ↓ 20%)
    - Complexity level (Low / Medium / High)

    Additional Constraints:
    - Focus on frontend and backend logic (not infrastructure)
    - Exclude common techniques like minification, image compression, or CDN
    - Use Web Vitals as the performance evaluation standard
    ```

- **Output:**

    ```text
    Here are five quantifiable website performance optimization strategies for a SpringBoot + Vue3 tech stack, focusing on frontend and backend logic and using Web Vitals as the evaluation standard. Each strategy includes specific implementation steps, estimated performance gains, and complexity levels.

    1. Lazy Loading of Components and Routes (Vue3)
    Goal: Reduce LCP (Largest Contentful Paint) and improve FID (First Input Delay).

    Implementation Steps:
        1. Enable lazy loading of Vue components by using dynamic imports in Vue 3. Example:
            const Home = () => import('@/components/Home.vue');
        2. Configure Vue Router to lazy load routes:
            const routes = [
                { path: '/home', component: () => import('@/views/Home.vue') }
            ];
        3. Ensure critical above-the-fold content is eagerly loaded, while deferring non-essential content.
        4. Add Vue Router prefetching for important routes that users might navigate to next.
        Estimated Performance Gain:
            - LCP ↓ 25% (Since initial JavaScript load is reduced)
            - FID ↓ 15% (Smaller initial bundle size leads to faster interaction readiness)
        Complexity Level: Medium
            - Requires restructuring the app and careful management of which components to lazy load and prefetch.

    2. Server-Side Rendering (SSR) or Static Site Generation (SSG) with Vue3
        ....
    ... more 
    ```

4️⃣ **Why This Prompt Works better:**
- **Tech stack defined:** Helps the model tailor suggestions to relevant tools/frameworks
- **Structure format** Encourages table-style or list-style consistency
- **Quantifiable metrics:** Makes suggestions measurable and goal-oriented
- **Constraint clarity:** Prevents duplication of known/common optimizations


### 4. Data Analysis with Visual Reporting
1️⃣ **Initial Prompt:** 
- **Input:** "Analyze sales data."
    
2️⃣ **Issues:**
- **No context:** Model doesn't know what the data contains
- **No analysis goal:** No indication of what insights are expected
- **No visualization:** No instruction to generate visuals or summarize findings
- **Format ambiguity:** Model might return unstructured text or incomplete insights

3️⃣ **Optimized Prompt:**
- **Input:** 

    ```text
    You are a senior data analyst. Please analyze the following sales dataset:

    Dataset Details:
    - Time Range: January to December, 2027
    - Fields: Date, Product Category, Sales Amount, Profit Margin

    Tasks:
    1. Identify the top 3 months by total sales and explain the reasons for their growth
    2. Detect product categories with profit margins below 5%
    3. Generate a Markdown report with insights and a trend line chart

    Output Format:
    ## Analysis Report

    ### Key Findings
    - Insight 1 (with data reference)
    - Insight 2 (with comparative reasoning)

    ### Visualization
    Include a short description of the trend and a base64-encoded line chart (Sales over time).
    ```

4️⃣ **Why This Prompt Works better:**
- **Role Definition:** Ensures tone and perspective aligns with business reporting
- **Data schema description:** Guides the model in understanding available fields and time range
- **Clear task objectives:** Ensures actionable and measurable insights
- **Visualization required:** Encourages dynamic output such as charts or summaries
- **Structured format:** Enables integration with reports, dashboards, or automated emails


## 👍🏻 Prompt Engineering Rule of Thumb

**“If the model didn’t do what you expected, the prompt probably didn’t say it clearly enough.”**





<br>

---

🚀 Stay tuned for more insights into AI!