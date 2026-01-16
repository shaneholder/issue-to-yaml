# Copilot Instructions for Cloud Environment Request Processing

## Repository Overview

This repository automates the processing of cloud environment requests submitted through GitHub issues. It converts structured issue data into validated YAML configuration files that can be used for infrastructure provisioning.

**Key Capabilities:**
- GitHub issue-based request intake via custom issue template
- Automated conversion from issue format to YAML
- JSON Schema validation for all generated YAML files
- CI/CD automation with GitHub Actions

## Repository Structure

```
.github/
  ├── copilot-instructions.md          # This file - Copilot guidance
  ├── agents/
  │   └── convert-issue-to-yaml.agent.md  # Custom agent for issue-to-YAML conversion
  ├── ISSUE_TEMPLATE/
  │   └── cloud-environment-request.yml   # Issue template for cloud requests
  └── workflows/
      ├── process-cloud-request.yml       # Auto-assigns Copilot to new issues
      └── validate-yaml.yml               # Validates YAML against schema
requests/                               # Output directory for generated YAML files
schemas/
  └── cloud-environment-request.schema.json  # JSON Schema for validation
package.json                            # Node.js dependencies (ajv, js-yaml)
sampleissue.md                         # Example issue format
```

## Custom Agent: `convert-issue-to-yaml`

**When to use:** This specialized agent should be used whenever you need to process a cloud environment request issue.

**What it does:**
- Parses GitHub issue body content following the cloud-environment-request template
- Extracts all field values (environments, application details, compliance, security, etc.)
- Converts data to structured YAML format with proper type conversions (Yes/No → boolean)
- Validates output against `schemas/cloud-environment-request.schema.json`
- Saves YAML file to `requests/` directory with naming format: `<application-name>-<issue-number>.yaml`

**How to use:**

When assigned to an issue, simply instruct Copilot to use the agent:
- "Please use the convert-issue-to-yaml agent to process this issue"
- "Use the convert-issue-to-yaml agent to convert this cloud request to YAML"

The agent handles all parsing, validation, and file creation automatically. It will create a properly formatted YAML file in the `requests/` directory.

## Validating YAML Files

### Manual Validation

To validate a YAML file against the schema manually:

```bash
# Install dependencies first (if not already installed)
npm install

# Validate using ajv-cli
npx ajv validate \
  -s schemas/cloud-environment-request.schema.json \
  -d requests/your-file.yaml \
  --spec=draft7 \
  -c ajv-formats
```

### Automatic Validation

The `validate-yaml.yml` workflow automatically runs on pull requests that modify files in the `requests/` directory. It:
1. Detects changed YAML files
2. Validates each file against the schema
3. Reports validation errors as GitHub check failures

## GitHub Workflows

### 1. `process-cloud-request.yml`
**Trigger:** When an issue is opened or labeled with `cloud-request`

**Actions:**
- Assigns the issue to Copilot
- Adds `copilot-processing` label
- Posts a comment with instructions to use the `convert-issue-to-yaml` agent

### 2. `validate-yaml.yml`
**Trigger:** Pull requests modifying files in `requests/**/*.yaml` or `requests/**/*.yml`

**Actions:**
- Validates all changed YAML files against the JSON Schema
- Reports validation results as PR check status
- Fails the check if any file is invalid

## Schema Structure

The `cloud-environment-request.schema.json` defines the required structure for all YAML files:

**Required sections:**
- `metadata`: Issue tracking and generation info
- `description`: Request description
- `environments`: Production, development, test, staging configurations
- `application`: Name, legal entity, cloud provider, contact email
- `compliance`: HIPAA, CCPA, GDPR, PCI, PIPEDA requirements
- `data`: Classification level (Public/Internal/Confidential/Restricted)
- `governance`: Model (Hybrid/Centralized/Decentralized) and user base
- `configuration`: Network, GitHub, and AI usage settings
- `security`: Data exposure, special controls, GxP standards

**Important validation rules:**
- Boolean fields must be `true` or `false` (not strings)
- Email addresses must be valid format
- Cloud provider must be one of: Azure, AWS, GCP
- Legal entity must be numeric string
- Schema version must follow pattern `\d+\.\d+` (e.g., "1.0")

## Issue Template Format

Cloud environment requests use a structured GitHub issue template (`cloud-environment-request.yml`) that includes:

**Field mapping (Issue → YAML):**
- "Yes"/"No" dropdown values → boolean `true`/`false`
- "true"/"false" dropdown values → boolean `true`/`false`
- Text inputs → string values
- Checkboxes → individual boolean compliance fields

**Key sections in the issue:**
1. Description (textarea)
2. Environment Selection (4 environments: production, dev, test, staging)
3. Application Details (name, legal entity, cloud provider, email)
4. Compliance Requirements (checkboxes for various standards)
5. Data Classification (dropdown)
6. Governance Model (dropdown)
7. Additional Configuration (network, GitHub, AI settings)
8. Security Questions (data exposure, special controls, GxP)

## Testing and Validation

### Before submitting changes:

1. **Validate generated YAML files:**
   ```bash
   npm install
   npx ajv validate -s schemas/cloud-environment-request.schema.json -d requests/*.yaml --spec=draft7 -c ajv-formats
   ```

2. **Check file naming:**
   - Format: `<application-name>-<issue-number>.yaml`
   - Use lowercase with hyphens for application names
   - Example: `customer-portal-123.yaml`

3. **Verify YAML structure:**
   - All required fields present
   - Boolean values are `true`/`false` (not "Yes"/"No" strings)
   - Email addresses are valid
   - Enum values match schema exactly (case-sensitive)

### Example output:

See `requests/example-output.yaml` for a complete example of properly formatted output.

## Common Tasks

### Processing a new cloud request issue
1. Issue is created with `cloud-request` label (automatic via template)
2. Workflow assigns Copilot and adds `copilot-processing` label
3. Instruct Copilot: "Please use the convert-issue-to-yaml agent to process this issue"
4. Agent generates YAML file in `requests/` directory
5. Create PR with the generated file
6. Validation workflow runs automatically on PR

### Updating the schema
1. Modify `schemas/cloud-environment-request.schema.json`
2. Update agent instructions in `.github/agents/convert-issue-to-yaml.agent.md` if field mappings change
3. Update this documentation to reflect schema changes
4. Test with existing YAML files to ensure backward compatibility

### Adding new fields
1. Add field to issue template (`.github/ISSUE_TEMPLATE/cloud-environment-request.yml`)
2. Add field to schema (`schemas/cloud-environment-request.schema.json`)
3. Update agent field mapping in `.github/agents/convert-issue-to-yaml.agent.md`
4. Update this documentation

## Best Practices

1. **Always use the custom agent** for issue-to-YAML conversion rather than manual parsing
2. **Validate before committing** - run schema validation locally before pushing
3. **Use descriptive commit messages** that reference the issue number
4. **Check PR validation** - ensure the validate-yaml workflow passes
5. **Follow naming conventions** for output files
6. **Don't commit to main directly** - always use pull requests for review

## Dependencies

This repository uses Node.js packages for validation:
- `ajv` (v8.17.1+): JSON Schema validator
- `ajv-formats` (v3.0.1+): Additional format validators (email, date-time, etc.)
- `js-yaml` (v4.1.1+): YAML parser and serializer

Install with:
```bash
npm install
```

## Troubleshooting

### Validation fails with "invalid email"
- Ensure `distribution_list_email` field contains a valid email address
- Format: `user@domain.com`

### Validation fails with "must be boolean"
- Check that Yes/No values were converted to `true`/`false` (not strings)
- Compliance and security fields must be boolean, not quoted strings

### Validation fails with "must be one of"
- Verify enum values match schema exactly (case-sensitive)
- Cloud provider: Azure, AWS, or GCP (not lowercase)
- Data classification: Public, Internal, Confidential, or Restricted
- Governance model: Hybrid, Centralized, or Decentralized

### Agent doesn't process issue
- Verify issue has `cloud-request` label
- Check that issue follows the template format
- Review workflow run logs in Actions tab

## Additional Resources

- [GitHub Copilot Best Practices](https://docs.github.com/en/copilot/tutorials/coding-agent/get-the-best-results)
- [JSON Schema Documentation](https://json-schema.org/)
- [GitHub Actions Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
