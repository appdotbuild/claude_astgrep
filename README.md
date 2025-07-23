# Claude AST-Grep Integration

Convert your coding standards or annoying issues from AI assistants into deterministic automatically enforced rules using Claude Code and [ast-grep](https://ast-grep.github.io/).

![Demo](demo.gif)

## What is this?

A Claude Code command that transforms problematic code patterns or coding guidelines into ast-grep rules that automatically catch violations.

## Quick Start

```bash
# Install ast-grep. You can use other package managers instead!
uv sync

# In Claude Code, run:
/ast_grep
```

## Input Examples

### Example 1: Fix a Bad Pattern

You provide:
```javascript
// This is bad - using console.log in production
console.log("User data:", userData);
```

### Example 2: Enforce Team Guidelines

You provide your team's CLAUDE.md or .cursorrules:
```
- Always use logger instead of console
- Never return None to hide errors
- Use modern Python patterns (match over if-elif)
```
Claude creates rules that enforce these automatically.

## Why Use This?
- Converts human/AI-readable guidelines into machine-enforceable rules;
- Ensures AI assistants (Cursor, Claude, etc.) follow your standards and are not prone to reward hacking;
- Reduces manual code reviews for common issues;
- Helps maintain consistent code quality across code base.

Generated rules are validated during generatation and can be tested against your codebase.
With Claude Code hooks, you can apply them automatically during code generation. Furtermore, consider integrating them into your CI/CD pipeline to automatically enforce coding standards.

## Resources

- [ast-grep documentation](https://ast-grep.github.io/)
- [Claude Code hooks for automation](https://docs.anthropic.com/en/docs/claude-code/hooks)
