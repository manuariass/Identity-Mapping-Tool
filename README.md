# Identity-Mapping-Tool
When a Microsoft 365 Copilot connector indexes content from a third-party system, it does not only bring in documents — it brings in permissions. Every indexed item carries an access control list (ACL) from the source system, and that ACL is expressed in the source system's own language of identity: a Salesforce FederationIdentifier, a ServiceNow sys_id, a Jira AccountId, an internal user record, an email alias.

Microsoft 365 security trimming understands exactly one thing: Microsoft Entra identity.

Identity mapping is the translation layer between those two worlds. It decides, for every indexed item, who is allowed to see it. Today that translation is authored by a person, by hand, during connector setup — expressions, formulas, regular expressions, property selection — and the administrator has to know how their source system stores identities and get the formula right on the first attempt.

There is no feedback loop. The expression looks valid. The connection publishes. The crawl completes successfully. Nothing tells you the mapping is wrong until users report that Copilot cannot find anything.

Incorrect identity mapping is the number one cause of access problems with data indexed by Copilot connectors, and in most cases it is not a product defect — it is human error at configuration time.

What makes it hard
Identity data is rarely clean or consistent:

The same person can be an email address in one system, a username in another, and an opaque internal identifier in a third.
Some identities already exist in Entra (ME-ID). Others never will.
Contractors, service accounts, shared mailboxes, and people who have left the company complicate almost every rule you write.
A source field that looks like a UPN often is not — an email stored in FederationIdentifier that does not match the Entra UPN will silently break retrieval for every affected user.
The cost of getting it wrong
When the mapping is wrong, users lose access to content they are entitled to. Copilot returns nothing. The data is indexed, but unreachable. The customer concludes that Copilot does not work, and adoption stalls — while teams spend weeks troubleshooting search quality and content coverage, when the real cause was a single expression written during setup.

This tool moves that mistake from discovery-after-rollout to prevention-at-setup.

What it does
The Identity Mapping Builder walks you through how your data source actually stores its identities, then generates the expressions and formulas you paste into the connector configuration.

Classify the source identity. Determine whether the data source stores identifiers that require translation (non-ME-ID) or identities already present in Entra (ME-ID).
Select the source property. Choose the field that carries the identity on indexed items.
Define the desired output. Specify the Entra property to resolve against (UPN, mail, object ID).
Generate the configuration. The tool produces the correct mapping expression — using the same parameters as the connector setup itself, so the result is directly usable.
Review before you publish. Inspect the generated expression against sample values before committing it to the connection.
Because the output uses the connector's own parameter model, there is no translation step between what the tool produces and what you paste into the admin experience.

Who it is for
Audience	Use case
Customer / tenant administrators	Configuring a Copilot connector for the first time
Technical Program Managers	Supporting a customer through connector deployment
Field and pre-sales	Answering "how will permissions work?" during scoping
Support engineers	Diagnosing why indexed content is not reachable for some users
Getting started
Prerequisites
Access to the data source's identity schema (you need to know which field carries the identity)
Permissions to configure the connector in the Microsoft 365 admin center
Install
git clone <!-- TODO: repo URL -->
cd identity-mapping-builder
<!-- TODO: install command, e.g. npm install -->
Run
<!-- TODO: run command, e.g. npm start -->
Usage
Example: Salesforce
A Salesforce org stores the user identity in FederationIdentifier, and those values are email addresses that match the Entra UPN.

Select the data source type.
Indicate that the identity is not already an Entra ID (non-ME-ID).
Choose FederationIdentifier as the source property.
Choose userPrincipalName as the Entra target property.
Copy the generated expression into the connector's identity-mapping step.
⚠️ If FederationIdentifier holds an email that does not match the Entra UPN, mapping to userPrincipalName will resolve to nobody and users will retrieve nothing. Verify a sample of real values before publishing — this is the single most common failure in production.

Supported data sources
Data source	Typical identity field	Status
Salesforce	FederationIdentifier	
ServiceNow	sys_id / user email	
Jira	AccountId	
Zendesk	user record / email	
Confluence	user key / email	
Limitations
The tool generates configuration; it does not validate against a live tenant. It cannot confirm that the identities in your source system actually resolve in Entra — verify against real sample values before publishing.
It does not detect identities that have been deprovisioned in Entra (for example, users who have left the company). Those will fail at query time regardless of how the expression is written.
It does not address ACL problems that originate in the source system or in the connector itself — only the mapping expression you author during setup
