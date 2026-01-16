# OpenAI-translations-pydantic

Python framework for building reliable, typed translation pipelines using OpenAI and Pydantic.

This project provides a small foundation for validating, structuring, and running translations through OpenAI's models while keeping inputs, outputs, and configuration strictly typed using Pydantic models. It is intended as a starting point for building translation services, pipelines, and experiments where input/output validation, clear schemas, and reproducible release practices matter.

## Features

- Pydantic models for typed input and output schemas
- Helpers for calling OpenAI translation/generation endpoints
- Configuration via environment variables (API keys, model, temperature)
- Example pipeline for translating text with structured metadata
- Tests and linting ready (suggested setup)
- Release helper scripts (inspired by Pydantic's release tooling) for version bumping and changelog management

## Requirements

- Python 3.10+
- An OpenAI-compatible API key (set via `OPENAI_API_KEY`)
- Recommended: `poetry` or `uv` (Astral's uv) for dependency management

## Installation

Prefer installing in a virtual environment.

Using pip:
```
pip install -r requirements.txt
```

Using uv (if you include a uv lockfile / configuration):
```
uv sync --python 3.12 --group default
```

(Adjust to your project's chosen dependency manager.)

## Quickstart

1. Set your OpenAI API key:
```
export OPENAI_API_KEY="sk-..."
```

2. Example Pydantic models and translation call:

```python
from pydantic import BaseModel, Field
from typing import Optional
import os
import requests  # or openai if using their official client

OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
OPENAI_API_URL = "https://api.openai.com/v1/chat/completions"  # example

class TranslationRequest(BaseModel):
    source_text: str = Field(..., description="Text to translate")
    source_lang: Optional[str] = Field(None, description="Source language code (e.g. 'en')")
    target_lang: str = Field(..., description="Target language code (e.g. 'de')")

class TranslationResult(BaseModel):
    translated_text: str
    model: str
    source_lang: Optional[str]
    target_lang: str

def translate_with_openai(req: TranslationRequest, model: str = "gpt-4o-mini"):
    prompt = (
        f"Translate the following text from {req.source_lang or 'auto-detect'} "
        f"to {req.target_lang}.\n\nText:\n{req.source_text}\n\n"
        "Return only the translated text in the response."
    )
    headers = {
        "Authorization": f"Bearer {OPENAI_API_KEY}",
        "Content-Type": "application/json",
    }
    data = {
        "model": model,
        "messages": [{"role": "user", "content": prompt}],
    }
    resp = requests.post(OPENAI_API_URL, json=data, headers=headers, timeout=30)
    resp.raise_for_status()
    content = resp.json()
    # NOTE: adapt to the response format of the client you use
    translated_text = content["choices"][0]["message"]["content"].strip()
    return TranslationResult(
        translated_text=translated_text,
        model=model,
        source_lang=req.source_lang,
        target_lang=req.target_lang,
    )
```

3. Use it:

```python
req = TranslationRequest(source_text="Hello, world!", target_lang="de")
res = translate_with_openai(req)
print(res.translated_text)
```

## Configuration

- OPENAI_API_KEY — required. Your OpenAI API key.
- MODEL — optional. Default model used for translations.
- TIMEOUT — optional network timeout for API calls.

Consider using a `.env` file or a configuration system based on Pydantic's BaseSettings for production use:

```python
from pydantic import BaseSettings

class Settings(BaseSettings):
    openai_api_key: str
    model: str = "gpt-4o-mini"
    timeout: int = 30

    class Config:
        env_file = ".env"

settings = Settings()
```

## Testing & Linting

- Use pytest for tests.
- Use ruff for linting and formatting.
- Optionally use pre-commit hooks to enforce checks before commits.

Example test command:
```
pytest
```

Example lint command:
```
ruff check .
```

## Release process (recommended)

This repository includes simple release helper scripts inspired by the Pydantic project's release tooling:

- `release/prepare.py` — update version and generate release notes (dry-run supported).
- `release/push.py` — push the release branch, open a PR and create a draft release.

Typical steps:
1. Run `python release/prepare.py {NEW_VERSION}` (use `--dry-run` to preview).
2. Edit and curate the generated HISTORY/CHANGELOG entry.
3. Run `python release/push.py` to create the release branch, PR and draft.

Adjust these scripts for your repository layout and authentication method (they may call `gh` and expect a token).

## Contributing

Contributions are welcome.

- Open an issue to discuss larger changes.
- Follow the coding style: ruff + Black style guidelines and Pydantic-style typing.
- Add tests for new features and run the test suite before submitting a PR.

If you adopt the Pydantic project's pre-commit setup, include `.pre-commit-config.yaml` and run:
```
pre-commit install
pre-commit run --all-files
```

## Examples

Provide real examples in the `examples/` directory for:

- Batch translation pipeline
- Translating files (CSV/JSON)
- Handling language detection and metadata
- Error handling and retries for rate limits

## Security

Never commit API keys to the repository. Use environment variables or secret stores. Review and sanitize any inputs before posting to third-party APIs.

## License

Add your license here (e.g. MIT, Apache-2.0). Example:

```
MIT License
```

(Replace with your project's chosen license and include a LICENSE file.)

## Contact

Maintainers: add contact info / links to issues/PRs.

````
