# Neti | Website AI assistant

I designed, built, and launched Neti for Network Experts to help visitors find relevant services and reach the right team. The project combines an embedded website widget with an AWS backend that answers only from the company’s reviewed website content and invites visitors to have the team follow up.

[Visit the website](https://networkexperts.la/) · [Read the launch announcement](https://www.linkedin.com/feed/update/urn:li:share:7524227618863439872/) · [Back to my portfolio](https://armandabiri99.github.io/#projects)

## The problem

A visitor may know the problem they need to solve before they know the name of the service. I wanted the website to answer in everyday language, point to the right page, and offer a clear next step. That meant giving the assistant a way to acknowledge missing information and letting visitors control when an inquiry is sent.

## Architecture

![Neti architecture using AWS service icons. The embedded website widget connects to API Gateway and Lambda for chat, with DynamoDB and Bedrock on AWS. Consented inquiries follow a separate website path.](architecture-web-integration.svg)

*Conceptual view of the Network Experts deployment. Service artwork from the [official AWS Architecture Icons package](https://aws.amazon.com/architecture/icons/).*

1. **Chat.** The embedded widget sends a request through Amazon API Gateway to AWS Lambda. The backend uses DynamoDB for session and usage state. It retrieves candidate passages through Amazon Bedrock Knowledge Bases from a reviewed S3 corpus indexed with S3 Vectors. Bedrock selects the supporting facts, and a second short model pass rewords only those facts into a natural reply. The wording must pass a deterministic claim check, Bedrock Guardrails safety screening and a contextual grounding check before it is shown; otherwise the reply falls back to the verbatim approved facts or to a referral.
2. **Contact.** The widget collects an inquiry separately. The visitor can edit the details and area of interest, review the draft, and give contact consent. A separate **Send inquiry** action hands the reviewed submission to the website’s contact workflow. Structured inquiry fields stay outside the model request.

## Engineering decisions

| Decision | Reason and tradeoff |
| --- | --- |
| Retrieve from reviewed website content | A controlled corpus lets me trace claims to a source and preserve service qualifications. Content changes need review and a new knowledge release. A supported partial answer or a clear referral is preferable to guessing. |
| Natural wording behind fixed checks | Verbatim page excerpts were accurate but read like page dumps. A second model pass now phrases the selected facts, but it can only reword them: a deterministic check rejects added claims, unrequested brand names and unsupported numbers before the managed guardrails run. |
| Fix false positives without loosening safety | When the safety screen blocked ordinary requests, such as asking us to take over a company’s IT, I gave it clearer context about the business setting instead of lowering any threshold, and confirmed that malicious variants were still blocked at high confidence. |
| Tell a project from an incident | Someone asking for better email protection is a sales conversation; someone who clicked a link is a possible incident. The assistant separates the two, and any sign of actual compromise keeps the cautious path that points to the phone line and support portal. |
| Separate inquiry delivery from chat | Conversational intake reduces friction, while review and explicit consent give the visitor control over the submission. Service answers and troubleshooting both invite a follow-up, timed so that a question awaiting the visitor’s answer is not interrupted. If delivery is uncertain, the interface says so, and replay handling prevents duplicate submissions. |
| Short-lived conversation memory | Each chat keeps a small server-side note of the software or device under discussion, the symptoms described and the questions already asked. It stores no visitor wording, expires with the session, and lets the server itself stop repeated questions. Memory across visits was deliberately left out. |
| Limit automated troubleshooting | Basic help uses low-risk steps with a stopping point. Account access, unsafe actions, practice software and problems outside the approved scope go to the team. |
| Gated releases on Terraform | Infrastructure lives in Terraform with separate development and production environments. Each release pauses answering, applies a plan that must contain only the expected changes, and passes automated gates with the assistant off and again with it on before visitors see it. A deliberate limit change ships with its own plan check. |

## What I verified

The November 2026 release work measured retrieval, answer quality and the live visitor experience separately:

- **Retrieval in production.** The expected page appeared for **92 of 93 natural-language queries** and **64 of 68 title-based queries** in a 161-query evaluation.
- **Answer quality.** On a 33-conversation benchmark, accepted answers rose from **27 of 33** to **33 of 33**, and a 10-conversation email-security set went from **4 of 10** to **10 of 10**, before the natural-wording release was promoted.
- **Every listed service.** I asked one realistic visitor question about each of the **46 services** on the Solutions page in production. **42** got a grounded answer with a follow-up invitation on the first pass. The four misses traced to three defects (a safety-screen false positive and two routing issues), which were fixed and re-tested in production.
- **Regression safety.** More than **400 automated tests** cover routing, claim checks, safety screening, session memory and inquiry handling, plus browser tests of the widget that never send a real inquiry.

## Designed for more than one site

The backend is separate from the host website. The widget loads into a CMS through a plugin or embed, or into a custom frontend, and each deployment carries its own branding, approved knowledge corpus, help boundaries, allowed origins and contact destination. Local tests with two fictional sites confirmed that content, sessions and inquiry destinations stay separate. The diagram shows the Network Experts deployment.

This case study shares the architecture and evaluation results. Application source and deployment configuration are private.
