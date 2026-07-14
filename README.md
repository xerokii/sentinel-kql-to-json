# sentinel-kql-to-json

# KQL → Sentinel JSON

A single-file, offline web app that turns KQL queries into an **ARM template** (`Scheduled` analytics rules) ready to import into **Microsoft Sentinel**. Fill in the rule parameters by hand, add as many rules as you need, and preview, copy, or download the resulting JSON.

No build step, no dependencies, no internet connection required — just open the HTML file in a browser.

## Features

- Convert one or more KQL queries into a valid Sentinel ARM deployment template.
- Manually set every rule parameter: display name, description, severity, KQL query, frequency and lookup period, trigger operator/threshold, MITRE ATT&CK tactics and techniques, incident/grouping configuration, suppression, and entity mappings.
- Add, edit, and delete multiple rules — all of them are merged into a single output template.
- MITRE sub-techniques (format `Txxxx.xxx`) are automatically routed to the `subTechniques` field.
- Entity mappings of the same `entityType` are automatically grouped into a single block.
- Live JSON preview; one-click **Copy** and **Download**.
- Query text is escaped automatically (line breaks become `\r\n`, quotes are escaped) so the JSON is always valid.

## Usage

1. Open `sentinel-kql-to-json.html` in any modern browser (double-click it).
2. (Optional) Adjust the **workspace parameter name** and **API version** at the top.
3. Fill in the rule form and paste your KQL query.
4. Click **Add rule to JSON**. Repeat for additional rules.
5. Click **Download JSON** (or **Copy**) to get the file.

## Importing into Microsoft Sentinel

You can use the generated file in two ways:

- **Sentinel UI** — In Microsoft Sentinel go to **Analytics → Import**, then select the downloaded JSON. The current workspace is used automatically.
- **ARM deployment** — Deploy the template via the Azure portal (*Deploy a custom template*) or the CLI:

  ```bash
  az deployment group create \
    --resource-group <your-resource-group> \
    --template-file Azure_Sentinel_analytics_rules.json \
    --parameters workspace=<log-analytics-workspace-name>
  ```

### What is the `workspace` value?

It is the **name of the Log Analytics workspace** that backs your Sentinel instance (just the name, e.g. `law-security-prod` — not a GUID or a resource group).

Find it in the Azure portal under **Log Analytics workspaces**, or in Microsoft Sentinel under **Settings → Workspace settings**. You only need to supply it when deploying via ARM; when importing through the Sentinel UI the current workspace is assumed.

## Generated structure

The output follows the standard Sentinel export format:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": { "workspace": { "type": "String" } },
    "resources": [
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/<guid>')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/<guid>')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules",
            "kind": "Scheduled",
            "apiVersion": "2023-12-01-preview",
            "properties": { "displayName": "...", "query": "...", "severity": "...", "tactics": [], "techniques": [], "entityMappings": [] }
        }
    ]
}
```

Each rule gets a fresh GUID generated in the browser.

## Notes

- Durations use the ISO 8601 format (e.g. `PT1H` = 1 hour, `PT10M` = 10 minutes, `P1D` = 1 day). A custom option is available for any other value.
- Everything runs client-side; no data ever leaves your machine.
- Only `Scheduled` rules are generated (the most common type for KQL-based detections).

## License

MIT
