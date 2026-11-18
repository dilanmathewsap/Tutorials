---
parser: v2
auto_validation: true
time: 50
primary_tag: software-product>sap-ai-core
tags: [ tutorial>beginner, topic>artificial-intelligence, topic>machine-learning, software-product>sap-ai-core ]
author_name: Dilan
author_profile: https://github.com/
---

# Leveraging Orchestration (V2) Capabilities to Enhance Responses — with Claude Code and AI Core Skills
<!-- description --> In this tutorial, we explore optional Orchestration Service V2 capabilities available in the Generative AI Hub — Data Masking, Translation, and Content Filtering — using a sentiment analysis use case. Alongside the official AI Launchpad, Python SDK, JavaScript SDK, and Bruno paths, this version adds a **Claude Code** path: driving the same V2 modules through natural-language instructions to the `genai-hub-orchestration` skill from the SAP AI Skills Library, installed via `npx skills add`.

> **⚠️ SDK Version Note**: The code in this tutorial was validated against a specific installed version of `generative-ai-hub-sdk`. In particular, `LLMModelDetails` takes the keyword argument **`params`**, not `parameters`, in the version tested here — even though some published documentation uses `parameters`. If any class in this tutorial throws a `pydantic.ValidationError` with `extra_forbidden`, run `help(<ClassName>)` or `inspect.signature(<ClassName>.__init__)` on your installed version before trusting this document's exact keyword names. See the **Troubleshooting** section at the end for the verification command.

## You will learn
- Inference of GenAI models using Orchestration Service V2 along with Data Masking, Translation, and Content Filtering features.
- How to configure each module via AI Launchpad, Python SDK, JavaScript SDK, and Bruno.
- How to drive the same V2 modules through Claude Code using the `genai-hub-orchestration` skill, without hand-writing SDK calls.
- How to verify SDK class signatures directly when installed package versions drift from published documentation.

## Prerequisites
1. **BTP Account**
   Set up your SAP Business Technology Platform (BTP) account.
   [Create a BTP Account](https://developers.sap.com/group.btp-setup.html)
2. **For SAP Developers or Employees**
   Internal SAP stakeholders should refer to the following documentation: [How to create BTP Account For Internal SAP Employee](https://me.sap.com/notes/3493139), [SAP AI Core Internal Documentation](https://help.sap.com/docs/sap-ai-core)
3. **For External Developers, Customers, or Partners**
   Follow this tutorial to set up your environment and entitlements: [External Developer Setup Tutorial](https://developers.sap.com/tutorials/btp-cockpit-entitlements.html), [SAP AI Core External Documentation](https://help.sap.com/docs/sap-ai-core?version=CLOUD)
4. **Create BTP Instance and Service Key for SAP AI Core**
   Follow the steps to create an instance and generate a service key for SAP AI Core and ensure to use service plan **extended**:
   [Create Service Key and Instance](https://help.sap.com/docs/sap-ai-core/sap-ai-core-service-guide/create-service-key?version=CLOUD)
5. **AI Core Setup Guide**
   Step-by-step guide to set up and get started with SAP AI Core:
   [AI Core Setup Tutorial](https://developers.sap.com/tutorials/ai-core-genaihub-provisioning.html)
6. An **Extended** SAP AI Core service plan is required, as the Generative AI Hub is not available in the Free or Standard tiers. For more details, refer to
   [SAP AI Core Service Plans](https://help.sap.com/docs/sap-ai-core/sap-ai-core-service-guide/service-plans?version=CLOUD)
7. **AI Launchpad Setup Guide**
   Step-by-step guide to set up AI Launchpad:
   [AI Launchpad Tutorial](https://developers.sap.com/tutorials/ai-launchpad-provisioning.html)
8. **Orchestration Deployment**:
   Refer to the tutorial [the basic consumption of GenAI models using orchestration](https://developers.sap.com/tutorials/ai-core-orchestration-consumption.html) and ensure at least one orchestration deployment is ready to be consumed during this process.
9. **For the Claude Code path specifically**:
   - **Claude Code** installed and signed in, with Node.js and `npx` available.
   - The public SAP AI Skills Library installed using `npx skills add SAP/ai-skills-library/plugins/aicore-skills`.
   - SAP AI Core credentials configured locally for skill execution, using `aicore configure` or the supported environment-variable method. Verify access with the `aicore-admin-resources` skill. Claude Code's own sign-in/model configuration is separate from SAP AI Core credentials.
10. Basic Knowledge:
    Familiarity with the orchestration workflow is recommended.


### Pre-Read

This tutorial builds on the foundational orchestration concepts introduced in the [beginner's tutorial](https://developers.sap.com/tutorials/ai-core-orchestration-consumption.html) and focuses on enhancing GenAI responses using orchestration service V2 modules: **Data Masking**, **Translation**, and **Content Filtering**.

Previously, the beginner's tutorial used a **resume processing use case** to illustrate the harmonized API. In this tutorial, we use a **sentiment analysis use case** to demonstrate how these optional V2 modules can be applied to protect sensitive information, translate multilingual support requests, and filter out undesirable or non-compliant content — enhancing the quality, safety, and compliance of generative AI outputs.

**Data masking** anonymizes or pseudonymizes personal or confidential data before it's sent to the model.
🔗 [Learn more about Data Masking in SAP AI Core](https://help.sap.com/docs/sap-ai-core/sap-ai-core-service-guide/data-masking?version=CLOUD)

**Translation** enables automatic language conversion of inputs and outputs during LLM processing.
🔗 [Learn more about Translation in SAP AI Core](https://help.sap.com/docs/sap-ai-core/sap-ai-core-service-guide/input-translation?version=CLOUD)

**Content filtering** identifies and blocks inappropriate, offensive, or non-compliant input and output content within an orchestration workflow.
🔗 [Learn more about Content Filtering in SAP AI Core](https://help.sap.com/docs/sap-ai-core/sap-ai-core-service-guide/input-filtering?version=CLOUD)

Other orchestration modules such as **grounding** are also available in SAP AI Core and are covered in separate tutorials.

You will learn how to:
- Integrate data masking within the orchestration flow to safeguard personal or confidential information.
- Apply content filtering to identify and restrict inappropriate or non-compliant responses.
- Apply translation for multilingual input/output handling.
- Use relevant SAP AI Core features and configurations to support these capabilities — via AI Launchpad, Python SDK, JavaScript SDK, Bruno, and Claude Code.

By the end of this tutorial:
- You'll understand how to design a secure and controlled orchestration pipeline suitable for **enterprise-grade GenAI applications**.
- You'll be able to implement the solution using **SAP AI Launchpad**, **Python SDK**, **JavaScript SDK**, **Bruno**, or **Claude Code** (via the SAP AI Skills Library).
- You'll know how to resolve `ValidationError: extra_forbidden` issues caused by SDK version drift from published documentation.

### Accessing Orchestration Capabilities

We build upon the orchestration framework introduced in the [beginner's tutorial](https://developers.sap.com/tutorials/ai-core-orchestration-consumption.html). The focus shifts from basic orchestration to leveraging optional modules to enhance data privacy and refine response quality:

- **Data Masking**: Hiding sensitive information like phone numbers, organizational details, or personal identifiers.
- **Content Filtering**: Screening for hate speech, self-harm, explicit content, and violence to ensure safe and relevant responses.
- **Translation**: Automatically converts input and/or output text between source and target languages to support multilingual processing.

We use a **sentiment analysis** use case, where orchestration is enhanced by incorporating data masking, translation, and content filtering.

[OPTION BEGIN [AI Launchpad]]

**Access the Generative AI Hub:**
- Navigate to the resource group where your orchestration has been deployed.
- Go to Generative AI Hub.
- Select Orchestration and click on create.

![img](img/image_ail_orch.png)
![img](img/image_ail_orch1.png)

[OPTION END]

[OPTION BEGIN [Python SDK]]

**NOTE**: If you are continuing with the same notebook from the previous tutorial, skip steps 1 and 2. Otherwise, create a new notebook using the already deployed orchestration URL to access the Harmonized API.

- The `support-request.txt` file, containing the support request content, must be added to the working directory. Use the following code to load the file content:

```python
from gen_ai_hub.orchestration_v2.utils import load_text_file
# Load the support request file content
support_request_path = "support-request.txt"  # Specify the correct path to the file
support_request = load_text_file(support_request_path)
# Print the content to verify it has been loaded
print(support_request)
```

[OPTION END]

[OPTION BEGIN [JavaScript SDK]]

**NOTE**: If you are continuing with the same project from the previous tutorial, skip steps 1 and 2. Otherwise, create a new project using the already deployed orchestration URL to access the Harmonized API.

```javascript
const txtContent = await Deno.readTextFile('./support-request.txt');
console.log(txtContent);
```

[OPTION END]

[OPTION BEGIN [Bruno]]

If you have already completed the environment setup, configuration, and deployment as described in the [beginner's tutorial](https://developers.sap.com/tutorials/ai-core-orchestration-consumption.html), you can directly proceed to the Data Masking Configuration. Otherwise, follow that tutorial first.

[OPTION END]

[OPTION BEGIN [Claude Code]]

Claude Code's own sign-in and model configuration are separate from SAP AI Core credentials. In this tutorial, Claude Code uses the `aicore-skills` to interact with SAP AI Core, while the local AI Core CLI configuration supplies the credentials used by those skills and any generated SDK code.

| Layer | What it authenticates | Where it is configured |
|---|---|---|
| **Claude Code** | The coding agent that reads your instructions and generates or runs code | Claude Code's own sign-in and settings |
| **SAP AI Core credentials** | The SAP AI Core APIs and SDK operations performed by the skills or generated code | `~/.aicore/config.json`, created by `aicore configure`, or supported environment variables |

**Step 1 — Get a service key for SAP AI Core**

In the [BTP Cockpit](https://developers.sap.com/tutorials/btp-cockpit-entitlements.html), open **Instances and Subscriptions**, select your SAP AI Core instance, and open an existing service key or create one. Keep the service-key JSON available locally. You will need the authentication URL, client ID, client secret, and AI Core API URL. Use the resource group that contains the orchestration deployment; do not assume that its name is `default`.

**Step 2 — Install the SAP AI Core skills**

Open a terminal with Node.js and `npx` available, then install the public SAP AI Skills Library package:

```bash
npx skills add SAP/ai-skills-library/plugins/aicore-skills
```

The package provides skills including `aicore-admin-resources`, `aicore-lifecycle-management`, and `genai-hub-orchestration`. Installing the skills makes their instructions available to Claude Code; it does not authenticate to AI Core or grant resource-group permissions.

**Step 3 — Configure the local AI Core credentials**

Run the AI Core CLI configuration command:

```bash
aicore configure
```

The configuration is stored in:

```text
~/.aicore/config.json
```

If `aicore` is not found, locate the CLI executable and run it using its full path. If the CLI prompts for X.509 certificate and key files but you intend to use client-secret authentication, do not enter certificate paths. Check the options supported by your installed CLI with:

```bash
aicore configure --help
```

Alternatively, set the supported connection values in the terminal before starting Claude Code:

```bash
export AICORE_AUTH_URL="<your-auth-url>"
export AICORE_CLIENT_ID="<your-client-id>"
export AICORE_CLIENT_SECRET="<your-client-secret>"
export AICORE_BASE_URL="<your-ai-core-api-url>"
```

Use the values from your service key. Keep secrets out of prompts, screenshots, notebooks, and committed files. Environment variables set this way apply to processes launched from that terminal session.

**Step 4 — Verify AI Core access**

Start or restart Claude Code after installing the skills and configuring credentials. Run:


> `/aicore-admin-resources "Validate my current AI Core connection and list the resource groups I can access."`

![img](img/admin1.png)

A successful response confirms that the skill can access the AI Core resources available to those credentials. If authentication succeeds but the expected deployment is not visible, check the selected resource group and the service key's permissions.

You are now ready to follow the **Claude Code** path through the rest of this tutorial.

Use the lifecycle skill to identify the existing orchestration deployment:


> `/aicore-lifecycle-management "List the existing deployments in my active resource group. Identify the orchestration deployment and report its deployment ID, configuration ID, status, and deployment URL."`

![img](img/admin2.png)

Note the orchestration deployment URL from the output — the `genai-hub-orchestration` skill will use it for the steps below. Place the `support-request.txt` file (containing the German support message used later in this tutorial) in your project root so Claude Code can reference it.

[OPTION END]

### Template Configuration

The templating module is a mandatory step in orchestration. It defines dynamic inputs using placeholders, constructs structured prompts, and generates the final query passed to the model configuration module. We create a template with two message components:

- `system`: Defines assistant behavior and task.
- `user`: Provides the support request input.

[OPTION BEGIN [AI Launchpad]]

- In the **Templating** section, locate the message icon with three tabs: **User, Assistant, System**.
- Click the **User** tab and enter:

```PROMPT
Please analyze the sentiment of the following support request: {{ ?support_text }}
```

- The variable `support_text` will be created. Enter default values based on your use case:

```TEXT
"Subject: Bestellung #1234567890 Verspätet - John Johnson Nachricht: Halle, ich schreibe ihnen um mich nach dem Status meiner Bestellung mit der Bestellnr. +1234567890 zu erkundigen. Die Lieferung war eigentlich für gestern geplant, ist bisher jedoch nicht erfolgt. Mein Name ist John Johnson und meine Lieferadresse lautet 125 Cole Meadows Drive Palo Alto, California 94301. Bitte lassen Sie mich per Telefon unter der Nummer +1 505802 2172 wissen, wann ich mit meiner Lieferung rechnen kann. Danke!"
```

![img](img/image_ail_019.png)

- Click **Add**. Configure the **System** tab:

```PROMPT
You are a customer support assistant. Analyze the sentiment of the user request provided and return whether the sentiment is positive, neutral, or negative. Also provide a one-line justification for your classification.
```

![img](img/image005.png)

[OPTION END]

[OPTION BEGIN [Python SDK]]

```python
from gen_ai_hub.orchestration_v2.models.message import SystemMessage, UserMessage
from gen_ai_hub.orchestration_v2.models.template import Template

template = Template(
    template=[
        SystemMessage(content="""You are a customer support assistant. Analyze the sentiment of the user request provided and return whether the sentiment is positive, neutral, or negative. Also provide a one-line justification for your classification."""
        ),
        UserMessage(content="Please analyze the sentiment of the following support request: {{ ?support_text }}"
        ),
    ],
    defaults=
        {"support_text":"User is unhappy with the latest update and facing usability issues."}
)
```

Select the model to be used for this orchestration:

```python
from gen_ai_hub.orchestration_v2.models.llm_model_details import LLMModelDetails

# NOTE: verified against installed SDK via help(LLMModelDetails) — the
# keyword is "params", not "parameters". If yours differs, run:
#   python -c "from gen_ai_hub.orchestration_v2.models.llm_model_details import LLMModelDetails; help(LLMModelDetails)"
llm = LLMModelDetails(name="gpt-5-nano", params={"max_completion_tokens": 1028})
```

[OPTION END]

[OPTION BEGIN [JavaScript SDK]]

```javascript
import { OrchestrationClient } from '@sap-ai-sdk/orchestration';

const orchestrationClient = new OrchestrationClient({
  promptTemplating: {
    model: {
      name: 'gpt-4o',
      params: { max_completion_tokens: 200, temperature: 0 }
    },
    prompt: {
      template: [
        { role: 'system', content: 'You are a customer support assistant. Analyze the sentiment of the user request provided and return whether the sentiment is positive, neutral, or negative. Also provide a one-line justification.' },
        { role: 'user', content: 'Please analyze the sentiment of the following support request: {{ ?support_text }}' }
      ]
    }
  }
});

const response = await orchestrationClient.chatCompletion({
  placeholderValues: {
    support_text: 'User is unhappy with the latest update and facing usability issues.'
  }
});
```

[OPTION END]

[OPTION BEGIN [Bruno]]

Continue in the **05_orchestration** collection — the template is set via the request body's `prompt_templating` section, shown combined with the other modules at the end of this tutorial.

[OPTION END]

[OPTION BEGIN [Claude Code]]

> `/genai-hub-orchestration "Create a V2 sentiment-analysis orchestration using gpt-5-nano (max_completion_tokens=1028). System: Classify support-request sentiment as positive, neutral, or negative, with a one-line justification. User template: 'Analyze this support request: {{ ?support_text }}'. Inspect the installed SDK's LLMModelDetails signature first and use the correct argument name (params or parameters)."`

![img](img/admin4.png)

Claude Code should generate the equivalent `Template` / `SystemMessage` / `UserMessage` / `LLMModelDetails` Python code shown in the Python SDK option above, since both paths use the same `gen_ai_hub.orchestration_v2` package underneath.

[OPTION END]

### Setting Up Data Masking Parameters

The **Data Masking** module ensures data privacy by anonymizing or pseudonymizing sensitive information before it is processed:

- **Anonymization**: Irreversibly replaces personal identifiers with placeholders (e.g., `MASKED_ENTITY`).
- **Pseudonymization**: Substitutes identifiers with reversible tokens (e.g., `MASKED_ENTITY_ID`).

[OPTION BEGIN [AI Launchpad]]

- Navigate to the **Data Masking** section.
- Choose **Anonymize** for enhanced privacy.
- Check the boxes for: Email Address, Organization Name, Person's Name, Person's Phone Number, Username & Password.

![img](img/image009.png)

[OPTION END]

[OPTION BEGIN [Python SDK]]

```python
from gen_ai_hub.orchestration_v2.models.data_masking import MaskingModuleConfig, MaskingProviderConfig, MaskingMethod, DPIStandardEntity, ProfileEntity

data_masking_config = MaskingModuleConfig(
    masking_providers=[MaskingProviderConfig(
        method=MaskingMethod.ANONYMIZATION,
        entities=[
            DPIStandardEntity(type=ProfileEntity.ADDRESS),
            DPIStandardEntity(type=ProfileEntity.EMAIL),
            DPIStandardEntity(type=ProfileEntity.PHONE),
            DPIStandardEntity(type=ProfileEntity.PERSON),
        ]
    )],
)
```

**NOTE**: We anonymize name, phone number, address (location), and email to protect user privacy in the support text. These are the verified V2 class names — `MaskingModuleConfig` / `MaskingProviderConfig` (not a merged `DataMaskingDetails`), with `method=` (not `masking_method=`).

[OPTION END]

[OPTION BEGIN [JavaScript SDK]]

```javascript
import { buildDpiMaskingProvider } from '@sap-ai-sdk/orchestration';

const maskingProvider = buildDpiMaskingProvider({
  method: 'anonymization',
  entities: [
    'profile-person',
    'profile-email',
    'profile-phone',
    {
      type: 'custom',
      regex: '\\b(TICKET|CASE)-[0-9]{4,}\\b',
      replacement_strategy: { method: 'constant', value: 'MASKED_REFERENCE_ID' }
    }
  ],
  allowlist: ['SAP']
});
```

[OPTION END]

[OPTION BEGIN [Bruno]]

```JSON
"masking": {
        "masking_providers": [
          {
            "type": "sap_data_privacy_integration",
            "method": "anonymization",
            "entities": [
              { "type": "profile-email" },
              { "type": "profile-person" },
              { "type": "profile-phone" },
              { "type": "profile-org" },
              { "type": "profile-location" }
            ]
          }
        ]
      }
```

![img](img/data_masking.png)

[OPTION END]

[OPTION BEGIN [Claude Code]]

>`/genai-hub-orchestration "Add V2 anonymization for ADDRESS, EMAIL, PHONE, and PERSON using MaskingMethod.ANONYMIZATION. Use only MaskingModuleConfig, MaskingProviderConfig, DPIStandardEntity, and ProfileEntity from gen_ai_hub.orchestration_v2.models.data_masking. Apply it to the sentiment template and test with German support-request.txt. Do not use DataMaskingDetails.""`

![img](img/admin8.png)

If a `TypeError` or `ValidationError` occurs, verify the installed constructor signature rather than guessing:

```bash
python -c "from gen_ai_hub.orchestration_v2.models.data_masking import MaskingProviderConfig; import inspect; print(inspect.signature(MaskingProviderConfig.__init__))"
```

[OPTION END]

### Translation

The Translation module enables multilingual processing by translating content sent to and received from the model, using SAP's Document Translation service. Target language is mandatory; source language auto-detects if omitted.

[OPTION BEGIN [AI Launchpad]]

- Navigate to the **Translation** section.
- Specify source/target languages for Input and Output Translation:
  - Input: German ➝ English
  - Output: English ➝ German

![img](img/image004.png)
![img](img/image025.png)

[OPTION END]

[OPTION BEGIN [Python SDK]]

```python
from gen_ai_hub.orchestration_v2.models.translation import TranslationModuleConfig, SAPDocumentTranslation, TranslationConfig

translation_config = TranslationModuleConfig(
    input=SAPDocumentTranslation(
        config=TranslationConfig(source_language="de-DE", target_language="en-US")
    ),
    output=SAPDocumentTranslation(
        config=TranslationConfig(source_language="en-US", target_language="de-DE")
    )
)
```

**NOTE**: The correct structure uses `input=` / `output=` wrapping a `SAPDocumentTranslation(config=TranslationConfig(...))` object — not flat `input_translation` / `output_translation` / `provider` keys (an error hit when this was attempted directly via raw REST/curl).

[OPTION END]

[OPTION BEGIN [JavaScript SDK]]

```javascript
import { buildTranslationConfig } from '@sap-ai-sdk/orchestration';

const inputTranslation = buildTranslationConfig('input', { sourceLanguage: 'de-DE', targetLanguage: 'en-US' });
const outputTranslation = buildTranslationConfig('output', { sourceLanguage: 'en-US', targetLanguage: 'de-DE' });

const translationConfig = { input: inputTranslation, output: outputTranslation };
```

[OPTION END]

[OPTION BEGIN [Bruno]]

```JSON
"translation": {
          "input": { "type": "sap_document_translation", "config": { "source_language": "de-DE", "target_language": "en-US" } },
          "output": { "type": "sap_document_translation", "config": { "source_language": "en-US", "target_language": "de-DE" } }
  }
```

![img](img/translation.png)

[OPTION END]

[OPTION BEGIN [Claude Code]]


> `/genai-hub-orchestration "Add V2 bidirectional translation using TranslationModuleConfig, SAPDocumentTranslation, and TranslationConfig from gen_ai_hub.orchestration_v2.models.translation. Input: de-DE → en-US; output: en-US → de-DE. Use type='sap_document_translation' with nested config for both. Do not use input_translation, output_translation, or provider. Add alongside existing data masking in the sentiment pipeline."`

![img](img/admin5.png)

[OPTION END]

### Defining Content Filtering Rules

The **Content Filtering** module screens both input and output content to remove inappropriate or unwanted elements such as hate speech or violent content.

[OPTION BEGIN [AI Launchpad]]

- Navigate to **Input Filtering**. Adjust levels (Hate, Self-Harm, Sexual Content, Violence).

![img](img/image013.png)
![img](img/image026.png)

- In **Model Configuration**, select your Deployment ID and an LLM. Ensure the deployment is Running.

![img](img/image015.png)
![img](img/image024.png)

- In **Output Filtering**, adjust levels the same way.

![img](img/image019.png)

[OPTION END]

[OPTION BEGIN [Python SDK]]

```python
from gen_ai_hub.orchestration_v2.models.azure_content_filter import AzureContentFilter, AzureThreshold
from gen_ai_hub.orchestration_v2.models.llama_guard_3_filter import LlamaGuard38bFilter
from gen_ai_hub.orchestration_v2.models.content_filtering import FilteringModuleConfig, InputFiltering, OutputFiltering
from gen_ai_hub.orchestration_v2.models.content_filter import ContentFilter, ContentFilterProvider

content_filter_config = FilteringModuleConfig(
    input=InputFiltering(filters=[
        ContentFilter(type=ContentFilterProvider.AZURE, config=AzureContentFilter(hate=AzureThreshold.ALLOW_SAFE,
                                                                                  violence=AzureThreshold.ALLOW_SAFE,
                                                                                  self_harm=AzureThreshold.ALLOW_SAFE,
                                                                                  sexual=AzureThreshold.ALLOW_SAFE)),
        ContentFilter(type=ContentFilterProvider.LLAMA_GUARD_3_8B, config=LlamaGuard38bFilter(hate=True))
        ]),
    output=OutputFiltering(filters=[
        ContentFilter(type=ContentFilterProvider.AZURE, config=AzureContentFilter(hate=AzureThreshold.ALLOW_SAFE,
                                                                                  violence=AzureThreshold.ALLOW_SAFE,
                                                                                  self_harm=AzureThreshold.ALLOW_SAFE,
                                                                                  sexual=AzureThreshold.ALLOW_SAFE)),
        ContentFilter(type=ContentFilterProvider.LLAMA_GUARD_3_8B, config=LlamaGuard38bFilter(hate=True))
    ])
)
```

**NOTE**: Adjust thresholds for hate, sexual, self-harm, and violence categories based on your use case. If this raises a `ValidationError`, verify with `help(FilteringModuleConfig)` before changing field names.

Combine the template, model, and modules into the orchestration configuration:

```python
from gen_ai_hub.orchestration_v2.models.template import Template, PromptTemplatingModuleConfig
from gen_ai_hub.orchestration_v2.models.config import ModuleConfig, OrchestrationConfig

prompt_template = PromptTemplatingModuleConfig(prompt=template, model=llm)

module_config = ModuleConfig(prompt_templating=prompt_template, filtering=content_filter_config, masking=data_masking_config,
                            translation=translation_config)

config = OrchestrationConfig(modules=module_config)
```

**NOTE**: Ensure that your orchestration deployment is in Running status and ready to be consumed during this process.

[OPTION END]

[OPTION BEGIN [JavaScript SDK]]

```javascript
import { buildAzureContentSafetyFilter } from '@sap-ai-sdk/orchestration';

const inputFilter = buildAzureContentSafetyFilter('input', {
  hate: 'ALLOW_SAFE_LOW', self_harm: 'ALLOW_SAFE_LOW', sexual: 'ALLOW_SAFE_LOW', violence: 'ALLOW_SAFE_LOW', prompt_shield: true
});

const outputFilter = buildAzureContentSafetyFilter('output', {
  hate: 'ALLOW_SAFE', self_harm: 'ALLOW_SAFE', sexual: 'ALLOW_SAFE', violence: 'ALLOW_SAFE'
});
```

```javascript
import { OrchestrationClient } from '@sap-ai-sdk/orchestration';

const orchestrationClient = new OrchestrationClient({
  resourceGroup: 'grounding',
  promptTemplating: {
    model: { name: 'gpt-4o', params: { max_completion_tokens: 200, temperature: 0 } },
    prompt: {
      template: [
        { role: 'system', content: 'You are a customer support assistant. Analyze the sentiment of the user request provided and return whether the sentiment is positive, neutral, or negative. Also provide a one-line justification.' },
        { role: 'user', content: 'Please analyze the sentiment of the following support request: {{ ?support_text }}' }
      ]
    }
  },
  translation: translationConfig,
  masking: { masking_providers: [maskingProvider] },
  filtering: { input: { filters: [inputFilter] }, output: { filters: [outputFilter] } }
});
```

[OPTION END]

[OPTION BEGIN [Bruno]]

```JSON
"filtering": {
        "input": { "filters": [ { "type": "azure_content_safety", "config": { "hate": 4, "self_harm": 4, "sexual": 4, "violence": 4 } } ] },
        "output": { "filters": [ { "type": "azure_content_safety", "config": { "hate": 0, "self_harm": 0, "sexual": 0, "violence": 0 } } ] }
      }
```

![img](img/image028.png)

[OPTION END]

[OPTION BEGIN [Claude Code]]


> `/genai-hub-orchestration "Add V2 content filtering to the sentiment pipeline using FilteringModuleConfig, InputFiltering, OutputFiltering, ContentFilter, and ContentFilterProvider from gen_ai_hub.orchestration_v2.models. For both input and output, add an Azure filter (AzureContentFilter with AzureThreshold.ALLOW_SAFE for hate, violence, self_harm, and sexual) and a LlamaGuard filter (LlamaGuard38bFilter with hate=True). Do not guess class or field names"`

![img](img/admin9.png)

If a `TypeError` or `ValidationError` occurs, use this instruction:

```text
Do not guess the parameter name. Run help() or inspect.signature() on the
failing class's __init__ and show the exact expected arguments before retrying.
```

[OPTION END]

### Executing the Orchestration Workflow

[OPTION BEGIN [AI Launchpad]]

- Click the **Test** icon and run the orchestration.
- Check the **Result** section for the response.

![img](img/image023.png)

- Save the orchestration for future use.

![img](img/image_ail_sav.png)

[OPTION END]

[OPTION BEGIN [Python SDK]]

```python
from gen_ai_hub.orchestration_v2.service import OrchestrationService

orchestration_service = OrchestrationService()

result = orchestration_service.run(config=config, placeholder_values={"support_text": support_request})

response = result.final_result.choices[0].message.content
print(response)
```

![img](img/image_py_resp.png)

[OPTION END]

[OPTION BEGIN [JavaScript SDK]]

```javascript
try {
  const response = await orchestrationClient.chatCompletion({
    placeholderValues: { support_text: txtContent }
  });
  console.log(response.getContent());
} catch (error: any) {
  console.error('❌ Error during support sentiment analysis');
  console.error(error.message);
  console.error(error.cause?.response?.data);
}
```

![img](img/image_js_resp.png)
![img](img/image_js_v2.png)

[OPTION END]

[OPTION BEGIN [Bruno]]

Click **Send**. Validate the response contains masked results, translated text, and filtered content flags.

![img](img/image027.png)

[OPTION END]

[OPTION BEGIN [Claude Code]]


>`/genai-hub-orchestration "Run the combined sentiment orchestration (template + masking + translation + filtering) on support-request.txt and print result.final_result.choices[0].message.content."`

![img](img/admin6.png)

Once confirmed working, consolidate:


> `/genai-hub-orchestration "Save all Python code generated across the template, data masking, translation, content filtering, and execution steps into a single file named orchestration_v2_pipeline.py in this project."`

![img](img/admin7.png)

Run the generated file:

```bash
python orchestration_v2_pipeline.py
```

Report any friction:

```
Using the feedback-aicore-skills skill, review this session and raise a
GitHub issue noting: (1) LLMModelDetails uses "params" not "parameters",
and (2) the sap-tutorials doc for this page is out of sync with the
installed SDK version.
```

[OPTION END]

### Troubleshooting: SDK / Documentation Drift

During this tutorial, the installed `generative-ai-hub-sdk` version was found to diverge from the published tutorial text in one confirmed place:

| Class | Documented keyword | Verified keyword (via `help()`) |
|---|---|---|
| `LLMModelDetails` | `parameters` | `params` |

If any other class in this pipeline raises `TypeError: unexpected keyword argument` or `pydantic.ValidationError: extra_forbidden`, do not assume the tutorial text is correct — verify against your installed package directly:

```bash
python -c "from <module_path> import <ClassName>; help(<ClassName>)"
```
or, for just the constructor signature:
```bash
python -c "from <module_path> import <ClassName>; import inspect; print(inspect.signature(<ClassName>.__init__))"
```

Check the installed SDK version so you know what you're actually working against:
```bash
pip show generative-ai-hub-sdk
```

### Conclusion

Once the orchestration completes, the output is more refined — sensitive information masked, inappropriate content filtered, and (if configured) responses translated. This demonstrates the power of modules like Data Masking, Translation, and Content Filtering to enhance privacy and ensure response quality.

While this tutorial used a sentiment analysis use case, the same principles apply to other use cases. Customize the Data Masking and Content Filtering settings based on your specific requirements to handle sensitive or categorized data effectively.

By incorporating these optional modules — whether through AI Launchpad, Python SDK, JavaScript SDK, Bruno, or Claude Code via the SAP AI Skills Library — you can tailor responses to meet organizational data security policies and ensure safe, reliable responses across diverse scenarios. When working through Claude Code against a live SDK, treat published tutorial code as a strong starting point rather than ground truth — verify class signatures directly against your installed package version whenever a validation error suggests drift.
