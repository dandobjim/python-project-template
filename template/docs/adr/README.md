# Architecture Decision Records

An Architecture Decision Record (ADR) captures one significant design decision: the context, the options considered and why one was chosen. ADRs are immutable once accepted; to change a decision, write a new ADR that supersedes the old one.

This project uses the [MADR](https://adr.github.io/madr/) format.

## Creating a new ADR

1. Copy the template with the next free number and a kebab-case title:

   ```bash
   cp docs/adr/0000-template.md docs/adr/0001-use-postgresql-for-persistence.md
   ```

2. Fill in every section and set `Status: proposed`.
3. Open a pull request; set `Status: accepted` (or `rejected`) when it is merged.
4. Add it to the index below.

## Index

| ADR | Title | Status |
| --- | ----- | ------ |
| [0000](0000-template.md) | Template | — |
