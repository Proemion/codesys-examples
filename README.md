# CODESYS Examples

A collection of **CODESYS project examples** and utilities to help you quickly prototype and learn. Each example is packaged as a `.projectarchive` and includes a README Markdown file explaining its purpose and usage.

[What is a projectarchive file?](https://forge.codesys.com/forge/talk/Engineering/thread/f283955619/)

## Repository Structure

Each example has its own folder containing:

```text
example-folder/
├── example.projectarchive   # The actual CODESYS project
└── README.md                # Description and instructions
```

- **`.projectarchive`** – The CODESYS project file, ready to open.
- **`README.md`** – Explains the example, its purpose, and any setup instructions.


## How to Use

1. Clone this repository
2. Open the `.projectarchive` file in **CODESYS**.
3. Read the `README.md` inside the example folder to understand the example and any setup steps.
4. Modify or combine examples for your own projects.


## Contribution Guidelines

- The main branch is protected to ensure stability. Direct pushes are not allowed.
- Contributions must be made via Pull Requests (PRs).
- Add new examples in separate folders.
- Each folder **must** contain:
  - A `.projectarchive` file
  - A `README.md` file describing the example
- Use **clear and descriptive folder names** to make examples easy to find.
- PRs will be reviewed before merging to ensure quality and consistency.


## Notes

- Only `.projectarchive` files and `.md` documentation are included.
- This repository is intended to share ready-to-use CODESYS examples with all dependencies included.