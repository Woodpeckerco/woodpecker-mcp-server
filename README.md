# Woodpecker MCP Server

The Woodpecker MCP (Model Context Protocol) integration transforms cold email campaign management into a conversational experience.

By connecting your AI assistant to Woodpecker's powerful automation platform, you can create, manage and optimize email campaigns through natural language interactions.

Start with simple commands like listing campaigns and creating basic outreach sequences. Then gradually explore advanced features like A/B testing, complex follow-up sequences and detailed analytics reporting.

## Features

- **Campaign Management**: Create, update, run, pause and delete email and LinkedIn campaigns
- **Prospect Operations**: Add prospects to your account and campaigns, update and delete prospect data, search prospects
- **Email Composition**: Create multistep email sequences with A/B testing capabilities
- **Analytics & Reporting**: Retrieve campaign statistics and performance metrics
- **Account Integration**: Assign email and LinkedIn accounts to campaigns
- **Agency Companies**: List accessible companies and select a company for each tool call
- **Advanced Configuration**: Support for delivery schedules, timezone settings and GDPR compliance

## Installation

### Prerequisites

Before setting up the integration, ensure you have:

- **Woodpecker Account**: An account with the API & Integration add-on enabled
- **API Credentials**: Woodpecker API key (found in your account settings)
- **Docker**: Installed on your system (for running the MCP server) [https://docs.docker.com/desktop/](https://docs.docker.com/desktop/)
- **AI Agent**: Claude Desktop, Continue.dev or another MCP-compatible AI platform

### Getting Your Woodpecker API Key

1. Log in to the Woodpecker account ([https://login.woodpecker.co/](https://login.woodpecker.co/))
2. Go to the Marketplace in the top-right corner → Integrations → 'API keys'
3. Click `Create a key`

### AI Agent Configuration

#### Claude Desktop

##### Step 1: Locate Configuration File

Find your Claude Desktop configuration file:

**macOS:**
```
~/Library/Application Support/Claude/claude_desktop_config.json
```

**Windows:**
```
%APPDATA%\Claude\claude_desktop_config.json
```

**Linux:**
```
~/.config/Claude/claude_desktop_config.json
```

##### Step 2: Add MCP Server Configuration

Edit the configuration file to include the Woodpecker MCP server:

```json
{
    "mcpServers": {
        "woodpecker": {
            "command": "docker",
            "args": [
                "run",
                "-i",
                "--rm",
                "-e",
                "WOODPECKER_API_KEY",
                "woodpeckerco/woodpecker-mcp-server"
            ],
            "env": {
                "WOODPECKER_API_KEY": "<YOUR_API_KEY>"
            }
        }
    }
}
```

##### Step 3: Restart Claude Desktop

1. Quit Claude Desktop completely
2. Restart the application
3. Look for the Woodpecker tools in the interface

### Other MCP-Compatible Agents

For other AI platforms that support MCP:

1. **Ensure MCP Support**: Verify your AI agent supports the Model Context Protocol
2. **Authentication**: Provide Woodpecker API credentials (`WOODPECKER_API_KEY`) through environment variables
3. **Protocol**: Use stdio transport as supported by your agent

### Verification and Testing

In your AI agent, try asking:

```
"What Woodpecker tools do you have access to?"
"List my Woodpecker campaigns"
```

### Troubleshooting

#### MCP Server Not Connecting

**Symptoms:** AI agent can't access Woodpecker tools

**Solutions:**
1. Try absolute Docker path (e.g. `/usr/local/bin/docker`) in MCP Server Configuration
2. Check if Docker container is running: `docker ps`
3. Test API key: Make direct API call to Woodpecker

#### Invalid API Credentials

**Symptoms:** "Unauthorized" or "Invalid API key" errors

**Solutions:**
1. Regenerate API key in Woodpecker settings
2. Update environment variables with new key
3. Restart AI Agent application

#### Tools Not Available

**Symptoms:** AI agent doesn't see Woodpecker functions

**Solutions:**
1. Check MCP server configuration in AI agent
2. Restart AI agent application
3. Check AI agent logs for MCP connection errors

#### Campaign Creation Fails

**Symptoms:** Error when creating campaigns

**Solutions:**
1. Ensure you have at least one email account configured in Woodpecker
2. Verify email account IDs with `listMailboxes` function
3. Check campaign parameters match required format
4. Ensure daily enrollment limit is > 0

## Usage Examples

### Creating Your First Campaign

```
"Create a new email campaign with these details:
- Name: Product Demo Outreach
- Subject: Quick demo of our new feature
- Message: Hi {{FIRST_NAME}}, I'd love to show {{COMPANY}} our new feature. Are you available for a 15-minute demo?
- Send Monday-Friday, 9 AM to 5 PM
- Daily limit: 25 prospects
- Use my main email account"
```

### Adding Prospects in Bulk

```
"Add these prospects to campaign 12345:
1. John Doe, john@example.com, Example Corp, Marketing Director
2. Jane Smith, jane@example.com, Tech Solutions, CEO
3. Bob Johnson, bob@example.com, Startup Inc, CTO

Use this personalization for each:
- John: 'I saw your recent blog post about email marketing'
- Jane: 'Congratulations on your recent funding round'
- Bob: 'Your product launch looked impressive'"
```

### Campaign Management

```
"Pause campaign 12345 and show me its performance statistics"
"Update campaign 'Product Demo Outreach' to send only 15 prospects per day"
"Add a follow-up email to campaign 12345 that sends 3 days after the first email"
```

### Analytics and Reporting

```
"Show me statistics for all my running campaigns"
"Which campaign has the highest open rate?"
"Create a summary report of my campaign performance this month"
```

## Best Practices

### Campaign Management
1. **Start Small**: Test with 5-10 prospects before scaling
2. **Monitor Performance**: Check statistics regularly
3. **Respect Limits**: Stay within daily enrollment limits
4. **A/B Testing**: Use multiple email versions for optimization

### Prospect Data
1. **Data Quality**: Ensure email addresses are valid
2. **Personalization**: Use meaningful snippets and custom fields
3. **Segmentation**: Organize prospects with tags and industries
4. **Compliance**: Include unsubscribe links and respect opt-outs

### AI Agent Interactions
1. **Be Specific**: Provide clear instructions for campaign creation
2. **Verify Results**: Check campaign details before running
3. **Use Examples**: Include sample content and personalization
4. **Monitor Automation**: Review AI-generated campaigns before deployment

## Support

Contact Woodpecker Support Team at [Woodpecker Support](https://woodpecker.co/contact-support/)

## License

Permission is granted to use this Docker image for personal and commercial purposes.
Redistribution, modification and reverse engineering are prohibited.
No warranty is provided.

Copyright 2025 Woodpecker.co S.A.

## Available Tools

For agency accounts, use `listCompanies` to find a company ID. Pass its
`companyId` on each tool call for that company. Omit `companyId` to use the
main account represented by the API key; company selection does not carry over
between calls.

For campaign and step creation, `deliveryTimes` contains `deliveryDays`
(an array of weekday names from `MONDAY` through `SUNDAY`),
`deliveryTimeStart`, and `deliveryTimeStop` (times such as `09:30`).
`followupAfter` contains `range` (`DAY`, `HOUR`, or `MINUTE`) and `value`
(number). Email `versions` contain optional `subject` and `body` strings;
the body is HTML without enclosing `html` or `body` tags.

### Campaign Management

#### `createEmailCampaign`
Create a Woodpecker email campaign with one email step and minimal configuration.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `name` (string, optional): Campaign name
- `timezone` (string, optional): Campaign timezone (defaults to user's timezone if omitted)
- `dailyEnroll` (number, optional): Daily prospect enrollment limit (safe default if omitted)
- `emailAccountIds` (array, optional): SMTP account IDs
- `versions` (array, optional): Email content versions (max 5; one empty version if omitted)
- `deliveryTimes` (object): Delivery times
- `followupAfter` (object, optional): Delay before processing the next step (1 day if omitted)

#### `createLinkedinProfileVisitCampaign`
Create a Woodpecker LinkedIn campaign with one profile visit step.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `name` (string, optional): Campaign name
- `timezone` (string, optional): Campaign timezone (defaults to user's timezone if omitted)
- `dailyEnroll` (number, optional): Daily prospect enrollment limit (safe default if omitted)
- `linkedinAccountId` (number, optional): LinkedIn account ID
- `followupAfter` (object, optional): Delay before processing the next step (1 day if omitted)

#### `createLinkedinConnectionRequestCampaign`
Create a Woodpecker LinkedIn campaign with one connection request step.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `name` (string, optional): Campaign name
- `timezone` (string, optional): Campaign timezone (defaults to user's timezone if omitted)
- `dailyEnroll` (number, optional): Daily prospect enrollment limit (safe default if omitted)
- `linkedinAccountId` (number, optional): LinkedIn account ID
- `followupAfter` (object, optional): Delay before processing the next step (1 day if omitted)
- `bodyVersions` (array, optional): Connection request message versions (max 5; one empty version if omitted)

#### `createLinkedinDirectMessageCampaign`
Create a Woodpecker LinkedIn campaign with one direct message step.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `name` (string, optional): Campaign name
- `timezone` (string, optional): Campaign timezone (defaults to user's timezone if omitted)
- `dailyEnroll` (number, optional): Daily prospect enrollment limit (safe default if omitted)
- `linkedinAccountId` (number, optional): LinkedIn account ID
- `followupAfter` (object, optional): Delay before processing the next step (1 day if omitted)
- `bodyVersions` (array, optional): Direct message content versions (max 5; one empty version if omitted)

#### `createLinkedinInMailMessageCampaign`
Create a Woodpecker LinkedIn campaign with one InMail message step.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `name` (string, optional): Campaign name
- `timezone` (string, optional): Campaign timezone (defaults to user's timezone if omitted)
- `dailyEnroll` (number, optional): Daily prospect enrollment limit (safe default if omitted)
- `linkedinAccountId` (number, optional): LinkedIn account ID
- `followupAfter` (object, optional): Delay before processing the next step (1 day if omitted)
- `bodyVersions` (array of objects, optional): InMail versions with optional `subject` and `body` strings (max 5; one version with no subject and an empty body if omitted)

#### `listCampaigns`
Retrieve campaigns with optional status filtering.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `pageNumber` (number): Page number (1-based)
- `statuses` (array, optional): Filter by status (RUNNING, DRAFT, EDITED, PAUSED, STOPPED, COMPLETED)

#### `retrieveCampaignDetails`
Get detailed campaign structure including all steps and configurations.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID

#### `retrieveCampaignStatistics`
Fetch campaign performance metrics and analytics.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID

#### `updateCampaignSettings`
Modify campaign-wide settings including name, email accounts, daily limits and timezone.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID
- `name` (string, optional): Campaign name
- `emailAccountIds` (array, optional): List of email account IDs
- `timezone` (string, optional): Campaign timezone
- `dailyEnroll` (number): Daily enrollment limit

#### `buildCampaignUrl`
Generate Woodpecker app URL for campaign access.

**Parameters:**
- `companyId` (number, optional): Agency company ID returned by `listCompanies`; omit to build a URL for the current account
- `campaignId` (number): Campaign ID

#### Campaign Control

Each campaign control tool also accepts optional `companyId` for an agency company.

- `runCampaign(campaignId)`: Start campaign execution
- `pauseCampaign(campaignId)`: Pause campaign
- `stopCampaign(campaignId)`: Stop campaign
- `deleteCampaign(campaignId)`: Remove campaign entirely
- `makeCampaignEditable(campaignId)`: Enable campaign modifications

### Step Management

#### `addEmailStep`
Add an email follow-up step to a campaign.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID
- `parentId` (string): Parent step ID
- `versions` (array, optional): Email content versions (max 5; one empty version if omitted)
- `deliveryTimes` (object): Delivery times
- `followupAfter` (object, optional): Delay before processing the next step (1 day if omitted)

#### `addLinkedinProfileVisitStep`
Add a LinkedIn profile visit follow-up step to a campaign.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID
- `parentId` (string): Parent step ID
- `linkedinAccountId` (number, optional): LinkedIn account ID
- `followupAfter` (object, optional): Delay before processing the next step (1 day if omitted)

#### `addLinkedinConnectionRequestStep`
Add a LinkedIn connection request follow-up step to a campaign.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID
- `parentId` (string): Parent step ID
- `linkedinAccountId` (number, optional): LinkedIn account ID
- `followupAfter` (object, optional): Delay before processing the next step (1 day if omitted)
- `bodyVersions` (array, optional): Connection request message versions (max 5; one empty version if omitted)

#### `addLinkedinDirectMessageStep`
Add a LinkedIn direct message follow-up step to a campaign.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID
- `parentId` (string): Parent step ID
- `linkedinAccountId` (number, optional): LinkedIn account ID
- `followupAfter` (object, optional): Delay before processing the next step (1 day if omitted)
- `bodyVersions` (array, optional): Direct message content versions (max 5; one empty version if omitted)

#### `addLinkedinInMailMessageStep`
Add a LinkedIn InMail message follow-up step to a campaign in DRAFT or EDITED status.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID
- `parentId` (string): Parent step ID
- `linkedinAccountId` (number, optional): LinkedIn account ID
- `followupAfter` (object, optional): Delay before processing the next step (1 day if omitted)
- `bodyVersions` (array of objects, optional): InMail versions with optional `subject` and `body` strings (max 5; one version with no subject and an empty body if omitted)

#### `updateCampaignStep`
Modify step delivery times and scheduling.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID
- `stepId` (string): Step ID
- `payload` (string): Updated delivery configuration

#### `updateEmailStepVersion`
Update email content, subject lines, signatures and tracking settings.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID
- `stepId` (string): Step ID
- `versionId` (string): Version ID
- `subject` (string): Email subject
- `message` (string): Email body (HTML supported)
- `signature` (string): SENDER or NO_SIGNATURE
- `trackOpens` (boolean): Enable open tracking

#### `updateLinkedinStepVersion`
Update LinkedIn connection request, direct message, or InMail content.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID
- `stepId` (string): Step ID
- `versionId` (string): Version ID
- `subject` (string, optional): InMail subject, at most 200 characters and without snippets; omit for other LinkedIn step types
- `message` (string): Message body content

#### `deleteCampaignStep`
Remove steps from campaigns.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID
- `stepId` (string): Step ID

### Prospect Operations

#### `addProspectsToDatabase`
Adds new prospects to your global prospect list without enrolling them in any campaign.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `prospectsPayload` (string): JSON array of prospect data

**Notes:**
- Prospects are added to your account but not to any campaign
- Available for future campaign enrollment
- Useful for building a prospect database before campaign creation

#### `addProspectsToCampaign`
Bulk add prospects with full contact information and custom snippets.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID
- `prospectsPayload` (string): Array of prospect objects

**Note**: Always check for `DUPLICATE` prospects in response. Use `updateProspectsInCampaign` for duplicates if data updates are needed.

#### `updateProspectsInDatabase`
Updates existing prospects in your global database or adds new ones if they don't exist.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `prospectsPayload` (string): JSON array of prospect data with updates

**Notes:**
- Existing prospects are updated based on email address
- New prospects are added if email doesn't exist
- Only include fields you want to update
- Updates apply globally (affects all campaigns using these prospects)

#### `updateProspectsInCampaign`
Update existing prospect data (requires explicit user request).

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID
- `prospectsPayload` (string): Array of prospect objects with updates

#### `listProspectsInDatabase`
Lists prospects from your global prospect database (not tied to any specific campaign).

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `pageNumber` (integer): Page number (1-based indexing)

**Notes:**
- Returns paginated results of all prospects in your account
- These prospects can be added to any campaign
- Useful for managing your overall prospect database

#### `listProspectsInCampaign`
Paginated retrieval of campaign prospects.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `campaignId` (number): Campaign ID
- `pageNumber` (number): Page number (1-based)

#### `searchProspects`
Searches for prospects that match specific criteria across your entire database.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `pageNumber` (integer): Page number (1-based indexing)
- `searchCriteria` (object, optional): Search field names mapped to arrays of string values
- `filterCriteria` (object, optional): Filter field names mapped to string values

**Available search fields:**
- `email` - Email address
- `first_name` - First name
- `last_name` - Last name
- `company` - Company name
- `organization_id` - Organization ID
- `industry` - Industry
- `website` - Website URL
- `tags` - Tags (case-sensitive, without leading #)
- `title` - Job title
- `phone` - Phone number
- `address` - Street address
- `city` - City
- `state` - State/Province
- `country` - Country
- `snippet1` through `snippet15` - Custom fields

**Available filter fields:**
- `id` - Comma-separated list of prospect IDs
- `status` - Prospect's global status: ACTIVE, BOUNCED, REPLIED, BLACKLIST, INVALID
- `campaigns_id` - Comma-separated list of campaign IDs that prospects are enrolled in
- `contacted` - Whether a prospect has ever been contacted
- `interested` - Interest level: INTERESTED, MAYBE-LATER, NOT-INTERESTED, NOT-MARKED

**Notes:**
- Tag searching is case-sensitive, don't use leading # when searching by tags
- Search criteria uses OR for same field, AND for different fields
- To filter OPT-OUT prospects, use "BLACKLIST" status
- Multiple filter values are comma-separated

#### `deleteProspects`
Permanently deletes prospects from your database and/or specific campaigns.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `prospectIds` (string): Comma-separated list of prospect IDs to delete
- `campaignIds` (string, optional): Comma-separated list of campaign IDs from which to remove prospects

**Notes:**
- **Without campaignIds**: Deletes prospects globally from your entire database
- **With campaignIds**: Removes prospects only from specified campaigns
- This action is permanent and cannot be undone
- Requires explicit user confirmation before execution
- Use prospect IDs (not email addresses) obtained from list/search operations

**Warning:** Global deletion removes prospects from all campaigns and your database. Local deletion (with campaignIds) only removes them from specified campaigns while keeping them in your global database.

### Account Management

#### `listCompanies`
List one page of up to 50 companies available through an agency account.

**Parameters:**
- `pageNumber` (integer): Page number (1-based indexing)
- `active` (boolean, optional): Filter by active status

#### `listMailboxes`
Retrieve available email accounts for campaign assignment.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `filter` (string): Filter to use when listing mailboxes (SMTP, SMTP_AVAILABLE_FOR_CAMPAIGN, IMAP, ALL)

#### `listLinkedinAccounts`
Retrieve available LinkedIn accounts for campaign assignment.

**Parameters:**
- `companyId` (number, optional): Agency company ID from `listCompanies`; omit for the current account
- `filter` (string): Filter to use when listing accounts (ALL, AVAILABLE_FOR_CAMPAIGN)

## Changelog

### v0.1.0 (2026-09-16)
- Support agency company selection across Woodpecker tools and add the `listCompanies` tool
- Improve choosing mailboxes available for a campaign
- Require additional validation before running a campaign
- Support LinkedIn InMail steps
- Loosen validation and add default values in various create campaign and add step tools
- Campaign management improvements
- Support Linkedin campaigns
- Requested returning more details when listing campaigns, mailboxes and prospects
- **Behavioral compatibility note:** This version introduces the agency HQ and client company account structure. Older or less capable LLMs may not reliably distinguish between these account scopes or pass `companyId` consistently, which can cause a tool to operate on the current account instead of the intended client company.

### v0.0.9 (2025-07-28)
- Internal improvements and bug fixes

### v0.0.8 (2025-06-24)
- Added global prospects list tools

### v0.0.7 (2025-06-11)
- Initial release with campaigns related tools

## Documentation
- **Woodpecker MCP Server**: [https://developers.woodpecker.co/docs/mcp/](https://developers.woodpecker.co/docs/mcp/)
- **Woodpecker API**: [https://developers.woodpecker.co/docs/](https://developers.woodpecker.co/docs/)
- **MCP Protocol**: [https://modelcontextprotocol.io](https://modelcontextprotocol.io)
- **Claude Desktop**: [https://claude.ai/download](https://claude.ai/download)
