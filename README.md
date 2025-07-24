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

## Rule examples

### Disallow silent exception handling in Python
```python
id: require-exception-logging
message: Exception handlers should include logging for observability
severity: error
language: python
rule:
  pattern: |
    try:
      $$$
    except $$$:
      $$$BODY
  not:
    has:
      pattern: $$$BODY
      any:
        - regex: ".*log.*"
        - regex: ".*logger.*"
        - regex: ".*raise.*"
```

### Disallow empty string values in Select components (React)
```
id: no-empty-select-value
language: tsx
message: Select item components cannot have an empty string as value. Use a meaningful default value.
severity: error
rule:
  all:
    - pattern:
        context: '<$COMP value="" $$$ATTRS />'
        selector: jsx_attribute
    - inside:
        any:
          - kind: jsx_self_closing_element
            has:
              kind: identifier
              regex: '^(Select|Option|MenuItem|DropdownMenuItem|RadioGroupItem|.*Select.*|.*Option.*)'
          - kind: jsx_opening_element
            has:
              kind: identifier
              regex: '^(Select|Option|MenuItem|DropdownMenuItem|RadioGroupItem|.*Select.*|.*Option.*)'
```

## Resources

- [ast-grep documentation](https://ast-grep.github.io/)
- [Claude Code hooks for automation](https://docs.anthropic.com/en/docs/claude-code/hooks)
