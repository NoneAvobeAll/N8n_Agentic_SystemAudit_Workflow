# Contributing to N8n Multi-Server Audit Workflow

Thank you for your interest in contributing! This document provides guidelines for contributing to this project.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How to Contribute](#how-to-contribute)
- [Development Workflow](#development-workflow)
- [Pull Request Process](#pull-request-process)
- [Coding Standards](#coding-standards)
- [Documentation Guidelines](#documentation-guidelines)
- [Testing Guidelines](#testing-guidelines)
- [Issue Reporting](#issue-reporting)
- [Feature Requests](#feature-requests)

## Code of Conduct

We are committed to providing a welcoming and inclusive community. Please:

- Be respectful and constructive
- Welcome newcomers and help them learn
- Focus on what is best for the community
- Show empathy towards other community members

## How to Contribute

### Reporting Bugs

1. Check existing issues to avoid duplicates
2. Create a detailed bug report with:
   - Clear description of the problem
   - Steps to reproduce
   - Expected vs actual behavior
   - N8n version and environment details
   - Screenshots/logs if applicable
   - Server configuration (sanitized)

### Suggesting Enhancements

1. Check existing feature requests
2. Use the feature request template
3. Provide:
   - Clear use case description
   - Proposed solution or approach
   - Alternative approaches considered
   - Impact on existing functionality

### Submitting Code

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## Development Workflow

### Setting Up Development Environment

```bash
# Clone your fork
git clone https://github.com/YOUR_USERNAME/N8n_Agentic_SystemAudit_Workflow.git
cd N8n_Agentic_SystemAudit_Workflow

# Create a development branch
git checkout -b feature/your-feature-name
```

### Making Changes

1. **Modify the Workflow**
   - Open N8n instance
   - Import `workflows/multi-server-audit-workflow.json`
   - Make changes in N8n editor
   - Export the modified workflow
   - Replace `workflows/multi-server-audit-workflow.json`

2. **Update Documentation**
   - Update relevant documentation files
   - Keep README.md in sync with changes
   - Update CHANGELOG.md

3. **Test Changes**
   - Test in non-production environment
   - Verify all connections work
   - Check error handling
   - Verify alerts are sent correctly

## Pull Request Process

### PR Checklist

Before submitting a PR, ensure:

- [ ] Code follows project style guidelines
- [ ] All tests pass (if applicable)
- [ ] Documentation is updated
- [ ] CHANGELOG.md is updated
- [ ] Commit messages are clear and descriptive
- [ ] No sensitive data is included
- [ ] Workflow is tested in N8n

### PR Template

```markdown
## Description
Brief description of changes made

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update
- [ ] Performance improvement
- [ ] Security fix

## Testing
Describe how you tested your changes

## Screenshots (if applicable)
Add screenshots showing the changes

## Checklist
- [ ] My code follows the style guidelines
- [ ] I have performed a self-review
- [ ] I have commented my code where complex
- [ ] I have updated documentation
- [ ] My changes generate no new warnings
- [ ] Testing has been performed
```

### Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `chore`: Maintenance tasks
- `security`: Security fixes

**Example:**
```
feat(windows): add Active Directory replication check

Add PowerShell command to check AD replication health
on domain controllers. Alerts if replication is failing.

Closes #123
```

## Coding Standards

### JavaScript/TypeScript (N8n Code Nodes)

```javascript
// ✅ Good - Clear variable names
const serverHostname = $json.hostname;
const memoryUsagePercentage = $json.mem_pct;

// ❌ Bad - Unclear abbreviations
const h = $json.hostname;
const mp = $json.mem_pct;

// ✅ Good - Error handling
try {
  const result = execSync(command, { encoding: 'utf8', timeout: 10000 });
  auditData.result = result;
} catch (error) {
  auditData.error = error.message;
  auditData.has_error = true;
}

// ❌ Bad - No error handling
const result = execSync(command, { encoding: 'utf8' });
auditData.result = result;

// ✅ Good - Comments for complex logic
// Calculate memory percentage with error handling
// Return 0 if parsing fails
const memPct = parseFloat(memory.trim()) || 0;

// ❌ Bad - Magic numbers without comments
const threshold = 85;
if (usage > threshold) { ... }
```

### Workflow Design

- Use clear, descriptive node names
- Add sticky notes for documentation
- Keep nodes logically organized
- Use consistent spacing and alignment
- Group related functionality

### Security Guidelines

- **NEVER** commit credentials or API keys
- Use environment variables for sensitive data
- All audit commands must be read-only
- No harmful commands (rm, delete, format, etc.)
- Validate all user inputs
- Use parameterized queries/commands

```javascript
// ✅ Good - Parameterized command
const command = `${sshBase} "journalctl -p err -n ${maxLines} --no-pager"`;

// ❌ Bad - Command injection vulnerability
const command = `${sshBase} "journalctl ${userInput}"`;
```

## Documentation Guidelines

### README.md

- Keep sections up-to-date
- Add clear installation instructions
- Include troubleshooting steps
- Provide examples for common use cases
- Use proper formatting (markdown, tables, code blocks)

### API/Configuration Documentation

- Document all configuration options
- Provide default values
- Explain what each option does
- Give example configurations

```markdown
### mem_critical_pct

Memory usage percentage that triggers a critical alert.

- **Type**: Number
- **Default**: 85
- **Example**: 90

When memory usage exceeds this percentage, a critical alert is sent.
```

### Code Comments

- Comment complex logic
- Explain why (not just what)
- Keep comments up-to-date
- Use clear, concise language

```javascript
// Calculate memory usage percentage
// Free command returns total and used memory in MB
// We calculate percentage as (used / total) * 100
const memPct = (usedMb / totalMb) * 100;
```

## Testing Guidelines

### Manual Testing Checklist

Before submitting changes, test:

- [ ] Workflow imports without errors
- [ ] OS detection works for Linux and Windows
- [ ] SSH connections establish successfully
- [ ] WinRM connections establish successfully
- [ ] All audit commands return data
- [ ] Threshold evaluation works correctly
- [ ] Critical alerts are sent via Telegram
- [ ] AI analysis generates responses
- [ ] Error handling works (test with invalid credentials)
- [ ] Workflow completes without hanging

### Test Environments

1. **Development**: Local N8n instance with test servers
2. **Staging**: Non-production servers that mirror production
3. **Production**: Test on single server first, then roll out

### Testing Edge Cases

- Empty server list
- Invalid server credentials
- Network timeouts
- Server down/unreachable
- AI service unavailable
- Telegram bot offline
- Mixed OS environments
- Large numbers of servers

## Issue Reporting

### Bug Report Template

```markdown
## Description
Clear and concise description of the bug

## Steps to Reproduce
1. Go to '...'
2. Click on '...'
3. Scroll down to '...'
4. See error

## Expected Behavior
Description of what you expected to happen

## Actual Behavior
Description of what actually happened

## Environment
- N8n Version:
- Workflow Version:
- OS Type (Linux/Windows):
- N8n Host OS:

## Server Configuration
(Provide sanitized configuration)

## Logs
```
Paste relevant logs here
```

## Screenshots
If applicable, add screenshots

## Additional Context
Any other context about the problem
```

## Feature Requests

### Feature Request Template

```markdown
## Problem Statement
Clear description of the problem or use case

## Proposed Solution
Description of the proposed solution

## Alternatives Considered
Description of alternative approaches

## Additional Context
Any other context or screenshots
```

## Areas for Contribution

We welcome contributions in these areas:

### New OS Support
- macOS monitoring
- BSD monitoring
- Container monitoring (Kubernetes pods, etc.)

### Additional Metrics
- Database-specific metrics
- Application performance metrics
- Network throughput monitoring
- SSL certificate monitoring
- Backup status monitoring

### New Alert Channels
- Email notifications
- Slack integration
- Discord integration
- Microsoft Teams
- PagerDuty
- Opsgenie

### Enhanced AI Analysis
- Specialized prompts for different scenarios
- Multi-model support
- Custom AI providers

### Documentation
- Translation to other languages
- Video tutorials
- Additional examples
- Troubleshooting guides

### Performance
- Parallel processing optimizations
- Caching strategies
- Connection pooling
- Batch processing improvements

## Questions or Need Help?

- Open an issue for bugs or feature requests
- Start a discussion for questions
- Check existing issues and documentation first

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

Thank you for contributing! 🙏