# Claude Skills Repository

A collection of custom skills that extend Claude's capabilities for specialized tasks and workflows.

## What Are Skills?

Skills are folders of instructions, scripts, and resources that Claude loads dynamically to improve performance on specialized tasks. Think of them as modular expertise modules that teach Claude how to complete specific tasks in a repeatable, consistent way.

Skills enable Claude to:
- **Follow your specific workflows** - Teach Claude your organization's processes and standards
- **Apply brand guidelines** - Ensure consistent styling, tone, and formatting
- **Automate complex tasks** - Break down multi-step processes into reliable, repeatable operations
- **Integrate with your tools** - Work better with your specific software and platforms
- **Maintain quality standards** - Enforce best practices and quality checks

Unlike one-off prompts, skills create persistent, reusable capabilities that Claude can apply across many conversations and use cases.

## Why Use Skills?

**Consistency**: Get reliable, standardized outputs every time without having to re-explain your requirements.

**Efficiency**: Skip lengthy explanations - activate a skill and Claude immediately knows your processes and preferences.

**Scalability**: Share skills across your team so everyone benefits from the same enhanced capabilities.

**Specialization**: Teach Claude domain-specific knowledge, from your company's design system to industry-specific terminology and workflows.

## How Skills Work

When you activate a skill, Claude reads the skill's instructions and resources before starting work. This gives Claude the context needed to complete tasks according to your specific requirements. Skills can contain:

- Detailed instructions and guidelines
- Examples and templates
- Code snippets and scripts
- Best practices and quality standards
- Reference materials and resources

Claude intelligently decides when to use skills based on your request and automatically loads the most relevant ones for the task at hand.

## Skill Structure

Every skill is a self-contained folder with a `SKILL.md` file at its root. Here's the basic structure:

```
my-skill-name/
├── SKILL.md           # Required: Instructions and metadata
├── examples/          # Optional: Example files
├── templates/         # Optional: Template resources
└── scripts/          # Optional: Helper scripts
```

### SKILL.md Format

Each `SKILL.md` file contains YAML frontmatter followed by markdown instructions:

```markdown
---
name: my-skill-name
description: A clear, complete description of what this skill does and when Claude should use it
---

# My Skill Name

## Overview
[Explain what this skill does and its purpose]

## Instructions
[Detailed step-by-step instructions for Claude to follow]

## Examples
[Concrete examples showing expected inputs and outputs]

## Guidelines
[Best practices, quality standards, and edge cases to handle]
```

**Required Fields:**
- `name` - A unique identifier (lowercase, use hyphens instead of spaces)
- `description` - A comprehensive description that helps Claude understand when to use this skill

## Using Skills in This Repository

### In Claude.ai

1. **For Team/Pro subscribers**: Some skills may already be available in Claude.ai's skill library
2. **For custom skills**:
   - Download the skill folder from this repository
   - Go to your Claude.ai account settings
   - Navigate to the Skills section
   - Click "Upload skill" and select the skill folder
   - The skill will now be available in your projects

For detailed instructions, see: [Using skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude)

### In Claude Code

Claude Code can install skills directly from GitHub repositories.

**One-time setup**: Register this repository as a marketplace:
```bash
/plugin marketplace add [your-username]/[your-repo-name]
```

**Installing skills**: 
```bash
/plugin install [skill-name]@[marketplace-name]
```

Or use the interactive browser:
1. Type `/plugin marketplace add [your-username]/[your-repo-name]`
2. Select `Browse and install plugins`
3. Choose your marketplace
4. Select the skill you want
5. Click `Install now`

**Using an installed skill**:
Just mention it naturally in your request:
```
"Use the [skill-name] skill to create a report from this data"
```

For more details, see: [Claude Code Skills Documentation](https://docs.claude.com/en/docs/claude-code/skills)

### Via Claude API

You can use skills programmatically through the Claude API:

**Upload a skill**:
```python
import anthropic

client = anthropic.Anthropic(api_key="your-api-key")

# Read your SKILL.md file
with open("path/to/SKILL.md", "r") as f:
    skill_content = f.read()

# Create the skill
skill = client.skills.create(
    name="my-skill-name",
    description="Complete description of what this skill does",
    skill_markdown=skill_content
)
```

**Use a skill in a conversation**:
```python
message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    skills=[{"type": "skill", "skill_id": skill.id}],
    messages=[
        {"role": "user", "content": "Your request here"}
    ]
)
```

For comprehensive API documentation, see: [Skills API Quickstart](https://docs.claude.com/en/api/skills-guide)

## Skills in This Repository

<!-- List your skills here with brief descriptions -->

### [Skill Name 1]
Brief description of what this skill does and when to use it.

### [Skill Name 2]
Brief description of what this skill does and when to use it.

<!-- Add more skills as needed -->

## Creating Your Own Skills

Want to create a new skill? Follow these steps:

1. **Create a new folder** with a descriptive, hyphenated name
2. **Add a SKILL.md file** with proper frontmatter and instructions
3. **Test thoroughly** - Try the skill with various inputs to ensure it works reliably
4. **Document clearly** - Include examples, edge cases, and guidelines
5. **Iterate** - Refine based on real-world usage

### Skill Development Best Practices

**Be Specific**: Provide concrete, detailed instructions rather than vague guidance.

**Include Examples**: Show Claude exactly what good outputs look like.

**Handle Edge Cases**: Anticipate and document how to handle unusual situations.

**Test Iteratively**: Start simple and add complexity based on what works.

**Use Clear Language**: Write instructions as if explaining to a knowledgeable colleague.

For an in-depth guide, see: [Creating Custom Skills](https://support.claude.com/en/articles/12512198-creating-custom-skills)

## Resources

### Official Documentation
- [What are skills?](https://support.claude.com/en/articles/12512176-what-are-skills) - Overview and introduction
- [Using skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude) - Usage guide
- [Creating custom skills](https://support.claude.com/en/articles/12512198-creating-custom-skills) - Development guide
- [Claude Code Skills](https://docs.claude.com/en/docs/claude-code/skills) - Claude Code integration
- [Skills API Guide](https://docs.claude.com/en/api/skills-guide) - API reference

### Blog Posts & Articles
- [Introducing Skills](https://www.anthropic.com/news/skills) - Official announcement
- [Equipping agents for the real world with Agent Skills](https://anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) - Technical deep dive

### Example Repositories
- [Anthropic Skills Repository](https://github.com/anthropics/skills) - Official examples and document skills

## Requirements

- **For Claude.ai**: Team or Pro subscription (for custom skill uploads)
- **For Claude Code**: Claude Code CLI tool installed
- **For API**: Anthropic API key with skills access

## Contributing

<!-- Customize this section based on whether you want contributions -->

Contributions are welcome! Please:
1. Fork this repository
2. Create a new branch for your skill
3. Follow the skill structure guidelines
4. Test your skill thoroughly
5. Submit a pull request with a clear description

## License

<!-- Add your chosen license here -->

## Support

For issues or questions:
- Open an issue in this repository
- Check the [official documentation](https://docs.claude.com)
- Visit [Anthropic Support](https://support.claude.com)

---

**Note**: These skills are provided as-is. Always test skills thoroughly in your own environment before relying on them for critical tasks.
