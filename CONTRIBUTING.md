# Best Practices for Technical Guides (Quick Reference Format)

This document outlines the best practices for creating concise, action-oriented technical guides designed for quick reference, especially for users who will become familiar with the procedure over time.

## 1\. Core Principles

  * **Action-First Directives:** Every step should begin immediately with the primary action verb. The focus is on *what the user needs to do*.
      * **Example:** "Click **Create a tunnel**..." (NOT "Log in and then click...")
  * **Conciseness & Scannability:** Eliminate unnecessary words, jargon, and lengthy explanations. The goal is rapid comprehension and scannability for quick reference.
  * **Clarity for Quick Reference:** While concise, ensure clarity. Assume a user who will eventually be familiar with the platform/procedure, allowing for shorthand where context is implied.
  * **Leverage Online Medium:** Utilize hyperlinks effectively for navigation and external references to keep the core text brief.

## 2\. Formatting Guidelines

  * **UI Elements (Buttons, Labels, Options):**
      * **Buttons:** Represent buttons using the `<button>Button Name</button>` format.
      * **UI Labels/Options:** Use **bold** (`**Label Name**`) for names of fields, checkboxes, dropdown options, or sections within a UI.
          * **Example:** "Check `Allow HTTP traffic`"
  * **Code, Commands, & Exact Values:**
      * Use backticks (`` ` ``) for specific code snippets, commands, exact input values, or file/folder names.
          * **Example:** `n8n-free-tier`, `e2-micro`, `Container Optimized OS`
  * **Hyperlinks:**
      * For primary navigation points (e.g., leading to a service dashboard), combine the service name with a hyperlink and an arrow symbol (`↗`) for external links.
      * **Format:** `[Service Name↗](URL)`
      * **Example:** `[Zero Trust↗](https://dash.cloudflare.com/zero-trust)`
  * **Navigation Paths:**
      * Place navigation instructions within **parentheses** immediately following the primary action.
      * Use `>` to denote hierarchy in the navigation path.
      * **Format:** `(Service Name↗ > Section > Page)`
      * **Example:** "Click **Create a tunnel** ([Zero Trust↗](https://www.google.com/url?sa=E&source=gmail&q=https://dash.cloudflare.com/zero-trust) \> **Networks** \> **Tunnels**)"
  * **Structuring Steps:**
      * Use markdown hyphen (`-`) for main steps.
      * Indent with hyphens or asterisks (`-` or `*`) for sub-configurations or nested options. Maintain consistent indentation.
      * **Example:**
        ```markdown
        - Click <button>Action</button> and configure:
            - **Setting 1**: `Value`
            - **Setting 2**: `Another Value`
        ```
  * **Introducing Configuration Steps:**
      * When an action leads to a set of configurations, use "and configure:" to smoothly transition.
      * **Example:** "Click \<button\>Create Instance\</button\>... and configure:"

## 3\. Handling Variable and Dynamic Inputs

  * **Flexible Naming/Values:**
      * Provide a suggested default or example, followed by a clear indicator of user choice in parentheses.
      * **Format:** `[Suggested Value]` (or your preferred name/value)
      * **Example:** `Name: n8n-free-tier` (or your preferred name)
  * **Selecting from Specific Options (e.g., Regions):**
      * Provide a primary recommended option.
      * List alternative choices using the `|` (pipe) character as a separator within parentheses.
      * **Avoid:** Using backticks (`` ` ``) for the list of options themselves within the parentheses to prevent visual confusion with fixed values.
      * **Avoid:** Adding leading or trailing `|` characters.
      * **Format:** `[Primary Option]` (or your preferred [setting] from: `Option 1 | Option 2 | Option 3`)
      * **Example:** `Region: us-central1 (Iowa)` (or your preferred region from: `us-west1 (Oregon) | us-east1 (South Carolina)`)
  * **Copying Dynamic Data (e.g., Tokens):**
      * Clearly state the action (`Copy`).
      * Specify *what* to copy (e.g., "token value").
      * Provide clear instructions on *where* to find it, referencing nearby fixed elements.
      * Consider displaying the full command/string separately for context.
      * **Example:** "Copy the **token** value (the alphanumeric string immediately after `--token`) from the displayed command."

## 4\. Tone and Language

  * **Standard English:** Maintain a professional and clear tone.
  * **Avoid Overly Informal Language:** Replace casual phrases (e.g., "whatever you want") with more professional alternatives (e.g., "your preferred name").
  * **Proofreading:** Always proofread for grammatical correctness, typos, and overall clarity.

## 5\. Licensing and Translations

This project's content is licensed under the [Creative Commons Attribution-NoDerivatives-NonCommercial 4.0 International (CC BY-NC-ND 4.0) License](https://creativecommons.org/licenses/by-nc-nd/4.0/).

While this license generally prevents the distribution of modified versions of the work, we warmly welcome contributions for **translations** directly to this repository. If you wish to contribute a translation:

* Please submit your translated content via a Pull Request.
* By submitting a translation, you agree for your contribution to be incorporated into this project, which will then be distributed by the project maintainers under the terms of the CC BY-NC-ND 4.0 license.