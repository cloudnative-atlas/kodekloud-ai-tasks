# AI Bug Description Clarifier

A Python utility that uses the OpenAI Chat Completion API to transform informal, developer-written bug reports into clear, structured, and professional issue summaries.

---

## Task

Bug Description Clarifier

The `nautilus` AI Engineering team is developing tools to improve the clarity of developer-reported bugs. Developers often report issues informally, which makes them difficult to understand or reproduce.

You are tasked to build a Python-based AI Bug Description Clarifier that transforms such informal bug reports into clear, structured, and professional issue summaries.

Inside `/root/openaiproject/bug_clarifier.py`:

1. Initialize the `OpenAI` client using environment values (`api_key` and `base_url`).
2. Define a function `clarify_bug(description: str) -> str` that builds a parameterized prompt to rewrite the raw bug description.
3. Send this prompt to the OpenAI Chat Completion API.
4. Use the following configuration for the API call:
   * model: `openai/gpt-4.1-mini`
   * messages: user → the constructed prompt
   * max_tokens: `100`
   * temperature: `0.0`
5. Use the input bug report:

```text
App keeps crashing when I click save.
```

6. Store the AI response in a variable named `response` and print the clarified bug summary to the console.

**Notes:**

1. Function must use the developer's input description dynamically in the prompt.
2. Ensure you are working inside `/root/openaiproject`.
3. OpenAI credentials are available in `/root/.bash_profile`.
4. Use hardcoded values for `api_key` and `base_url` when initializing the `OpenAI client` or read them from environment variables via `os.environ.get('API_KEY')` and `os.environ.get('BASE_URL')`.
5. Before running `bug_clarifier.py`, set up a virtual environment:

```bash
python3 -m venv venv && source venv/bin/activate && pip install openai
```

6. Maximum of 10 API requests allowed before rate limiting.

---

## Environment Setup

Before running the script, create and activate a virtual environment, then install the OpenAI SDK:

```bash
cd /root/openaiproject
python3 -m venv venv
source venv/bin/activate
pip install openai
```

Make sure the credentials from `/root/.bash_profile` are loaded into your shell:

```bash
source /root/.bash_profile
echo $API_KEY
echo $BASE_URL
```

---

## Solution

`/root/openaiproject/bug_clarifier.py`:

```python
import os
from openai import OpenAI

# Initialize the OpenAI client using environment variables
client = OpenAI(
    api_key=os.environ.get('API_KEY'),
    base_url=os.environ.get('BASE_URL')
)


def clarify_bug(description: str) -> str:
    """
    Transforms informal bug reports into professional, structured summaries.
    """
    # Parameterized prompt for clarity and structure
    prompt = (
        f"Rewrite the following informal bug report into a clear, "
        f"professional, and structured issue summary:\n\n'{description}'"
    )

    completion = client.chat.completions.create(
        model="openai/gpt-4.1-mini",
        messages=[
            {"role": "user", "content": prompt}
        ],
        max_tokens=100,
        temperature=0.0
    )

    return completion.choices[0].message.content


if __name__ == "__main__":
    raw_report = "App keeps crashing when I click save."

    # Store and print the AI response
    response = clarify_bug(raw_report)
    print(response)
```

---

## Explanation of the Solution

The script is structured into three logical layers: client setup, the reusable transformation function, and the script entry point.

### 1. Client Initialization

```python
client = OpenAI(
    api_key=os.environ.get('API_KEY'),
    base_url=os.environ.get('BASE_URL')
)
```

The OpenAI SDK client is created at module level so that it is initialized exactly once per process, rather than being rebuilt on every function call. Both `api_key` and `base_url` are read from environment variables sourced from `/root/.bash_profile`. Passing `base_url` explicitly is what allows the same SDK to target a routed or proxied endpoint instead of the default public OpenAI API. Reading from the environment instead of hardcoding keeps the credentials out of source control.

### 2. The `clarify_bug` Function

```python
def clarify_bug(description: str) -> str:
    prompt = (
        f"Rewrite the following informal bug report into a clear, "
        f"professional, and structured issue summary:\n\n'{description}'"
    )
```

The function accepts any raw bug description as input and builds a parameterized prompt using an f-string. The developer's text is interpolated dynamically rather than hardcoded, so the same function can be reused for any bug report. Wrapping the description in quotes inside the prompt helps the model treat it as the content to transform rather than as part of the instruction itself.

### 3. The API Call

```python
completion = client.chat.completions.create(
    model="openai/gpt-4.1-mini",
    messages=[{"role": "user", "content": prompt}],
    max_tokens=100,
    temperature=0.0
)
```

Each parameter has a deliberate purpose:

- **`model="openai/gpt-4.1-mini"`** uses a lightweight model that is fast and inexpensive while still being capable of structured rewriting tasks.
- **`messages=[{"role": "user", "content": prompt}]`** uses a single user-role message, which is sufficient for a one-shot transformation. There is no system message or prior context required.
- **`max_tokens=100`** caps the response length, preventing the model from drifting into long-form prose and keeping the summary concise enough to fit cleanly inside an issue tracker.
- **`temperature=0.0`** forces deterministic output. The same bug report will produce the same rewritten summary across runs, which is critical when this is integrated into automated tooling where reproducibility matters.

### 4. Extracting the Response

```python
return completion.choices[0].message.content
```

The OpenAI SDK returns a structured response object. The actual text generated by the model lives at `choices[0].message.content`. The `choices` list supports multiple completions per request (controlled by the `n` parameter), but since we did not request more than one, we take the first and only entry.

### 5. The Entry Point

```python
if __name__ == "__main__":
    raw_report = "App keeps crashing when I click save."
    response = clarify_bug(raw_report)
    print(response)
```

The `if __name__ == "__main__"` guard ensures the script only executes the demo call when run directly, not when imported as a module. This makes `clarify_bug` reusable from other scripts or services without triggering an unintended API call. The result is stored in `response` (matching the task requirement) and printed for inspection.

---

## Running the Script

With the venv active and credentials sourced:

```bash
python bug_clarifier.py
```

### Sample Output

```
Issue Summary:
The application consistently crashes when the user clicks the "Save" button.

Steps to Reproduce:
1. Open the application.
2. Perform any action that enables the Save functionality.
3. Click the "Save" button.

Expected Behavior:
The application should save the data without crashing.

Actual Behavior:
The application crashes immediately upon clicking Save.
```

Output will vary slightly based on model behavior, but with `temperature=0.0` it remains stable across runs for the same input.

---

## Common Pitfalls

| Symptom | Likely Cause | Fix |
|---|---|---|
| `401 Incorrect API key provided` | `base_url` is `None`, so the SDK falls back to `api.openai.com` and rejects the proxy key | Verify the env variable name matches what is set in `.bash_profile` (run `env \| grep -i api`) |
| `ModuleNotFoundError: No module named 'openai'` | Running outside the activated venv | Run `source venv/bin/activate` before invoking Python |
| Empty or `None` response content | Token cap too low for the response | Confirm `max_tokens` is set to `100` as required |
| Rate limit error | Used more than 10 requests during the session | Add inline assertions and print statements before each API call to validate inputs without spending requests |
