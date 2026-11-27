# Career Profile Q&A Assistant

A small command-line application that extracts text from a PDF or text profile and answers career questions through OpenRouter. The app keeps conversation history in memory for follow-up questions.

This is a basic LLM application. It puts the extracted profile in the system message. It does **not** use embeddings, a vector database, retrieval-augmented generation (RAG), or autonomous tools.

## Features

- Read a text-based PDF or UTF-8 `.txt` file.
- Ask repeated questions while retaining chat context.
- Configure the OpenRouter API key and model outside the source code.
- Use a fictional sample profile for a safe first run.
- Limit profile text to 30,000 characters.

## Requirements

- Python 3.10 or newer
- An OpenRouter API key
- A model identifier enabled for your OpenRouter account

## Setup

Create and activate a virtual environment from this directory.

Windows PowerShell:

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
```

Linux or macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install the dependencies:

```bash
python -m pip install -r requirements.txt
```

Create your private `.env` file from the template.

Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

Linux or macOS:

```bash
cp .env.example .env
```

Edit `.env` and replace both placeholders:

```dotenv
OPENROUTER_API_KEY=YOUR_REAL_API_KEY
OPENROUTER_MODEL=YOUR_ENABLED_MODEL_ID
```

Never commit `.env`. It is excluded by `.gitignore`; `.env.example` contains placeholders only.

## Run

Try the fictional sample profile:

```bash
python main.py sample_resume.txt
```

Or provide your own text-based PDF:

```bash
python main.py /path/to/profile.pdf
```

Type `exit` or `quit` to end the chat. Scanned PDFs without an OCR text layer may not contain extractable text.

## How it works

1. `python-dotenv` reads `.env` into the process environment.
2. `pypdf` extracts text from a PDF; normal file reading handles `.txt` profiles.
3. The program creates an OpenAI-compatible client pointed at OpenRouter.
4. A `system` message carries instructions and the profile; each question and answer is appended to `messages`.
5. The chat completion response is printed in the terminal.

See [CODE_WALKTHROUGH_EN.md](CODE_WALKTHROUGH_EN.md) for a detailed English explanation and [CODE_WALKTHROUGH_DE.md](CODE_WALKTHROUGH_DE.md) for the German version.

## Privacy and safety

- The extracted profile and questions are sent to OpenRouter and the selected model provider for inference. Use the fictional sample or a properly redacted file unless you have reviewed the providers' privacy terms and chosen to send real personal data.
- The API key stays in a local `.env` file and is never printed by the app. Do not upload `.env` to GitHub.
- The prompt asks the model to use only facts supported by the profile, but prompt instructions cannot guarantee correct behavior. Check important claims against the source.
- Do not use this demo to make hiring decisions or send generated claims to employers without verifying them.

## Limitations

- The full profile is included in each API request; this is not RAG and is not a scalable document-search design.
- No OCR, chunking, vector database, persistent storage, graphical interface, or automated evaluation is included.
- Conversation history is stored in memory only until the program exits.

## References

- [OpenRouter developer documentation](https://openrouter.ai/developers)
- [python-dotenv documentation](https://github.com/theskumar/python-dotenv)

---
Educational project; not an employment decision system.

