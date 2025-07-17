# Game Server Eggs Repository

This repository contains **Git-driven egg definitions** for deploying popular server software in a standardized, maintainable way.

## Repository Layout

- **nests.yaml**  
  Top-level file listing major software categories as a plain YAML array  
  (not wrapped in a key). Each nest has a unique `id`, `name`, and concise `description`.

- **eggs/**  
  Directory containing all egg definitions. Each egg is a YAML file describing a deployable server (e.g. Minecraft, databases), organized in folders by software type or family.

  ```
  eggs/
    minecraft/
      java.yaml
      ...
    ...
  ```

### Example: Adding a New Egg

1. **Add or update a nest** in `nests.yaml` if creating a new category.
2. **Create a YAML file** under `eggs/<category>/<egg>.yaml` following the schema used in existing eggs.
3. **Define environment variables** and metadata as shown in the egg examples.

See existing files for structure and best practices.

---

## Egg file schema

Each egg YAML must follow this schema:

```yaml
# Example egg.yaml schema (with comments)
id: "minecraft-java"              # Unique, lowercase, URL-safe ID
nest: "minecraft"                 # Must match an ID in nests.yaml
name: "Minecraft Java Edition"    # Human-friendly name
description: "Vanilla Minecraft server for Java"  # Short description
image: "itzg/minecraft-server:latest"   # Docker image reference
startup: "java -Xmx{{SERVER_MEMORY}} -jar server.jar nogui"   # Startup command template
envSchema:                        # Describes required ENV variables
  SERVER_MEMORY:
    type: string
    description: "Amount of RAM (e.g. 2G, 4096M)"
    default: "2G"
  EULA:
    type: boolean
    description: "Accept Mojang EULA"
    default: true
  # ... more variables as needed
test:                             # [optional] List of simple test commands/checks
  - "test -f server.jar"
  - "grep 'Done' logs/latest.log"
```

## Contributing: Adding new eggs

- **id:** Must be unique, lowercase, URL-safe (no spaces or special chars).
- **nest:** Must reference an existing nest ID in `nests.yaml`.
- **Directory naming:** Place your egg at `eggs/<category>/<variant>.yaml` (e.g. `eggs/minecraft/java.yaml`). Use the nest/category name consistently.
- **envSchema:**  
  - Every variable should have a type (`string`, `boolean`, `integer`), description, and a safe default if possible.
  - Keep variable names UPPERCASE and concise.
  - Only expose variables that must be set by users.
- **Testing tips:**  
  - Use the `test:` array for simple validation commands (e.g. checking files, logs).
  - Ensure the egg can be parsed/used without errors before submitting PRs.
  - Validate that your `startup` command works with the provided envSchema defaults.
- **ID/nest rules:**  
  - The `id` must not collide with any other egg.
  - The `nest` must exist in `nests.yaml` before your egg can be merged.

---

## Backend integration / Syncing

Integrators can programmatically fetch and sync egg definitions using the GitHub API or any HTTP client.

### Required .env variables (example)

```
EGG_REPO_URL=https://github.com/yourorg/game-eggs
EGG_REPO_BRANCH=main
EGG_REPO_TOKEN=ghp_xxx   # (for private repos or higher rate limits)
EGG_SYNC_INTERVAL=600    # (in seconds, how often to pull updates)
```

### TypeScript example: EggSyncService (simplified)

```ts
import axios from "axios";
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

export class EggSyncService {
  async syncEggs() {
    const url = process.env.EGG_REPO_URL + "/raw/" +
      (process.env.EGG_REPO_BRANCH || "main") +
      "/eggs/minecraft/java.yaml";
    const { data } = await axios.get(url, {
      headers: process.env.EGG_REPO_TOKEN
        ? { Authorization: `token ${process.env.EGG_REPO_TOKEN}` }
        : undefined,
    });

    // Parse YAML, validate, and upsert (pseudo-code)
    const egg = parseEggYaml(data); // implement YAML parsing
    await prisma.egg.upsert({
      where: { id: egg.id },
      update: egg,
      create: egg,
    });
  }
}
```
- See the blueprint for a full-featured EggSyncService and error handling.
- Use `js-yaml` or similar to parse YAML.

### Fetching eggs via GitHub API

- To fetch a raw egg YAML file, use the `raw.githubusercontent.com` endpoint, or GitHub API with a personal access token if needed.
- For private repos, add an `Authorization: token ...` header.

#### Minimal cURL example

```sh
curl -s \
  https://raw.githubusercontent.com/yourorg/game-eggs/main/eggs/minecraft/java.yaml
```

For private repos:

```sh
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/yourorg/game-eggs/contents/eggs/minecraft/java.yaml?ref=main" \
  | jq -r .content | base64 -d
```

---