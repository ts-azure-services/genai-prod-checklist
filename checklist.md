# Azure OpenAI / GenAI Production Checklist
The goal of these section are to share "evolving" best practices for productionalizing LLM applications with Azure OpenAI. Some
items are prescriptive in nature while others are more open-ended. These should be adapted to your unique situation. Additionally,
many of these aspects could apply at a specific use case or workload level, while others could apply more generically (for
example, at a platform level).

## Security & Networking
- Ensure all Azure OpenAI resources are secured behind a private endpoint and connected to a primary VNET. Leverage the [baseline E2E chat reference architecture](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/architecture/baseline-openai-e2e-chat) for a deeper
  understanding of mock setups.
- If using Azure API Management, ensure all traffic is configured to route from relevant routes (e.g., Azure Front Door).

## Identity
- Authentication:
  - Ensure all authentication is done through MSFT Entra (formerly, Azure AD).
  - As much as possible, restrict access to the OpenAI instance through API keys.
    - If using keys, consider using Azure Key Vault and routine re-generation of keys as security practices.
  - For isolation of certain workflows to various users or groups, leverage MSFT Entra at both the front-end and through the various
    workflows to ensure isolation to user groups. Certain services also integrate well with MSFT Entra to provide user isolation for
    various features. (For example, you have [security filters for trimming Azure AI Search results](https://learn.microsoft.com/en-us/azure/search/search-security-trimming-for-azure-search-with-aad) to provide results
    specific to certain users/groups.)
- Authorization:
  - Ensure the appropriate authorization is in place depending upon the right Azure RBAC defined [here](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/role-based-access-control#summary). (Note: `Owner` and `Contributor` are both Azure Resource Manager roles so they only cover control plane permissions.

## Use Cases
- For logging and analytical purposes, having distinct service principals aligned to specific use cases or workloads is
  recommended. This can also help with specific application throttling logic implemented in custom APIM policies to prioritize
  some workloads over others.
- As a part of ongoing monitoring, periodically review the OAI resource usage across various use cases and workloads. Work with
  the right stakeholders to evaluate priorities of various workloads and if revisions need to be made in the setups to reflect
  these priorities (e.g. implementing application throttling for requests with Azure API Management).
- Building custom chatbots:
  - "Pre-caching semantically relevant results". In many use cases, a chatbot will be used in connection with a specific knowledge base. This offers an opportunity to anticipate questions
  that a user may pose and pre-cache those results for a faster response. Implementing strategies to look for semantically similar
  questions first before querying an LLM offer performance benefits. A key consideration is ensuring the pre-cached answers do not
  turn stale. 
  - "Building questions off a knowledge base". Another strategy could be to build questions using an LLM off this knowledge base.

## Data Stores
- It is fairly common for a RAG pattern to implement Azure AI Search as the vector store, and include other data stores like
  Azure CosmosDB, or Azure Redis to aid other functions, e.g. logging of transactions, semantic caching of questions, etc.. With the expanded
  capability of various data stores to do vector search retrieval, many of these data stores could be good substitutes for the
  primary data store.

## Logging
- Enable diagnostic logs at the OAI resource level.
- Enable diagnostic logs for Azure APIM (if implemented). With APIM, there is an opportunity to define more of what does not
  natively get tracked through the OAI resource (for ex, prompts, completions, token costs, etc.).
- Enable function app logs.
- LLM Observability:
  - A reasonable goal should be to be able to see how a single request works its way through the system so you can diagnose production
    performance for performance regressions while also respond to feedback around model responses (e.g. accuracy, lack of
    robustness, etc.)
- If streaming mode is enabled, there may be some constraints in implementing consistent logging. Evaluate this on a case by case
  basis.

## Managing OAI Capacity
- Leverage Azure APIM in a secured manner to be the entry point to multiple OAI backends/deployments.
- For multiple regional deployments, you need only maintain one OAI resource per region, which in turn can connect to multiple
  deployments, each containing different models.
- If your OAI backend capacity is split between PTUs and PAYGO capacities, consider using custom APIM policies to handle how
  traffic will be directed between various backends:
  - A popular technique laid out <here> it to maximize the utilization of PTUs and allow overflowing traffic to be sent to PAYGO.
    Note that this is within the same model (e.g. gpt-4, version 0613).
  - Many are also interested in a "hybrid model" pattern where OAI backends are separated by different model families, and a
    single user query may interface with multiple models before returning a request.
  - When selecting regions, ensure there are supporting model families available (e.g. text-embedding-ada-002) if the use case
    requires it, including if PAYGO capacity is also needed to buffer a PTU capacity.
- Leverage Azure APIM to provide the following benefits:
  - Load balancing: to help with request routing. A sample repo [here](https://github.com/Azure/aoai-apim).
  - Restrict all authentication through MSFT Entra. (This can limit access given data plane keys cannot be restricted.)
  - Log appropriate detail on the request and response.
  - Application throttling. Depending upon the user group or use case, implement logic to determine application priorities.
  - Chargebacks: Identify usage by use case, department or however access is organized. Sample repo [here](https://github.com/Azure-Samples/private-openai-with-apim-for-chargeback).

## LLM Development Practices
< Pending:
- LLMOps, use of Azure AI Studio
- LLM Observability:
evaluation: how should I evaluate?
- Certain use cases may opt for the use of more native functionality specific to Azure OpenAI, e.g. the Assistants API. Using this, you
  can leverage a sandbox to run code and maintain stateful applications. Where appropriate, these should be implemented
  considering stakeholder timelines and requirements.
>

## Application Layer
<Pending: 
- Frameworks like Langchain and Semantic Kernel
- Bot framework vs. function apps.
- Add the link on how to choose various compute options.
- [GPT-RAG implementation](https://github.com/Azure/gpt-rag)
>

## Software Engineering Practices
- For prototyping use cases, leverage [Azure AI Studio](https://ai.azure.com/) to both test models and iterate through development
  workflows using [Prompt Flow](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/prompt-flow). Once a use case has gained
  approval by relevant stakeholders, leverage the above practices to bring it to production.
- Dev/Prod environments:
  - Consider maintaining resources separately for a dev and a production environment (separate subscription or resource group). 
  - The dev subscription should mimic the production environment closely and allow for testing of new functionality and workflows.
  - This may require good naming conventions to identify resources tied to dev vs. production environments.

## Resources
- Azure [WAF perspective](https://learn.microsoft.com/en-us/azure/well-architected/service-guides/azure-openai) on Azure OpenAI.

### Pending for consideration
- Load testing or performance testing for applications
