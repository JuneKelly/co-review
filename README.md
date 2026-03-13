# co-review

Co-review changes with your LLM Agent

## Installation

Install with [jolene](https://github.com/JuneKelly/jolene):

```bash
jolene install --github JuneKelly/co-review
```

Or by hand:

```bash
ln -s $(pwd)/commands/co-review.md ~/.config/opencode/command/co-review.md

ln -s $(pwd)/commands/co-review.md ~/.claude/commands/co-review.md
```

## Usage

```
/co-review remote branch foo
```

```
/co-review this branch
```

```
/co-review current changes
```
