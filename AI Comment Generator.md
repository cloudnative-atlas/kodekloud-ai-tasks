# AI Comment Generator

A Python utility that uses the OpenAI Chat Completion API to analyze a code snippet and generate a clear, one-line comment or docstring describing its purpose. Designed to improve code readability and maintainability through automated documentation.

---

## Task

AI Comment Generator

The `nautilus` AI Development Team is exploring how artificial intelligence can improve code readability and maintainability by automatically generating meaningful comments. You are tasked with building a Python-based AI module that analyzes a code snippet and produces a clear, one-line comment or docstring describing its purpose.

Inside `/root/openaiproject/commenter.py`, create an `OpenAI` client using the API key and base URL provided in `/root/.bash_profile`. Define a function `generate_comment(code_snippet: str) -> str` that constructs a parameterized prompt instructing the AI to generate a one-line comment explaining the provided code.

After building the function, send the constructed `prompt` to the OpenAI chat model:

* model: `openai/gpt-4.1-mini`
* messages: user → `prompt`
* max_tokens: `30`
* temperature: `0.2`

Save the result in a variable named `response` and print the generated comment or docstring to the console.

Use the following snippet for testing inside your script:

```python
def calculate_area(length, width):
    return length * width
```

**Notes:**

1. Ensure you are working inside `/root/openaiproject`.
2. `api_key` and `base_url` are stored in `/root/.bash_profile`.
3. The function must be parameterized using the input snippet.
4. The function must return the generated comment AND print it.
5. Use hardcoded values for `api_key` and `base_url` when initializing the OpenAI client or read them from environment variables via `os.environ.get('API_KEY')` and `os.environ.get('BASE_URL')`.
6. Before running, create a virtual environment and install OpenAI:

```bash
python3 -m venv venv && source venv/bin/activate && pip install openai
```

7. You are allowed up to `10` OpenAI requests. Past this limit you may encounter a `rate limiter error`, so optimize your request usage.

---

## Environment Setup

Before running the script, create and activate a virtual environment, then install the OpenAI SDK:

```bash
cd /root/openaiproject
python3 -m venv venv
source venv/bin/activate
pip install openai
```

Source the credentials and confirm the actual variable names that are exported:

```bash
source /root/.bash_profile
env | grep -i openai
```

---

## Solution

`/root/openaiproject/commenter.py`:

```python
import os
from openai import OpenAI

# Initialize the OpenAI client using environment variables
client = OpenAI(
    api_key=os.environ.get('OPENAI_API_KEY'),
    base_url=os.environ.get('OPENAI_API_BASE'),
)


def generate_comment(code_snippet: str) -> str:
    """
    Analyzes a code snippet and returns a clear, one-line comment
    describing its purpose. Also prints the comment to the console.
    """
    # Parameterized prompt that instructs the model to summarize the code
    prompt = (
        f"Generate a clear, one-line comment or docstring that explains "
        f"the purpose of the following code:\n\n{code_snippet}"
    )

    completion = client.chat.completions.create(
        model="openai/gpt-4.1-mini",
        messages=[
            {"role": "user", "content": prompt}
        ],
        max_tokens=30,
        temperature=0.2
    )

    comment = completion.choices[0].message.content
    print(comment)
    return comment


if __name__ == "__main__":
    code_snippet = """def calculate_area(length, width):
    return length * width"""

    # Store and print the generated comment
    response = generate_comment(code_snippet)
```

---

## Explanation of the Solution

The script is structured into three logical layers: client setup, the parameterized comment-generation function, and the script entry point.

### 1. Client Initialization

```python
client = OpenAI(
    api_key=os.environ.get('API_KEY'),
    base_url=os.environ.get('BASE_URL')
)
```

The OpenAI SDK client is created at module level so it is initialized exactly once per process. Both `api_key` and `base_url` are read from environment variables sourced from `/root/.bash_profile`. Reading from the environment instead of hardcoding keeps secrets out of source control, and passing `base_url` explicitly is what allows the SDK to target a routed proxy endpoint instead of the public OpenAI API.

### 2. The `generate_comment` Function

```python
def generate_comment(code_snippet: str) -> str:
    prompt = (
        f"Generate a clear, one-line comment or docstring that explains "
        f"the purpose of the following code:\n\n{code_snippet}"
    )
```

The function accepts any code snippet as input and builds a parameterized prompt using an f-string. The snippet is interpolated dynamically rather than hardcoded, so the same function can be reused to comment on any function, class, or block of code. Separating the instruction from the code with a blank line (`\n\n`) gives the model a visual cue that one part is the directive and the other is the content to analyze, which improves output quality.

### 3. The API Call

```python
completion = client.chat.completions.create(
    model="openai/gpt-4.1-mini",
    messages=[{"role": "user", "content": prompt}],
    max_tokens=30,
    temperature=0.2
)
```

Each parameter is tuned to match the task's requirement of producing a single-line summary:

- **`model="openai/gpt-4.1-mini"`** is a fast, lightweight model that handles short summarization tasks efficiently. The `openai/` prefix is required because the lab routes requests through a proxy.
- **`messages=[{"role": "user", "content": prompt}]`** sends the prompt as a single user message, which is enough for a one-shot generation task.
- **`max_tokens=30`** is a deliberately tight cap. A one-line comment rarely needs more than 20 to 30 tokens, and capping aggressively prevents the model from drifting into multi-line explanations or extended docstrings.
- **`temperature=0.2`** keeps the output mostly deterministic with just a small amount of variation. This is the right tradeoff for code commentary: you want consistency (the same code should produce roughly the same comment), but a small temperature lets the model phrase things naturally rather than always producing the exact same wording.

### 4. Both Returning and Printing

```python
comment = completion.choices[0].message.content
print(comment)
return comment
```

The task explicitly requires the function to both **return** the comment and **print** it. This is a small but important distinction: returning makes the function composable (you can pipe its output into another step, write it to a file, or feed it to a linter), while printing satisfies the immediate console-output requirement. Doing both in the same function is a common pattern when a utility must support both interactive and programmatic use.

### 5. The Test Snippet

```python
code_snippet = """def calculate_area(length, width):
    return length * width"""
```

The test snippet is wrapped in a triple-quoted string so its formatting (the `def` line and the indented `return`) is preserved when interpolated into the prompt. Preserving indentation matters: it gives the model a properly formatted Python function rather than a flattened single-line blob, which tends to produce cleaner, more accurate comments.

### 6. The Entry Point

```python
if __name__ == "__main__":
    code_snippet = """..."""
    response = generate_comment(code_snippet)
```

The `if __name__ == "__main__"` guard ensures the demo call runs only when the script is executed directly, not when imported as a module. The result is stored in a variable named `response` to satisfy the task requirement, even though `generate_comment` already prints internally.

---

## Running the Script

With the venv active and credentials sourced:

```bash
python commenter.py
```

### Sample Output

```
# Calculates the area of a rectangle given its length and width.
```

Because `temperature=0.2`, output remains highly consistent across runs. Minor phrasing variations may occur, but the meaning will stay the same.

---

## Common Pitfalls

| Symptom | Likely Cause | Fix |
|---|---|---|
| `401 Incorrect API key provided` pointing at `platform.openai.com` | `base_url` is `None` because the env variable name in the script does not match the one in `.bash_profile`. The SDK falls back to public OpenAI and rejects the proxy key. | Run `env \| grep -i openai` and use the exact variable names seen in the output |
| `ModuleNotFoundError: No module named 'openai'` | Running Python outside the activated venv | Run `source venv/bin/activate` and confirm the `(venv)` prefix appears in the prompt |
| Output is truncated mid-sentence | `max_tokens=30` is hit before the model finishes | Confirm the prompt is asking for a *one-line* comment; if you genuinely need longer output, raise `max_tokens` |
| Output includes the function code itself, not just a comment | The model echoed the snippet back | Tighten the prompt wording, for example: "Return ONLY a one-line comment, no code." |
| Rate limit error before the task is verified | More than 10 requests sent during debugging | Validate inputs with `assert` and `print` statements before the API call so you do not spend a request on a typo |
