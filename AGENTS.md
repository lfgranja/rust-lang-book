# AGENTS.md - Context and Rules for AI Agents

This file provides context, workflows, and strict rules for AI agents working on "The Rust Programming Language" book repository.

## 1. Project Architecture

This repository contains the source code for the book and its supporting tools.

-   **`src/`**: The book content (Markdown). This is the primary workspace. **Code blocks here often include files from `listings/`.**
-   **`listings/`**: Source code for the book's examples. **Edit code here, not in the markdown files**, unless the markdown contains inline code blocks.
-   **`packages/trpl/`**: The `trpl` crate, used in book examples. It provides a stable API for readers. **Note:** This is a standalone crate (edition 2024), NOT part of the root workspace.
-   **`packages/tools/`**: Internal maintenance scripts (Rust binaries). Part of the root workspace.
-   **`nostarch/`**: **READ-ONLY**. Snapshots for the print publisher. **NEVER EDIT.**
-   **`ci/`**: Validation scripts (`validate.sh`, `spellcheck.sh`).

## 2. Environment Setup

Agents must ensure the following tools are available. If missing, install them:

```bash
# Core build tool
cargo install mdbook --version 0.4.40  # Check for latest version if unsure

# Formatting tools
cargo install dprint
rustup component add rustfmt
```

## 3. Build & Test Workflows

### Build the Book
Generates HTML output in `book/`.
```bash
mdbook build
```

### Run Tests (Critical)

**A. Test the Book (Doctests)**
This extracts code blocks from Markdown (and included listings) and runs them.
*Prerequisite*: You must build the `trpl` crate first or point to its deps.

```bash
# 1. Build support crate (Required for many examples)
cargo build --manifest-path packages/trpl/Cargo.toml

# 2. Run ALL book tests (from root)
mdbook test --library-path packages/trpl/target/debug/deps

# 3. Run SINGLE chapter tests (faster/targeted) - RECOMMENDED
mdbook test --library-path packages/trpl/target/debug/deps src/ch04-01-what-is-ownership.md
```

**B. Test Code Listings Directly**
When modifying a code example in `listings/`, test it locally first:
```bash
cd listings/ch04-understanding-ownership/listing-04-01
cargo run
# OR
cargo test
```

**C. Test the Support Crate**
For changes to `packages/trpl` itself:
```bash
cd packages/trpl
cargo test
```

### Validation
Run these before submitting ANY changes:
```bash
./ci/validate.sh      # Checks internal links and references
./ci/spellcheck.sh    # Checks spelling (add exceptions to ci/dictionary.txt)
```

## 4. Code Style & Formatting (Strict)

**Violations of these rules will cause CI failure.**

### General
-   **Line Length**: **HARD LIMIT 80 characters**. This applies to Markdown prose AND code blocks.
-   **Indentation**: 4 spaces.

### Prose (Markdown)
-   **Formatting**: Run `dprint fmt` on changed files.
-   **Headings**: Title Case (e.g., `## Handling Potential Failure`).
-   **Terminology**: Use *italics* for first definitions of terms. NEVER use single quotes.
-   **Methods**: Refer to methods without parentheses (e.g., "call `read_line`", NOT "call `read_line()`").
-   **Mixing**: Do not mix code and prose in one word.
    -   *Bad*: "We `use`d the library."
    -   *Good*: "We used the `use` keyword."

### Code Listings (Rust)
-   **Source**: Most code blocks are included from `listings/`. **Edit the source file in `listings/`, NOT the markdown.**
-   **Formatting**: Run `rustfmt` on Rust code.
-   **File Names**: If a listing corresponds to a file, put the filename in a seemingly-commented path above the block (check existing files for syntax).
-   **Output**: Use ````text` or ````console` for output, not `rust`.
-   **Hiding**: Use `#` to hide lines in `mdbook`, but keep code compilable.
    ```rust
    fn main() {
        // # loop {
        // #    // hidden logic
        // # }
    }
    ```

## 5. Agent Behavior Protocols

1.  **Pedagogical Value**: You are writing a book, not just code. Explanations must be clear, accurate, and beginner-friendly.
2.  **Accuracy**: All Rust code must compile (unless demonstrating a compiler error).
3.  **Workflow for Code Changes**:
    -   Identify the source file in `listings/` (look at the `{{#rustdoc_include ...}}` path in the markdown).
    -   Modify the file in `listings/`.
    -   Run `rustfmt` on the modified file.
    -   Test the listing locally (`cargo run` / `cargo test`).
    -   Run `mdbook test` on the relevant chapter to ensure integration.
4.  **No Starch Constraints**: Never touch `nostarch/`. generally rely on `src/` validation.
5.  **Refactoring**: If refactoring code examples, ensure the surrounding prose still describes the code accurately. Code and text are tightly coupled.
