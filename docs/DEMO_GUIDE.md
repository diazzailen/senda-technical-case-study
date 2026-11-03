# SENDA: 60–90 second technical walkthrough

**Demo setup:** Use a fictional profile and patient case. Prepare a locally permitted educational document and a prechecked, general question if you want to show a sourced answer; external model output may vary. Never display personal information or present test text as medically approved guidance.

| Time | Show / say |
| --- | --- |
| 0–12 s | Open the mobile home screen. “I built SENDA as a mobile-first MVP to explore accessible transplant education.” |
| 12–30 s | Open the educational assistant and ask a general question. “The React Native client sends this to an authenticated NestJS endpoint; Python handles the evidence-based generation step.” |
| 30–48 s | If the prechecked answer returns, show the answer and source metadata. “NestJS retrieves versioned document chunks; the AI service maps model-selected citations back to those records.” |
| 48–63 s | Show a prechecked no-evidence response. “When no eligible sources are retrieved, the assistant withholds an answer rather than inventing one.” |
| 63–78 s | Open a fictional case and timeline. “Case access is enforced server-side for owners and invited support contacts.” |
| 78–90 s | Show the [architecture diagram](../README.md#architecture-at-a-glance). “The core engineering work is the backend boundaries, semantic retrieval, authorization and testable AI behavior.” |

**Close:** “This is an engineering MVP; its information has not been clinically validated and it is not a production healthcare service.”
