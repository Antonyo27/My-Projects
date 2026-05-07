# Templates

Reusable scaffolds for adding new projects to this showcase.

## Files

| Template | Use For |
|----------|---------|
| [`project-readme-template.md`](./project-readme-template.md) | The top-level `README.md` of a new project folder under `projects/<name>/` |
| [`case-study-template.md`](./case-study-template.md) | The narrative case study under `projects/<name>/docs/case-study.md` |

## How to Add a New Project

1. **Create the folder structure**:
   ```text
   projects/{NewProjectName}/
   ├── README.md            ← copy from project-readme-template.md
   ├── tech-stack.md
   ├── screenshots/
   │   └── .gitkeep
   ├── demo/
   │   └── .gitkeep
   ├── architecture/
   │   └── .gitkeep
   └── docs/
       ├── case-study.md    ← copy from case-study-template.md
       ├── challenges.md
       ├── deployment.md
       └── scalability.md
   ```

2. **Fill in placeholders** — the templates use `{PLACEHOLDER}` syntax for fields you need to replace.

3. **Add the project to the root README** — append a card to the project showcase grid in `/README.md`.

4. **Drop in screenshots** — replace the `.gitkeep` files with real captures. Sanitize anything client-confidential before publishing.

## Conventions

- **Folder names** match the project's display name (preserve spaces and capitalization for readability — GitHub handles URL-encoding automatically).
- **Image placeholders** in READMEs reference paths like `./screenshots/home-page.png` so they render the moment the file is added.
- **Confidentiality** — for client work, always include the confidentiality notice and never reveal client identity, branding, or proprietary business rules.
