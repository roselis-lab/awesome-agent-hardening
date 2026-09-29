# Contribution Guidelines

Please note that this project is released with a [Contributor Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/). By participating in this project you agree to abide by its terms.

## What belongs here

This list is for people who build agents. A link belongs here if it helps a developer decide what an agent may reach and what stops it outside the model.

- **Incidents** need a primary source: the researchers' write-up, the vendor's advisory or post-mortem, a CVE record, or a court or regulator document. News coverage is fine only when it quotes the primary source and adds something to it.
- Say plainly in the description whether damage was confirmed. A researcher's demo is a demo, not a breach.
- **Controls** must hold outside the model: permissions, isolation, data flow rules, output handling, backups and rollback. A classifier or a guardrail is welcome, but describe it as a signal, not a boundary.
- No product landing pages, no vendor marketing, no paywalled content.

## Format

- One link per line: `- [Title](https://example.com) - Description.`
- Use the resource's own title.
- The description is one sentence in your own words, starts with a capital letter and ends with a period. Do not repeat the title in it.
- For controls, start the description with the principle it applies: `Least privilege:`, `Isolation:`, `Data flow control:`, `Output handling:` or `Recoverability:`.
- Put the link in the section for the capability it is about. If it fits several, pick the one where a developer would look first. Every link appears only once.
- New items go at the end of their section.

## Pull requests

- One item per pull request, unless the items are one story, such as a write-up and its advisory.
- Check your spelling and that the link opens.
- The CI runs [awesome-lint](https://github.com/sindresorhus/awesome-lint) and a link checker; fix what they report.

Thank you for your suggestions.
