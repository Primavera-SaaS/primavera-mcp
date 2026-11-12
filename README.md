# Primavera plugin for Claude Code

Build B2B prospect lists from Claude. [Primavera](https://primaverapp.com) is a database of businesses: the Google Maps
places of France and of other countries, read with their websites (email addresses, phone numbers, websites, social
networks, technologies, executives in France). With this plugin, Claude turns a request in plain words ("independent car
repair shops around Lyon, with an email") into a list: it finds the categories and the area, counts, shows sample rows,
tells the cost, creates the list once you agree, and downloads it as a CSV or Excel file.

## What it contains

- A remote MCP server, `https://primaverapp.com/api/mcp` (Streamable HTTP): the Primavera API as tools.
- A skill, `prospect-lists`: the path from a request to a downloaded file, and the money rules.

## What you need

- A Primavera account with a Pro or Agency subscription (the API is not open to trial or free accounts).
- An API key, created at https://primaverapp.com/dashboard/api. The plugin asks for it at install time and keeps it as
  a sensitive setting; it is sent only to `https://primaverapp.com`, in the `Authorization` header.

## Install

```
/plugin marketplace add Primavera-SaaS/primavera-mcp
/plugin install primavera@primavera
```

Then ask Claude, for example: "Find independent bakeries in Bordeaux with an email address and give me a CSV."

## Money

The tools spend the credits of your Primavera account, at the prices of the web app: free lookups, counts, previews
and estimates; 100 credits per category suggestion; 1 credit per new place exported (Agency: included under fair use);
the jobs and the email verification at their usual prices. Every paid tool needs a credit limit (`maxCredits`, never
exceeded) and an idempotency key, so that a retry never charges twice. Claude asks you before every paid call.

## Data and network

The plugin sends requests to one destination only: `https://primaverapp.com` (the MCP server, which runs the Primavera
API). It sends your API key, your requests (targets, areas, filters, list names, questions asked to websites) and reads
back counts, sample rows without contact details, lists and download links. Downloaded files are written where you ask
Claude to write them. Privacy policy: https://primaverapp.com/privacy. Terms: https://primaverapp.com/terms.

## Without the plugin

Any MCP client can use the server directly with the header `Authorization: Bearer <your key>`: configuration for
Claude Code, Claude Desktop, Cursor and Codex at https://primaverapp.com/docs/api#from-claude-cursor-or-codex-mcp.

## License

MIT.
