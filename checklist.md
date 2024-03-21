# Azure OpenAI / GenAI Production Checklist

## Authentication
- Ensure all authentication is done through MSFT Entra (formerly, Azure AD).
- As much as possible, restrict access to the OpenAI instance through API keys.
  - If using keys, consider using Azure Key Vault and routine re-generation of keys as security practices.

## Authorization
- Ensure the appropriate authorization is in place depending upon the right [Azure RBAC](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/role-based-access-control#summary).
- Note that `Owner` and `Contributor` are both Azure Resource Manager roles. They only cover control plane permissions.

## Security and Networking
- Ensure all Azure OpenAI resources are behind a private endpoint.
- If using APIM, ensure all traffic is configured to 
  
- add the detail about Front Door.


## Logging
- Enable diagnostic logs at the OAI resource level.
- Enable diagnostic logs for Azure APIM (if implemented). With APIM, there is an opportunity to define more of what does not
  natively get tracked through the OAI resource (for ex, prompts, completions, token costs, etc.).
- Enable function app logs.
- LLM Observability:
  - The goal should be to be able to see how a single request works its way through the system so you can diagnose production
    performance for performance regressions while also respond to feedback around model responses (e.g. accuracy, lack of
    robustness, etc.)

## Managing OAI Capacity
- Leverage Azure APIM in a secured manner to be the entry point to multiple OAI backends/deployments.
- For multiple regional deployments, you need only maintain one OAI resource per region, which in turn can connect to multiple
  deployments, each containing different models.
- If your OAI backend capacity is split between PTUs and PAYGO capacities, consider using techniques to prioritize 
- Leverage Azure APIM to provide the following benefits:
  - Load balancing: to help with request routing. A sample repo [here](https://github.com/Azure/aoai-apim).
  - Restrict all authentication through MSFT Entra. (This can limit access given data plane keys cannot be restricted.)
  - Log appropriate detail on the request and response.
  - Application throttling. Depending upon the user group or use case, implement logic to determine application priorities.
  - Chargebacks: Identify usage by use case, department or however access is organized. Sample repo [here](https://github.com/Azure-Samples/private-openai-with-apim-for-chargeback).
- For 

## LLM Development Practices
- LLMOps
- 


## Software Engineering Practices
- Dev/Prod subscription:
  - Consider maintaining resources separately for a dev and a production environment (separate subscription or resource group). 
  - The dev subscription will mimic the production environment, and allow for testing of new functionality and workflows.
  - This may require good naming conventions to identify resources tied to dev vs. production environments.
- 

## Resources
- Azure [WAF perspective](https://learn.microsoft.com/en-us/azure/well-architected/service-guides/azure-openai) on Azure OpenAI:

- App Layer: 
  
- Relevant frameworks:

- Implementation of 
- LLM Ops
  - Use of Azure AI Studio to 
    
- For use cases, define specific service principals to log behavior.
- Identify high priority use cases vs. others.
