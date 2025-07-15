# SPA Redirector/Content Server

## Overview

This project consists of a single `index.html` file that acts as a smart router for a statically hosted website (like on GitHub Pages). It allows you to create simple, memorable URLs that can either redirect to another website or serve content directly from your repository, all without needing a server-side language.

This system has two primary modes of operation based on the URL used to access it.

## How It Works

The script inspects the URL's query parameters to decide what to do.

1. **Default Behavior (No Parameters):** If you load the `index.html` page without any parameters, it looks for a file named `root.txt` in the same directory.

2. **Parameter-Based Behavior:** If you load the page with a parameter (e.g., `?folder=file`), it uses the parameter to find a specific file within a sub-folder.

The crucial difference lies in whether the `file` part of the parameter has an extension or not.

## Core Concepts & Usage

### Mode 1: Redirection or Indirect Serving (No Extension in URL)

This mode is used for creating short, clean redirect links or for pointing to other files. The script automatically appends `.txt` to the filename provided in the URL.

* **URL Format:** `/?<folder>=<file_without_extension>`

* **Action:** The script looks for a file at `<folder>/<file_without_extension>.txt`.

* **Behavior:**

  1. It reads the `.txt` file.

  2. If the **first line** of the file is a valid URL (starts with `http`), the browser is immediately redirected to that URL.

  3. If the first line is *not* a URL, the content of the `.txt` file is interpreted as a **filename**. The script then serves the content of *that* file from the same directory.

**Example:**

* URL: `https://<user>.github.io/nav/?sfdc=1.0`

* Looks for file: `/sfdc/1.0.txt`

* If `1.0.txt` contains `https://salesforce.com`, the user is redirected there.

* If `1.0.txt` contains `data.json`, the page will display the content of the file `/sfdc/data.json`.

### Mode 2: Direct File Serving (Extension in URL)

This mode is used to serve any file from your repository directly, just as if the user had navigated to it. This is useful for displaying HTML pages, images, PDFs, or any other file type the browser can handle.

* **URL Format:** `/?<folder>=<file.with.extension>`

* **Action:** The script looks for the exact file at `<folder>/<file.with.extension>`.

* **Behavior:**

  * The file is served directly to the browser. The script infers the correct `Content-Type` (MIME type) from the file's extension (e.g., `.html` becomes `text/html`, `.png` becomes `image/png`).

  * **No redirect check is performed.** The content of the file is always displayed.

**Example:**

* URL: `https://<user>.github.io/nav/?content=page.html`

* Looks for file: `/content/page.html`

* Action: Displays the `page.html` file as a web page inside an iframe.

* URL: `https://<user>.github.io/nav/?assets=logo.png`

* Looks for file: `/assets/logo.png`

* Action: Displays the `logo.png` image on the page.

## Fallback Logic

In any scenario, if the script tries to fetch a file and it doesn't exist (e.g., a 404 Not Found error), it will automatically **fall back to processing the `root.txt` file**. This provides a safe, predictable default behavior for any broken or mistyped links.

## Example File Structure

Here is a sample directory structure to illustrate the concepts:

```
/ (your repository root)
├── index.html
├── root.txt
|
├── sfdc/
│   ├── 1.0.txt
│   └── data.json
|
└── content/
    ├── page.html
    └── document.pdf

```

**How URLs would work with this structure:**

| URL | Action |
| :-- | :-- |
| `.../nav/` | Reads `root.txt`. If it's a URL, redirects. If it's a filename, serves that file. |
| `.../nav/?sfdc=1.0` | Reads `sfdc/1.0.txt`. If it's a URL, redirects. If it contains `data.json`, serves `sfdc/data.json`. |
| `.../nav/?content=page.html` | Serves the file `content/page.html` as an HTML page. |
| `.../nav/?content=document.pdf` | Serves the file `content/document.pdf` as a PDF. |
| `.../nav/?foo=bar` | Fails to find `foo/bar.txt`, falls back to processing `root.txt`. |
