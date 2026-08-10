# AI Instagram Content Automation — n8n Workflow Guide

## 1. Project Overview

This project is an AI-powered Instagram content automation workflow built with **n8n**.

The workflow automatically:

1. Runs on a schedule.
2. Selects the Instagram platform and content topic.
3. Generates an AI-written post.
4. Generates hashtags and a call-to-action.
5. Checks the content length.
6. Shortens the content if necessary.
7. Creates an Instagram media container using the Instagram Graph API.
8. Publishes the post to Instagram.

### Main Technologies

- n8n
- Instagram Graph API
- Facebook Graph API
- Google Gemini
- Groq / LLM
- Instagram Business Account

---

# 2. Workflow Data Flow

The overall workflow is:

```text
Schedule Trigger
       ↓
Select Platform
       ↓
Basic LLM Chain
       ↓
Gemini Hashtags
       ↓
Gemini CTA
       ↓
Merge Content & Check Limit
       ↓
Too Long?
   ↙       ↘
 Yes        No
 ↓           ↓
Gemini      Final Content
Shorten     (Original)
 ↓           ↓
Final Content (Shortened)
       ↓
Merge Final Content
       ↓
IG - Create Container
       ↓
IG - Publish
       ↓
Instagram Post
```

---

# 3. Schedule Trigger

### Purpose

Starts the automation automatically according to the configured schedule.

Example:

```text
Every 6 hours
```

The schedule can be changed according to the project's requirements.

---

# 4. Select Platform

### Purpose

Provides the basic information required by the rest of the workflow.

Example data:

```json
{
  "platform": "instagram",
  "topic": "AI Automation",
  "tone": "professional",
  "keywords": [
    "AI",
    "automation",
    "productivity"
  ],
  "imageUrl": "DIRECT_IMAGE_URL"
}
```

### Important

The `imageUrl` must be a publicly accessible **direct image URL**.

Do not use a Markdown link such as:

```text
[image](https://example.com/image.jpg)
```

Use:

```text
https://example.com/image.jpg
```

---

# 5. Basic LLM Chain

### Purpose

Generates the main Instagram post content using an AI language model.

The generated content is based on:

- Topic
- Tone
- Keywords
- Platform

The result is passed to the following content-generation nodes.

---

# 6. Gemini Hashtags

### Purpose

Generates relevant hashtags for the Instagram post.

Example:

```text
#AI
#Automation
#Productivity
#Tech
#Innovation
```

---

# 7. Gemini CTA

### Purpose

Generates a Call-To-Action (CTA).

Example:

```text
Follow us for more AI updates!
```

The CTA is added to the final Instagram caption.

---

# 8. Merge Content & Check Limit

### Purpose

Combines the generated:

- Main content
- Hashtags
- CTA

into one final content field.

Example:

```text
Unlock the power of AI automation and transform your workflow
with increased productivity and efficiency!

#AI #Automation #Productivity #Tech #Innovation

Follow us for more AI updates!
```

The node also checks whether the generated content exceeds the required length.

---

# 9. Too Long? — IF Node

### Purpose

Checks whether the generated Instagram content is too long.

The workflow has two possible paths:

```text
Content too long?
       ↓
      YES ──→ Gemini - Shorten
       │
       NO
       ↓
Continue with original content
```

This prevents excessively long content from being sent to Instagram.

---

# 10. Gemini - Shorten

### Purpose

If the content exceeds the required length, Gemini shortens it.

The shortened version keeps the important:

- Message
- Hashtags
- CTA

while reducing unnecessary text.

---

# 11. Final Content / Final Content (Shortened)

### Purpose

Stores the final version of the Instagram caption.

Depending on the IF condition, the workflow uses either:

- Original content
- Shortened content

The final output is stored in:

```text
finalText
```

---

# 12. Merge Final Content

### Purpose

Combines the final caption with the other required Instagram data.

Example:

```json
{
  "platform": "instagram",
  "imageUrl": "https://example.com/image.jpg",
  "finalText": "AI automation can transform your workflow..."
}
```

This node provides the final data required by the Instagram API.

---

# 13. IG - Create Container

### Purpose

Creates an Instagram media container using the Instagram Graph API.

### HTTP Method

```text
POST
```

### Endpoint

```text
https://graph.facebook.com/v23.0/17841439384790740/media
```

The number:

```text
17841439384790740
```

is the Instagram Business Account ID used by this workflow.

### Authentication

```text
Facebook Graph API
```

The configured Facebook Graph API credential provides the required access token.

### Body Type

```text
Form URL Encoded
```

### Body Fields

#### image_url

```text
{{ $json.imageUrl }}
```

#### caption

```text
{{ $json.finalText }}
```

### Successful Response

A successful request returns a media container ID.

Example:

```json
{
  "id": "18083735057312827"
}
```

This ID is passed to the publishing node.

---

# 14. IG - Publish

### Purpose

Publishes the previously created Instagram media container.

### HTTP Method

```text
POST
```

### Endpoint

```text
https://graph.facebook.com/v23.0/17841439384790740/media_publish
```

### Body

The `creation_id` is taken from the previous node:

```text
{{ $json.id }}
```

Example:

```text
18083735057312827
```

This tells Instagram which media container should be published.

### Successful Result

A successful response returns an Instagram media ID and confirms that the media was submitted for publishing.

---

# 15. Instagram Graph API Requirements

The workflow requires an Instagram account that supports publishing through the Instagram Graph API.

The Instagram account should be properly connected to the required Meta/Facebook setup.

The workflow also requires:

- Instagram Business/Professional account
- Facebook Page connection where required
- Meta Developer application
- Valid access token
- Required Instagram publishing permissions
- Correct Instagram Business Account ID

---

# 16. Authentication

The workflow uses an n8n credential:

```text
Facebook Graph API
```

The access token should be stored inside the n8n credential system.

### DO NOT

Do not place access tokens directly inside:

- GitHub files
- README files
- GUIDE.md
- JavaScript code
- Public workflow JSON
- Screenshots

---

# 17. Environment Variable Issue

During development, the following expression caused an error:

```text
{{ $env.IG_BUSINESS_ACCOUNT_ID }}
```

The error was:

```text
access to env vars denied
```

This happened because the n8n environment variable was not accessible from the workflow.

The working solution was to use the Instagram Business Account ID directly in the API endpoint:

```text
https://graph.facebook.com/v23.0/17841439384790740/media_publish
```

### Security Note

For production deployment, sensitive configuration should be managed through secure credentials or environment configuration supported by the deployment platform.

The Instagram Business Account ID itself is not a secret, but access tokens are.

---

# 18. Common Errors and Solutions

## Error: `access to env vars denied`

### Cause

The workflow attempted to access an environment variable using:

```text
$env.IG_BUSINESS_ACCOUNT_ID
```

but the n8n environment did not allow that variable to be accessed from the workflow.

### Solution

Use the correctly configured account ID directly in the endpoint or configure the deployment environment appropriately.

---

## Error: Instagram post does not appear

Check:

1. `IG - Create Container` succeeded.
2. `IG - Publish` succeeded.
3. There is no `error` field in the node output.
4. The correct Instagram Business Account ID is being used.
5. The Instagram account is correctly connected.
6. The access token has the required permissions.

---

## Error: Image cannot be processed

Make sure `imageUrl` is a publicly accessible direct image URL.

Correct:

```text
https://example.com/image.jpg
```

Incorrect:

```text
[https://example.com/image.jpg](https://example.com/image.jpg)
```

The image should also be accessible by Meta's servers.

---

## Error: `creation_id` missing

The `IG - Publish` node should receive the ID from `IG - Create Container`.

Use:

```text
{{ $json.id }}
```

Example:

```text
18083735057312827
```

---

# 19. Testing Procedure

Before enabling automatic scheduling, test the workflow manually.

### Step 1

Run the workflow using the appropriate manual testing method.

### Step 2

Check:

```text
Select Platform
```

Make sure the platform is:

```text
instagram
```

### Step 3

Check the AI content.

Make sure:

```text
finalText
```

contains the expected caption.

### Step 4

Check the image URL.

Make sure it is a direct publicly accessible URL.

### Step 5

Run:

```text
IG - Create Container
```

Expected result:

```json
{
  "id": "MEDIA_CONTAINER_ID"
}
```

### Step 6

Run:

```text
IG - Publish
```

Expected result:

```json
{
  "id": "INSTAGRAM_MEDIA_ID"
}
```

### Step 7

Open the Instagram account and verify that the post appears.

---

# 20. Security and `.gitignore`

Never upload secrets to GitHub.

The `.gitignore` file should normally exclude sensitive/local files such as:

```text
.env
.env.*
!.env.example

node_modules/
venv/
.venv/
__pycache__/
*.pyc

.n8n/
logs/
*.log

.DS_Store
Thumbs.db
```

Do not commit:

- API keys
- Access tokens
- Passwords
- Private credentials
- `.env` files
- Local n8n credential data

---

# 21. Recommended Repository Structure

A possible GitHub repository structure is:

```text
ai-instagram-automation/
│
├── n8n/
│   ├── instagram-content-automation.json
│   └── GUIDE.md
│
├── .gitignore
├── README.md
└── .env.example
```

The exported n8n workflow JSON can be stored in:

```text
n8n/instagram-content-automation.json
```

and this documentation can be stored as:

```text
n8n/GUIDE.md
```

---

# 22. Workflow Status

The final tested workflow successfully completed:

```text
AI Content Generation       ✅
Hashtag Generation          ✅
CTA Generation              ✅
Content Length Check        ✅
Content Shortening          ✅
Instagram Container         ✅
Instagram Publishing        ✅
Instagram Post              ✅
```

The workflow was successfully tested end-to-end.

---

# 23. Final Summary

This n8n automation removes the need to manually create and publish Instagram posts.

The workflow automatically generates content with AI, prepares the final caption, attaches an image, creates an Instagram media container, and publishes the post through the Instagram Graph API.

The final automation flow is:

```text
Schedule
   ↓
AI Content
   ↓
Hashtags + CTA
   ↓
Length Check
   ↓
Shorten if Required
   ↓
Final Content
   ↓
Instagram Container
   ↓
Instagram Publish
   ↓
Published Instagram Post
```

**Project Status: COMPLETED AND TESTED ✅**