# AGENTS.md

## About BobKit and BobKit

**GitHub BobKit** is a comprehensive toolkit for implementing Spec-Driven Development (SDD) - a methodology that emphasizes creating clear specifications before implementation. The toolkit includes templates, scripts, and workflows that guide development teams through a structured approach to building software.

**BobKit** is the command-line interface that bootstraps projects with the BobKit framework. It sets up the necessary directory structures, templates, and AI agent integrations to support the Spec-Driven Development workflow.

The toolkit supports multiple AI coding assistants, allowing teams to use their preferred tools while maintaining consistent project structure and development practices.

---

## General practices

- Any changes to `__init__.py` for the BobKit require a version rev in `pyproject.toml` and addition of entries to `CHANGELOG.md`.

## Adding New Agent Support

This section explains how to add support for new AI agents/assistants to the BobKit. Use this guide as a reference when integrating new AI tools into the Spec-Driven Development workflow.

### Overview

BobKit supports multiple AI agents by generating agent-specific command files and directory structures when initializing projects. Each agent has its own conventions for:

- **Command file formats** (Markdown, TOML, etc.)
- **Directory structures** (`.claude/commands/`, `.cursor/commands/`, etc.)
- **Command invocation patterns** (slash commands, CLI tools, etc.)
- **Argument passing conventions** (`$ARGUMENTS`, `{{args}}`, etc.)

### Current Supported Agents

| Agent | Directory | Format | CLI Tool | Description |
|-------|-----------|---------|----------|-------------|
| **Bob-IDE** | `.bob/commands/` | Markdown | N/A (IDE-based) | Bob-IDE in VS Code |


### Step-by-Step Integration Guide

Follow these steps to add a new agent (using Bob-IDE as an example):

#### 1. Update AI_CHOICES Constant

Add the new agent to the `AI_CHOICES` dictionary in `src/bobkit/__init__.py`:

```python
AI_CHOICES = {
    "bob-ide": "Bob-IDE",
}
```

Also update the `agent_folder_map` in the same file to include the new agent's folder for the security notice:

```python
agent_folder_map = {
    "bob-ide": ".bob/",
}
```

#### 2. Update CLI Help Text

Update all help text and examples to include the new agent:

- Command option help: `--ai` parameter description
- Function docstrings and examples
- Error messages with agent lists

#### 3. Update README Documentation

Update the **Supported AI Agents** section in `README.md` to include the new agent:

- Add the new agent to the table with appropriate support level (Full/Partial)
- Include the agent's official website link
- Add any relevant notes about the agent's implementation
- Ensure the table formatting remains aligned and consistent

#### 4. Update Release Package Script

Modify `.github/workflows/scripts/create-release-packages.sh`:

##### Add to ALL_AGENTS array:
```bash
ALL_AGENTS=(bob-ide)
```

##### Add case statement for directory structure:
```bash
case $agent in
  bob-ide)
    mkdir -p "$base_dir/.bob/commands"
    mkdir -p "$base_dir/.bob/rules"
    generate_commands bob-ide prompt.md "\$ARGUMENTS" "$base_dir/.bob/commands" "$script" ;;
esac
```

#### 4. Update GitHub Release Script

Modify `.github/workflows/scripts/create-github-release.sh` to include the new agent's packages:

```bash
gh release create "$VERSION" \
  .genreleases/BobKit-template-bob-ide-sh-"$VERSION".zip \
  .genreleases/BobKit-template-bob-ide-ps-"$VERSION".zip \
```

#### 5. Update Agent Context Scripts

##### Bash script (`scripts/bash/update-agent-context.sh`):

Add to case statement:
```bash
case "$AGENT_TYPE" in
  bob-ide) update_agent_file "$COPILOT_FILE" "Bob-IDE" ;;
  "")
    if [[ -f "$COPILOT_FILE" ]]; then
      update_agent_file "$COPILOT_FILE" "Bob-IDE"
    fi
    ;;
esac
```

##### PowerShell script (`scripts/powershell/update-agent-context.ps1`):

Add file variable:
```powershell
$windsurfFile = Join-Path $repoRoot '.windsurf/rules/specify-rules.md'
```

Add to switch statement:
```powershell
switch ($AgentType) {
    'bob-ide' { Update-AgentFile $copilotFile 'Bob-IDE' }
    '' {
        if (Test-Path $copilotFile) { Update-AgentFile $copilotFile 'Bob-IDE' }
    }
}
```

#### 6. Update CLI Tool Checks (Optional)

For agents that require CLI tools, add checks in the `check()` command and agent validation:

```python
# In check() command

```

**Note**: Skip CLI checks for IDE-based agents (Copilot, Windsurf).

## Agent Categories

### IDE-Based Agents
Work within integrated development environments:
- **Bob-IDE**: Built into VS Code/compatible editors

## Command File Formats

### Markdown Format
Used by: Claude, Cursor, opencode, Windsurf

```markdown
---
description: "Command description"
---

Command content with {SCRIPT} and $ARGUMENTS placeholders.
```

### TOML Format
Used by: (No longer used in current agents)

```toml
description = "Command description"

prompt = """
Command content with {SCRIPT} and {{args}} placeholders.
"""
```

## Directory Conventions

- **IDE agents**: Follow IDE-specific patterns:
  - Bob-IDE: `.bob/commands/`

## Argument Patterns

Different agents use different argument placeholders:
- **Markdown/prompt-based**: `$ARGUMENTS`
- **TOML-based**: `{{args}}`
- **Script placeholders**: `{SCRIPT}` (replaced with actual script path)
- **Agent placeholders**: `__AGENT__` (replaced with agent name)

## Testing New Agent Integration

1. **Build test**: Run package creation script locally
2. **CLI test**: Test `bobkit init --ai <agent>` command
3. **File generation**: Verify correct directory structure and files
4. **Command validation**: Ensure generated commands work with the agent
5. **Context update**: Test agent context update scripts

## Common Pitfalls

1. **Forgetting update scripts**: Both bash and PowerShell scripts must be updated
2. **Missing CLI checks**: Only add for agents that actually have CLI tools
3. **Wrong argument format**: Use correct placeholder format for each agent type
4. **Directory naming**: Follow agent-specific conventions exactly
5. **Help text inconsistency**: Update all user-facing text consistently

## Future Considerations

When adding new agents:
- Consider the agent's native command/workflow patterns
- Ensure compatibility with the Spec-Driven Development process
- Document any special requirements or limitations
- Update this guide with lessons learned

---

*This documentation should be updated whenever new agents are added to maintain accuracy and completeness.*