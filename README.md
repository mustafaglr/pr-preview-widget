# Azure DevOps Extension: Active PR Overview Widget

This document provides a full overview of how to build and configure an Azure DevOps dashboard widget named **Active PR Overview Widget**. The widget displays active pull requests across selected projects and supports rich filtering via a configuration panel.

---

## 🧰 Prerequisites

Ensure the following tools are installed:

```bash
npm install -g tfx-cli
```

Other requirements:

* Node.js (LTS recommended)
* Azure DevOps organization access
* Basic knowledge of HTML, JavaScript, and Azure DevOps APIs

---

## 📁 File Structure

```
├── configuration.html        # Configuration screen
├── index.html                # Main widget display
├── scripts/
│   └── VSS.SDK.min.js        # SDK provided by Microsoft
├── images/
│   ├── icon.png              # Widget icon
│   └── preview.png           # Widget preview image
├── vss-extension.json        # Manifest file for the extension
```

---

## 🧩 Widget Overview

The widget (`index.html`) shows active pull requests from Azure Repos Git projects. It supports:

* Filtering by selected projects
* Regex filtering by project name
* Filtering by reviewer team
* Source and target branch filtering

### Main Features:

* Fetches PRs using `TFS/VersionControl/GitRestClient`
* Displays project, repo, branches, title, creator, and creation date
* Uses avatars or initials
* Opens PRs in a new tab when clicked

---

## ⚙️ Configuration Panel (configuration.html)

This file allows users to configure the widget settings:

* `Include All Projects`: Show PRs from all projects
* `Allowed Projects Regex`: Filter project names by regex
* `Select Projects`: Manual selection via checkboxes
* `Reviewer Team Filter`: Only include PRs where a team member is a reviewer
* `Source/Target Branch Regex`: Filter branches using regex

### Logic Highlights:

* Uses `TFS/Core/RestClient` to list all available projects
* Applies filter logic and saves custom settings as JSON
* Disables dropdown if `Include All Projects` or regex is filled

---

## 🔧 Key Functions in the Code

Here’s a quick explanation of core functions across the widget and configuration files:

### `getShortBranchName(fullBranchName)`

Simplifies full branch names (e.g. `refs/heads/feature/xyz`) to display-friendly format (`feature/xyz`).

### `getInitials(name)`

Extracts initials from a user’s display name, used when an avatar image is not available.

### `load(widgetSettings)`

Main lifecycle hook for both the widget and configuration. Loads saved settings, fetches projects and PRs, and renders them.

### `onSave()`

Gathers selected configuration values and returns them as widget settings.

### `updateWidgetData()`

Triggered on input changes in the config panel to live-update settings and notify Azure DevOps of configuration state.

### `toggleProjectListState()`

Enables or disables the manual project list UI based on whether all projects are selected or regex is used.

---

## 📦 Manifest File: vss-extension.json

Defines the widget and configuration contribution:

```json
{
  "id": "active-pr-overview-widget",
  "version": "1.0.12",
  "name": "Active PR Overview Widget",
  "publisher": "mustafa",
  "targets": [ { "id": "Microsoft.VisualStudio.Services" } ],
  "contributions": [ ... ],
  "files": [ ... ],
  "categories": ["Code", "Collaboration"],
  "tags": ["Dashboard", "Widget", "Pull Request"]
}
```

### Key Sections Explained

* `manifestVersion`: Always `1`
* `id`: Unique ID for the extension
* `version`: Semantic version
* `description`: What your widget does
* `publisher`: Your publisher ID (must be registered in DevOps Marketplace)
* `targets`: Platform target, always include `Microsoft.VisualStudio.Services`

### Contributions

* `type: ms.vss-dashboards-web.widget`: The widget itself
* `type: ms.vss-dashboards-web.widget-configuration`: Configuration UI
* `uri`: Points to `index.html` and `configuration.html` respectively

---

## ▶️ Build and Package

Use the following command to build your extension package:

```bash
tfx extension create --manifest-globs vss-extension.json
```

This command will generate a `.vsix` file you can upload to Azure DevOps.

---

## 📌 VSS.SDK.min.js

This file is necessary for your widget to work inside the Azure DevOps dashboard environment. It provides the `VSS.init()`, `VSS.require()` and registration methods.

### Example:

```js
VSS.init({
  explicitNotifyLoaded: true,
  usePlatformStyles: true,
  usePlatformScripts: true
});

VSS.require(["TFS/Dashboards/WidgetHelpers"], function (WidgetHelpers) {
  VSS.register("your-widget-id", function () {
    return {
      load: function(widgetSettings) {
        // Your logic here
      }
    };
  });

  VSS.notifyLoadSucceeded();
});
```

---

## ✅ Final Notes

* The widget is designed to be responsive and informative.
* Branch and reviewer filters make it ide al for multi-team environments.
* Project selection via regex or dropdown ensures flexibility.
