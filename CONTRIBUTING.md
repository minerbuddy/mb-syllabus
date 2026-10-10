# Contributing to Mino API

Thank you for your interest in improving Mino! Whether you are fixing a typo, reporting a bug, or building a new feature, your help is welcome.

By participating, you agree to follow our [Code of Conduct](CODE_OF_CONDUCT.md).

## Ways to Contribute

- **Report bugs** you find while using the API
- **Suggest features** or improvements
- **Improve documentation**, including the README and code comments
- **Fix issues** or implement items from the roadmap in the [README](README.md#roadmap)
- **Add tests**, since the project currently has none

## Before You Start

For anything bigger than a small fix, please **open an issue first** to discuss your idea. This avoids duplicate work and makes sure the change fits the direction of the project.

## Reporting Bugs

Please open an issue and include:

1. What you did (the request body, with any secrets removed)
2. What you expected to happen
3. What actually happened, including error messages or stream `error` events
4. Your environment: OS, Python version, and the `GEMINI_MODEL` you used

> **Never post your `GOOGLE_API_KEY`** or any other secret in an issue, log, or screenshot. If you accidentally expose a key, revoke it immediately and create a new one.

## Suggesting Features

Open an issue describing the problem you want to solve, your proposed solution, and any alternatives you considered.

## Development Setup

1. **Fork** the repository and clone your fork:

   ```bash
   git clone <your-fork-url>
   cd rag_app-main
   ```

2. **Create a virtual environment** and install dependencies:

   ```bash
   python -m venv .venv
   source .venv/bin/activate        # Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. **Create a `.env` file** in the project root:

   ```env
   GOOGLE_API_KEY=your_gemini_api_key
   GEMINI_MODEL=gemini-3.5-flash
   ALLOWED_ORIGINS=http://localhost:3000,http://127.0.0.1:3000
   ```

   Do not commit this file. Make sure `.env` is listed in your `.gitignore`.

4. **Run the server:**

   ```bash
   uvicorn app.main:app --reload --port 8000
   ```

5. **Try it out** at `http://localhost:8000/docs`, or with curl:

   ```bash
   curl -N -X POST http://localhost:8000/query/ \
     -H "Content-Type: application/json" \
     -d '{"query": "What is the powder factor?"}'
   ```

## Making Changes

1. Create a branch from `main` with a descriptive name:

   ```bash
   git checkout -b fix/empty-query-validation
   ```

   Suggested prefixes: `feat/`, `fix/`, `docs/`, `refactor/`, `test/`.

2. Make your changes, keeping each pull request **focused on one thing**.
3. Test your change manually (and with automated tests if you added them).
4. Commit with clear messages, for example:

   ```
   fix: return 413 when image payload is too large
   docs: explain SSE event types in README
   ```

5. Push your branch and open a pull request against `main`.

## Code Style

The project does not yet enforce a linter or formatter, so please:

- Follow [PEP 8](https://peps.python.org/pep-0008/) and match the style of the surrounding code
- Use type hints where practical
- Keep functions small and well named
- Add short comments that explain *why*, not *what*, for anything non-obvious
- Do not add new dependencies without a good reason, and mention it in your pull request

## Changing the System Prompt

The Mino persona and answering rules live in `MINO_SYSTEM_PROMPT` in `app/core/prompts.py`. Because users may rely on Mino for safety-related information, changes here need extra care:

- Do not weaken the instructions about admitting uncertainty or reminding users to follow site protocols and the latest DGMS circulars
- Do not add regulation numbers, limits, or figures unless you can verify them against an official source, and mention that source in your pull request
- Describe in your pull request how you tested the new behavior (sample questions and answers are helpful)

## API Compatibility

The `POST /query/` stream format (`token`, `done`, `error` events) is used by frontends. If you need to change request fields or event shapes, call it out clearly in your pull request and prefer backward-compatible changes (for example, adding optional fields).

## Pull Request Checklist

Before requesting a review, please check that:

- [ ] The change is focused and described clearly in the pull request
- [ ] The app starts and `GET /health` works
- [ ] `POST /query/` still streams correctly, including the error case
- [ ] No secrets, API keys, or `.env` files are included
- [ ] Documentation (README or comments) is updated if behavior changed
- [ ] New dependencies, if any, are added to `requirements.txt` and justified
- [ ] Tests are added or updated where it makes sense

## Review Process

A maintainer will review your pull request as soon as they can. You may be asked to make changes. This is a normal part of the process, so please don't take it personally. Once approved, a maintainer will merge it.

## Security Issues

If you find a security vulnerability (for example, a way to bypass CORS restrictions or expose credentials), please **do not open a public issue**. Contact the maintainers privately at **`[ADD CONTACT EMAIL HERE]`** instead.

## Questions

If something here is unclear, open an issue and ask. Improving this guide is a contribution too.
