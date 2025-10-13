# Claude Code - Status Line Documentation

Complete reference for creating and customizing the Claude Code status line.

## What is the Status Line?

The status line is a customizable display at the bottom of the Claude Code interface, similar to terminal prompts (PS1) in shells like Oh-my-zsh. It shows contextual information about your current session.

### Features
- **Dynamic updates**: Updates when conversation messages change
- **Rich information**: Access to session data, model, cost, workspace info
- **ANSI colors**: Full color support for styling
- **Custom scripts**: Use any language (Bash, Python, Node.js, etc.)
- **Automatic setup**: Claude can configure it for you

## Quick Start

### Automatic Setup
```bash
/statusline
```

Claude will:
1. Detect your current terminal prompt
2. Create a status line script
3. Configure it in settings.json
4. Reproduce your PS1-style prompt by default

### Custom Setup
```bash
/statusline show the model name in orange and current directory

/statusline add git branch, current time, and session duration

/statusline make it look like my zsh prompt with colors
```

## Configuration

### Settings Location
```json
// ~/.claude/settings.json or .claude/settings.json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 0
  }
}
```

### Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `type` | string | Always "command" |
| `command` | string | Path to script that generates status line |
| `padding` | number | Optional: set to 0 to extend to edge |

### Example Configuration
```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 0
  }
}
```

## How It Works

### Update Mechanism
- Status line updates when conversation messages update
- Updates run at most every 300ms (throttled)
- First line of stdout becomes the status line text
- ANSI color codes are supported

### Data Flow
```
Claude Code Session
       ↓
   JSON Data (stdin)
       ↓
  Your Script
       ↓
  Formatted Text (stdout)
       ↓
  Status Line Display
```

## JSON Input Structure

Your script receives JSON data via stdin with the following structure:

```json
{
  "hook_event_name": "Status",
  "session_id": "abc123...",
  "transcript_path": "/path/to/transcript.json",
  "cwd": "/current/working/directory",
  "model": {
    "id": "claude-opus-4-1",
    "display_name": "Opus"
  },
  "workspace": {
    "current_dir": "/current/working/directory",
    "project_dir": "/original/project/directory"
  },
  "version": "1.0.80",
  "output_style": {
    "name": "default"
  },
  "cost": {
    "total_cost_usd": 0.01234,
    "total_duration_ms": 45000,
    "total_api_duration_ms": 2300,
    "total_lines_added": 156,
    "total_lines_removed": 23
  }
}
```

### Field Reference

| Field | Description |
|-------|-------------|
| `hook_event_name` | Always "Status" for status line |
| `session_id` | Unique session identifier |
| `transcript_path` | Path to session transcript |
| `cwd` | Current working directory |
| `model.id` | Full model identifier |
| `model.display_name` | Short name (Opus, Sonnet, Haiku) |
| `workspace.current_dir` | Current directory |
| `workspace.project_dir` | Project root directory |
| `version` | Claude Code version |
| `cost.total_cost_usd` | Total API cost for session |
| `cost.total_duration_ms` | Total session duration |
| `cost.total_api_duration_ms` | Time spent in API calls |
| `cost.total_lines_added` | Lines added in session |
| `cost.total_lines_removed` | Lines removed in session |

## Example Scripts

### Simple Status Line (Bash)

```bash
#!/bin/bash
# ~/.claude/statusline.sh

# Read JSON from stdin
input=$(cat)

# Extract current directory (replace $HOME with ~)
dir=$(echo "$input" | jq -r '.workspace.current_dir' | sed "s|^$HOME|~|")

# Extract model name
model=$(echo "$input" | jq -r '.model.display_name')

# Print status line with colors
echo -e "\\033[1;32m➜\\033[0m \\033[36m$(basename $dir)\\033[0m \\033[90m[$model]\\033[0m"
```

**Result**: `➜ project-name [Sonnet]`

### Git-Aware Status Line (Bash)

```bash
#!/bin/bash
# ~/.claude/statusline.sh

input=$(cat)
dir=$(echo "$input" | jq -r '.workspace.current_dir' | sed "s|^$HOME|~|")
model=$(echo "$input" | jq -r '.model.display_name')

# Get git branch
git_branch=""
if git rev-parse --git-dir > /dev/null 2>&1; then
  git_branch=$(git branch --show-current 2>/dev/null)
  if [ -n "$git_branch" ]; then
    git_branch=" \\033[35m($git_branch)\\033[0m"
  fi
fi

echo -e "\\033[1;32m➜\\033[0m \\033[36m$(basename $dir)\\033[0m$git_branch \\033[90m[$model]\\033[0m"
```

**Result**: `➜ project-name (main) [Sonnet]`

### Python Status Line

```python
#!/usr/bin/env python3
# ~/.claude/statusline.py

import json
import sys
import os

# Read JSON from stdin
data = json.load(sys.stdin)

# Extract data
current_dir = data['workspace']['current_dir']
model = data['model']['display_name']
cost = data['cost']['total_cost_usd']

# Format directory (replace home with ~)
home = os.path.expanduser('~')
if current_dir.startswith(home):
    current_dir = '~' + current_dir[len(home):]

# Format status line with colors
print(f"\033[1;32m➜\033[0m \033[36m{os.path.basename(current_dir)}\033[0m \033[90m[{model}]\033[0m \033[33m${cost:.4f}\033[0m")
```

**Result**: `➜ project-name [Sonnet] $0.0123`

### Node.js Status Line

```javascript
#!/usr/bin/env node
// ~/.claude/statusline.js

const fs = require('fs');
const path = require('path');

// Read JSON from stdin
let input = '';
process.stdin.on('data', chunk => input += chunk);
process.stdin.on('end', () => {
  const data = JSON.parse(input);
  
  // Extract data
  let dir = data.workspace.current_dir;
  const model = data.model.display_name;
  const duration = Math.floor(data.cost.total_duration_ms / 1000);
  
  // Replace home with ~
  const home = process.env.HOME;
  if (dir.startsWith(home)) {
    dir = '~' + dir.slice(home.length);
  }
  
  // Format time
  const minutes = Math.floor(duration / 60);
  const seconds = duration % 60;
  const time = `${minutes}m${seconds}s`;
  
  // Print status line
  console.log(`\x1b[1;32m➜\x1b[0m \x1b[36m${path.basename(dir)}\x1b[0m \x1b[90m[${model}]\x1b[0m \x1b[33m${time}\x1b[0m`);
});
```

**Result**: `➜ project-name [Sonnet] 5m23s`

### Advanced Status Line (Bash with Git + Time)

```bash
#!/bin/bash
# ~/.claude/statusline-advanced.sh

input=$(cat)

# Extract data
dir=$(echo "$input" | jq -r '.workspace.current_dir' | sed "s|^$HOME|~|")
model=$(echo "$input" | jq -r '.model.display_name')
cost=$(echo "$input" | jq -r '.cost.total_cost_usd')
duration=$(echo "$input" | jq -r '.cost.total_duration_ms')

# Calculate time
seconds=$((duration / 1000))
minutes=$((seconds / 60))
seconds=$((seconds % 60))
time="${minutes}m${seconds}s"

# Get git info
git_info=""
if git rev-parse --git-dir > /dev/null 2>&1; then
  branch=$(git branch --show-current 2>/dev/null)
  
  # Check for changes
  if ! git diff-index --quiet HEAD -- 2>/dev/null; then
    status="*"
  else
    status=""
  fi
  
  if [ -n "$branch" ]; then
    git_info=" \\033[35m($branch$status)\\033[0m"
  fi
fi

# Current time
current_time=$(date +%H:%M:%S)

# Build status line
echo -e "\\033[1;32m➜\\033[0m \\033[36m$(basename $dir)\\033[0m$git_info \\033[90m[$model]\\033[0m \\033[33m$time\\033[0m \\033[90m$current_time\\033[0m"
```

**Result**: `➜ project-name (main*) [Sonnet] 5m23s 14:30:45`

## ANSI Color Codes

### Basic Colors

| Code | Color | Example |
|------|-------|---------|
| `\033[0m` | Reset | `echo -e "\033[0m"` |
| `\033[30m` | Black | `echo -e "\033[30mBlack\033[0m"` |
| `\033[31m` | Red | `echo -e "\033[31mRed\033[0m"` |
| `\033[32m` | Green | `echo -e "\033[32mGreen\033[0m"` |
| `\033[33m` | Yellow | `echo -e "\033[33mYellow\033[0m"` |
| `\033[34m` | Blue | `echo -e "\033[34mBlue\033[0m"` |
| `\033[35m` | Magenta | `echo -e "\033[35mMagenta\033[0m"` |
| `\033[36m` | Cyan | `echo -e "\033[36mCyan\033[0m"` |
| `\033[37m` | White | `echo -e "\033[37mWhite\033[0m"` |
| `\033[90m` | Bright Black (Gray) | `echo -e "\033[90mGray\033[0m"` |

### Text Styles

| Code | Style |
|------|-------|
| `\033[1m` | Bold |
| `\033[2m` | Dim |
| `\033[3m` | Italic |
| `\033[4m` | Underline |

### Combining Styles

```bash
# Bold green
echo -e "\033[1;32mBold Green\033[0m"

# Bold cyan
echo -e "\033[1;36mBold Cyan\033[0m"

# Dim gray
echo -e "\033[2;90mDim Gray\033[0m"
```

## Common Patterns

### Show Current Directory
```bash
dir=$(echo "$input" | jq -r '.workspace.current_dir' | sed "s|^$HOME|~|")
echo "$dir"
```

### Show Directory Basename
```bash
dir=$(echo "$input" | jq -r '.workspace.current_dir')
echo "$(basename $dir)"
```

### Show Model with Color
```bash
model=$(echo "$input" | jq -r '.model.display_name')
echo -e "\\033[35m[$model]\\033[0m"
```

### Show Session Cost
```bash
cost=$(echo "$input" | jq -r '.cost.total_cost_usd')
echo -e "\\033[33m\$$(printf '%.4f' $cost)\\033[0m"
```

### Show Session Duration
```bash
duration=$(echo "$input" | jq -r '.cost.total_duration_ms')
seconds=$((duration / 1000))
minutes=$((seconds / 60))
seconds=$((seconds % 60))
echo "${minutes}m${seconds}s"
```

### Show Git Branch
```bash
if git rev-parse --git-dir > /dev/null 2>&1; then
  branch=$(git branch --show-current 2>/dev/null)
  echo "($branch)"
fi
```

### Show Git Status
```bash
if git rev-parse --git-dir > /dev/null 2>&1; then
  if ! git diff-index --quiet HEAD -- 2>/dev/null; then
    echo "*"  # Has changes
  fi
fi
```

### Show Current Time
```bash
echo $(date +%H:%M:%S)
```

### Show Username and Host
```bash
echo "$USER@$(hostname -s)"
```

## Advanced Customizations

### Battery Status (macOS)
```bash
battery=$(pmset -g batt | grep -o "[0-9]*%" | head -1)
echo "🔋 $battery"
```

### Docker Container Count
```bash
containers=$(docker ps -q 2>/dev/null | wc -l | tr -d ' ')
if [ "$containers" -gt 0 ]; then
  echo "🐳 $containers"
fi
```

### Active Kubernetes Context
```bash
if command -v kubectl &> /dev/null; then
  context=$(kubectl config current-context 2>/dev/null)
  echo "☸️  $context"
fi
```

### Python Virtual Environment
```bash
if [ -n "$VIRTUAL_ENV" ]; then
  venv=$(basename "$VIRTUAL_ENV")
  echo "🐍 $venv"
fi
```

### Node.js Version
```bash
if command -v node &> /dev/null; then
  node_version=$(node --version | cut -d'v' -f2)
  echo "⬢ $node_version"
fi
```

## Conditional Display

### Show Only When in Git Repository
```bash
if git rev-parse --git-dir > /dev/null 2>&1; then
  # Show git info
else
  # Show something else
fi
```

### Show Cost Only When Significant
```bash
cost=$(echo "$input" | jq -r '.cost.total_cost_usd')
if (( $(echo "$cost > 0.01" | bc -l) )); then
  echo -e "\\033[33m\$$cost\\033[0m"
fi
```

### Show Warning on High Cost
```bash
cost=$(echo "$input" | jq -r '.cost.total_cost_usd')
if (( $(echo "$cost > 1.0" | bc -l) )); then
  echo -e "\\033[31m⚠ High cost: \$$cost\\033[0m"
fi
```

## Performance Optimization

### Caching Expensive Operations
```bash
#!/bin/bash
# Cache git status for 1 second

cache_file="/tmp/claude-statusline-cache-$$"
cache_age=1

if [ -f "$cache_file" ]; then
  age=$(($(date +%s) - $(stat -f %m "$cache_file" 2>/dev/null || stat -c %Y "$cache_file")))
  if [ $age -lt $cache_age ]; then
    cat "$cache_file"
    exit 0
  fi
fi

# Generate status line (expensive operations)
result="..."

# Save to cache
echo "$result" > "$cache_file"
echo "$result"
```

### Parallel Execution
```bash
# Run multiple checks in background
git_info=$(git branch --show-current 2>/dev/null &)
docker_count=$(docker ps -q 2>/dev/null | wc -l &)

wait  # Wait for background jobs

# Use results
```

## Troubleshooting

### Status Line Not Appearing
1. Check script is executable:
   ```bash
   chmod +x ~/.claude/statusline.sh
   ```

2. Test script manually:
   ```bash
   echo '{"model":{"display_name":"Test"},"workspace":{"current_dir":"/test"}}' | ~/.claude/statusline.sh
   ```

3. Check configuration in settings.json:
   ```bash
   cat ~/.claude/settings.json | jq .statusLine
   ```

4. Verify script path is correct

### Script Errors
- Check for syntax errors: `bash -n ~/.claude/statusline.sh`
- Test with sample input
- Check jq is installed: `which jq`
- Review error messages in verbose mode: `claude --verbose`

### Colors Not Working
- Ensure ANSI codes are properly escaped
- Test in terminal: `echo -e "\033[32mGreen\033[0m"`
- Check terminal supports colors
- Verify script outputs to stdout (not stderr)

### Slow Updates
- Reduce complexity of script
- Cache expensive operations (git, docker, etc.)
- Profile script execution time
- Consider removing slow checks

## Testing Status Line

### Test with Mock Data
```bash
echo '{
  "hook_event_name": "Status",
  "session_id": "test123",
  "model": {"display_name": "Sonnet"},
  "workspace": {"current_dir": "/test/project"},
  "cost": {"total_cost_usd": 0.5, "total_duration_ms": 120000}
}' | ~/.claude/statusline.sh
```

### Test Different Scenarios
```bash
# Test with different models
echo '{"model":{"display_name":"Opus"},"workspace":{"current_dir":"/test"}}' | ./statusline.sh

# Test with high cost
echo '{"model":{"display_name":"Opus"},"workspace":{"current_dir":"/test"},"cost":{"total_cost_usd":5.0}}' | ./statusline.sh

# Test long directory
echo '{"model":{"display_name":"Sonnet"},"workspace":{"current_dir":"/very/long/path/to/project"}}' | ./statusline.sh
```

### Debug Mode
```bash
#!/bin/bash
# Add to script for debugging

input=$(cat)
echo "[DEBUG] Input: $input" >&2
# ... rest of script
```

View debug output:
```bash
claude --verbose 2>&1 | grep DEBUG
```

## Best Practices

### Keep It Simple
- Start with basic info (directory, model)
- Add features incrementally
- Test each addition

### Performance First
- Cache expensive operations
- Avoid slow commands
- Optimize for common case
- Profile execution time

### Clear Visual Hierarchy
- Use colors meaningfully
- Most important info first
- Consistent spacing
- Clear separators

### Contextual Information
- Show what's relevant to current task
- Hide unnecessary details
- Adapt to git repository vs non-git
- Consider project type

### Error Handling
```bash
# Handle missing jq
if ! command -v jq &> /dev/null; then
  echo "Status line requires jq"
  exit 1
fi

# Handle invalid JSON
if ! echo "$input" | jq . > /dev/null 2>&1; then
  echo "Invalid input"
  exit 1
fi
```

## Examples by Use Case

### Minimalist
```bash
dir=$(basename $(echo "$input" | jq -r '.workspace.current_dir'))
echo "➜ $dir"
```

### Developer
```bash
# Directory, git branch, model
dir=$(basename $(echo "$input" | jq -r '.workspace.current_dir'))
model=$(echo "$input" | jq -r '.model.display_name')
branch=$(git branch --show-current 2>/dev/null || echo "")
echo -e "\\033[32m➜\\033[0m $dir \\033[35m$branch\\033[0m \\033[90m[$model]\\033[0m"
```

### Cost-Conscious
```bash
# Show cost prominently
cost=$(echo "$input" | jq -r '.cost.total_cost_usd')
duration=$(echo "$input" | jq -r '.cost.total_duration_ms')
minutes=$((duration / 60000))
echo -e "\\033[33m\$$(printf '%.4f' $cost)\\033[0m \\033[90m${minutes}m\\033[0m"
```

### DevOps
```bash
# Kubernetes context, docker containers, git branch
k8s=$(kubectl config current-context 2>/dev/null || echo "")
containers=$(docker ps -q 2>/dev/null | wc -l | tr -d ' ')
branch=$(git branch --show-current 2>/dev/null || echo "")
echo "☸️  $k8s 🐳 $containers ($branch)"
```

## Related Documentation
- [Main documentation](README.md)
- [CLI Reference](cli.md)
- [Hooks Documentation](hook.md)
- [Settings Documentation](settings.md)
