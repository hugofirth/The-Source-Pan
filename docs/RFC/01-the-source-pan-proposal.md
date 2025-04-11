---
Title: "The Source Pan" initial project proposal
Status: DRAFT
Outcome: "" # To be filled upon resolution
Description: RFC detailing the high level concept for The Source Pan (tsp), a CLI toolkit for building brilliant context in complex codebases 
Author: Hugo Firth
Created: 2025-04-11
---

# Project Proposal for “The Source Pan” (tsp)

Below is a structured summary of the key capabilities, workflows, and design considerations we’ve discussed so far. This spec can serve as an initial blueprint to hand off to a developer.


## 1. Overview

The Source Pan (tsp) is a command-line toolkit for creating and managing non-technical artifacts (RFCs, tasks, ADRs, etc.) alongside the code they relate to, as well as generating and maintaining a rich knowledge graph of the codebase. It aims to provide humans and LLMs with consistent, up-to-date “context” about the project—covering both source code details and domain/organizational knowledge.


## 2. Installation and Setup

	1.	Installation
	•	Installable via standard OS package managers (e.g., brew on macOS, apt on Debian).
	•	A system-wide command tsp is made available upon installation.
	2.	Project Initialization
	•	Run tsp init inside a Git repository to set up tsp for that project.
	•	If not a Git repo, the command fails with an appropriate error.
	•	If already initialized, tsp init does nothing but confirm it’s already set up.
	•	Guided “setup wizard” collects user preferences, such as:
	•	Which non-technical artifacts to manage.
	•	Where (or whether) existing artifacts are stored, or if new directories/templates should be created.
	•	(Optional) Integration with a language server (e.g., via LSP) to generate codebase knowledge (AST, references, etc.).
	•	(Optional) Integration with Git history for authorship data.
	•	Preferred LLM provider and API key (saved in a local .env or similar, ignored by Git).
	•	Whether to automatically generate source file descriptions using the LLM.
	•	Offers a preview of token usage and cost.
	•	Allows the user to narrow scope (exclude certain modules, only generate minimal summaries, etc.).
	•	Whether to generate vector embeddings for any artifacts (for future RAG usage).
	•	Whether to auto-update artifacts and knowledge graph on every commit or only on demand.
	3.	Configuration File (.tsp.conf)
	•	Stored in the project’s root directory.
	•	Persists all user choices from tsp init (e.g., artifact directories, LLM type, update strategies).
	•	Does not store sensitive API keys (those go in a .env-type file).
	•	Potentially in a properties, JSON, or TOML format.
	•	Users can also edit .tsp.conf directly (via tsp configure) or globally (via tsp configure --global).


## 3. Managing Artifacts

Artifacts are non-technical items like tasks, RFCs, ADRs, design docs, etc. They’re stored in plain-text files (commonly Markdown), one file per artifact. Users can define new artifact types and create templates for each.
	1.	Artifact Templates
	•	Found in a .tsp/templates or similarly named directory within the repo.
	•	Typically markdown files containing placeholder variables (e.g., {{author}}).
	•	Users can also define system-wide templates in a global directory (for reuse across multiple repos).
	2.	Creating Artifacts

`tsp create <artifact-type> <artifact-name>`

	•	Prompts the user to fill in template variables.
	•	After completion, the user can open the new file in their chosen editor or simply finalize the creation.
	•	The new file is saved in the location configured for that artifact type (e.g., ./rfc folder).

	3.	Listing and Finding Artifacts
	•	tsp list <artifact-type> lists existing artifacts (or tsp list <artifact-type> <artifact-name> to get details).
	•	tsp find <artifact-type> <query-string> performs a basic text search within artifacts of that type.
	4.	Removing Artifacts
	•	tsp remove <artifact-type> <artifact-name> deletes or archives the corresponding file, prompting for confirmation.
	5.	Exporting Artifacts
	•	tsp export-artefact <artifact-type> <artefact-names-list> [--all] [--static-site] <target-folder>
	•	Exports the specified artifacts to either raw text or a basic static site format for external consumption.


## 4. Knowledge Graph and Codebase Insights

Knowledge Graph
	•	tsp creates a codebase knowledge graph capturing:
	1.	Folder Hierarchy: directories and files with contains relationships.
	2.	Project Hierarchy: AST info via tree-sitter or language servers.
	3.	References and Definitions: symbol definitions and references across files.
	4.	Authorship: commits, commit messages, authors (extracted from Git history).
	5.	Links to Non-Technical Artifacts: references (e.g., an RFC mentioning a specific file or symbol).
	•	The data is intended for:
	•	Human consumption (easy visual navigation).
	•	Machine consumption (e.g., RAG indexing, code navigation, or advanced queries).

	1.	Storage Format
	•	A text-based (likely JSON) structure, stored in version control.
	•	Possibly split across multiple files for layering (e.g., main code structure vs. authorship vs. references).
	•	Merges/branch conflicts resolved similarly to code merges (by re-generating as needed).
	2.	Generating and Updating
	•	Created initially by tsp init wizard (if the user agrees).
	•	Maintained via tsp update or automatically via Git commit hooks (user choice).
	•	Each update references the current Git SHA to note the state of the codebase.
	3.	Exporting the Graph
	•	tsp export-graph <target-folder> writes the graph to disk (likely in JSON) for integration with external tools (e.g., Neo4j) or visualization software.


## 5. LLM Integration

	1.	LLM Setup
	•	During tsp init, the user can specify an LLM provider and place their API key in a .env-style file (gitignored).
	•	The .tsp.conf references the LLM provider but never stores the key itself.
	2.	Generating Summaries & Descriptions
	•	The user can choose to have tsp generate documentation for source files, modules, classes, etc.
	•	Before sending context to the LLM, tsp warns the user about approximate token usage and cost.
	•	Summaries can be placed inline as comments, in sidecar files (e.g., .tsp-info), or purely in the knowledge graph.
	3.	Vector Embeddings (Optional)
	•	The user can specify which artifact types or code segments to embed for similarity search or other RAG workflows.
	•	tsp can manage these embeddings in a version-controlled manner (although large embeddings might be stored separately or compressed).


## 6. Everyday Commands & Workflows

	1.	Create Artifacts

`tsp create <artifact-type> <artifact-name>`

	•	Interactive prompts for filling template variables.
	•	Option to open the newly generated file in the user’s default editor.

	2.	Get Context

`tsp get-context <artifact-type> <artifact-id|name> --for-humans [--for-machines=target-file]`

	•	Gathers relevant code files, related artifacts, and key contributors.
	•	Human-readable version: a neatly formatted CLI output with headings like “Related Source Files,” “Key Contributors,” etc.
	•	Machine-readable version: appended to a specified file in a structured format (e.g., JSON) containing more extensive detail (like entire source file contents) for LLM usage.

	3.	Update

`tsp update`

	•	Manually triggers knowledge graph generation or re-generation for the current Git state.
	•	If configured, may run automatically on each commit hook.

	4.	Configure

`tsp configure`
`tsp configure --global`

	•	Opens .tsp.conf in the default editor for local or global settings.

	5.	List/Find/Remove Artifacts
	•	tsp list <artifact-type> to see all items of that type.
	•	tsp find <artifact-type> <query-string> for basic searches.
	•	tsp remove <artifact-type> <artifact-name> for cleanup.
	6.	Export Artifacts or Graph
	•	tsp export-artefact ... to publish documents externally.
	•	tsp export-graph ... to export the knowledge graph for external visualization or ingestion.


## 7. Conflict Resolution & Concurrency

	1.	Non-Technical Artifacts and Configs
	•	Since artifacts and .tsp.conf are plain text in version control, teams resolve conflicts via standard Git merge workflows.
	2.	Knowledge Graph Files
	•	In case of merge conflicts (e.g., two parallel updates to the graph on different branches), it’s recommended to finalize merges at the artifact/code level, then run tsp update on the merged result. This ensures the knowledge graph is re-generated or updated according to the final code state.


## 8. Future/Optional Ideas

	•	Custom Graph Database Integration
	•	Provide an optional “live” link to a Neo4j instance or similar for teams wanting more advanced queries.
	•	Could be enabled via an additional configuration in .tsp.conf.
	•	Expanded LLM Features
	•	Automatic generation of “task plans” that highlight relevant references in code, plus recommended next steps.
	•	More advanced or compressed embedding workflows for large codebases.
	•	Visual CLI or Web UI
	•	Potential to develop a web-based “dashboard” mode or a TUI (text-based UI) for browsing artifacts and code references.

## 9. Potential Issues

	1.	Balancing Simplicity and Power
	•	Potential Pitfall: If “tsp” tries to do everything (managing artifacts, building a knowledge graph, generating vector embeddings, hooking into LLMs, and so on), the codebase and user experience could become too complex.
	•	Mitigation: Start with a well-defined, minimal core workflow. Provide clear extension points for advanced features to avoid bloating the core tool.
	2.	Maintaining the Knowledge Graph Over Time
	•	Potential Pitfall: In large, fast-moving codebases, the knowledge graph might become stale or out-of-sync quickly, especially if auto-updates are disabled, or if teams branch heavily.
	•	Mitigation: Emphasize best practices around update triggers (commit hooks, periodic CI tasks, or explicit tsp update usage). Make it very easy to run and verify the latest state.
	3.	Handling Large Codebases and Token Costs
	•	Potential Pitfall: For huge repositories, generating per-file or per-method summaries with an LLM could be expensive and time-consuming.
	•	Mitigation: Provide robust options to filter or throttle which parts of the code get summarized. Possibly allow partial or incremental summarization, or alternate summarization strategies (e.g., short docstrings vs. full-file introspection).
	4.	User Adoption and Workflow Fit
	•	Potential Pitfall: If the tool requires too much extra setup or imposes new processes, developers may bypass it.
	•	Mitigation: Ensure the default workflows feel natural (e.g., it doesn’t disrupt typical Git usage too heavily). Offer value quickly—like giving a neat “context summary” or “map” on demand—so devs see immediate benefit.
	5.	Schema Evolution & Merging
	•	Potential Pitfall: Over time, the structure of the knowledge graph, artifact metadata, or config might need to evolve. Merges could get tricky if the format changes significantly or if multiple branches alter the same files.
	•	Mitigation: Provide versioning in the .tsp.conf schema or knowledge graph schema. Make re-generation of the graph a common practice to avoid messy merges.
	6.	Managing Sensitivity of Data
	•	Potential Pitfall: Summaries, commit messages, or artifact contents might contain private or proprietary information, so some teams might want to limit what gets sent to LLM APIs.
	•	Mitigation: Clearly separate the steps that transmit data to LLM providers. Provide ways to skip or mask certain files, directories, or commit messages.

## 10. Summary

tsp offers a holistic approach to managing technical and non-technical context in one place. By embedding the project’s domain knowledge, code structure, authorship, and more, it enables both human developers and LLMs to more quickly find and reference the details they need. The key pillars are:
	•	Artifact Management (RFCs, tasks, docs, etc.) via templates and direct Git storage.
	•	Knowledge Graph capturing how files, symbols, commits, and documents interrelate.
	•	LLM Support for summarizing code and artifacts, generating embeddings, and providing context.
	•	Configurable Updates (manual or commit-hook-based) to keep everything in sync as the codebase evolves.

This spec should give developers a solid starting point for building out the initial version of tsp, with room to refine details as the project proceeds.
