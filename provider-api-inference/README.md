# Provider API inference evaluation

**Status:** ACTIVE  
**Started:** 2026-11-01  
**Scope:** Provider APIs and hosted-model access available to FlossWare

## Question

Which external inference APIs provide useful, reliable model capabilities for FlossWare, with emphasis on genuinely free or low-cost access, provider-neutral invocation, and evidence-backed capability profiles?

## Boundaries

- This is cross-project research, not a production implementation.
- `FlossWare/evaluation` contains the experimental catalogue-inventory and inference smoke-test utilities for this investigation. These scripts are research tooling, not part of the published evaluation package or a production provider transport layer.
- `FlossWare/model-gateway` owns provider-neutral invocation, credentials, resources, usage/cost, and provenance.
- `FlossWare/ensemble` may later consume verified provider/model capability profiles; it must not own provider-specific discovery or evaluation.
- EnMaaS is retained as a separate enterprise-provider baseline.
- Never place credentials or credential values in reports, fixtures, logs, provenance, or repository content.

## Initial authenticated catalogue baseline

Observed on 2026-11-01 using FlossWare-scoped environment variables and provider model-list endpoints.

| Provider | Models returned | Discovery result |
|---|---:|---|
| Cerebras | 2 | Authenticated |
| Cohere | 20 | Authenticated |
| DeepInfra | 173 | Authenticated |
| DeepSeek | 2 | Authenticated |
| Google Gemini API | 50 | Authenticated |
| Groq | 11 | Authenticated |
| Hugging Face | 135 | Authenticated |
| Mistral | 55 | Authenticated |
| NVIDIA | 85 | Authenticated |
| OpenAI | 128 | Authenticated |
| OpenRouter | 416 | Authenticated |
| Voyage AI | unknown | HTTP 404 from attempted `/v1/models`; endpoint mismatch, not evidence of invalid credentials |

**Total catalogue entries reported:** 1,077 across the 11 successful catalogue requests.

Additional configured credentials not yet probed with provider-specific discovery endpoints: Cloudflare, Eden AI, Pollinations, Poolside, Thinking Machines, Unstructured, and ZeroLimit AI.

These counts are catalogue observations, not counts of chat-capable, free, available, or supported models. A successful model-list request proves authentication and catalogue access only.

## Preliminary access classification

This is a research lead, not a final eligibility decision. Terms, quotas, model access, and account-specific limits must be rechecked before tests.

| Provider | Current public access signal | Evaluation treatment |
|---|---|---|
| OpenRouter | Free model variants are marked `:free`; free usage is rate-limited. | First priority for zero-priced model probes. Record exact model ID and provider route. |
| Groq | A Free tier exists with model-specific limits; exact account limits are authoritative. | First priority. Capture rate-limit headers and account/model limits. |
| Google Gemini API | Free tier gives limited access to selected models and free input/output tokens; free-tier content may be used to improve products. | First priority, but explicitly record data-use implications. |
| Mistral | Studio Free mode is enabled by default and does not require a credit card; usage/rate limits apply. | First priority. |
| Cohere | Free evaluation keys exist; trial Chat API keys are limited to 1,000 calls/month and generally 20 requests/minute per model. | First priority for chat, embeddings, and reranking; distinguish trial from production key. |
| NVIDIA NIM | NVIDIA Developer Program advertises free NIM API access for prototyping. | First priority; discover per-model access and actual quota. |
| Hugging Face Inference Providers | Free accounts receive $0.10/month in routed inference credits, subject to change. Custom provider keys bypass those HF credits and bill the provider directly. | Specialized/secondary priority; distinguish HF-routed billing from custom-provider-key billing. |
| Cerebras | Current pricing advertises a one-time $5 promotional credit requiring a valid payment method, expiring after 30 days. | Conditional trial, not recurring free access. Confirm account terms before inference. |
| DeepInfra | Current pricing page says a card or prepayment is required. | Low-cost paid comparison, not a free-tier candidate. |
| DeepSeek | API is token-priced; free consumer chat is not free API inference. | Low-cost paid comparison, not a free-tier candidate. |
| OpenAI | API catalogue access is available; no free API inference entitlement is assumed. | Exclude from free-first tests unless account credits are explicitly confirmed. |
| Voyage AI | Model-list endpoint used in initial probe returned 404. | Separate embeddings/reranking API discovery. |

### Public references checked 2026-11-01

- OpenRouter free models and limits: https://openrouter.ai/collections/free-models/
- OpenRouter plan comparison: https://openrouter.ai/pricing/
- Groq rate limits: https://console.groq.com/docs/rate-limits
- Groq billing tiers: https://console.groq.com/docs/billing-faqs
- Gemini pricing and data-use distinction: https://ai.google.dev/gemini-api/docs/pricing
- Gemini rate limits: https://ai.google.dev/gemini-api/docs/rate-limits
- Mistral Free mode: https://docs.mistral.ai/getting-started/quickstarts/studio/activate-and-generate-api-key
- Cohere evaluation keys and limits: https://docs.cohere.com/v2/docs/rate-limits
- NVIDIA Developer Program: https://docs.api.nvidia.com/nim/docs/product
- Hugging Face inference credits and billing modes: https://huggingface.co/docs/inference-providers/pricing
- Cerebras current promotional credit: https://www.cerebras.ai/pricing
- DeepInfra pricing/payment requirements: https://deepinfra.com/pricing
- DeepSeek API pricing: https://api-docs.deepseek.com/quick_start/pricing/

## Evaluation sequence

### Phase 1: Catalogue normalization

For each provider, preserve:
- Provider and API family
- Exact model identifier
- Model display name, when supplied
- Context and output limits, when supplied
- Input/output modalities
- Supported operations or endpoints
- Published pricing metadata
- Catalogue retrieval timestamp
- Raw response digest and source URL, excluding credentials

Do not infer capabilities from a model name. Missing metadata is `unknown`, not `false`.

### Phase 2: Minimal inference smoke test

Run one deliberately small request against one selected model per eligible provider. Record:
- Request/response protocol and endpoint
- HTTP status and provider request ID
- Time to first byte/token and total latency
- Completion status and stop reason
- Input/output/cache/reasoning usage when returned
- Cost and billing metadata when returned
- Rate-limit headers
- Error body and retry metadata on failure

A successful HTTP response is not proof of semantic correctness or a free request.

### Phase 3: Representative capability suite

Run the full suite against one representative model per API family/provider:
1. Basic non-streaming inference
2. Streaming, event ordering, and final usage
3. Tool-call generation and tool-result round trip
4. Constrained/structured output
5. Reasoning/thinking controls and observable reasoning metadata
6. Context-window boundary behavior
7. Error semantics, rate limits, and retry metadata
8. Usage, cache, cost, and provenance

Then run targeted model-specific checks for differences. Avoid repeating the full suite for every model.

### Phase 4: Specialized APIs

Evaluate embeddings, reranking, tokenization, and document processing separately. A provider that does not offer general chat inference can still be valuable.

## Provider-neutral capability evidence

Represent each capability with:
- `status`: `supported`, `unsupported`, `unknown`, or `conditional`
- `evidence`: test identifier and observed behavior
- `scope`: provider, API family, model, account tier, and endpoint
- `observed_at`: timestamp
- `source`: official documentation or test result
- `limitations`: relevant constraints

Preserve native provider payloads alongside normalized events. Normalization must not discard provider-specific details, ordering, usage fields, signatures, request IDs, or stop reasons.

## Safety and cost controls

- Never print, serialize, or persist API key values.
- Do not send real user data, proprietary code, or secrets in probes.
- Use synthetic prompts and minimal token budgets.
- Prefer confirmed $0 model variants or explicitly documented free tiers.
- Do not assume a zero cost field means free; missing cost means unknown.
- Do not retry automatically on 429, 402, or provider-specific billing errors.
- Record the account tier and billing mode used for each result.
- Treat free-tier terms and model catalogues as time-sensitive observations.

## Initial inference smoke-test observations

Observed on 2026-11-01 using the experimental harness in `FlossWare/evaluation`. These are request-level observations for the configured account and exact model IDs, not general provider guarantees.

### OpenRouter: NVIDIA Nemotron 3.5 Lightning Free

| Field | Observation |
|---|---|
| Requested model | `nvidia/nemotron-3.5-lightning:free` |
| Returned model | `nvidia/nemotron-3.5-lightning:free` |
| Access classification | `catalogue-zero-price` |
| HTTP status | 200 |
| Latency | 3.75 s |
| Finish reason | `stop` |
| Prompt tokens | 36 |
| Completion tokens | 382 |
| Reasoning tokens | 369 |
| Total tokens | 418 |
| Visible response | `FLOSSWARE_PROVIDER_SMOKE_OK` (27 characters) |
| Semantic match | true |
| Response completeness | `complete` |
| Reported cost | 0 |
| Retry performed | false |

The request succeeded with `max-output-tokens=512`. An earlier request with a 32-token completion budget returned HTTP 200 but no visible response and a `length` finish reason, with all reported completion tokens consumed by reasoning. The harness default was subsequently raised to 128; the 128-token attempt also did not complete. The 512-token request is therefore the **first observed successful budget**, not a proven minimum.

The successful request reported 369 reasoning tokens out of 382 completion tokens, leaving 13 tokens for visible output. This supports classifying the earlier empty responses as incomplete reasoning/output rather than semantic mismatches or provider-access failures. It does not establish a universal token requirement; reasoning consumption can vary by prompt, model revision, and provider behavior.

The provider-reported zero cost applies only to this observed request. A catalogue `:free` label and a zero cost field are not guarantees of future availability, account eligibility, or billing treatment.

### Evidence limitations

- One successful semantic smoke test is not a capability-suite result.
- Streaming, tool calls, structured output, context limits, and error behavior remain untested for this model.
- The test establishes successful non-streaming text inference for this exact model ID, account configuration, endpoint, prompt, and observed date only.
- Provider request ID is retained in the local JSONL evidence file; credentials and credential values are not included in the research record.

## Initial conclusion

The authenticated catalogue baseline is established, and the first successful inference smoke-test observation is now recorded for OpenRouter's NVIDIA Nemotron 3.5 Lightning Free. Continue with one representative inference probe per eligible provider, then proceed to the representative capability suite. DeepInfra, DeepSeek, and OpenAI remain useful paid comparison candidates but should not be mislabeled as free APIs.
