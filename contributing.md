# Contribution Guidelines

Please note that this project is released with a [Contributor Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/). By participating in this project you agree to abide by its terms.

## What belongs here

The list is for people who build agents. A link belongs here if it helps a developer decide what an agent may reach and what stops it outside the model.

- An incident needs a primary source: the researchers' write-up, the vendor's advisory or post-mortem, a CVE record, or a court or regulator document. A news article is fine when it quotes the primary source and adds something to it.
- Say in the description whether the damage was confirmed. If researchers showed an attack on their own test setup, call it a demo.
- A control has to hold outside the model: permissions, isolation, data flow rules, output handling, backups and rollback. Classifiers and guardrails are welcome too, as long as the description says they catch only part of the cases.
- Please leave out product landing pages, vendor marketing and paywalled content.

## Format

- One link per line: `- [Title](https://example.com) - Description.`
- Use the resource's own title.
- Write the description yourself, in one sentence that starts with a capital letter and ends with a period. Don't repeat the title in it.
- Put the link in the section for the capability it is about. If it fits several, pick the one where a developer would look first. Each link appears once.
- Add new items at the end of their section.

## Pull requests

- One item per pull request, unless the items tell one story, such as a write-up and its advisory.
- Check your spelling and that the link opens.
- The CI runs [awesome-lint](https://github.com/sindresorhus/awesome-lint) and a link checker. Please fix what they report.

Thank you for your suggestions.
