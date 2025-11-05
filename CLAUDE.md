# Working with Cisco Workflow JSON Files

## Overview

This repository contains workflow automation definitions for Cisco XDR and Cisco Workflows platforms. These are JSON-formatted workflow definitions that can be imported, exported, and version-controlled.

## Workflow JSON Structure

### Main Workflow Object
```json
{
  "workflow": {
    "unique_name": "definition_workflow_<ID>",
    "name": "Human-readable workflow name",
    "title": "Display title",
    "type": "generic.workflow",
    "variables": [...],
    "properties": {...},
    "actions": [...]
  }
}
```

### Key Components

#### 1. **Variables**
Define inputs, outputs, and local variables:
- `scope`: "input", "output", or "local"
- `type`: "datatype.string", "datatype.integer", "datatype.boolean", "datatype.secure_string"
- `is_required`: Boolean flag for required inputs
- `variable_string_format`: Can be "json", "text", "html", etc.

#### 2. **Properties**
Workflow-level settings:
- `atomic.is_atomic`: Whether workflow is atomic (executes as single unit)
- `delete_workflow_instance`: Auto-delete instances after completion
- `target`: Execution target configuration
  - `no_target: true` for Cisco Workflows (no integration targets)
  - `target_id` or `target_group_id` for XDR (requires integrations)

#### 3. **Actions**
Workflow logic and execution steps:
- `HTTP Request`: API calls to external services
- `Execute Python Script`: Custom Python logic
- `Condition Block`: If/else logic
- `While Loop`: Iteration logic
- `Sub-workflow`: Call other workflows
- `Set Variables`: Update variable values

#### 4. **Categories**
Tags for organizing workflows:
```json
"categories": [
  "category_025W2J2HK90LP3gVw454IoXQ0Qd1anzcY85"
]
```

#### 5. **Subworkflows**
Referenced child workflows included in export.

## Platform Differences: XDR vs Workflows

### Cisco XDR (SecureX Orchestration)
- **Supports**: Module targets, target groups, automation rules, incident triggers
- **Target Format**: `{"override_workflow_target": true, "target_id": "$module_target;..."}`
- **Advanced Features**: Incident automation, integration modules

### Cisco Workflows
- **Simpler System**: No external integration targets
- **Target Format**: `{"no_target": true}`
- **Removed Features**: automation_rules, incident.rule_event, target_groups, module_targets

## Converting XDR Workflows to Cisco Workflows

### Required Changes

1. **Remove Module Targets**
   ```json
   // DELETE entire section
   "module_targets": [...]
   ```

2. **Remove Target Groups**
   ```json
   // DELETE entire section
   "target_groups": {...}
   ```

3. **Replace All Target References**
   ```json
   // FROM:
   "target": {
     "execute_on_this_target_group": true,
     "target_group_id": "target_group_..."
   }

   // TO:
   "target": {
     "no_target": true
   }
   ```

4. **Remove Automation Rules**
   ```json
   // DELETE from properties
   "properties": {
     "automation_rules": { ... },  // REMOVE THIS
     ...
   }
   ```

5. **Remove Rules Section**
   ```json
   // DELETE entire section (if present)
   "rules": {...}
   ```

## Common Tasks

### Importing a Workflow
1. Navigate to Cisco Workflows UI
2. Go to **Import** > **Browse**
3. Select the JSON file
4. Resolve any validation errors
5. Click **Import**

### Exporting a Workflow
1. Select workflow in UI
2. Click **Export**
3. Choose format: **JSON** (default)
4. Save to repository

### Validating JSON
```bash
# Use Python to validate JSON syntax
python3 -m json.tool workflow.json > /dev/null && echo "Valid"

# Or use jq
jq empty workflow.json && echo "Valid"
```

### Finding Workflow References
```bash
# Find all workflows calling a specific sub-workflow
grep -r "workflow_id.*02DCWQY555WLO3" .

# Find workflows with specific variables
jq '.workflow.variables[] | select(.properties.name=="i_agent_task")' workflow.json
```

### Modifying Variables

**Add Input Variable:**
```json
{
  "schema_id": "datatype.string",
  "properties": {
    "value": "",
    "scope": "input",
    "name": "i_new_parameter",
    "type": "datatype.string",
    "description": "Description here",
    "is_required": false,
    "display_on_wizard": false,
    "is_invisible": false
  },
  "unique_name": "variable_workflow_<UNIQUE_ID>",
  "object_type": "variable_workflow"
}
```

### Version Control Best Practices

1. **One workflow per directory**
   ```
   WorkflowName__definition_workflow_ID/
     └── definition_workflow_ID.json
   ```

2. **Commit message format**
   ```
   Workflow Name::author@email.com::description of change
   ```

3. **Use branches for major changes**
   ```bash
   git checkout -b feature/update-agent-logic
   ```

## Troubleshooting

### Common Import Errors

**Error: "missing XDR integration"**
- **Cause**: Workflow references `$module_target` or `target_id`
- **Fix**: Replace with `"no_target": true`

**Error: "Read not found schema errored: incident.rule_event"**
- **Cause**: Workflow has `automation_rules` property
- **Fix**: Remove `automation_rules` from properties and `rules` section

**Error: "Invalid workflow instance"**
- **Cause**: JSON syntax error or malformed structure
- **Fix**: Validate JSON, check for trailing commas, unmatched brackets

### Debugging Workflows

1. **Check execution logs** in workflow instance history
2. **Use HTTP Request activities** with `continue_on_error_status_code: true` to debug API calls
3. **Add Python Script activities** for complex debugging:
   ```python
   import json
   import sys
   print(f"DEBUG: Variable value = {sys.argv[1]}")
   ```

## Key Python Script Patterns

### JSON Processing
```python
import json
import sys

# Read input
data = json.loads(sys.argv[1])

# Process
result = process_data(data)

# Output for workflow
result_json = json.dumps(result)
```

### Date/Time Handling
```python
import datetime
import uuid

# Generate timestamps
time = datetime.datetime.utcnow().isoformat()

# Generate UUIDs
run_id = str(uuid.uuid4())
```

## API Integration Patterns

### HTTP Request with Bearer Auth
```json
{
  "type": "web-service.http_request",
  "properties": {
    "method": "POST",
    "relative_url": "/api/v1/resource",
    "custom_headers": [
      {
        "name": "Authorization",
        "value": "Bearer $workflow.global.api_token$"
      }
    ],
    "body": "$activity.previous.output.json_body$"
  }
}
```

## References

- **Cisco Workflows Documentation**: [Link to docs]
- **Cisco XDR Automation**: [Link to XDR docs]
- **Python 3 Script Reference**: Available functions and modules
- **JSONPath Queries**: For extracting data from JSON responses

## Workflow Naming Conventions

- **Prefix meaning**:
  - `i_` = Input variable
  - `o_` = Output variable
  - `l_` = Local variable
  - `definition_workflow_` = Workflow unique name
  - `definition_activity_` = Activity unique name
  - `variable_workflow_` = Variable unique name

## Additional Resources

- Workflow examples: See `/examples` directory
- Atomic actions: Reusable workflow components
- Target configurations: For API integrations
