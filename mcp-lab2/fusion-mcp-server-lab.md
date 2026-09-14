# Fusion MCP Server

Build an MCP server with a custom tool backed by an OpenAPI connection in Amplify Fusion / AI Gateway, then test it in Claude Desktop.

## Requirements

* Access to Amplify Fusion (https://emea-techlab.sandbox.fusion.services.axway.com) with a working project and MCP Server ready to configure
![connection](images/01-connection.png)
* Connection details to an OpenAPI to manage invoices
* Claude Desktop installed (requires Node.js), with a free Claude account or higher

## Steps

### 1. Update the MCP server

* Open your Amplify Fusion project
![open-project](images/02-open-project.png)
* Open the existing MCP Server `Invoice Fusion MCP Server`
![mcp-server-open](images/03-mcp-server-open.png)
* Edit the MCP server configuration and set a unique frontend base path (use your username, for example)
* Click **Save**
![mcp-server-config](images/04-mcp-server-config.png)


### 2. Add a tool

* Click **Add Tool**
* Give it the name `applyDiscount`
* Give it the title `Apply Discount to Invoice`
* Provide a precise description: `Apply a discount to a specified invoice. Discounts can only be applied to draft invoices.`
* Define the input JSON schema, for example:
  ```json
  {
    "type": "object",
    "properties": {
      "discount": {
        "type": "string",
        "example": "10%"
      },
      "invoice_id": {
        "type": "string"
      }
    },
    "required": [ "discount" , "invoice_id" ]
  }
  ```
![add-tool](images/05-add-tool.png)

### 3. Add an integration to the tool

* Click the link icon to add an integration.
![link-integration](images/06-link-integration.png)
* Choose **Create a new integration** and name it `mcpTool_applyDiscount`.
![create-new-integration](images/07-create-new-integration.png)
* Expand the **Tools** component in the integration flow.
* Add the API call step:
  * Click the **+** button and add a **OpenAPI client - Invoke operation** component
  ![add-api-invoke-step](images/08-add-api-invoke-step.png)
  * Expand the bottom panel.
  * In the center part, select the existing **Zoho Invoice API** connection in the center panel.
  * Select the **Invoice** object.
  * Select the **UpdateInvoice** action (the API operation).
  ![set-api-connection](images/09-set-api-connection.png)
  * On the left, locate **applyDiscountMCPServerRequest/body/discount** and **applyDiscountMCPServerRequest/body/invoice_id** in the input variables.
  * Map **applyDiscountMCPServerRequest/body/discount** to **body/discount**, by dragging a line from left to right.
  ![map-api-request-body](images/10-map-api-request-body.png)
  * Map **applyDiscountMCPServerRequest/body/invoice_id** to **pathParams/Invoice_id** (Warning: don't map with **body/invoice_id**)
  ![map-api-request-pathparam](images/11-map-api-request-pathparam.png)
  * On the right, locate **applyDiscountMCPServerResponse/TextContent/result/content/text** in the output variables.
  * Map **UpdateInvoiceOutput/response/message** to **applyDiscountMCPServerResponse/TextContent/result/content/text** on the right.
  * Map **UpdateInvoiceOutput/error/message** to the same **applyDiscountMCPServerResponse/TextContent/result/content/text** on the right.
  * Right-click **toolResponseType** on the right and set its value to `TextContent`.
  * Save the component configuration
  ![map-response](images/12-map-response.png)

### 4. Check back-end connection

* Click on **Navigate** next to the API connection
* Scroll down the backend API connection properties and click **Test**
![test-connection](images/13-test-connection.png)
* If a green check is returned, continue with step 5.
* In case you get a red cross, try **Generate token** just above and provide provided Zoho user credentials. 
  ![generate-token](images/14-generate-token.png)
  * If a green check is returned, click **Test** again. If a green check is returned, continue with step 5.
  * In other cases check connection details, and configuration on Zoho Invoice side


### 5. Activate the MCP server

* Go back to your MCP server and activate it
* Copy the MCP server URL
![activate-mcp-server](images/15-activate-mcp-server.png)

### 6. Test from Claude Desktop

* Open Claude Desktop config (File > Settings > Developers > Edit Config)
![claude-desktop-config](images/16-claude-desktop-config.png)
* Add your MCP server to `claude_desktop_config.json`:
  ```json
  {
    "mcpServers": {
      "Invoice Fusion MCP Server": {
        "command": "npx",
        "args": ["mcp-remote", "<your MCP server URL>"]
      }
    },
    "preferences": { ... }
  }
  ```
* Save the config file
* Restart Claude Desktop (File > Exit, then reopen)
* Check that the MCP server status is **running**
![mcp-server-running](images/17-mcp-server-running.png)
* Start a new chat, enable the MCP server, and send test prompts:
  * `What is the latest draft invoice?` -> should provide the invoice details, including total and due date
  * `Apply a 10% discount.` -> should update the invoice accordingly
  * `Send it.` -> should mark the invoice as sent
