# Usage Reports

{{% alert title="Enterprise Only Feature" %}}
This functionality is available only to users on Enterprise plans. Hosted Organizations support is planned but not yet implemented.
{{% /alert %}}

Saturn Cloud Enterprise provides admin users with detailed reports about **Saturn Cloud usage within your organization**. These reports show how your Saturn Cloud resources are used by users, resource types, and time periods. They do **not** include billing or usage data from your underlying cloud provider (such as AWS, Azure, or OCI); they only reflect Saturn Cloud platform usage.

You can access usage reports in the **Settings** tab of the Saturn Cloud application under **Usage Reports**. A report is generated for each month that Saturn Cloud has been running in your enterprise account. The data is updated hourly.

![Usage reports list](/images/docs/usage-reports-list.webp "doc-image")

### Download Formats

Each month has two download options:

- **HTML** – A full HTML report with charts and tables of usage data broken down by users, resource type, and time period.  
- **CSV** – A CSV file with one row per resource per hour it was active. This format is ideal for custom analysis or integration with your BI tools.

![Example usage report](/images/docs/usage-report.webp "doc-image")

### Direct Database Access

If you prefer direct access to your Saturn Cloud usage data, Saturn Cloud can provide read access to a Snowflake database containing your organization's usage information. This enables you to run your own queries or integrate usage data into your internal dashboards. Contact <a href="/docs">Saturn Cloud Support</a> to request database access or discuss additional options.

### Scope and Hosted Organizations

- **Scope** – Usage Reports show only Saturn Cloud platform activity (compute resources, notebooks, jobs, dashboards, etc.) inside your organization's Saturn Cloud instance.  
- **Hosted Organizations** – Support for Hosted Organizations is planned and will appear in a future release. Until then, Usage Reports only cover your primary Saturn Cloud Enterprise environment.
