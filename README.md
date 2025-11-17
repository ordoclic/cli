# CLI

A command-line interface (CLI) for managing professionals, entities, and drugbase licenses via the Ordoclic Partner API.

---

## 📦 Overview

The CLI provides convenient access to the Ordoclic Partner API for technical and administrative operations such as:

- Creating and retrieving professionals.
- Managing entities and linking professionals.
- Setting drugbase licenses (e.g., **VIDAL**).
- Handling authentication through local configuration and signing keys.

---

## ⚙️ Configuration

The CLI uses a configuration file located at:

- **Linux:** `~/.config/ordoclic/config.json`
- **macOS:** `~/Library/Application Support/ordoclic/config.json`
- **Windows:** `%AppData%\ordoclic\config.json`

### Example configuration file

```json
{
  "partner_id": "YOUR_PARTNER_ID",
  "rpps": "99988877766",
  "private_key_path": "/absolute/path/to/private_key.pem",
  "lang": "en"
}
```

### Configuration fields

> **Note (development):** You can override the API base using the `API_BASE_URL` environment variable.


| Field | Description |
|--------|-------------|
| `partner_id` | Your Ordoclic partner identifier (required). |
| `rpps` | Default RPPS identifier for your administrator professional (optional but required for entity creation). |
| `private_key_path` | Absolute path to your RSA private key file (required). |
| `lang` | Default CLI language (`en` or `fr`). Can be overridden with `--lang`. |

---

## 🌐 Language

The CLI supports two languages: **English (`en`)** and **French (`fr`)**.

- The language can be set in your configuration file (`lang` field).
- It can also be overridden per command using the `--lang` flag.

Example:

```bash
ordoclic --lang=fr professional create --file pro.json
```

---

## 🚀 Commands

### ⚠️ Version check & interactive blocking

Before any command is executed (except `version`), the CLI checks
whether a more recent version of Ordoclic is available.

-   If the local version is **ahead** of the remote one → the CLI
    **blocks** execution.
-   If the local version is **behind**, the CLI asks for confirmation:

```{=html}
<!-- -->
```
    A newer version is available.
    Continue? [y/N]

To skip this confirmation automatically (e.g. for CI pipelines):

```bash
--skip-version-check
```

Example:

```bash
ordoclic --skip-version-check professional get 123
```

### Version

Display the current CLI version and compare it with the latest available
release.

```bash
ordoclic version
```

### 🔐 Authentication

#### Get an authentication token

Retrieve an authentication token for a professional using their RPPS
number.

```bash
ordoclic auth get --rpps 12345678901
```

You may also define the RPPS value in the configuration file and omit
the flag:

```bash
ordoclic auth get
```

### 🧑‍⚕️ Professional Management

#### Create a professional

Create a professional using JSON input (from a file or stdin).

```bash
# From stdin
ordoclic professional create < pro.json

# From file
ordoclic professional create --file pro.json

# With notification
ordoclic professional create --file pro.json --should-notify
```

#### Get a professional by ID

```bash
ordoclic professional get <professional_id>
```

#### Set a drugbase license (e.g., VIDAL)

```bash
ordoclic professional set drugbase <professional_id> vidal --config='{"app_id":"...","app_key":"..."}'
```

Expected API call:
```
PUT /drugbase/v1/professionals/{professionalId}/drugbase-setting
{
  "drugbase": "VIDAL",
  "filename": "license.json",
  "config": { "app_id": "...", "app_key": "..." }
}
```

---

### 🏢 Entity Management

#### Create an entity

Creates a new entity and automatically links the administrator using the RPPS defined in your config.

```bash
ordoclic entity create --name="My Entity"
```

This command resolves the administrator’s Ordoclic ID from their RPPS via:
```
POST /v1/professionals/validate
{
  "rpps": "<your_rpps>"
}
```

Then creates the entity:
```
POST /v1/entities
{
  "name": "My Entity",
  "admins": ["<resolved_id>"]
}
```

#### Add one or more professionals to an entity

```bash
# Multiple IDs as arguments
ordoclic entity add professional --entity-id=aaaa-bbbb-cccc dddd-eeee ffff-gggg

# From a JSON file
ordoclic entity add professional --entity-id=aaaa-bbbb --file pros.json
```

Example `pros.json`:

```json
{
  "professionalIDs": [
    "bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
    "11111111-2222-3333-4444-555555555555"
  ]
}
```

---

## 🧩 Input from stdin or files

For commands supporting JSON input, you can provide data either via:

- **File**: `--file <path>`
- **Standard input**: `< data.json`

The CLI automatically detects which method you’re using.

---

## 🌍 Error messages and localization

All user-facing messages are translated based on the active language (`term` package).
Each package defines its own message catalog for maintainability.

Example localized error:
```
❌ Failed to load configuration: unable to locate config directory
```

---

## ⚠️ Exit codes

| Code | Meaning |
|------|----------|
| `0` | Success |
| `1` | Generic error (invalid input, API failure, etc.) |
