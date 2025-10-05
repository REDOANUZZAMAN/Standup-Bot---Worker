# 🤖 Standup Bot - Worker

An intelligent n8n workflow that automates hiring signal detection and lead management. Automatically enriches contact data, manages CRM records, triggers outreach campaigns, and notifies teams based on company status.

![Status](https://img.shields.io/badge/status-active-success.svg)
![n8n](https://img.shields.io/badge/n8n-workflow-FF6D5A?style=flat&logo=n8n&logoColor=white)
![HubSpot](https://img.shields.io/badge/HubSpot-FF7A59?style=flat&logo=hubspot&logoColor=white)
![Automation](https://img.shields.io/badge/automation-100%25-blue.svg)

## ✨ Features

- **Webhook Integration** - Real-time hiring signal capture from Lonescale
- **Contact Enrichment** - Automatic email finding and verification via Dropcontact
- **Smart CRM Management** - Intelligent company and contact creation/updates in HubSpot
- **Multi-Channel Outreach** - Automated Lemlist campaigns and LinkedIn task creation
- **Conditional Routing** - Different actions based on lead status (New, Contacted, Deal, Customer)
- **Team Notifications** - Targeted Slack alerts to Sales and Customer Success teams
- **Task Automation** - Auto-generated follow-up tasks with context
- **Error Handling** - Robust workflow with fallback options

## 🎯 Use Case

Perfect for sales and marketing teams who want to:
- Capture buying signals from job postings
- Automate lead qualification and enrichment
- Trigger personalized outreach at scale
- Keep teams informed of opportunities
- Maintain clean CRM data automatically

## 🚀 Quick Start

### 1. Import Workflow

```bash
# In n8n interface:
# 1. Go to Workflows → Import from File
# 2. Upload the JSON file
# 3. Workflow will be imported with all nodes
```

### 2. Configure Credentials

Set up authentication for each service:

| Service | Auth Type | Required Scopes |
|---------|-----------|----------------|
| Dropcontact | API Key | Full access |
| HubSpot | OAuth2 | contacts, companies, engagements |
| Lemlist | API Key | campaigns.write |
| Slack | OAuth2 | chat:write, channels:read |

### 3. Activate Webhook

```bash
# 1. Open "Lonescale - New Job Intent" node
# 2. Click "Test workflow"
# 3. Copy the Production webhook URL
# 4. Configure in Lonescale settings
```

### 4. Update Configuration

```javascript
// Update Slack channels
channel: "your-sales-channel"
channel: "your-cs-channel"

// Update Lemlist campaign
campaignId: "your-campaign-id"

// Update HubSpot properties (if needed)
properties: ["custom_field_1", "custom_field_2"]
```

### 5. Activate & Test

```bash
# 1. Set workflow to "Active"
# 2. Trigger a test webhook from Lonescale
# 3. Monitor execution in n8n
# 4. Verify data in HubSpot, Lemlist, and Slack
```

## 📁 Workflow Structure

```
Webhook Trigger (Lonescale)
    ↓
Email Enrichment (Dropcontact)
    ↓
Company Search (HubSpot)
    ↓
    ├─ Exists? → Update Company
    └─ New? → Create Company
    ↓
Contact Search & Create/Update
    ↓
Status-Based Routing:
    ├─ NEW/OPEN → Lemlist Campaign or LinkedIn Task
    ├─ ATTEMPTED_TO_CONTACT → Follow-up Task
    ├─ OPEN_DEAL → Sales Notification
    └─ CUSTOMER → CS Notification
```

## 🔄 Data Flow

### Input (Lonescale Webhook)
```json
{
  "company_name": "Acme Corp",
  "company_domain": "acme.com",
  "people_first_name": "John",
  "people_last_name": "Doe",
  "people_linkedin_url": "linkedin.com/in/johndoe",
  "job_offers": [{
    "job_name": "Sales Manager",
    "job_link": "...",
    "context_keywords": "..."
  }]
}
```

### Output Actions

| Lead Status | Action | Destination |
|------------|--------|-------------|
| New Lead | Add to campaign | Lemlist |
| New Lead (no email) | Create task | HubSpot |
| Contacted | Follow-up task | HubSpot |
| Open Deal | Notification | Slack (Sales) |
| Customer | Notification | Slack (CS) |

## 🎨 Customization

### Modify Lead Status Logic

```javascript
// Edit "New Company?" IF node
conditions: {
  string: [
    { value1: "status", value2: "NEW" },
    { value1: "status", value2: "OPEN" },
    { value1: "status", value2: "YOUR_CUSTOM_STATUS" }
  ]
}
```

### Customize Task Templates

```javascript
// Edit "HubSpot - Follow up task" node
body: `${company_name} is hiring a ${job_title}
      
Custom note: Your additional context
Link: ${job_link}
Keywords: ${keywords}`
```

### Add More Notifications

```javascript
// Duplicate Slack node
// Update channel and conditions
channel: "your-new-channel"
text: "Your custom message with {{ variables }}"
```

### Change Lemlist Campaign Logic

```javascript
// Add conditional routing before Lemlist
// Based on job title, company size, etc.
if (jobTitle.includes("Sales")) {
  campaignId: "sales-campaign"
} else {
  campaignId: "general-campaign"
}
```

## 🛠️ Technologies Used

- **[n8n](https://n8n.io/)** - Workflow automation platform
- **[Lonescale](https://lonescale.com/)** - Hiring signal detection
- **[Dropcontact](https://www.dropcontact.com/)** - Email enrichment
- **[HubSpot](https://www.hubspot.com/)** - CRM platform
- **[Lemlist](https://www.lemlist.com/)** - Email outreach
- **[Slack](https://slack.com/)** - Team communication

## 📊 Workflow Nodes

| Node | Type | Purpose |
|------|------|---------|
| Lonescale - New Job Intent | Webhook | Trigger point |
| Dropcontact | API | Email enrichment |
| HubSpot - Search company | API | Find existing records |
| Is Account in Hubspot | IF | Conditional routing |
| HubSpot Create/Update | API | CRM management |
| New Company? | IF | Status check |
| email found | IF | Email validation |
| Lemlist - Add lead | API | Campaign enrollment |
| HubSpot - Tasks | API | Task creation |
| Slack - Notifications | API | Team alerts |

## 🐛 Troubleshooting

### Webhook Not Triggering

```bash
# Check these items:
1. Workflow is Active (toggle in top-right)
2. Webhook URL is correct in Lonescale
3. Webhook path matches: fe426a62-eee5-4fed-bc74-45d4ac09b338-lonescale
4. Test with manual webhook execution
```

### Email Not Found

```bash
# Expected behavior:
- Dropcontact may not have data for all contacts
- Workflow creates LinkedIn outreach task as fallback
- No workflow failure, continues execution
```

### HubSpot Duplicate Contacts

```bash
# Prevention measures:
1. Workflow searches by email first
2. Uses HubSpot's native deduplication
3. Update operation if contact exists
4. Enable "Continue on Fail" for edge cases
```

### Missing Slack Notifications

```bash
# Verify:
1. Slack OAuth token is valid
2. Bot is invited to target channels
3. Channel names match exactly
4. Check n8n execution log for errors
```

### Lemlist Campaign Errors

```bash
# Common issues:
1. Campaign ID is incorrect
2. Email format is invalid
3. Lead already exists in campaign
4. API rate limits reached
```

## 📝 Best Practices

### 1. Test in Sandbox First
- Use test webhook data
- Verify all integrations work
- Check data mapping is correct

### 2. Monitor Regularly
- Review n8n execution logs weekly
- Check for failed executions
- Validate data quality in HubSpot

### 3. Optimize Performance
- Use execution timeout settings
- Enable error workflow for failures
- Set up execution retention

### 4. Maintain Data Quality
- Review Dropcontact match rate
- Audit HubSpot duplicates monthly
- Update Lemlist campaign regularly

### 5. Document Changes
- Note any workflow modifications
- Keep credential list updated
- Document custom logic additions

## 🔒 Security Considerations

- Store credentials securely in n8n
- Use environment variables for sensitive data
- Rotate API keys regularly
- Limit webhook access with authentication
- Review data access permissions
- Enable audit logging in HubSpot

## 📈 Performance Metrics

Track these KPIs:
- Webhook success rate
- Email enrichment match rate
- HubSpot record creation accuracy
- Lemlist campaign enrollment rate
- Task completion rate
- Average execution time

## 🌐 Browser Requirements

n8n web interface:
- ✅ Chrome (recommended)
- ✅ Firefox
- ✅ Safari
- ✅ Edge

## 💡 Enhancement Ideas

- [ ] Add lead scoring based on job title/seniority
- [ ] Integrate with LinkedIn Sales Navigator
- [ ] Add email validation before Lemlist
- [ ] Create custom HubSpot workflows
- [ ] Add AI-powered lead qualification
- [ ] Implement A/B testing for campaigns
- [ ] Add data enrichment from multiple sources
- [ ] Create executive dashboard for metrics

## 📄 License

This workflow is open source and available under the [MIT License](LICENSE).

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

1. Fork the project
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add lead scoring logic'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 👨‍💻 Author

**Redoanuzzaman**
- GitHub: [@REDOANUZZAMAN](https://github.com/REDOANUZZAMAN)
- Portfolio: [View Work](https://redoan.dev)

## 🙏 Acknowledgments

- n8n community for workflow automation patterns
- HubSpot for comprehensive CRM API
- Lonescale for buying signal intelligence
- The open-source automation community

## 💖 Show Your Support

Give a ⭐️ if this workflow helped you automate your sales process!

## 📞 Support

Need help?
1. Check [n8n Documentation](https://docs.n8n.io/)
2. Visit [n8n Community Forum](https://community.n8n.io/)
3. Open an issue in this repository
4. Contact via GitHub

---

Made with 🤖 and n8n automation

**Last Updated:** October 2025
