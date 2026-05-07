# AI ChatBot

A Python role-play chatbot built using the OpenAI Chat Completion API. The bot acts as a friendly travel guide that greets the user and asks where they want to travel.

---

## Task

AI ChatBot

The AI development team at `Devops` is tasked with building a role-play chatbot using OpenAI's API.

**Task Requirements:**

1. Navigate to the `/root/openaiproject/chatbot.py` directory.
2. Create a `client` instance using `api_key` and `base_url`.
3. Use openai model=`openai/gpt-4.1-mini`
4. Define a variable `prompt` with the following content:

```
You are a friendly travel guide. Greet the user and ask where they want to go.
```

5. Send this prompt to the OpenAI chat model and store the result in variable name `response`.
6. Extract and print the generated text reply from the `response`.
7. Run the file after installing OpenAI in a virtual environment.

**Notes:**

1. Ensure you are working inside `/root/openaiproject`.
2. `api_key` & `base_url` are in `/root/.bash_profile` (typically `OPENAI_API_KEY` and `OPENAI_API_BASE_URL`).
3. Install OpenAI inside a venv before running the script.

```bash
python3 -m venv venv && source venv/bin/activate && pip install openai
```

4. Use `temperature=0.7` & `max_tokens=100`.
5. Use hardcoded values for `api_key` & `base_url` when initializing the OpenAI client, or read them from environment variables via `os.environ.get('API_KEY')` and `os.environ.get('BASE_URL')`.
6. You are allowed a maximum of `10` requests. After this, you may encounter a `rate limiter error`. Therefore, use your requests judiciously.

---

## Environment Setup

Before running the script, create and activate a virtual environment, then install the OpenAI SDK:

```bash
cd /root/openaiproject
python3 -m venv venv
source venv/bin/activate
pip install openai
```

Make sure the credentials from `/root/.bash_profile` are loaded into your shell, and confirm the exact variable names that are set:

```bash
source /root/.bash_profile
env | grep -i openai
```

The output of `env | grep -i openai` will reveal the actual variable names. In this lab the variables are `OPENAI_API_KEY` and `OPENAI_API_BASE` (note: `OPENAI_API_BASE`, without the `_URL` suffix that the task notes suggest). Always use the exact name from the environment, not from the task description.

---

## Solution

`/root/openaiproject/chatbot.py`:

```python
import os
from openai import OpenAI

# Read credentials from the environment (sourced from /root/.bash_profile)
api_key = os.environ.get("OPENAI_API_KEY")
base_url = os.environ.get("OPENAI_API_BASE")

# Create the OpenAI client instance
client = OpenAI(
    api_key=api_key,
    base_url=base_url,
)

# Role-play prompt for the travel guide chatbot
prompt = "You are a friendly travel guide. Greet the user and ask where they want to go."

# Send the prompt to the chat model
response = client.chat.completions.create(
    model="openai/gpt-4.1-mini",
    messages=[
        {"role": "user", "content": prompt}
    ],
    temperature=0.7,
    max_tokens=100,
)

# Extract and print the generated reply
print(response.choices[0].message.content)
```

---

## Explanation of the Solution

The script is built in four logical parts: reading credentials, creating the client, defining the prompt, and making the API call.

### 1. Reading Credentials from the Environment

```python
api_key = os.environ.get("OPENAI_API_KEY")
base_url = os.environ.get("OPENAI_API_BASE")
```

Both values are sourced from `/root/.bash_profile` rather than hardcoded into the script. This keeps secrets out of source control and allows the same code to run against different endpoints (public OpenAI, KodeKloud proxy, internal gateway) by simply changing the environment. The `os.environ.get()` call returns `None` if the variable does not exist, which is important to be aware of: if `base_url` is `None`, the SDK silently falls back to `api.openai.com`, which is the most common cause of confusing 401 errors in this lab.

### 2. Creating the OpenAI Client

```python
client = OpenAI(
    api_key=api_key,
    base_url=base_url,
)
```

The modern OpenAI Python SDK (version 1.x and above) uses an `OpenAI` class that accepts `api_key` and `base_url` directly as constructor arguments. The client is created once at module level so that it is reused for any subsequent calls in the script, which is more efficient than rebuilding it per request.

### 3. Defining the Prompt

```python
prompt = "You are a friendly travel guide. Greet the user and ask where they want to go."
```

This is the role-play instruction that primes the model. Although it is technically more conventional to put this kind of role-setting text into a `system` message, the task explicitly asks for it as a `user` message, so the script follows that pattern. The model is capable of interpreting the instruction either way.

### 4. The API Call

```python
response = client.chat.completions.create(
    model="openai/gpt-4.1-mini",
    messages=[
        {"role": "user", "content": prompt}
    ],
    temperature=0.7,
    max_tokens=100,
)
```

Each parameter has a deliberate purpose:

- **`model="openai/gpt-4.1-mini"`** uses a fast, cost-efficient chat model. The `openai/` prefix is required because the lab routes requests through a proxy that supports multiple model providers, and this prefix tells the proxy which backend to use.
- **`messages=[{"role": "user", "content": prompt}]`** sends the prompt as a user-role message, matching the task requirement.
- **`temperature=0.7`** adds a moderate amount of randomness to the response. This makes the chatbot's greeting feel more natural and varied across runs, rather than producing the exact same output every time.
- **`max_tokens=100`** limits the reply length. A travel-guide greeting should be short, and this cap also avoids burning unnecessary tokens against the 10-request budget.

### 5. Extracting and Printing the Reply

```python
print(response.choices[0].message.content)
```

The OpenAI SDK returns a structured `ChatCompletion` object. The actual generated text lives at `response.choices[0].message.content`. The `choices` list can hold multiple completions if the `n` parameter is greater than 1, but here we requested only one, so we take the first element. `.message.content` then extracts the assistant's reply as a plain string for printing.

---

## Running the Script

With the venv active and credentials sourced:

```bash
python chatbot.py
```

### Sample Output

```
Hello there, traveler! I'm so excited to help you plan your next adventure. 
Tell me, where in the world would you like to go? Are you dreaming of sunny 
beaches, snowy mountains, bustling cities, or perhaps a quiet countryside escape?
```

Because `temperature=0.7` introduces randomness, the exact wording will differ on each run, but the structure (greeting plus a question about destination) remains consistent.

---

## Common Pitfalls

| Symptom | Likely Cause | Fix |
|---|---|---|
| `401 Incorrect API key provided` pointing at `platform.openai.com` | `base_url` is `None` because the env variable name in the script does not match the actual one in `.bash_profile`. The SDK falls back to public OpenAI and rejects the proxy key. | Run `env \| grep -i openai` and use the exact variable name from the output (in this lab, `OPENAI_API_BASE`, not `OPENAI_API_BASE_URL`) |
| `ModuleNotFoundError: No module named 'openai'` | Running Python outside the activated venv | Run `source venv/bin/activate` and confirm the `(venv)` prefix appears in the prompt |
| Empty environment variables when running the script | `.bash_profile` was not sourced into the current shell | Run `source /root/.bash_profile` before invoking Python, then verify with `echo $OPENAI_API_KEY` |
| Rate limit error before the task is verified | More than 10 requests sent during debugging | Use sanity-check `assert` statements and `print` debugging on the env variables before the API call so you do not spend a request to discover a typo |
| `model_not_found` or invalid model error | Forgot the `openai/` prefix on the model string | Use the exact model name `openai/gpt-4.1-mini` |
