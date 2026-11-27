# Code Walkthrough: Career Profile Q&A Assistant

This guide explains the project step by step. The user asks questions about a r�sum�. The model receives the extracted text as context and answers in the terminal.

## 1. What does the program do?

```text
PDF or TXT  extract text  OpenRouter API  answer in terminal
```

This is a basic LLM application. It is **not an autonomous agent** and does **not use RAG**: it does not search selected text chunks; it sends the profile text directly as context.

## 2. Packages and settings

- `pypdf` extracts text from PDF files.
- `openai` provides the Chat Completions client.
- `python-dotenv` loads the local `.env` file into environment variables.
- `argparse`, `os`, and `pathlib` are part of Python and handle arguments, settings, and paths.

```python
from dotenv import load_dotenv
from openai import APIError, OpenAI
from pypdf import PdfReader
```

`OPENROUTER_API_KEY` and `OPENROUTER_MODEL` live in `.env`. `.gitignore` excludes `.env` from Git. `.env.example` contains placeholders only; never put a real key there.

## 3. Read profile text from a file

```python
def load_profile(path: Path) -> str:
```

The function checks the file extension. For a PDF, `PdfReader` reads each page and joins the extracted text. For `.txt`, Python reads the file as UTF-8. Empty or unsupported input produces a clear error.

The 30,000-character limit is a simple guard against very large input. It does not guarantee that every model will accept the request; models have different context limits.

A scanned PDF is often just an image. `pypdf` does not perform OCR, so scanned files would need a separate OCR step.

## 4. Build the system message

```python
def make_system_message(profile_text: str) -> str:
```

The `system` message contains two things:

1. Instructions: use supported facts and clearly say when information is missing.
2. The extracted profile text as context.

The instruction to treat document instructions as document text is a useful prompt-injection precaution. It cannot guarantee that the model will resist every attack.

## 5. Load the API key and model

```python
load_dotenv()
api_key = os.getenv("OPENROUTER_API_KEY", "").strip()
model = os.getenv("OPENROUTER_MODEL", "").strip()
```

`load_dotenv()` loads local configuration. `os.getenv()` reads environment values. The API key is not written in the program and is never printed.

If a placeholder remains or a value is missing, `argparse` stops with a helpful message.

## 6. Use the OpenAI SDK with OpenRouter

```python
client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=api_key,
)
```

The project uses the OpenAI Python SDK but points its `base_url` to OpenRouter. The model name comes from `.env`, so it is not hard-coded in the source.

## 7. Conversation history and roles

```python
messages = [{"role": "system", "content": system_message}]
```

`messages` is the list sent to the model:

- `system`: instructions and profile context.
- `user`: the user's question.
- `assistant`: the model's answer.

After each turn, recent messages stay in memory, so the user can ask a follow-up. The history is lost when the program exits.

## 8. Main loop and answer

The loop reads questions, recognizes `exit` or `quit`, and then calls:

```python
response = client.chat.completions.create(
    model=model,
    messages=messages,
)
```

The API provider runs the model. `response.choices[0].message.content` contains the answer, which the program prints in the terminal.

`APIError` handles API failures so that an invalid model name or connection problem does not produce a long Python traceback. If available, an HTTP status is displayed; the API key is not.

## 9. Problems in the original notebook

- `google.colab.userdata` works only in Google Colab. This project uses `.env` instead.
- The final cell was marked as Markdown, so Python inside it would not run.
- The loop appeared twice. One loop stored the answer but did not display it.
- `Markdown` needed to be imported before `display(Markdown(...))` could work.
- Printing the first characters of an API key is unnecessary. The key should not be printed at all.
- The model was hard-coded in the notebook. Here it is set in local configuration.

## 10. Privacy and limitations

The profile text and questions are sent to OpenRouter and the selected model provider. Start with `sample_resume.txt`. Before sending real personal data, review the providers' privacy terms and make that choice deliberately.

This simple application puts the whole profile in the prompt. RAG would split documents into chunks, retrieve relevant chunks, and send only those as context. This project does not do that yet.

## Check your understanding

Before moving on, answer this yourself: Which three message roles can appear in `messages`, and what is each role for?

