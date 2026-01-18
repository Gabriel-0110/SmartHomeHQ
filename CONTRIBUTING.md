# Contributing to SmartHomeHQ

Thank you for your interest in contributing to SmartHomeHQ! This repository serves as documentation for a personal smart home setup, but contributions and suggestions are welcome.

## How to Contribute

### Documentation Updates

1. **Fork the repository** and create a new branch for your changes
2. **Make your changes** following the documentation standards below
3. **Submit a pull request** with a clear description of your changes

### Reporting Issues

- Use GitHub Issues to report documentation errors or suggest improvements
- Include specific file paths and line numbers when reporting errors
- Suggest corrections or improvements when possible

## Documentation Standards

### File Format

- **Markdown**: Use `.md` extension for all documentation files
- **CSV**: Use `.csv` for inventory and tabular data
- **Images**: Use PNG or JPEG formats, store in relevant folder

### Naming Conventions

- Use lowercase with hyphens for file names: `device-setup.md`
- Use descriptive names that reflect content: `network-topology.md`
- Prefix configuration files with date if version-specific: `2026-01-config.yaml`

### Markdown Style

- Use clear, descriptive headings (# H1, ## H2, ### H3)
- Include table of contents for documents longer than 200 lines
- Use code blocks with language specification: \`\`\`yaml
- Use tables for structured data when appropriate
- Include links to related documentation
- Keep line length reasonable (80-120 characters recommended)

### Content Guidelines

- **Be specific**: Include model numbers, versions, and exact configurations
- **Be current**: Update dates when modifying configurations
- **Be secure**: Never commit passwords, API keys, or sensitive information
- **Be clear**: Write for your future self who may not remember the setup

### Documentation Structure

Each folder should contain:

1. **README.md**: Overview and index of contents
2. **Template files**: Standardized formats for common documentation types
3. **Specific guides**: Detailed documentation for configurations and setups

### Security Guidelines

⚠️ **CRITICAL**: Never commit sensitive information:

- Passwords or API keys
- Private IP addresses (document using placeholders like `192.168.1.x`)
- MAC addresses
- Personal information
- WiFi credentials
- SSH keys or certificates

Instead, use:
- Placeholder values: `API_KEY_HERE`
- Environment variables: `${API_KEY}`
- References to external secrets management
- Generic examples that users can adapt

## Pull Request Process

1. Update the README.md or relevant index if adding new files
2. Follow the existing folder structure
3. Ensure all markdown is properly formatted
4. Remove any temporary or personal information
5. Update the roadmap if adding significant new documentation sections

## Questions or Suggestions?

- Open an issue for general questions
- Tag issues appropriately (documentation, enhancement, bug)
- Be respectful and constructive in all interactions

## Code of Conduct

- Be professional and respectful
- Focus on constructive feedback
- Help others learn and improve

Thank you for contributing to better smart home documentation!
