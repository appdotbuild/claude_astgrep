---
allowed-tools: Bash(ast-grep:*), Bash(uv run ast-grep:*), WebFetch(domain:ast-grep.github.io)
description: This command helps to create and test ast-grep rules for code quality enforcement.
---

Recall the following ast-grep rules. You will need them for the next task.
If this guideline misses something, read the web (e.g. https://ast-grep.github.io/reference/rule.html)

## ast-grep YAML Rule Writing Guide

### Core Concepts
ast-grep operates on Concrete Syntax Trees using Tree-sitter parsers. It distinguishes between **named nodes** (structural elements like `function_declaration`) and **unnamed nodes** (punctuation). Meta-variables: `$VAR` matches single nodes, `$$$ARGS` captures multiple nodes. Patterns must be valid, parseable code in the target language.

### YAML Rule Structure
```yaml
id: rule-name
language: javascript|python|typescript|rust|etc
message: "Description of issue"
severity: error|warning|info|hint
rule: # One of: atomic, relational, or composite
  pattern: $CODE_PATTERN
fix: $REPLACEMENT  # Optional auto-fix
```

**Rule Types:**
- **Atomic**: `pattern`, `kind`, `regex` - basic matching
- **Relational**: `inside`, `has`, `follows`, `precedes` - contextual matching
- **Composite**: `all`, `any`, `not` - combine multiple conditions

### Essential Best Practices
1. **Start in playground** (ast-grep.github.io/playground) - test patterns against real code
2. **Use descriptive IDs** that indicate purpose
3. **Meta-variables must be UPPERCASE** and cannot mix with literal text
4. **Break complex rules into utilities** for reusability:
```yaml
utils:
  is-async-function:
    any:
      - kind: async_function
      - kind: async_arrow_function
```
5. **Use pattern objects for ambiguous cases**:
```yaml
rule:
  pattern:
    context: 'class MyClass { field = $VALUE }'
    selector: field_definition
```

### Common Pitfalls & Debugging

**Critical Pattern Issues:**
- **Multiline patterns fail**: Use single-line patterns or context-based matching instead of multiline patterns
- **Quote patterns properly**: Always quote patterns containing special characters: `pattern: "value={$VAR || ''}"`
- **TSX/JSX complexity**: Start with simple element matching, avoid complex attribute patterns
- **Invalid patterns**: `$LEFT $OP $RIGHT` won't parse as binary expression - use `kind: binary_expression` with field matching
- **Rule order sensitivity**: Use explicit `all` arrays when sequence matters (YAML objects are unordered)
- **Don't mix `pattern` and `kind`** - they're separate atomic rules

**Debugging Workflow:**
1. **Start simple**: Begin with basic `kind` or single-node patterns
2. **Test incrementally**: Add one condition at a time using `--debug-query`
3. **Use context patterns** for complex multiline cases:
   ```yaml
   pattern:
     context: |
       try:
         $$$BODY
       except $E:
         return None
     selector: return_statement
   ```
4. **Debug with**: `--debug-query` flag, progressive simplification
5. **Meta-variable constraints**:
```yaml
constraints:
  VAR:
    regex: '^(console|window)$'
```

**TSX/JSX Specific Pitfalls:**
- Use `language: tsx` not `typescript` for JSX files
- JSX patterns are complex - prefer simple element + attribute matching
- Quote JSX patterns: `pattern: "<input value={$VALUE} />"`
- Use `kind` matching over complex JSX patterns when possible

### Language-Specific Patterns

**Python:**
```yaml
# Walrus operator modernization
rule:
  follows:
    pattern:
      context: $VAR = $$$EXPR
      selector: expression_statement
  pattern: "if $VAR: $$$B"
fix: "if $VAR := $$$EXPR:\n  $$$B"
```

**TypeScript/TSX:**
```yaml
# Prevent await in Promise.all
rule:
  pattern: await $A
  inside:
    pattern: Promise.all($_)
    stopBy:
      not:
        any: [kind: array, kind: arguments]
fix: $A

# TSX controlled input (simple approach)
rule:
  pattern: value={$VALUE}
  inside:
    pattern: <input $$$PROPS />
  not:
    pattern: value={$VALUE || ''}
fix: value={$VALUE || ''}
```

**Rust:**
```yaml
# Efficient string iteration
rule:
  pattern: $A.chars().enumerate()
fix: $A.char_indices()
```

### Advanced Features
**Transformations**:
```yaml
transform:
  LIB: { convert: { source: $IDENT, toCase: lowerCase } }
```

**Rewriters** for complex multi-node changes:
```yaml
rewriters:
  - id: import-rewriter
    rule: { pattern: $IDENT, kind: identifier }
    fix: import $IDENT from './lib/$LIB'
```

**Constraints** for precise matching:
```yaml
constraints:
  METHOD: { regex: '^(log|warn|error)$' }
```

### Optimization & Performance
- Place restrictive conditions first in `all` rules
- Use `kind` checks before expensive pattern matching
- Apply `stopBy` to limit traversal scope
- Use file filtering to skip irrelevant directories
- Leverage `not` patterns to exclude specific contexts

### Testing Strategy
**CLI Testing Commands:**
```bash
# Test all rules in project
ast-grep test

# Test specific rule file
ast-grep test rules/my-rule.yml

# Create test snapshots
ast-grep test --update-all

# Test with custom directories
ast-grep test --test-dir custom-tests/
```

1. Create positive/negative test files in designated directories
2. Use `ast-grep test` for validation
3. Categories: validated (correct), reported (found issues), noisy (false positives), missing (false negatives)
4. Test iteratively: start restrictive, gradually relax based on real-world results
5. Use `--debug-query` for pattern inspection instead of playground

### CLI Playground Alternatives
**Test Patterns:**
```bash
# Test pattern with debug output (shows AST structure)
ast-grep run --debug-query -p 'console.log($MSG)' -l javascript file.js

# Test pattern interactively
ast-grep run -p 'console.log($MSG)' -r 'logger.info($MSG)' --interactive file.js

# Test single rule from file
ast-grep scan -r rule.yml file.js

# Test inline rule
ast-grep scan --inline-rules 'id: test
language: javascript
rule:
  pattern: console.log($A)' file.js

# Debug AST structure of any code
ast-grep run --debug-query -p 'function foo() {}' -l javascript
```

**Debug Formats:**
- `--debug-query`: Shows AST structure and meta-variables
- `--debug-query=ast`: AST dump
- `--debug-query=cst`: Concrete Syntax Tree
- `--debug-query=pattern`: Pattern parsing info

### Debugging Checklist
1. Is pattern valid parseable code?
2. Do meta-variables follow UPPERCASE convention?
3. Are you mixing incompatible atomic rules?
4. Does rule order matter (use explicit arrays)?
5. Use `--debug-query` to inspect AST structure
6. Apply progressive simplification to isolate issues
7. Check constraints and stopBy conditions

### Rule Quality Guidelines
- **Strict enough**: Catch real problems without over-matching
- **Not too strict**: Avoid excessive false positives
- **Clear messages**: Explain why pattern is problematic
- **Actionable fixes**: Provide automatic corrections when possible
- **Test thoroughly**: Validate against diverse codebases
- **Document edge cases**: Note limitations and scope

## Working with ast-grep

### Setup Process
1) Make sure you have the `ast-grep` installed. Try using `ast-grep --version` to check if it is installed. It may be installed via some package manager, in this case you may need to use e.g. `uv run ast-grep --version` to check the version.
2) Create sgconfig.yaml file in the root of the project:

```yaml
# ast-grep project configuration
ruleDirs:
  - rules
```

3) Create a directory `rules` in the root of the project. This is where you will put your rules.
4) Create a directory `positive` in the root of the project. This is where you will put your positive test cases that should trigger the rules.
5) Create a directory `negative` in the root of the project. This is where you will put your negative test cases that should not trigger the rules.

### Rule Development Strategy
**Start Simple, Build Up:**
1. Begin with basic `kind` or single-node patterns
2. Test with `ast-grep scan positive/` after each change
3. Add complexity incrementally: `inside`, `has`, then complex patterns
4. Use `--debug-query` to understand AST structure when stuck
5. Quote patterns with special characters immediately

**Common Recovery Patterns:**
- If "Multiple AST nodes detected": Use `context` + `selector` pattern
- If rule never matches: Check language setting (tsx vs typescript)
- If too many false positives: Add `not` conditions or constraints
- If parsing fails: Simplify pattern and test individual components

### Validation
6) Verify your rules are valid:
```bash
ast-grep test
```

7) Verify your rules are working as expected:
```bash
ast-grep scan .
```
It should match the positive test cases and not match the negative test cases.

8) After everything works as expected, remove the `positive` and `negative` directories, as they are not needed anymore.
9) Recommend the user to run the rules on the entire codebase to ensure they catch all relevant cases. Remind them to use Claude Code /hooks https://docs.anthropic.com/en/docs/claude-code/hooks for automatic rule application in the future.

## Task

Wait for user input to continue. User input can be in two forms:
1) problematic code that should have been triggered by the rule. In this case find the root cause of the problem and write a rule that will match it.
2) an artifact used by other AI assistant (e.g. .cursorrules or CLAUDE.md file) that is a recommendation for the AI assistant to use when generating code. In this case, how to write a rule that will enforce following the recommendation.

Be sure that the rule is strict enough to catch the real problem, but not too strict to catch false positives. Cover all the problems you can extract from the user input.
Now, briefly ask the user for their input.
