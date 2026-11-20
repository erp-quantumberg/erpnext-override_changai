# Override Changai

`override_changai` is a Frappe app that makes a small set of targeted changes to
the ChangAI experience. It does not replace ChangAI or implement its chat,
retrieval, or vector-store functionality.

## Features

- **ChangAI chat rebrand:** Changes the chatbot heading from “ChangAI from
  ERPGulf” to “Q-DynamicsAI” and replaces its default ERP-assistant greeting.
  The script observes the chatbot's Shadow DOM so the changes also apply when
  its content is rendered or updated later.
- **FAISS notice suppression:** Removes the specific “FAISS MASTER store not
  found” notice from the login message log and from JSON responses. Other
  messages are left alone. This hides the notice; it does not create or repair
  the missing vector store, change ChangAI's retrieval behavior, or remove
  ChangAI's error log entries.
- **Asset cache versioning:** Adds a modification-time query string to the
  bundled rebrand script so browsers can fetch an updated asset after it
  changes.

## Requirements

### Required apps and tools

| Requirement                                   | Details                                                                                                             |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Frappe Framework site                         | The app must be installed in the site's bench.                                                                      |
| Frappe Bench CLI                              | Used to fetch, install, build, and manage the app.                                                                  |
| [ERPNext](https://github.com/frappe/erpnext)  | ChangAI is designed for ERPNext; install a version compatible with your Frappe bench.                               |
| [ChangAI](https://github.com/ERPGulf/changai) | Required on the same site. Install and configure it before this app; this app hooks into ChangAI-specific behavior. |
| Python 3.14 or newer                          | Required by this app's `pyproject.toml` and the bench Python environment.                                           |

This app declares no additional Python runtime packages or npm dependencies.
ChangAI may have its own requirements, including other apps or services; follow
[ChangAI's installation and configuration instructions](https://github.com/ERPGulf/changai)
for those. This app's static JavaScript is served as a Frappe asset and does not
have a separate frontend installation.

## Installation

Install ERPNext and ChangAI on the target site first, following
[ChangAI's installation instructions](https://github.com/ERPGulf/changai).
Generate a Gemini API key at
[Google AI Studio](https://aistudio.google.com/app/api-keys), then enter it in
the **Gemini** section of **ChangAI Settings** and save the settings.

From the directory of your Frappe bench, fetch this repository:

```bash
bench get-app https://github.com/erp-quantumberg/erpnext-override_changai.git --branch main
```

Then install the app on the site:

```bash
bench --site <site-name> install-app override_changai
```

Replace `<site-name>` with the site's actual name. If the app is already present
in the bench, skip `bench get-app`.

## Setup

Confirm that ERPNext, ChangAI, and this app are installed on the intended site:

```bash
bench --site <site-name> list-apps
```

Build the app's assets and clear the site's cache:

```bash
bench build --app override_changai
bench --site <site-name> clear-cache
```

For a production bench, restart the bench's services if needed so workers and
web processes pick up the installed app and its hooks:

```bash
bench restart
```

Open the site, sign in, and check the ChangAI chatbot heading and greeting. The
FAISS notice is suppressed only when it matches the ChangAI master-store
message; unrelated warnings and errors remain visible.

## Configuration notes

- The app registers `qdynamics_rebrand.js` through Frappe's `app_include_js`
  hook. The script looks for the ChangAI chatbot host with the ID
  `changai-chatbot-host` and updates matching heading and greeting text inside
  its Shadow DOM. If ChangAI changes that markup, host ID, or source text, the
  rebrand selectors may need to be updated.
- The message hooks are registered in `override_changai/hooks.py`. The
  session-creation hook filters the matching notice before login messages are
  serialized; the after-request hook handles matching notices already present
  in JSON responses. Unrelated messages are not intentionally filtered.
- Install this app **after ChangAI**. Its session hook relies on Frappe loading
  this app's hook after ChangAI's startup hooks.
- Suppressing the missing-store notice does not make the vector store
  available. If ChangAI needs that store for retrieval, configure or restore it
  using ChangAI's instructions.
- No extra site settings are defined by this app. It adds no DocTypes or
  database patches.

## Troubleshooting

### The chatbot still shows the old heading or greeting

1. Confirm `override_changai` appears in
   `bench --site <site-name> list-apps`.
2. Rebuild the app assets with `bench build --app override_changai`.
3. Clear the site cache and hard-refresh the browser to discard cached assets.
4. If ChangAI has changed its chatbot markup, check that its host element still
   uses `changai-chatbot-host` and that the heading and greeting still match
   the strings handled by the rebrand script.

### The FAISS master-store notice still appears

1. Confirm ChangAI is installed before `override_changai` on the same site.
2. Check that the message contains the expected
   `FAISS MASTER store not found at` marker. Other messages are intentionally
   unaffected.
3. If the underlying store is required, fix its setup through ChangAI. This app
   only suppresses the matching notice; it does not repair the store or remove
   its error log entry.

### The app is not recognized or its hooks do not run

Confirm the app is installed on the correct site with
`bench --site <site-name> list-apps`. If installation or code changes were made
after the site was created, run:

```bash
bench --site <site-name> migrate
bench build --app override_changai
bench restart
```

## Contributing

Contributions are welcome. Keep changes focused on this app's ChangAI
integration, and test them in a development bench with ChangAI installed.

Install and enable the repository's pre-commit hooks:

```bash
python -m pip install pre-commit
pre-commit install
```

Run the configured formatters and linters before opening a pull request:

```bash
pre-commit run --all-files
```

To run the app's Frappe tests on a configured bench:

```bash
bench --site <site-name> run-tests --app override_changai
```

Pull requests are also checked by the repository's GitHub Actions workflows,
including pre-commit, Semgrep, and dependency auditing.

## License

This app declares the MIT License. See [license.txt](license.txt) for the
license text. The copyright year and holder in that file are still placeholders
and should be completed by the maintainers before distribution.
