# Contributing to Midnight Learning Guide

Thank you for your interest in contributing to this Midnight programming learning resource! This guide will help you get started.

## Table of Contents

- [How Can I Contribute?](#how-can-i-contribute)
- [Getting Started](#getting-started)
- [Contribution Guidelines](#contribution-guidelines)
- [Style Guide](#style-guide)
- [Pull Request Process](#pull-request-process)
- [Code of Conduct](#code-of-conduct)

## How Can I Contribute?

There are many ways to contribute to this learning resource:

### 1. Report Issues
- Found a typo or error? Report it!
- Have suggestions for improvements? Let us know!
- Encountered broken links? File an issue!

### 2. Improve Documentation
- Fix typos and grammatical errors
- Clarify confusing explanations
- Add missing information
- Update outdated content

### 3. Add Examples
- Create new contract examples
- Add use case tutorials
- Share best practices
- Contribute code snippets

### 4. Write Tutorials
- Beginner-friendly guides
- Advanced topic deep-dives
- Project walkthroughs
- Video tutorials

### 5. Translate Content
- Translate documentation to other languages
- Help make learning accessible globally

### 6. Answer Questions
- Help others in discussions
- Review pull requests
- Share your knowledge

## Getting Started

### Prerequisites

- Git installed on your system
- GitHub account
- Basic knowledge of Markdown
- (Optional) Midnight development environment

### Fork and Clone

1. **Fork this repository**
   - Click the "Fork" button on GitHub
   - This creates your own copy

2. **Clone your fork**
   ```bash
   git clone https://github.com/YOUR_USERNAME/midnight.git
   cd midnight
   ```

3. **Add upstream remote**
   ```bash
   git remote add upstream https://github.com/arkilian/midnight.git
   ```

4. **Create a branch**
   ```bash
   git checkout -b your-feature-branch
   ```

## Contribution Guidelines

### Documentation

- **Clear and Concise**: Write in simple, understandable language
- **Examples**: Include code examples where relevant
- **Accuracy**: Ensure technical accuracy
- **References**: Link to official documentation when appropriate
- **Structure**: Follow existing document structure

### Code Examples

- **Working Code**: All examples should compile and run
- **Comments**: Add explanatory comments
- **Best Practices**: Follow Midnight best practices
- **Testing**: Include test cases when applicable
- **Documentation**: Add README with usage instructions

### File Organization

```
midnight/
├── docs/              # Documentation files
│   ├── *.md          # Individual doc files
├── examples/          # Code examples
│   ├── example-name/
│   │   ├── README.md
│   │   └── *.compact
├── tutorials/         # Step-by-step tutorials
│   └── *.md
└── README.md          # Main readme
```

## Style Guide

### Markdown

#### Headers
```markdown
# H1 - Main Title (one per file)
## H2 - Major Sections
### H3 - Subsections
#### H4 - Minor Points
```

#### Code Blocks
````markdown
```typescript
// Use appropriate language tags
contract Example {
  state value: uint;
}
```
````

#### Lists
```markdown
- Unordered lists use dashes
- Keep items concise

1. Ordered lists use numbers
2. Start from 1
```

#### Links
```markdown
[Link Text](URL)
[Internal Link](../docs/file.md)
```

#### Emphasis
```markdown
**Bold** for emphasis
*Italic* for terms
`code` for inline code
```

### Compact Code

```typescript
// Use clear, descriptive names
contract TokenContract {  // ✅ Clear
contract TC {             // ❌ Unclear

// Add comments for complex logic
witness complexProof(...): proof {
  // Explain what this proves
  assert(condition, "Error message");
  return proof;
}

// Format consistently
entry functionName(param: Type): ReturnType {
  // 2-space indentation
  if (condition) {
    // Nested properly
  }
}
```

### Documentation Style

```markdown
## Function Name

Brief description of what it does.

### Parameters
- `param1: Type` - Description
- `param2: Type` - Description

### Returns
- `ReturnType` - Description

### Example
```typescript
// Usage example
```

### Notes
Additional important information.
```

## Pull Request Process

### Before Submitting

1. **Test Your Changes**
   - Ensure code examples compile
   - Check all links work
   - Verify formatting looks correct

2. **Update Documentation**
   - Update README if needed
   - Add comments to code
   - Update relevant guides

3. **Commit Messages**
   ```bash
   # Good commit messages
   git commit -m "Add token transfer example"
   git commit -m "Fix typo in getting-started.md"
   git commit -m "Update ZK proof tutorial"
   
   # Bad commit messages
   git commit -m "update"
   git commit -m "fix stuff"
   ```

### Submitting Pull Request

1. **Push to Your Fork**
   ```bash
   git push origin your-feature-branch
   ```

2. **Create Pull Request**
   - Go to your fork on GitHub
   - Click "Pull Request"
   - Fill in the template

3. **PR Description Template**
   ```markdown
   ## Description
   Brief description of changes
   
   ## Type of Change
   - [ ] Bug fix
   - [ ] New feature
   - [ ] Documentation update
   - [ ] Example code
   
   ## Checklist
   - [ ] Code compiles/runs
   - [ ] Documentation updated
   - [ ] Links verified
   - [ ] Follows style guide
   
   ## Related Issues
   Closes #123
   ```

4. **Wait for Review**
   - Maintainers will review your PR
   - Address feedback if requested
   - Make additional commits if needed

### After Submission

- Respond to review comments
- Make requested changes
- Be patient and respectful
- Thank reviewers!

## Review Process

### What We Look For

✅ **Quality**
- Accurate information
- Clear explanations
- Working code examples

✅ **Style**
- Follows style guide
- Consistent formatting
- Proper grammar/spelling

✅ **Value**
- Adds useful information
- Improves clarity
- Helps learners

### Review Timeline

- Initial response: Within 7 days
- Full review: Within 14 days
- Feedback incorporated: Variable

## Types of Contributions

### High Priority
- Fixing errors in existing content
- Updating outdated information
- Adding missing fundamentals
- Improving clarity

### Medium Priority
- Adding new examples
- Creating tutorials
- Adding advanced topics
- Translations

### Nice to Have
- Additional resources
- Video tutorials
- Interactive examples
- Community content

## Recognition

Contributors will be:
- Listed in README contributors section
- Credited in relevant files
- Thanked in release notes
- Appreciated by the community! 🎉

## Getting Help

Need help contributing?

- 💬 [Discord](https://discord.com/invite/midnightntwrk) - Ask in #contributors
- 📧 Email: contributors@midnight.network (if available)
- 🐙 GitHub Discussions: For questions about contributions
- 📋 Issues: Open an issue with "question" label

## Code of Conduct

### Our Pledge

We pledge to make participation in this project a harassment-free experience for everyone, regardless of:
- Age, body size, disability
- Ethnicity, gender identity and expression
- Level of experience
- Nationality, personal appearance, race, religion
- Sexual identity and orientation

### Our Standards

**Positive behaviors:**
- Being respectful and inclusive
- Accepting constructive criticism
- Focusing on what's best for the community
- Showing empathy towards others

**Unacceptable behaviors:**
- Harassment or discriminatory language
- Trolling or insulting comments
- Personal or political attacks
- Publishing others' private information
- Other conduct inappropriate in a professional setting

### Enforcement

Instances of unacceptable behavior may be reported to project maintainers. All complaints will be reviewed and investigated.

Project maintainers have the right to remove, edit, or reject:
- Comments, commits, code, issues, and other contributions
- Ban temporarily or permanently any contributor

## Questions?

Don't hesitate to ask questions:

- Open an issue with the "question" label
- Ask in Discord #help channel
- Email the maintainers

## Thank You!

Your contributions make this learning resource better for everyone in the Midnight community. Thank you for your time and effort! 🌙

---

**Happy Contributing!**

Last updated: December 2025
