# Neti | Website AI assistant

I designed, built, and launched Neti for Network Experts to help visitors find relevant services and reach the right team. The project combines an embedded website widget with an AWS backend. Its design can be adapted to customers across industries and integrated with a CMS or a custom-built site.

[Visit the website](https://networkexperts.la/) · [Read the launch announcement](https://www.linkedin.com/feed/update/urn:li:share:7524227618863439872/) · [Back to my portfolio](https://armandabiri99.github.io/#projects)

## The problem

A visitor may know the problem they need to solve before they know the name of the service. I wanted the website to answer in everyday language, point to the right page, and offer a clear next step. That meant giving the assistant a way to acknowledge missing information and letting visitors control when an inquiry is sent.

## Architecture

![Neti architecture using AWS service icons. The embedded website widget connects to API Gateway and Lambda for chat, with DynamoDB and Bedrock on AWS. Consented inquiries follow a separate website path.](architecture-web-integration.svg)

*Conceptual view of the Network Experts deployment. Service artwork from the [official AWS Architecture Icons package](https://aws.amazon.com/architecture/icons/).*

1. **Chat.** The embedded widget sends a request through Amazon API Gateway to AWS Lambda. The backend uses DynamoDB for session and usage state. It retrieves candidate passages through Amazon Bedrock Knowledge Bases from a reviewed S3 corpus indexed with S3 Vectors. Bedrock helps select evidence for the answer. The application checks the source and its revision before rendering the supported facts and attaching links.
2. **Contact.** The widget collects an inquiry separately. The visitor can edit the details and area of interest, review the draft, and give contact consent. A separate **Send inquiry** action hands the reviewed submission to the website’s contact workflow. Structured inquiry fields stay outside the model request.

## Engineering decisions

| Decision | Reason and tradeoff |
| --- | --- |
| Retrieve from reviewed website content | A controlled corpus lets me trace claims to a source and preserve service qualifications. Content changes need review and a new knowledge release. A supported partial answer or a clear referral is preferable to guessing. |
| Separate inquiry delivery from chat | Conversational intake reduces friction, while review and explicit consent give the visitor control over the submission. If delivery is uncertain, the interface should say so. Request replay and duplicate handling help prevent retries from triggering a second submission. |
| Limit automated troubleshooting | Basic help uses low-risk steps with a stopping point. Account access, unsafe actions and problems outside the approved scope go to the appropriate support channel. |
| Use separate development and production environments | I used Terraform for infrastructure configuration and validated releases in development before promoting them. Feature controls allow answering and help to be paused while the website’s contact path remains available. |

## What I verified

The November 2026 release review tested retrieval, answer quality and the inquiry experience separately:

- **Retrieval in production.** The expected page appeared for **92 of 93 natural-language queries** and **64 of 68 title-based queries** in a 161-query evaluation. These checks measured whether retrieval surfaced the expected page.
- **Answer quality in development.** Manual acceptance improved from **23 of 33** to **26 of 33** on the same benchmark. A separate email-security set passed **4 of 10**, with gaps in follow-up specificity and unnecessary refusals. Finding the right source still leaves work to do on how the assistant uses it.
- **Public website behavior.** I reviewed two service answers and their citations after promotion. Earlier desktop and responsive mobile checks exercised inquiry intake, edits, review and consent reset without sending an inquiry. Delivery and retry outcomes were exercised with mocked receivers.

## Adapting it to another customer

The backend is separate from the host site. A CMS can load the widget through a plugin or embed, and a custom-built site can integrate it into its frontend. Each deployment needs the customer’s branding, an approved knowledge corpus, supported help boundaries, approved origins and a destination for contact handoffs.

For example, a retailer could answer product and policy questions, a manufacturer could explain capabilities and collect quote requests, and a professional services firm could guide visitors to the right team. Local testing used two fictional customers to check that source content, sessions and inquiry destinations stayed separate. The diagram shows the Network Experts deployment. A new customer integration needs its own answer-quality, privacy and delivery validation.

This case study shares the architecture and evaluation results. Application source and deployment configuration are private.
