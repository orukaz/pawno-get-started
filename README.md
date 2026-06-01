# Pawno Affiliate Signup

Plan for a dedicated Pawno affiliate signup flow that collects qualified leads before sending users into the full pawn application process.

## Goal

Create a lower-friction registration flow for affiliate traffic.

The current calculator/application flow is useful for users who are already ready to pawn an item, but it asks for too much too early for campaign traffic. The affiliate flow should first collect the user's core contact details, create their Pawno account through Clerk, sign them in, and then guide them to the next step.

## Campaign Context

This flow is mainly intended for direct-response traffic where the user has not necessarily decided what they want to pawn yet.

Important traffic sources:

- SMS campaigns
- Affiliate platforms
- LFF / Lead For Finance traffic
- Revenly affiliate redirect links, for example: https://app.revenly.net/affiliate/click/a1d113f7-137d-41a1-b712-bba67741d7dd

These users may be interested in getting money quickly, but they are often not ready to complete a full pawn application in the same session.

## Revenly Redirect Example

Example LFF / Revenly affiliate click URL:

```text
https://app.revenly.net/affiliate/click/a1d113f7-137d-41a1-b712-bba67741d7dd
```

Observed redirect target on June 1, 2026:

```text
https://www.pawno.io/es?click=rev_01kt1vn17t1fzz10ejrk2wxt2z&campaign=pawno-io&network=revenly&publisher=default&publisher_id=default&campaign_id=pawno-io&click_id=rev_01kt1vn17t1fzz10ejrk2wxt2z
```

The exact `rev_...` value is generated per click, so it should be treated as a dynamic click identifier.

Redirect query parameters:

| Parameter | Example value | Meaning | Recommended handling |
| --- | --- | --- | --- |
| `click` | `rev_01kt1vn17t1fzz10ejrk2wxt2z` | Revenly click identifier. It currently matches `click_id`. | Store as received. |
| `click_id` | `rev_01kt1vn17t1fzz10ejrk2wxt2z` | Canonical affiliate click identifier. | Store as the primary affiliate click ID. |
| `campaign` | `pawno-io` | Campaign slug/name passed by Revenly. | Store as campaign label. |
| `campaign_id` | `pawno-io` | Campaign ID passed by Revenly. | Store as campaign ID. |
| `network` | `revenly` | Affiliate network/source. | Store as affiliate network. |
| `publisher` | `default` | Publisher slug/name. | Store as publisher label. |
| `publisher_id` | `default` | Publisher ID. | Store as publisher ID. |

Recommended normalized attribution fields:

```text
affiliate_network=revenly
affiliate_click_id=rev_01kt1vn17t1fzz10ejrk2wxt2z
affiliate_campaign=pawno-io
affiliate_campaign_id=pawno-io
affiliate_publisher=default
affiliate_publisher_id=default
affiliate_landing_url=https://www.pawno.io/es?...
affiliate_raw_url=https://app.revenly.net/affiliate/click/a1d113f7-137d-41a1-b712-bba67741d7dd
affiliate_raw_query={...}
```

For the new Get Started flow, the Revenly link should ideally redirect directly to:

```text
https://www.pawno.io/es/get-started?click=...&campaign=...&network=revenly&publisher=...&publisher_id=...&campaign_id=...&click_id=...
```

If the affiliate platform cannot change the destination immediately, Pawno should still capture these parameters on the current landing page and preserve them when sending the user to `/es/get-started` or `/en/get-started`.

## Current Website References

Live pages:

- Home: https://www.pawno.io/en
- Sign in: https://www.pawno.io/en/sign-in
- Dashboard: https://www.pawno.io/en/dashboard
- Current pawn application flow: https://www.pawno.io/en/calculator?type=metals&metal=gold&metal_amount=3&purity=14k&weight_unit=gr

Captured analysis files:

- [Current website snapshot index](docs/analysis/current-website/README.md)
- [Home HTML](docs/analysis/current-website/home.html)
- [Home screenshot](docs/analysis/current-website/home.png)
- [Sign-in HTML](docs/analysis/current-website/sign-in.html)
- [Sign-in screenshot](docs/analysis/current-website/sign-in.png)
- [Dashboard HTML](docs/analysis/current-website/app-dashboard.html)
- [Dashboard screenshot](docs/analysis/current-website/app-dashboard.png)
- [Pawn application form HTML](docs/analysis/current-website/pawn-application-form.html)
- [Pawn application form screenshot](docs/analysis/current-website/pawn-application-form.png)
- [SMS verification screenshot](docs/analysis/current-website/sms-message.png)

## AS-IS Experience

In the current campaign journey, users can arrive from an SMS or affiliate link and are quickly pushed toward the existing website or calculator/application flow.

The existing application flow is optimized for users who are already prepared to pawn an item. It can require the user to think about the item, estimate details, upload photos or documents, create an account, and complete email or SMS verification before the lead is safely captured.

For warm but undecided campaign users, this is too much commitment too early.

Typical AS-IS journey:

1. User receives an SMS or clicks an affiliate campaign link.
2. User lands on the Pawno website or current calculator/application flow.
3. User is asked to start a relatively detailed pawn process.
4. User may need to choose an item, enter item details, prepare photos, and complete account verification.
5. If the user is unsure, busy, older, or missing photos/documents, they abandon the flow.
6. Pawno may lose the lead before collecting reliable contact details.

## AS-IS Problems

Main problems:

- Intent mismatch: campaign users are often curious, not ready to submit a full pawn application.
- Too much cognitive load: users must think about item type, item details, photos, documents, and verification at once.
- Missing readiness: users may not yet know which item they want to pawn.
- Photo/document friction: users may not have photos or identity documents available when clicking an SMS.
- Verification friction: asking for both email and SMS verification early can interrupt the lead capture.
- Short verification window: a 30-second code window is too short for slower or older users; 3-5 minutes would be more forgiving.
- Unclear next step: users can feel they are entering a formal loan/pawn process before they understand the benefit.
- Lost attribution: if the user abandons before account/lead creation, affiliate and campaign attribution may be lost.

## TO-BE Solution

Create a dedicated `/en/get-started` affiliate signup page that captures the user first and delays the heavier pawn application steps.

The TO-BE flow should:

- Match the intent of SMS and affiliate users.
- Ask only for information needed to create a useful lead.
- Validate email and, if required, verify it after the lead has already been saved.
- Capture mobile phone without requiring SMS verification in the first step.
- Save the lead as soon as the minimum valid information is available.
- Preserve affiliate and UTM attribution.
- Let the user continue later if they are not ready to submit item details immediately.

Recommended TO-BE journey:

1. User clicks an SMS or affiliate link.
2. User lands directly on `/en/get-started`.
3. Page explains the low-commitment benefit: get started, no obligation, fast estimate.
4. User enters basic contact details and rough item intent.
5. The lead is created or updated in draft status.
6. Email is validated and verified if required by account policy.
7. Mobile phone is saved without SMS verification.
8. A Clerk account is created and the user is signed in.
9. User is redirected to `/en/dashboard/thank-you`.
10. Thank-you page offers the next step: continue application, add item photos, or wait for contact.

## Draft Lead Handling

Save the lead as early as possible once the form has enough valid information.

Recommended minimum data for draft creation:

- First name
- Last name
- Email address
- Mobile phone
- Item category or `not sure yet`
- Campaign attribution metadata

Draft status:

```text
draft
```

If the draft ID is shown in the URL, use only an opaque identifier or signed token. Do not put email, phone, name, or personal ID in the URL.

Example safe URLs:

```text
/en/dashboard/thank-you?draft=ld_8s7K2x
/en/get-started?draft=ld_8s7K2x
```

This would allow the user to return to the draft without exposing personal data in browser history, analytics tools, referrers, or affiliate redirects.

## Product Decision

Use **Get started** instead of **Register** for the affiliate route and campaign messaging.

Recommended route:

```text
/en/get-started
```

Reasoning:

- "Register" sounds like an account-management task.
- "Get started" feels lower commitment and more natural for campaign traffic.
- The form itself can still clearly explain that a Pawno account will be created.

Suggested labels:

- Page title: `Get started with Pawno`
- Form heading: `Create your Pawno account`
- Primary CTA: `Create account`
- Existing user link: `Already have an account? Sign in`

## Proposed Flow

1. Affiliate visitor lands on `/en/get-started`.
2. Visitor fills a short signup form.
3. The form creates or updates a draft lead.
4. Clerk creates the user account.
5. User is signed in automatically after registration.
6. User is redirected to an authenticated thank-you page.
7. Thank-you page offers the next step toward valuation or full pawn application.

Recommended thank-you route:

```text
/en/dashboard/thank-you
```

## Form Fields

Required fields:

- First name
- Last name
- Email address
- Password
- Mobile phone
- Item category
- Terms/privacy consent

Recommended item category options:

- Gold or jewelry
- Watch
- Phone or laptop
- I am not sure yet
- Other

Optional fields:

- DNI/NIE or personal ID
- Timeline: `Today`, `This week`, `Just checking`
- Marketing/contact consent

Do not require mobile phone verification in the initial affiliate lead form.

## Recommended Form Structure

Keep the first screen focused on lead capture, not the full application.

Suggested form sections:

1. Contact details
2. What the user may want to pawn
3. Account security
4. Consent and submit

Suggested fields:

- First name
- Last name
- Email address
- Mobile phone
- Item category
- Timeline
- Password
- Terms/privacy consent
- Marketing/contact consent

Suggested helper copy:

```text
No obligation. Start with your contact details and continue the valuation when you are ready.
```

Suggested mobile phone helper copy:

```text
We may contact you about your estimate. No SMS verification is required now.
```

Suggested item helper copy:

```text
Not sure yet? Choose "I am not sure yet" and continue.
```

Recommended page copy:

- Eyebrow: `Fast start`
- Headline: `Get started with Pawno`
- Supporting text: `Create your account now and continue the valuation when you are ready. No obligation.`
- Trust points: `No obligation`, `Secure and private`, `Photos can be added later`
- Primary CTA: `Create account`
- Secondary link: `Already have an account? Sign in`

Recommended field order:

1. First name and last name
2. Email address
3. Mobile phone
4. Item category
5. Timeline
6. Password
7. Consent

This keeps the user moving from easy contact details into account creation instead of starting with the heaviest commitment.

## Identity Details

Personal ID should not be required in the first version.

Reasoning:

- It is sensitive information.
- It can reduce affiliate landing page conversion.
- It is more appropriate later in the verified pawn application or KYC step.

If included, it should be optional and clearly labelled as:

```text
DNI/NIE or personal ID (optional)
```

## Social Login

Google auth is not required for the first version.

Reasoning:

- Email and password keeps the first implementation simpler.
- Social login may still require an extra profile-completion step for phone and lead intent.
- Affiliate tracking and lead metadata should stay consistent across sign-up methods.

Google auth can be added later if account creation drop-off is high.

## Affiliate Tracking

Capture campaign metadata and attach it to the created user or lead record.

Recommended metadata:

- `utm_source`
- `utm_medium`
- `utm_campaign`
- `utm_content`
- `utm_term`
- `network`
- `click`
- `click_id`
- `campaign`
- `campaign_id`
- `publisher`
- `publisher_id`
- `affiliate_id` or `ref`
- `affiliate_network`
- `affiliate_click_id`
- `affiliate_campaign`
- `affiliate_campaign_id`
- `affiliate_publisher`
- `affiliate_publisher_id`
- `landing_url`
- `referrer`
- `gclid`
- `fbclid`
- raw query parameters

Attribution should be captured before the user starts filling the form, because campaign users may abandon at any step.

When both raw and normalized values are stored, raw values preserve audit/debug information while normalized values make reporting and CRM follow-up easier.

## Clerk Implementation Notes

Use a custom Clerk sign-up flow rather than only the hosted/default sign-up component.

Reasoning:

- The affiliate flow needs custom fields.
- The form needs campaign metadata.
- The redirect after sign-up should go to the authenticated thank-you page.
- Phone should be collected without forcing SMS verification at this stage.

Suggested implementation direction:

- Use Clerk custom sign-up APIs/hooks.
- Create the user with email, password, first name, and last name.
- Store lead-specific fields in metadata or in the app database after account creation.
- Redirect to `/en/dashboard/thank-you` after a successful sign-up.

Verification recommendation:

- Email format validation should happen immediately.
- Email verification can stay required for full account activation, because email is the main recovery and account identifier.
- Draft lead creation should not depend on the user successfully completing an email or SMS code step.
- SMS verification should not be required in the affiliate lead form.
- If verification codes are used, the code lifetime should be increased from roughly 30 seconds to at least 3-5 minutes.

## Design Direction

Use a split layout on desktop.

Left side:

- Pawno brand/product message.
- Real, relevant visual asset showing valuables such as jewelry, watches, gold, or electronics.
- Short trust points:
  - No obligation
  - Secure and private
  - Fast estimate

Right side:

- Compact signup form.
- Clear account creation copy.
- Primary CTA.
- Link to sign in for existing users.

Mobile:

- Put the signup form first.
- Keep trust points short and close to the form.
- Avoid a long marketing section before the form.

Recommended desktop layout:

- Left 45%: visual, value proposition, trust points, short explanation.
- Right 55%: form with a clear heading and compact fields.
- Keep the form above the fold on common laptop screens.
- Avoid making the user scroll before seeing the first input.

Recommended mobile layout:

- Header/logo at top.
- One short benefit statement.
- Form immediately visible.
- Trust points below or between form sections.
- Avoid large hero imagery before the form.

The page should feel like a fast start, not a formal loan application.

## Thank-You Page

The thank-you page should confirm that the lead/account was created and make the next step feel optional but useful.

Recommended content:

- Confirmation: `Your Pawno account is ready`
- Reassurance: `Your details were saved. You can continue now or come back later.`
- Primary action: `Continue with item details`
- Secondary action: `Add photos later`
- Optional support CTA: `Talk to Pawno`

If a draft exists, show its status:

```text
Draft saved
```

This reduces anxiety and makes it clear the user has not lost progress.

## Tasks

- [ ] Confirm final route: `/en/get-started`
- [ ] Confirm thank-you route: `/en/dashboard/thank-you`
- [ ] Confirm Revenly/LFF destination URL for Get Started campaigns
- [ ] Confirm final form fields
- [ ] Decide whether optional DNI/NIE field is shown in version one
- [ ] Confirm draft lead data model and status
- [ ] Confirm safe draft URL/token strategy
- [ ] Build affiliate signup page
- [ ] Implement Clerk custom sign-up flow
- [ ] Capture affiliate and UTM metadata
- [ ] Preserve Revenly query parameters when redirecting into Get Started
- [ ] Save lead as draft after minimum valid data is available
- [ ] Create authenticated thank-you page
- [ ] Add sign-in link for existing users
- [ ] Review email verification lifetime
- [ ] Disable SMS verification requirement for affiliate lead capture
- [ ] QA desktop layout
- [ ] QA mobile layout
- [ ] Test successful sign-up and redirect
- [ ] Test validation and error states
