---
name: mcp-recommendations
description: Curated list of MCPs and skills for optimal productivity
---

# 🚀 Recommended MCPs for Token Efficiency

## **Must-Have MCPs**
### 1. **Filesystem MCP** ✅ Already Added
- **Purpose**: Direct file system access
- **Token Efficiency**: Eliminates file reading/writing tokens
- **Use Case**: File operations without AI overhead

### 2. **GitHub MCP** ✅ Already Added  
- **Purpose**: GitHub repository management
- **Token Efficiency**: Uses API calls instead of Git CLI parsing
- **Use Case**: PR management, issue tracking, repository ops

### 3. **NPM Search MCP** ✅ Already Added
- **Purpose**: Package discovery and management
- **Token Efficiency**: Direct npm registry access
- **Use Case**: Finding packages, version management

### 4. **Playwright MCP** ✅ Already Added
- **Purpose**: Browser automation and testing
- **Token Efficiency**: Direct browser interaction vs. HTML parsing
- **Use Case**: Web scraping, testing, UI automation

## **Advanced MCPs to Consider**
### 5. **Database MCPs**
```json
"sqlite": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-sqlite"]
}
```
- **Purpose**: Direct database queries
- **Token Efficiency**: SQL results vs. code generation
- **Use Case**: Data analysis, migrations

### 6. **Kubernetes MCP**
```json
"k8s": {
  "command": "npx", 
  "args": ["-y", "@modelcontextprotocol/server-kubernetes"]
}
```
- **Purpose**: Kubernetes cluster management
- **Token Efficiency**: kubectl commands vs. YAML generation
- **Use Case**: Deployment, scaling, monitoring

## **Token-Saving Skills to Create**

### 1. **API Integration Skill**
- **Purpose**: Standardize API calls
- **Token Savings**: Reusable patterns vs. repetitive code
- **Create**: `.opencode/skills/api-integration.md`

### 2. **Testing Automation Skill**  
- **Purpose**: Generate test patterns
- **Token Savings**: Template-based vs. AI generation
- **Create**: `.opencode/skills/testing-automation.md`

### 3. **Documentation Generator Skill**
- **Purpose**: Generate consistent docs
- **Token Savings**: Structured templates vs. freeform writing
- **Create**: `.opencode/skills/doc-generator.md`

## **Configuration Tips**

### **Token Optimization**:
1. **Prefer MCPs over AI**: Direct tool calls vs. generated commands
2. **Use Templates**: Skills with patterns vs. AI improvisation  
3. **Cache Results**: MCPs often have built-in caching
4. **Batch Operations**: One MCP call vs. multiple AI prompts

### **Cost Impact Analysis**:
- **Without MCPs**: 100% AI tokens for all operations
- **With MCPs**: 30% AI tokens, 70% direct tool calls
- **Savings**: Up to 70% token reduction!

## **Installation Priority**:
1. **Immediate**: filesystem, github, npm-search (already done ✅)
2. **Next Week**: playwright, database MCPs  
3. **Advanced**: kubernetes, monitoring MCPs