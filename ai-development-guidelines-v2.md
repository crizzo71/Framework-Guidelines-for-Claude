# AI-Assisted Development: Human-in-the-Loop Guidelines

## Executive Summary

This document establishes comprehensive guidelines for integrating AI code assistants (claude.code, GitHub Copilot, etc.) into software development workflows while maintaining critical human oversight and quality control.
Guiding Principles:

- We should assume all code is AI-assisted from here on out.
- We cannot rely on a human to properly tag/label a change as AI-assisted to enforce more strict acceptance criteria. It is inevitable that the human misses tagging/labeling appropriatey and we apply less strict acceptance requirements to generated code.
- Automated acceptance should enforce AI-assisted development guidelines.
- Automated testing should validate that AI development guidelines were followed.
## 1. Development Workflow Requirements
### 1.1 Pre-Development Phase
**Human Validation Checkpoints**
- **Requirement Analysis**: Human must validate AI's understanding of requirements before implementation
- **Architecture Review**: Senior engineer approval required for AI-suggested architectural decisions

### 1.2 Development Phase Guardrails
**Code Generation Rules**
1 **Maximum AI Block Size**: AI-generated code blocks should ideally not exceed 50 lines.
Rule of thumb:  If you can't read, understand, and feel confident in the code block's purpose within a few minutes, it's too large.
All code human or AI generated requires human review.  
2. **Critical Path Restriction**: AI generated code that will require strict human review
   - Payment processing
   - User authentication
   - Data encryption/decryption
   - Database migration scripts
   - Production deployment scripts

**Human Oversight Requirements**
- **Test Coverage**:   
  -AI as a Starting Point: Allow the AI assistant to generate an initial set of unit tests. These can cover the basic "happy path" and common scenarios. This saves time on writing boilerplate.
  -Human Review and Expansion: Require a human to review and augment the AI-generated test
  -Use code coverage as a metric and tool to  measure the percentage of code lines executed by the test suite. Set a minimum threshold (80/90%)
  -A pull request that includes AI-generated code will not be merged until the test suite passes and meets the required coverage threshold. This automates the enforcement and creates a clear, measurable gate for quality.
- **Documentation**: Human must write/verify all documentation for AI-assisted code

## 2. Pull Request (PR) Review Process
Follow your PR Review process

### 2.1 Review Assignment Rules

**Mandatory Human Reviewers**
- **All PRs**: Minimum 2 human reviewers (1 senior engineer)
- **Critical System Changes**: Lead architect must review any AI-assisted changes to:
  - Core business logic
  - Security-sensitive components
  - Performance-critical paths
  - Integration points

**Review Focus Areas**
1. **Logic Validation**: Verify AI understood requirements correctly
2. **Security Analysis**: Check for common AI-generated security vulnerabilities
3. **Code Quality**: Ensure adherence to team coding standards
4. **Test Coverage**: Validate test completeness and quality

## 3. Technical Implementation Requirements

### 3.1 Code Quality Gates

**Pre-Commit Hooks**
```bash
#!/bin/bash
# AI-assisted code quality check
if grep -r "\[AI-GENERATED\]" --include="*.js" --include="*.py" --include="*.java" .; then
    echo "AI-generated code detected. Running enhanced checks..."
    
    # Enhanced linting for AI code
    eslint --config .eslintrc.ai-strict.js src/
    
    # Security scan
    npm audit --audit-level moderate
    
    # Test coverage check
    npm run test:coverage -- --threshold=90
fi
```

**CI/CD Pipeline Modifications**
```yaml
# .github/workflows/ai-assisted-review.yml
name: AI-Assisted Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  ai-code-analysis:
    if: contains(github.event.pull_request.body, 'AI assistant used')
    runs-on: ubuntu-latest
    steps:
      - name: Enhanced Security Scan
        run: |
          # Run additional security tools for AI-generated code
          bandit -r . -f json -o security-report.json
          semgrep --config=auto --json --output=semgrep-report.json
      
      - name: Code Complexity Analysis
        run: |
          # Check for overly complex AI-generated functions
          radon cc . --min=B --show-complexity
      
      - name: Dependency Vulnerability Check
        run: |
          # Enhanced dependency checking
          safety check --json --output safety-report.json
```

### 3.2 Monitoring and Metrics

**AI Usage Tracking**
```javascript
// Development metrics collection
const aiUsageMetrics = {
  trackAIAssistance: (file, linesOfCode, aiTool, timestamp) => {
    // Log AI usage for analysis
    metrics.increment('ai.code.generated', {
      tool: aiTool,
      fileType: path.extname(file),
      size: linesOfCode
    });
  },
  
  trackHumanReview: (file, reviewTime, issuesFound) => {
    // Track human review effectiveness
    metrics.histogram('ai.code.review_time', reviewTime, {
      file: file,
      issues: issuesFound
    });
  }
};
```

## 4. Quality Control Framework

### 4.1 Testing Requirements

**AI-Generated Code Testing**
- **Unit Test Coverage**: Minimum 95% for AI-generated functions
- **Integration Tests**: Required for all AI-generated API endpoints
- **Edge Case Testing**: Human must identify and test edge cases AI might miss

**Test Review Process**
```javascript
// Example test validation for AI-generated code
describe('AI-Generated Function: calculateUserScore()', () => {
  // Human-written tests for AI-generated function
  
  it('should handle null user input gracefully', () => {
    // Test case AI might not consider
    expect(calculateUserScore(null)).toBe(0);
  });
  
  it('should prevent integer overflow for large scores', () => {
    // Security-focused test case
    expect(calculateUserScore(Number.MAX_SAFE_INTEGER)).toBeLessThan(1000);
  });
  
  it('should validate input types correctly', () => {
    // Input validation test
    expect(() => calculateUserScore('invalid')).toThrow('Invalid input type');
  });
});
```

### 4.2 Code Review Checklists

**Code Review Checklist**

**Security & Safety**
Applicable to ALL code
- [ ] No hardcoded secrets or sensitive data
- [ ] Input validation implemented correctly
- [ ] SQL injection prevention measures in place
- [ ] XSS protection implemented
- [ ] Rate limiting considered for API endpoints

**Code Quality**
- [ ] Follows established coding patterns
- [ ] Error handling is comprehensive
- [ ] Logging is appropriate and secure
- [ ] Performance implications assessed
- [ ] Memory leaks prevented

**Business Logic**
- [ ] Requirements correctly implemented
- [ ] Edge cases handled appropriately
- [ ] Data integrity maintained
- [ ] Business rules enforced correctly

## 5. Risk Mitigation Strategies

### 5.1 High-Risk Code Categories

**Prohibited AI Usage**
- Production database migration scripts
- Security credential management
- Financial transaction processing
- User permission/role management
- Cryptographic implementations

**Restricted AI Usage** (Requires Lead Review)
- API authentication mechanisms
- Data serialization/deserialization
- Performance-critical algorithms
- Third-party integration code
- Configuration management scripts

### 5.2 Rollback Procedures

**AI-Assisted Code Rollback Plan**
1. **Immediate Rollback Triggers**:
   - Security vulnerability discovered in AI-generated code
   - Performance degradation > 20% attributed to AI code
   - Business logic errors in AI-generated functions

2. **Rollback Process**:
   ```bash
   # Emergency rollback script
   #!/bin/bash
   echo "Rolling back AI-assisted changes..."
   
   # Identify AI-generated commits
   git log --grep="\[AI-" --oneline --since="1 week ago"
   
   # Create rollback branch
   git checkout -b rollback-ai-changes-$(date +%Y%m%d)
   
   # Selective revert with human review
   echo "Manual review required for each revert"
   ```

## 6. Training and Certification Requirements

### 6.1 Engineer Certification

**AI-Assisted Development Certification**
- **Level 1**: Basic AI tool usage (Junior developers)
  - Understanding AI limitations
  - Code review requirements
  - Security awareness

- **Level 2**: Advanced AI assistance (Senior developers)
  - Architecture review capabilities
  - Security assessment skills
  - Team mentoring responsibilities

### 6.2 Ongoing Education

**Monthly Training Topics**
- AI-generated security vulnerabilities
- Code review best practices for AI-assisted development
- New tool capabilities and limitations
- Industry incident case studies

## 7. Compliance and Auditing

### 7.1 Audit Trail Requirements

**Documentation Standards**
```markdown
## AI Usage Documentation Template

### Development Session: [Date]
**AI Tool Used**: claude.code v1.2.3
**Files Modified**: 
- `src/api/userService.js` (Lines 45-120, AI-generated)
- `tests/userService.test.js` (Lines 1-50, Human-written)

**Human Review Process**:
- [ ] Code logic validated by [Engineer Name]
- [ ] Security review completed by [Senior Engineer Name]
- [ ] Test coverage verified: 98%
- [ ] Performance impact assessed: +2ms average response time

**Issues Identified and Resolved**:
- Fixed potential null pointer exception in line 67
- Added input sanitization in line 89
- Improved error messages for user feedback
```

### 7.2 Metrics and KPIs

**Success Metrics**
- AI-assisted code defect rate < traditional development
- Human review time efficiency (target: 30% faster than full manual review)
- Security vulnerability detection rate in AI code
- Developer satisfaction with AI assistance

**Risk Metrics**
- Number of AI-generated security issues discovered post-deployment
- Rollback frequency for AI-assisted changes
- Performance regression incidents attributed to AI code

## 8. Tool Configuration and Setup

### 8.1 Claude.code Configuration

```json
{
  "claude_code_config": {
    "safety_checks": {
      "enabled": true,
      "security_scan": true,
      "complexity_limit": 10,
      "max_function_length": 50
    },
    "human_review_triggers": {
      "file_types": [".env", ".config", "migration.sql"],
      "keywords": ["password", "secret", "token", "crypto"],
      "complexity_threshold": 8
    },
    "integration": {
      "git_hooks": true,
      "ci_cd_integration": true,
      "metrics_collection": true
    }
  }
}
```

### 8.2 Development Environment Setup

**Required Tools Integration**
- **Static Analysis**: ESLint, SonarQube, CodeQL
- **Security Scanning**: Bandit, Semgrep, npm audit
- **Code Coverage**: Jest, Coverage.py, JaCoCo
- **Performance Monitoring**: Lighthouse CI, Web Vitals

## 9. Escalation Procedures

### 9.1 Issue Classification

**Severity Levels**
- **P0-Critical**: Security vulnerability in AI-generated production code
- **P1-High**: Business logic error causing data integrity issues
- **P2-Medium**: Performance degradation > 10% attributed to AI code
- **P3-Low**: Code quality issues not meeting standards

### 9.2 Response Procedures

**P0-Critical Response**
1. Immediate production rollback if safe
2. Security team notification within 15 minutes
3. Post-incident review within 24 hours
4. Process improvement plan within 48 hours

## Conclusion

This framework ensures that AI code assistants enhance developer productivity while maintaining the highest standards of code quality, security, and reliability through comprehensive human oversight and systematic quality controls.