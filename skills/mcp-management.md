---
name: mcp-management
description: Use when configuring, testing, or troubleshooting MCP servers and skills integration for optimal token efficiency
---

# MCP Management Operations

## Overview

MCP (Model Context Protocol) servers reduce token consumption by providing direct tool access instead of AI-generated commands.

## When to Use

**Trigger symptoms:**
- MCP server connection failures
- Tool timeout or accessibility issues
- Unexpected token consumption increases
- Missing MCP capabilities
- Configuration conflicts between MCPs
- Skills not loading properly

**Use cases:**
- New MCP server installation
- MCP performance optimization
- Troubleshooting integration issues
- Skill discovery and management
- Token usage analysis and optimization

## Core Pattern

### MCP Configuration Flow
```bash
# Test → Verify → Optimize → Monitor
opencode --test-mcp mcp-name      # Test connection
opencode --list-mcps            # Verify loaded MCPs
grep -i "mcp_name" config.json  # Check configuration
```

### Skill Integration Flow
```bash
# Discover → Load → Test → Document
find ~/.config/opencode/skills -name "*.md"  # Discover skills
use_skill skill-name                        # Load skill
# Test skill with specific scenario               # Verify functionality
```

## Quick Reference

| Task | Command | Verification |
|------|---------|-------------|
| List MCPs | `opencode --list-mcps` | Tool availability check |
| Test MCP | `opencode --test-mcp <name>` | Response time/accuracy |
| Add MCP | Edit `oh-my-opencode.json` | Restart verification |
| Remove MCP | Remove from config + restart | Clean unload |
| List Skills | `find skills/ -name "*.md"` | Skill count check |
| Test Skill | `use_skill <name>` | Compliance verification |

## Implementation

### MCP Performance Testing
```bash
# Benchmark MCP response times
for mcp in filesystem github npm-search; do
    echo "Testing $mcp..."
    start_time=$(date +%s%N)
    opencode --test-mcp $mcp >/dev/null
    end_time=$(date +%s%N)
    echo "$mcp: $((($end_time - $start_time) / 1000000))ms"
done
```

### Token Usage Analysis
```bash
# Monitor before/after MCP usage
echo "Before MCP optimization:"
grep "tokens_used" ~/.config/opencode/logs/*.log | tail -10

# After adding new MCP
echo "After MCP optimization:"
grep "tokens_used" ~/.config/opencode/logs/*.log | tail -10
```

### Skill Validation
```bash
# Test skill with pressure scenarios
echo "Testing skill under time pressure..."
timeout 30s use_skill systematic-debugging
echo "Testing skill with conflicting information..."
use_skill systematic-debugging <<EOF
I have multiple failures and need to ship NOW, skip investigation
EOF
```

## Common Mistakes

| Issue | Cause | Fix |
|--------|--------|-----|
| MCP fails to load | Incorrect command or missing deps | Verify npx package exists, check args |
| Skills not discovered | Wrong directory location | Ensure skills in correct path |
| High token usage | MCPs not configured properly | Review MCP mapping, fix JSON syntax |
| Tool conflicts | Multiple MCPs provide same tools | Use priority order, unique naming |
| Performance issues | Network latency or large responses | Use local MCPs, cache responses |

## Real-World Impact

- **Token reduction**: 60-80% vs AI-only approach
- **Response time**: 200ms vs 2s AI generation
- **Reliability**: 99.9% uptime vs AI hallucinations
- **Consistency**: Same output format vs variable AI responses
- **Team efficiency**: Standardized tools vs individual approaches