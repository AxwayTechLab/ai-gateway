# Serveur MCP Fusion

Créez un serveur MCP avec un tool personnalisé reposant sur une connexion OpenAPI dans Amplify Fusion / AI Gateway, puis testez-le dans Claude Desktop.

## Prérequis

* Accès à Amplify Fusion (https://fr-techlab.sandbox.fusion.services.axway.com) et à un projet avec un Server MCP en cours de création
![connection](images/01-connection.png)
* Détails de connexion à une Open API de gestions de factures
* Claude Desktop installé (nécessite Node.js), avec un compte Claude gratuit ou supérieur

## Étapes

### 1. Mettre à jour le serveur MCP

* Ouvrez votre projet Amplify Fusion.
![open-project](images/02-open-project.png)
* Ouvrez le serveur MCP existant `Invoice Fusion MCP Server`.
![mcp-server-open](images/03-mcp-server-open.png)
* Modifiez la configuration du serveur MCP et définissez un frontend base path unique (utilisez par exemple votre nom d'utilisateur).
* Cliquez sur **Save**.
![mcp-server-config](images/04-mcp-server-config.png)

### 2. Ajouter un tool

* Cliquez sur **Add Tool**.
* Donnez-lui le nom `applyDiscount`.
* Donnez-lui le titre `Apply Discount to Invoice`.
* Saisissez une description précise : `Apply a discount to a specified invoice. Discounts can only be applied to draft invoices.`
* Définissez le schéma JSON d'entrée, par exemple :
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

### 3. Ajouter une intégration au tool

* Cliquez sur l'icône de lien pour ajouter une intégration.
![link-integration](images/06-link-integration.png)
* Choisissez **Create a new integration** et nommez-la `mcpTool_applyDiscount`.
![create-new-integration](images/07-create-new-integration.png)
* Agrandir le composant **Tools** dans le flux d'intégration.
* Ajoutez un appel API :
  * Cliquez le bouton **+** et ajoutez un composant de type **OpenAPI client - Invoke operation**
  ![add-api-invoke-step](images/08-add-api-invoke-step.png)
  * Agrandir le panneau inférieur.
  * Sélectionnez la connexion existante **Zoho Invoice API** dans le panneau central.
  * Sélectionnez l'objet **Invoice**.
  * Sélectionnez l'action **UpdateInvoice** (l'opération d'API).
  ![set-api-connection](images/09-set-api-connection.png)
  * Dans l'arborescence des variables d'entrée, à gauche, localisez **applyDiscountMCPServerRequest/body/discount** et **applyDiscountMCPServerRequest/body/invoice_id**.
  * Mappez **applyDiscountMCPServerRequest/body/discount** vers **body/discount** en tirant une ligne de gauche à droite entre ces variables
  ![map-api-request-body](images/10-map-api-request-body.png)
  * Mappez **applyDiscountMCPServerRequest/body/invoice_id** vers **pathParams/Invoice_id** (Attention: ne pas confondre avec **body/invoice_id**).
  ![map-api-request-pathparam](images/11-map-api-request-pathparam.png)
  * Dans l'arborescence des variables de sortie, à droite, localisez **applyDiscountMCPServerResponse/TextContent/result/content/text**.
  * Mappez **UpdateInvoiceOutput/response/message** vers **applyDiscountMCPServerResponse/TextContent/result/content/text** à droite.
  * Mappez **UpdateInvoiceOutput/errors/message** vers le même **applyDiscountMCPServerResponse/TextContent/result/content/text** à droite.
  * Faites un clic droit sur **toolResponseType** à droite et définissez sa valeur (**Set Value**) à `TextContent`.
  * Enregistrez la configuration du composant.
  ![map-response](images/12-map-response.png)

### 4. Vérifier la connection back-end

* Cliquez sur **Navigate** à côté de la connexion API.
* Faites défiler les propriétés de connexion de l'API backend et cliquez sur **Test**.
![test-connection](images/13-test-connection.png)
* Si une coche verte s'affiche, continuez avec l'étape 5.
* En cas de croix rouge, essayez **Generate token** juste au-dessus et fournissez les identifiants utilisateur Zoho fournis.
  ![generate-token](images/14-generate-token.png)
  * Si une coche verte s'affiche, cliquez sur **Test** à nouveau. Si une coche verte s'affiche, continuez avec l'étape 5.
  * Dans les autres cas, vérifiez les détails de connexion et la configuration du côté Zoho Invoice.


### 5. Activer le serveur MCP

* Revenez à votre serveur MCP et activez-le.
* Copiez l'URL du serveur MCP.
![activate-mcp-server](images/15-activate-mcp-server.png)

### 6. Tester depuis Claude Desktop

* Ouvrez la configuration de Claude Desktop (File > Settings > Developers > Edit Config).
![claude-desktop-config](images/16-claude-desktop-config.png)
* Ajoutez votre serveur MCP dans `claude_desktop_config.json` :
  ```json
  {
    "mcpServers": {
      "Invoice Fusion MCP Server": {
        "command": "npx",
        "args": ["mcp-remote", "<votre URL de serveur MCP>"]
      }
    }
  }
  ```
* Enregistrez le fichier de configuration.
* Redémarrez Claude Desktop (File > Exit, puis rouvrez l'application).
* Vérifiez que le statut du serveur MCP est `running`.
![mcp-server-running](images/17-mcp-server-running.png)
* Démarrez une nouvelle conversation, activez le serveur MCP et testez les prompts suivants :
  * `Quelle est la dernière facture à l'état brouillon?` -> doit fournir les détails de la facture, y compris le total et la date d'échéance.
  * `Appliquer une remise de 10%.` -> doit mettre à jour la facture en conséquence.
  * `L'envoyer.` -> doit marquer la facture comme envoyée.

