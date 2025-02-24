# PostGPT: GPT-to-LinkedIn Integration

> **Warning:** This integration is not for the faint of heart. It involves advanced configuration, OAuth setup with LinkedIn, and manually obtaining your LinkedIn URN via Postman. Expect some bumps along the way. For support, join our [Discord](https://discord.gg/z5DgD5ZHNJ) .

## Overview

This project demonstrates how to hook a GPT directly into your LinkedIn account. The process involves:
- Creating a custom GPT action in ChatGPT.
- Setting up OAuth on [LinkedIn Developer](https://www.linkedin.com/developers/).
- Obtaining your personal URN via [Postman](https://www.postman.com/).
- Configuring your GPT with the appropriate schema and credentials.

> **Note:** A company account (with a company email) on LinkedIn is required to leverage the API.

## Prerequisites

- **ChatGPT Access:** Ensure you have access to ChatGPT’s [Explore GPTs](https://chat.openai.com/chat) feature.
- **LinkedIn Developer Account:** Sign up or log in at [LinkedIn Developers](https://www.linkedin.com/developers/).
- **Company Account:** You may need a company account on LinkedIn for API access.
- **Postman:** [Sign up](https://www.postman.com/) for a free account to obtain your URN.
- **Basic OAuth Knowledge:** Familiarity with OAuth flows will help.

## Setup Instructions

### 1. Create a GPT & New Action

1. Open ChatGPT and navigate to **Explore GPTs**.
2. Click **Create** and then select **Configure**.
3. Scroll down and click the **Create New Action** button.

### 2. Set Up OAuth with LinkedIn

1. Visit the [LinkedIn Developer Console](https://www.linkedin.com/developers/) and sign in.
2. Under **My Apps**, create a new app:
   - **App Name:** For example, `PostGPT`
   - **Company:** (e.g., *Synaptic Labs* – or your own company)
   - **App Logo:** Upload your logo.
3. Agree to the terms and click **Create App**.

4. In your new app’s dashboard:
   - Go to the **Products** tab.
   - Request access for:
     - **Share on LinkedIn**
     - **Sign in with LinkedIn using OpenID Connect**

5. In the **Settings** tab:
   - Click **Verify** to confirm your account.
   - Generate your OAuth credentials:
     - Copy the **Client ID** and **Client Secret**.
     - Note the **Auth URL**, **Access Token URL**, and **Scopes** required (these will be provided in your app’s documentation and should be added to your GPT configuration).
       - Auth URL: `https://www.linkedin.com/oauth/v2/authorization`
       - Token URL: `https://www.linkedin.com/oauth/v2/accessToken`
       - Scopes: `openid profile w_member_social email`

### 3. Configure GPT Authentication

1. In your GPT configuration under **Authentication Settings**, input:
   - **Client ID** and **Client Secret** (from LinkedIn Developer Console)
   - The **Auth URL**, **Access Token URL**, and **Scopes**.
2. Save your changes.

**Useful Links:**
- [LinkedIn Developer Console](https://www.linkedin.com/developers/)
- [LinkedIn API Documentation (Access Token)](https://docs.microsoft.com/en-us/linkedin/shared/authentication/authorization-code-flow)

### 4. Obtain Your LinkedIn URN Using Postman

1. Go to [Postman](https://www.postman.com/) and log in or sign up.
2. Create a **New Request** and set the URL to:
   ```
   https://api.linkedin.com/v2/userinfo
   ```
3. Under the **Authorization** tab:
   - Choose **OAuth 2.0**.
   - Click **Configure New Token** and fill in the following details:
     - **Token Name:** e.g., `LinkedIn`
     - **Callback URL:** Write this down for the next step.
     - **Auth URL:** https://www.linkedin.com/oauth/v2/authorization)
     - **Access Token URL:** https://www.linkedin.com/oauth/v2/accessToken
     - **Client ID:** (from LinkedIn Developer Console)
     - **Client Secret:** (from LinkedIn Developer Console)
     - **Scopes:** openid profile w_member_social email
     - **State:** random.
   - **Important:** Ensure the option to send credentials in the **body** is enabled.
4. Head back to the developer console, and under the `Auth` tab, look for the OAuth url call back setting and add the **Callback URL** from postman (probably `https://oauth.pstmn.io/v1/browser-callback`).
5. Click **Get New Token** and complete the login and authorization process.
6. Once the token is received, click **Use Token**.
7. Send the request and look for the `sub` field in the response. This string (typically around 9 alphanumeric characters) is your LinkedIn URN.

### 5. Configure the Schema in GPT

1. In your GPT’s configuration, locate the **Schema** section.
2. Copy and paste your schema into this section. A placeholder snippet is provided below:
   
```json
openapi: 3.1.0
info:
  title: LinkedIn Personal Post API
  version: 1.0.0
  description: |
    API to post content on your personal LinkedIn account using the UGC API.
servers:
  - url: https://api.linkedin.com/v2
paths:
  /ugcPosts:
    post:
      operationId: createUgcPost
      summary: Create a new post on LinkedIn
      description: Post text, links, or media to your personal LinkedIn profile.
      security:
        - linkedin_auth:
            - openid
            - profile
            - w_member_social
            - email
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/UgcPostRequest"
      responses:
        "201":
          description: Post created successfully.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/UgcPostResponse"
        "401":
          description: Unauthorized – Invalid or expired access token.
        "403":
          description: Forbidden – Insufficient permissions.
components:
  schemas:
    UgcPostRequest:
      type: object
      required:
        - author
        - lifecycleState
        - specificContent
        - visibility
      properties:
        author:
          type: string
          description: The URN of the author (e.g., urn:li:person:{ID} or urn:li:organization:{ID}).
          default: urn:li:person:[insert URN here (no brackets)]
        lifecycleState:
          type: string
          description: The state of the post (should be "PUBLISHED").
          example: PUBLISHED
        specificContent:
          type: object
          description: The content details of the post.
          properties:
            com.linkedin.ugc.ShareContent:
              type: object
              properties:
                shareCommentary:
                  type: object
                  description: The commentary content for the post.
                  properties:
                    text:
                      type: string
                      description: The text content of the post (max 3000 characters).
                      example: This is my first post using the LinkedIn API!
                shareMediaCategory:
                  type: string
                  description: Type of media being shared (use NONE for text-only posts).
                  example: NONE
        visibility:
          type: object
          description: Visibility settings for the post.
          properties:
            com.linkedin.ugc.MemberNetworkVisibility:
              type: string
              description: Who can see this post (e.g., PUBLIC).
              example: PUBLIC
    UgcPostResponse:
      type: object
      properties:
        id:
          type: string
          description: The unique identifier of the created post.
          default: urn:li:ugcPost:[insert URN here (no brackets)]
  securitySchemes:
    linkedin_auth:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://www.linkedin.com/oauth/v2/authorization
          tokenUrl: https://www.linkedin.com/oauth/v2/accessToken
          scopes:
            openid: Use your name and photo
            profile: Use your name and photo
            w_member_social: Create, modify, and delete posts, comments, and reactions on your behalf
            email: Use the primary email address associated with your LinkedIn account
   ```
   
3. Replace the placeholder `[insert URN here (no brackets)]` with the URN value obtained via Postman for example `default: urn:li:ugcPost:abc123xyz`. (Note: There are two spots in the schema that need this update.)

### 6. Finalize Callback URL Configuration

1. In your GPT configuration (the main configuration page where you insert the prompt, and choose the tools), locate the **Callback URL**.
2. Copy this URL and return to your LinkedIn Developer Console.
3. In your app’s `Auth` settings tab, add the GPT Callback URL to the authorized redirect URLs like you did for the postman request.
4. Save the changes.

### 7. Test Your Integration

1. In your GPT interface, execute a test command (for example, type “post it”).
2. You should see the command processed, and if successful, your post will appear live on LinkedIn.
3. If the process fails, double-check the OAuth settings, URN insertion, and Callback URL configuration.

## Troubleshooting & Support

- **Issues?** The process can be finicky—ensure every step is followed exactly.
- **Join our [Discord](https://discord.gg/z5DgD5ZHNJ) community** for real-time help.
- Consult the [LinkedIn API Documentation](https://docs.microsoft.com/en-us/linkedin/) if you run into problems with OAuth or API access.

## Advanced Flow (Premium)

For a more robust solution—including planning and scheduling posts directly from GPT—we offer an advanced version of this integration. Visit our [website](flows.synapticlabs.ai) for more details. *Note: This is a paid service.*

## Summary

1. **Create a GPT action** in ChatGPT.
2. **Set up OAuth** credentials on the LinkedIn Developer Console.
3. **Obtain your URN** via Postman.
4. **Insert your schema** (with your URN) into the GPT configuration.
5. **Configure the Callback URL** in both GPT and LinkedIn Developer settings.
6. **Test the integration** and post to LinkedIn.

Thanks for checking out PostGPT! Happy posting, and good luck with your integration.
