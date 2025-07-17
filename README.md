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