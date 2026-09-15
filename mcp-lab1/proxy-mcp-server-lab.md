# Proxy MCP Server

Expose a remote MCP server as a secure proxy with Amplify Fusion / AI Gateway, then test it in Claude Desktop.

## Requirements

* Access to Amplify Fusion: [https://emea-techlab.sandbox.fusion.services.axway.com](https://emea-techlab.sandbox.fusion.services.axway.com)
  ![connection](images/01-connection.png)
* URL of a remote MCP server to proxy ([https://mcpdemo.tools](https://mcpdemo.tools) provides an online MCP server, free and handy for test purposes)
* [Claude Desktop](https://claude.com/download) installed, with a free Claude account or higher
* [Node.js](https://nodejs.org/) installed on the same machine

## Steps

### 1. Create an MCP proxy

* Open your Amplify Fusion project in the **Designer** module
![open-project](images/02-open-project.png)
* Add a new MCP server
![add-mcp-server](images/03-add-mcp-server.png)
  * Creation Method: **External MCP Server**
  * Server Name: `Order Proxy MCP Server`
  * Fusion MCP Connection: choose **Create new connection**
  ![create-mcp-server](images/04-create-mcp-server-form.png)
    * Artifact Name: `Order Remote MCP Server`
    ![create-mcp-client](images/05-create-mcp-client.png)
    * Service Root URL: `https://mcpdemo.tools/mcp`
    * Click **Update** to save the connection
  ![mcp-client-properties](images/06-mcp-client-properties.png)
  * Click **Resume MCP Proxy Creation** to return to the MCP server creation form
  * Make sure **Order Remote MCP Server** is selected in the **Fusion MCP Connection** field
  * Click **Create**
  ![validate-mcp-server-creation](images/07-validate-mcp-server-creation.png)
* Review the MCP configuration and tools
* Add a unique frontend base path (for example, use your username)
![mcp-server-config](images/08-mcp-server-config.png)
* Click **Save**

### 2. Activate the MCP proxy

* Activate the MCP server on the Fusion data plane with the ![▶](images/09-activate-button.png) button
![activate-mcp-server](images/09a-activate-mcp-server.png)
* Copy the MCP proxy URL
![activate-mcp-server](images/09b-mcp-server-url.png)

### 3. Test from Claude Desktop

* Open Claude Desktop config (File > Settings > Developers > Edit Config)
![claude-config](images/10-claude-config.png)
* Add your MCP proxy to **claude_desktop_config.json**:
  ```json
  {
    "mcpServers": {
      "Order Proxy MCP Server": {
        "command": "npx",
        "args": ["mcp-remote", "<your MCP proxy URL>"]
      }
    },
    "preferences": { ... }
  }
  ```
* Save the config file
* Restart Claude Desktop (File > Exit, then reopen the app)
* Verify that the MCP server status is **running**
![claude-config-status](images/11-claude-config-status.png)
* Start a new chat, enable the MCP proxy, and send test prompts:
  * `Where is order 123?` -> should provide the order status
  * `Cancel it.` -> should cancel the order


### 4. (Optional) Choose which MCP server tools to proxy

* Deactivate the MCP server
* Unselect the **cancel_my_order** and **check_product_availability** tools
![disable-tools](images/12-disable-tools.png)
* Reactivate the MCP server
* Reload Claude Desktop (View > Reload)
* Start a new chat, enable the MCP proxy, and send test prompts:
  * `Where is order 456?` -> should provide the order status
  * `Cancel it.` -> should not work, because the tool is no longer exposed


### 5. (Optional) Add security

* Deactivate the MCP server
* Change the version to `2.0.0`
![mcp-server-version](images/13-mcp-server-version.png)
* Open the **Security** tab
* Choose **API key** or **OAuth 2.0 (JWT Validation)** for inbound security
* Select the corresponding governance rule
* Click **Save**
![mcp-server-security](images/14-mcp-server-security.png)
* Reactivate the MCP server
* Open the **Applications** menu in the **Manager** module
![applications](images/15-applications.png)
* Edit the client application **claude-desktop**
* In the MCP Server tab, add your Proxy MCP Server
![application-mcp-access](images/16-application-mcp-access.png)
* If you chose **API Key** security, copy the existing key in the **API Key** tab.
![application-api-key](images/17-application-api-key.png)
* If you chose **OAuth**, copy the **client ID** declared in **OAuth 2.0 Credentials** and the **client secret** temporarily shown in the description (the secret is normally only visible from the Identity Provider side)
![application-oauth-creds](images/18-application-oauth-creds.png)
* Update the Claude Desktop config accordingly:

  If you use an API key, add it as a header:
  ```json
  {
    "mcpServers": {
      "Order Proxy MCP Server": {
        "command": "npx",
        "args": [ "mcp-remote", "<your MCP proxy URL>", "--header", "apikey:<your api key>" ]
      }
    },
    "preferences": { ... }
  }
  ```

  If you use OAuth 2, switch to `mcp-remote-static` and add the client information provided:
  ```json
  {
    "mcpServers": {
      "Order Proxy MCP Server": {
          "command": "C:\\PROGRA~1\\nodejs\\npx.cmd",
          "args": [ "mcp-remote-static", "<your MCP proxy URL>", "--static-oauth-client-info", "{\"client_id\":\"<your client id>\",\"client_secret\":\"<your client secret>\"}" ]
        }
    },
    "preferences": { ... }
  }
  ```
* Save the config file
* Restart Claude Desktop (File > Exit, then reopen the app)
* Verify that the MCP server status is **running**
  > A sign-in with user credentials may be required in case of OAuth - use the same credentials as for Fusion)![oauth-user-authentication](images/19-oauth-user-authentication.png)
