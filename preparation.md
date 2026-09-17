# TechLab - AI Gateway
Prepare client desktop:
1. [Firefox](https://download.mozilla.org/?product=firefox-stub&os=win&lang=en-US)
    * Homepage = [fr](fr.md)
2. [Claude Desktop](https://claude.com/download) 
    * Create account: `axwaytechlabs+userX@gmail.com`
3. [Node.js](https://nodejs.org/dist/v24.21.0/node-v24.21.0-x64.msi) 
    * Test `npx --version`
4. [Notepad++](https://github.com/notepad-plus-plus/notepad-plus-plus/releases/download/v8.9.8/npp.8.9.8.portable.x64.zip)
    * Associate json file
5. Test connect to [Amplify](https://fr-techlab.sandbox.fusion.services.axway.com) Username: `techlabs+userX@axway.com`
    * Disable password save
    * Disable page translation
5. Test MCP in Claude config
    ```json
    {
        "mcpServers": {
            "Order Proxy MCP Server": {
                "command": "npx",
                "args": ["mcp-remote", "https://demo-design.sandbox.fusion.services.axway.com:4443/mcpdemo"]
            }
         },
        "preferences": {  }
    }
    ```
