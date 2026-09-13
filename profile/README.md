# Dr. Goulu Ecosystem & Companion Projects

This document provides a comprehensive technical overview of the companion modules, tools, and custom forks powering **[drgoulu.com](https://drgoulu.com/)**.

---

## 🧭 Architectural Overview

The site is built as a static site using [Hugo](https://gohugo.io/) and the [Hugo Blox](https://hugoblox.com/) framework. To deliver a seamless content-editing experience without relying on traditional server-side databases, it combines customized Hugo modules with a client-side Git-based CMS editor ([Sveltia CMS](https://github.com/drgoulu/sveltia-cms)) integrated via [Headless CMS for Hugo](https://github.com/drgoulu/headless-cms).

```
 drgoulu.com (Hugo site)
 ├── Hugo Modules:
 │   ├── altmetric4hugo  (github.com/drgoulu/altmetric4hugo)
 │   ├── openbook4hugo   (github.com/drgoulu/openbook4hugo)
 │   └── headless-cms    (github.com/drgoulu/headless-cms) [Fork of razonyang/hugo-mod-headless-cms]
 └── Static / Local Assets:
     └── sveltia-cms     (github.com/drgoulu/sveltia-cms)  [Fork of sveltia/sveltia-cms]
```

---

## 📦 Companion Modules & Forks

### 1. `altmetric4hugo`
* **Repository**: [`drgoulu/altmetric4hugo`](https://github.com/drgoulu/altmetric4hugo)
* **Purpose**: Embeds dynamic [Altmetric Badges](https://www.altmetric.com/) (donuts, bars, line badges, and attention scores) in static articles and publication pages.
* **Key Features**:
  * Acts as a modern Hugo replacement for the legacy WordPress *Altmetric* plugin.
  * Supports identifiers: DOIs, arXiv IDs, PubMed IDs (PMID), PubMed Central IDs (PMCID), ISBNs, URIs, Handles, and Altmetric IDs.
  * Responsive layout, customizable alignment, interactive popovers, automatic identifier reference display, and direct hyperlinks.

### 2. `openbook4hugo`
* **Repository**: [`drgoulu/openbook4hugo`](https://github.com/drgoulu/openbook4hugo)
* **Purpose**: Generates rich book data cards and citations directly from [Open Library](https://openlibrary.org/) at build time.
* **Key Features**:
  * Modern Hugo replacement for the WordPress *OpenBook Book Data* plugin.
  * Retrieves book metadata, cover art, author information, external library search links (WorldCat, Google Books, LibraryThing, BookFinder), and COinS citation metadata using ISBNs.
  * Build-time caching with Hugo's `resources.GetRemote`.

---

### 3. `headless-cms` (Fork)
* **Repository**: [`drgoulu/headless-cms`](https://github.com/drgoulu/headless-cms)
* **Upstream Project**: Forked from [`razonyang/hugo-mod-headless-cms`](https://github.com/razonyang/hugo-mod-headless-cms)
* **Purpose**: Hugo module providing the admin interface and automatic CMS configuration generation for static sites.

#### 🔄 Key Changes Compared to Upstream:
1. **Sveltia CMS Engine Support**:
   * Upstream only supported Decap CMS / Netlify CMS (`engine: "decap"` or `engine: "netlify"`).
   * This fork adds native support for `engine: "sveltia"`, generating the corresponding CMS script tags, meta links, and configuration endpoints.
2. **Dynamic Hugo Shortcode Discovery (`/headless-cms/shortcodes.js`)**:
   * Inspects all shortcodes available in the Hugo site layout as well as all imported Hugo modules (e.g., `altmetric`, `openbook`, `figure`, `youtube`, `vimeo`, `gist`, `highlight`).
   * Automatically exports a JavaScript manifest (`window.__hugoShortcodes = [...]`) accessible to Sveltia CMS so the editor preview can render any Hugo shortcode.
3. **Dual Development Modes for the CMS Editor**:
   * **`dev_server`**: Injects Vite HMR and development scripts (`http://localhost:5173`) for live development of Sveltia CMS with instant hot-reloading.
   * **`local_script`**: Allows serving a locally built standalone bundle (`/sveltia-cms.js`).
4. **Sveltia Configuration Generator**:
   * Generates `/headless-cms-config.yaml` with collections, fields, and options automatically inferred from site configuration.

---

### 4. `sveltia-cms` (Fork)
* **Repository**: [`drgoulu/sveltia-cms`](https://github.com/drgoulu/sveltia-cms)
* **Upstream Project**: Forked from [`sveltia/sveltia-cms`](https://github.com/sveltia/sveltia-cms)
* **Purpose**: Lightweight, Git-based single-page application CMS built with Svelte 5.

#### 🔄 Key Changes Compared to Upstream:
1. **Hugo Shortcode Live Previews in Rich Text Editor**:
   * Upstream lacked support for Hugo shortcode syntax (`{{< ... >}}` and `{{% ... %}}`).
   * Added dedicated preview components for:
     * `figure`: Renders image, caption, title, link, and sizing attributes.
     * `youtube` and `vimeo`: Embedded responsive video players.
     * `openbook`: Interactive book metadata preview cards.
     * `altmetric`: Live Altmetric badge donut embed.
     * `gist`: GitHub Gist embed preview.
     * `highlight`: Syntax-highlighted code blocks.
     * **Generic shortcodes**: Dynamically supported using definitions loaded from `headless-cms`.
2. **Native Subfolder Support in Collection File Matcher**:
   * In upstream, files in subfolders (like `content/posts/2012/my-post.md`) required explicit Decap `nestedDepth` configuration.
   * This fork matches nested subpaths at any depth by default without breaking flat collections.
3. **Hierarchical Subfolder Tree View in Sidebar**:
   * Automatically builds and displays a subfolder tree (e.g., `1980`...`2026`) in the primary left navigation sidebar for collections with subfolders, even without Decap `nested` options in the configuration.
4. **Hierarchical Subfolder Filtering**:
   * Selecting a subfolder in the tree filters the entry list to show only entries belonging to that folder and all its descendant subfolders.
   * Selecting the collection root displays all entries across all subfolders.
5. **Global Search with Contextual Subfolder Auto-Selection**:
   * Search remains global across the whole site, all collections, and all subfolders.
   * Selecting an entry from search results (or opening it directly) automatically selects and highlights its parent subfolder in the left sidebar tree, expanding ancestor folders as needed.

---

## ⚙️ Installation & Hugo Configuration

### 1. Hugo Modules (`config/_default/module.yaml`)

Import the modules into your site:

```yaml
imports:
  - path: github.com/HugoBlox/kit/modules/integrations/netlify
  - path: github.com/HugoBlox/kit/modules/blox
  - path: github.com/drgoulu/openbook4hugo
  - path: github.com/drgoulu/altmetric4hugo
  - path: github.com/drgoulu/headless-cms
```

### 2. Go Modules & Local Development (`go.mod`)

In `go.mod`, declare the module dependencies. For local development with active sibling checkouts, use `replace` directives:

```go
module github.com/HugoBlox/kit/templates/blog

go 1.19

require (
	github.com/HugoBlox/kit/modules/blox v0.0.0-...
	github.com/HugoBlox/kit/modules/integrations/netlify v0.0.0-...
	github.com/drgoulu/altmetric4hugo v0.0.0-...
	github.com/drgoulu/headless-cms v0.0.0-...
	github.com/drgoulu/openbook4hugo v0.0.0-...
)

replace github.com/drgoulu/altmetric4hugo => ../altmetric4hugo
replace github.com/drgoulu/openbook4hugo => ../openbook4hugo
replace github.com/drgoulu/headless-cms => ../headless-cms
```

---

### 3. CMS Configuration in `config/_default/hugo.yaml`

Configure `params.headless_cms` in `config/_default/hugo.yaml`:

```yaml
############################
## HEADLESS CMS CONFIGURATION
############################

params:
  headless_cms:
    # Use our customized Sveltia CMS engine
    engine: "sveltia"

    # Option A (Development): Live Vite dev server running in sveltia-cms
    dev_server: "http://localhost:5173"

    # Option B (Production / Standalone): Compiled static bundle
    # local_script: "/sveltia-cms.js"

    # Site base URL for preview and asset resolution
    site_url: "https://drgoulu.com"

    # Git backend configuration for direct GitHub synchronization
    backend:
      name: "github"
      repo: "drgoulu/drgoulu.com"
      branch: "main"

    # Asset storage paths
    media_folder: "static/uploads"
    public_folder: "/uploads"

    # Content Collections
    collections:
      posts:
        name: "posts"
        label: "Articles"
        label_singular: "Article"
        folder: "content/posts"
        fields:
          - { label: Titre, name: title, widget: string }
          - { label: Date, name: date, widget: datetime, type: date }
          - { label: Corps, name: body, widget: richtext }
```

---

## 🚀 Local Development Workflow

To run the local development environment with Hugo and live Sveltia CMS:

1. Execute the development helper script from the root of `drgoulu.com`:
   ```bash
   bash ./scripts/dev-sveltia.sh
   ```
   This concurrently starts:
   * **Vite dev server** (`http://localhost:5173`) in `sveltia-cms`
   * **Hugo server** (`http://localhost:1313`) in `drgoulu.com`

2. Access the CMS editor at:
   ```
   http://localhost:1313/admin/
   ```

3. Any edits made to `sveltia-cms` will hot-reload instantly in the browser.
