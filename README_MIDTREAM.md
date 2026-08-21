# WordPress Custom Build: Midstream Branch

Welcome to the **midstream** branch. This repository/branch combination automates the process of merging pristine WordPress core releases with our own custom modifications to generate a secure, production-ready WordPress build.

---

## Purpose of the Midstream Branch

The **midstream** branch is where all of our custom WordPress configurations, plugins, themes, `mu-plugins`, and automation templates live. 

Rather than modifying WordPress core files directly (which makes updating difficult and error-prone), we isolate all of our custom changes here. Everything committed to `midstream` is treated as an integration layer that will be automatically overlaid on top of the latest stable WordPress core.

---

## Architecture & Pipeline Workflow

Our automated CI/CD pipeline runs on a scheduled and on-demand basis, following a strict multi-stage workflow:

1. **Upstream Sync (`upstream` branch):**
   * WordPress updates frequently. Once a day (or on-demand), the pipeline checks for the latest **stable** release from the official WordPress repository and syncs it to the `upstream` branch. We strictly track stable releases to ensure reliable maintenance.

2. **Midstream Overlay (`downstream` branch):**
   * Once `upstream` is up-to-date, the pipeline pulls the payload from this `midstream` branch.
   * It takes a pristine copy of WordPress core, resets it, and overlays our midstream modifications on top of it.
   * This creates our **`downstream`** branch and tags it with a unique release identifier (e.g., `wp-stable-[VERSION]-[COMMIT]`).

3. **Security & Validation (`test` & `secret-detection`):**
   * Before any code goes live, the generated `downstream` build is subjected to automated security scans, including **SAST (Semgrep)** and **Secret Detection**. 
   * The pipeline ensures no credentials have leaked and no vulnerabilities have been introduced before allowing a deployment.

4. **External Push (`push-external`):**
   * Only after passing all security gates will the pipeline push the clean `downstream` build and its release tag out to our external production repositories on GitLab and GitHub.

---

## Design Philosophy & Best Practices

* **Leave Core Alone:** Core WordPress files are never manually modified. All customizations belong in `wp-content/themes`, `wp-content/plugins`, or `wp-content/mu-plugins`.
* **Modularity:** Midstream acts as an aggregation point. Many of the components pulled into `midstream` may originate from separate upstream addons, repositories, or automated templates. 
* **Stable Tracking:** By anchoring our builds exclusively to stable WordPress releases, we minimize breaking changes and keep our custom modifications reliable over time. 

---

## How to Make Changes

If you need to add a plugin, update a theme, or modify a configuration:
1. Make your changes **exclusively on the `midstream` branch** (or push them into their respective sources that feed into midstream).
2. Allow the scheduled or manual pipeline to rebuild the `downstream` branch.
3. Verify that the build passes security checks and successfully syncs to external production repositories.