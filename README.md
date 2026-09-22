# Identity-Mapping-Tool

[Open the hosted Identity Mapping Builder](https://manuariass.github.io/Identity-Mapping-Tool/)

# Identity Mapping Builder for Microsoft 365 Copilot Connectors

> Generate correct identity-mapping expressions for a Copilot connector — before a misconfiguration reaches production.

<!-- TODO: add badges once the repo is public — build status, license, release version -->

---

## Why this exists

When a Microsoft 365 Copilot connector indexes content from a third-party system, it does not only bring in documents — **it brings in permissions**. Every indexed item carries an access control list (ACL) from the source system, and that ACL is expressed in the source system's own language of identity: a Salesforce `FederationIdentifier`, a ServiceNow `sys_id`, a Jira `AccountId`, an internal user record, an email alias.

Microsoft 365 security trimming understands exactly one thing: **Microsoft Entra identity**.

Identity mapping is the translation layer between those two worlds. It decides, for every indexed item, who is allowed to see it. Today that translation is authored **by a person, by hand, during connector setup** — expressions, formulas, regular expressions, property selection — and the administrator has to know how their source system stores identities and get the formula right on the first attempt.

There is no feedback loop. The expression looks valid. The connection publishes. The crawl completes successfully. **Nothing tells you the mapping is wrong** until users report that Copilot cannot find anything.

Incorrect identity mapping is the **number one cause of access problems with data indexed by Copilot connectors**, and in most cases it is not a product defect — it is human error at configuration time.

### What makes it hard

Identity data is rarely clean or consistent:

- The same person can be an email address in one system, a username in another, and an opaque internal identifier in a third.
- Some identities already exist in Entra (ME-ID). Others never will.
- Contractors, service accounts, shared mailboxes, and people who have left the company complicate almost every rule you write.
- A source field that *looks* like a UPN often is not — an email stored in `FederationIdentifier` that does not match the Entra UPN will silently break retrieval for every affected user.

### The cost of getting it wrong

When the mapping is wrong, users lose access to content they are entitled to. Copilot returns nothing. The data is indexed, but unreachable. The customer concludes that Copilot does not work, and adoption stalls — while teams spend weeks troubleshooting search quality and content coverage, when the real cause was a single expression written during setup.

**This tool moves that mistake from discovery-after-rollout to prevention-at-setup.**

---

## What it does

The Identity Mapping Builder walks you through how your data source actually stores its identities, then generates the expressions and formulas you paste into the connector configuration.

1. **Classify the source identity.** Determine whether the data source stores identifiers that require translation (non-ME-ID) or identities already present in Entra (ME-ID).
2. **Select the source property.** Choose the field that carries the identity on indexed items.
3. **Define the desired output.** Specify the Entra property to resolve against (UPN, mail, object ID).
4. **Generate the configuration.** The tool produces the correct mapping expression — using the same parameters as the connector setup itself, so the result is directly usable.
5. **Review before you publish.** Inspect the generated expression against sample values before committing it to the connection.

Because the output uses the connector's own parameter model, there is no translation step between what the tool produces and what you paste into the admin experience.

---

## Who it is for

| Audience | Use case |
|---|---|
| **Customer / tenant administrators** | Configuring a Copilot connector for the first time |
| **Support engineers** | Diagnosing why indexed content is not reachable for some users |

---

## Getting started

Open the [hosted Identity Mapping Builder](https://manuariass.github.io/Identity-Mapping-Tool/)
in a modern browser. No installation is required, and all processing stays in the browser.

You need access to the data source's identity schema and permission to configure the
connector in the Microsoft 365 admin center.

---

## Usage

<!-- TODO: add a screenshot or GIF of the tool here — this is the single highest-value addition to this README -->

### Example: Salesforce

A Salesforce org stores the user identity in `FederationIdentifier`, and those values are email addresses that match the Entra UPN.

1. Select the data source type.
2. Indicate that the identity is **not** already an Entra ID (non-ME-ID).
3. Choose `FederationIdentifier` as the source property.
4. Choose `userPrincipalName` as the Entra target property.
5. Copy the generated expression into the connector's identity-mapping step.

> ⚠️ If `FederationIdentifier` holds an email that does **not** match the Entra UPN, mapping to `userPrincipalName` will resolve to nobody and users will retrieve nothing. Verify a sample of real values before publishing — this is the single most common failure in production.

<!-- TODO: add one worked example per supported connector (ServiceNow, Jira, Zendesk, Confluence) -->

---

## Supported data sources

- Supported Copilot Connectors Data Sources

---

## Limitations

- The tool **generates configuration; it does not validate against a live tenant.** It cannot confirm that the identities in your source system actually resolve in Entra — verify against real sample values before publishing.
- It does not detect identities that have been deprovisioned in Entra (for example, users who have left the company). Those will fail at query time regardless of how the expression is written.
- It does not address ACL problems that originate in the source system or in the connector itself — only the mapping expression you author during setup.
- <!-- TODO: add any known constraints specific to your implementation -->

---

## Contributing

<!-- TODO: confirm your contribution model before publishing -->

Issues and pull requests are welcome. If you hit an identity-mapping scenario the tool does not handle, please open an issue and include:

- The data source and the identity field involved
- What the source values look like (**redact real identities** — use `user@example.com` style placeholders)
- The Entra property you expected to resolve against

**Never include real customer identities, tenant IDs, incident numbers, or connector credentials in an issue or pull request.**

---

## Support

This is a community tool, not a supported Microsoft product. It generates configuration that you remain responsible for reviewing before applying to a production connector.

<!-- TODO: add the contact/Teams channel for questions -->

---

## Further reading

- [Microsoft 365 Copilot connectors overview](https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/overview-copilot-connector)
- <!-- TODO: link to the official identity-mapping documentation -->
