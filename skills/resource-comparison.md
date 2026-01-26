---
name: resource-comparison
description: Compare AWS resources between dev-qa and prod environments to identify and safely clean up unused resources (SQS, Lambda, DynamoDB, S3, etc.)
---

# Dev-QA vs Prod Resource Comparison & Cleanup

**Purpose:** Compare AWS resources between dev-qa and prod environments to identify deletion candidates and safely clean up unused resources.

**When to use:** When you need to audit and clean up AWS resources that exist in dev-qa but not in prod, or identify resources that are no longer being used.

**Applicable to:** SQS Queues, Lambda Functions, DynamoDB Tables, S3 Buckets, SNS Topics, EventBridge Rules, etc.

---

## Core Methodology

### Phase 1: Export & Initial Comparison

**Goal:** Identify resources in dev-qa that don't exist in prod

**Steps:**

1. **Export resource lists from both environments**
   - Use ControlMonkey exports (preferred)
   - Or AWS CLI to list resources
   - Format: CSV with resource names/identifiers

2. **Normalize resource names**
   - Strip environment identifiers (dev, qa, intg, prod, stg, sbox)
   - Handle prefixes, suffixes, and infixes
   - Example: `dev-myqueue` → `myqueue`, `myqueue-qa` → `myqueue`

3. **Use fuzzy matching**
   - Account for minor naming differences
   - Default threshold: 85% similarity
   - Prevents false positives from typos/variations

4. **Generate deletion candidates CSV**
   - Resources in dev-qa but not in prod
   - Include normalized names
   - Include match scores

**Tools Pattern:**
```python
# compare_resources.py
def normalize_name(name: str, env_identifiers: List[str]) -> str:
    # Strip env prefixes, suffixes, infixes
    pass

def fuzzy_match(name: str, prod_names: Set[str], threshold: int = 85) -> Tuple[Optional[str], int]:
    # Use Levenshtein distance
    pass

def compare_resources(dev_qa_list: List, prod_list: List) -> List[Dict]:
    # Compare and return candidates
    pass
```

---

### Phase 2: Filter Out Known Patterns

**Goal:** Remove resources that should be excluded from cleanup

**Common filters:**
- Automation/test session resources (temporary by design)
- Personal test resources (dev-specific, not expected in prod)
- Environment-specific infrastructure

**Pattern:**
```bash
# Filter out automation sessions
grep -v "automation_session" candidates.csv > filtered_candidates.csv

# Filter out personal test resources
grep -v -E "(test|demo|sandbox)" filtered_candidates.csv > final_candidates.csv
```

**Output:** Filtered candidates CSV for deeper analysis

---

### Phase 3: Check Resource Activity

**Goal:** Determine when resources were last used

**Metrics to check:**

1. **Resource-specific attributes:**
   - Last modified timestamp
   - Current state/status
   - Associated data (messages, records, etc.)

2. **CloudWatch metrics (last 90 days):**
   - Usage metrics (invocations, requests, reads/writes)
   - Activity timestamps
   - Error rates

3. **Resource age:**
   - Creation timestamp
   - Days since creation

**Tools Pattern:**
```python
# check_resource_activity.py
def check_cloudwatch_metric(cloudwatch, resource_id: str, metric_name: str, days_back: int = 90) -> Dict:
    # Query CloudWatch for activity
    pass

def get_resource_attributes(client, resource_id: str) -> Dict:
    # Get resource metadata
    pass

def analyze_activity(resource: Resource, lookback_days: int = 90) -> ActivityReport:
    # Combine metrics into activity report
    pass
```

**Output columns:**
- `resource_name`
- `exists`
- `days_since_activity`
- `activity_type`
- `has_activity_90d` (Yes/No)
- `last_activity_date`
- `current_state`

**Interpretation:**
- **Safe to delete:** No activity in 30+ days, no current usage
- **Review required:** Activity in last 30 days
- **Keep:** Activity in last 7 days or currently in use

---

### Phase 4: Create Backup (CRITICAL!)

**Goal:** Create complete backup for safe restoration

**NEVER skip this step before deletion!**

**What to backup:**
- Resource configuration (all attributes)
- Tags
- Policies/permissions
- Associated resources (DLQs, alarms, etc.)
- Dependencies/relationships

**Format:** JSON with full resource definitions

**Tools Pattern:**
```python
# backup_resources.py
def backup_resource(client, resource_id: str) -> Dict:
    config = get_resource_configuration(client, resource_id)
    tags = get_resource_tags(client, resource_id)
    policies = get_resource_policies(client, resource_id)
    
    return {
        'resource_id': resource_id,
        'configuration': config,
        'tags': tags,
        'policies': policies,
        'backed_up_at': datetime.utcnow().isoformat()
    }

def export_backup(backups: List[Dict], output_file: str):
    # Save as JSON with metadata
    pass
```

**Backup metadata:**
```json
{
  "backup_metadata": {
    "resource_type": "AWS::SQS::Queue",
    "created_at": "2026-01-25T12:00:00Z",
    "total_resources": 100,
    "resources_found": 95,
    "account_id": "123456789012",
    "region": "us-east-1"
  },
  "resources": [...]
}
```

**Storage:**
- Multiple copies (local + shared drive + git)
- Clear naming: `{resource_type}_backup_{date}.json`
- Test restore before proceeding

---

### Phase 5: Manual Verification

**Goal:** Human validation before deletion

**Checklist for each resource:**

1. **Check AWS Console:**
   - Current state
   - Recent activity
   - Alarms/monitoring
   - Tags (owner, purpose, expiry)

2. **Search codebase:**
   ```bash
   git grep -i "resource-name"
   rg "resource-name" --type py --type ts
   ```

3. **Check IaC repositories:**
   - Terraform state
   - CloudFormation stacks
   - CDK definitions

4. **Check dependencies:**
   - What depends on this resource?
   - What does this resource depend on?
   - Will deletion break anything?

5. **Ask team:**
   - Slack/email resource owner
   - Check with platform/SRE team
   - Verify with service owners

**Red flags (don't delete):**
- Recent activity (< 7 days)
- Referenced in active code
- Has production-like tags
- Owner raises concerns
- Unclear purpose

---

### Phase 6: Safe Deletion

**Goal:** Delete resources in controlled, reversible manner

**Deletion strategy:**

1. **Start small:** 5-10 resources per batch
2. **Monitor:** Watch for errors/alerts for 24 hours
3. **Iterate:** Gradually increase batch size
4. **Document:** Keep log of what was deleted

**Deletion patterns:**

**Option A: Manual (safest):**
```bash
# Delete one by one via console
# Best for first few deletions
```

**Option B: Scripted with confirmation:**
```python
# delete_resources.py with dry-run
def delete_resource(client, resource_id: str, dry_run: bool = True):
    if dry_run:
        print(f"Would delete: {resource_id}")
        return
    
    print(f"Deleting: {resource_id}")
    client.delete_resource(ResourceId=resource_id)
    print("✓ Deleted")

# Always require explicit confirmation
confirmation = input("Type 'DELETE' to confirm: ")
if confirmation != 'DELETE':
    sys.exit(0)
```

**Safety features:**
- Dry-run mode by default
- Confirmation prompts
- Per-resource progress
- Error handling (continue on failure)
- Summary report

**Monitoring after deletion:**
- Application logs
- CloudWatch alarms
- Error tracking (Sentry, etc.)
- Team feedback

---

### Phase 7: Handle Resource-Specific Patterns

**Goal:** Clean up resources with special patterns (e.g., session-based)

**Example: Automation session resources**

Pattern: `{prefix}_automation_session_{hash}_{suffix}`

Strategy:
- Delete sessions older than X days
- Keep today's sessions (still active)
- Use pattern matching for bulk operations

**Tools Pattern:**
```python
# delete_pattern_resources.py
def list_matching_resources(client, pattern: str) -> List[Resource]:
    # Find resources matching pattern
    pass

def filter_by_age(resources: List[Resource], exclude_date: date) -> List[Resource]:
    # Keep resources created on exclude_date
    pass

def bulk_delete(client, resources: List[Resource], dry_run: bool = True):
    # Delete with safety checks
    pass
```

---

## Resource-Specific Adaptations

### SQS Queues (Implemented)

**Unique aspects:**
- Messages cannot be backed up (only config)
- Check CloudWatch: NumberOfMessagesSent, NumberOfMessagesReceived, ApproximateNumberOfMessagesVisible
- DLQ relationships matter
- FIFO vs Standard queues

**Scripts:**
- `compare_sqs_queues.py`
- `check_queue_activity.py` / `check_queue_usage.py`
- `backup_queues.py` / `restore_queues.py`
- `delete_automation_queues.py`

### Lambda Functions (Template)

**Unique aspects:**
- Check invocation metrics
- Code cannot be backed up (use versions/aliases)
- Environment variables
- IAM roles/permissions
- Event source mappings

**Metrics to check:**
- Invocations (last 90 days)
- Errors
- Duration
- Concurrent executions

**Backup needs:**
- Function configuration
- Environment variables
- IAM role ARN
- Event source mappings
- Layers
- Tags

### DynamoDB Tables (Template)

**Unique aspects:**
- Data can be backed up (export to S3)
- Check read/write metrics
- GSI/LSI configurations
- Point-in-time recovery
- Backup schedule

**Metrics to check:**
- ConsumedReadCapacityUnits
- ConsumedWriteCapacityUnits
- UserErrors
- SystemErrors

**Backup needs:**
- Table schema
- Indexes
- Capacity settings
- Point-in-time recovery settings
- Data export (optional but recommended)

### S3 Buckets (Template)

**Unique aspects:**
- Data size matters
- Versioning enabled?
- Lifecycle policies
- Public access settings
- Cross-region replication

**Metrics to check:**
- NumberOfObjects
- BucketSizeBytes
- AllRequests
- GetRequests
- PutRequests

**Backup needs:**
- Bucket policy
- CORS configuration
- Lifecycle rules
- Versioning settings
- Encryption settings
- Object inventory (for restoration)

---

## Workflow Summary (Apply to Any Resource)

```
1. Export Lists
   ├─ dev-qa: List all resources
   └─ prod: List all resources

2. Compare
   ├─ Normalize names
   ├─ Fuzzy match
   └─ Generate candidates CSV

3. Filter
   ├─ Remove automation/session resources
   └─ Remove known patterns

4. Check Activity
   ├─ CloudWatch metrics (90 days)
   ├─ Resource attributes
   └─ Calculate days since activity

5. Backup (CRITICAL!)
   ├─ Export configurations
   ├─ Save to JSON
   └─ Test restore

6. Verify
   ├─ Check console
   ├─ Search codebase
   ├─ Check IaC
   └─ Ask team

7. Delete
   ├─ Small batches (5-10)
   ├─ Monitor 24h
   ├─ Document
   └─ Iterate

8. Handle Patterns
   └─ Bulk delete session-based resources
```

---

## Key Principles

### 1. Safety First
- Always backup before deletion
- Start with small batches
- Monitor after each batch
- Be ready to restore

### 2. Evidence-Based
- Don't guess - check metrics
- Use CloudWatch for activity
- Verify with multiple sources
- Document findings

### 3. Reversibility
- Keep backups for 90 days minimum
- Test restore process
- Document what was deleted
- Store backups securely

### 4. Team Collaboration
- Share deletion candidates
- Get approval from owners
- Document rationale
- Update runbooks

### 5. Automation with Safety
- Dry-run by default
- Require explicit confirmation
- Per-resource feedback
- Error handling

---

## Files & Structure (Template)

```
resource_cleanup/
├── README.md                          # Overview
├── WORKFLOW.md                        # Detailed workflow
├── SKILL_resource_comparison.md       # This skill
├── compare_resources.py               # Phase 1: Compare
├── check_resource_activity.py         # Phase 3: Activity check
├── backup_resources.py                # Phase 4: Backup
├── restore_resources.py               # Phase 4: Restore
├── delete_pattern_resources.py        # Phase 7: Bulk delete
├── BACKUP_RESTORE_README.md          # Backup documentation
├── requirements.txt                   # Dependencies
├── dev-qa_export.csv                 # Input: dev-qa resources
├── prod_export.csv                   # Input: prod resources
├── deletion_candidates.csv           # Output: Candidates
├── deletion_candidates_filtered.csv  # Output: Filtered
├── activity_report.csv               # Output: Activity
├── backup_TIMESTAMP.json             # Output: Backup
└── resources_to_delete.txt           # Input: Final list
```

---

## Common Pitfalls & Solutions

### Pitfall 1: Deleting Active Resources

**Problem:** Resource looks inactive but is actually used

**Solution:**
- Check multiple metrics (not just one)
- Use 90-day lookback minimum
- Verify with team
- Start with obviously dead resources (180+ days)

### Pitfall 2: Breaking Dependencies

**Problem:** Deleting resource breaks dependent services

**Solution:**
- Map dependencies first
- Check what references this resource
- Search codebase thoroughly
- Test in sandbox first

### Pitfall 3: Insufficient Backup

**Problem:** Can't restore because backup incomplete

**Solution:**
- Backup ALL attributes (not just basic config)
- Include relationships (DLQs, triggers, etc.)
- Test restore in sandbox
- Keep multiple backup copies

### Pitfall 4: Lost Data

**Problem:** Deleted resource had important data

**Solution:**
- For data stores (S3, DynamoDB): export data first
- For queues: ensure no critical messages
- Document that data is not recoverable
- Get explicit approval for data loss

### Pitfall 5: Premature Bulk Deletion

**Problem:** Deleted too many resources at once, hard to identify issues

**Solution:**
- Start with 5-10 resources
- Monitor for 24 hours
- Gradually increase batch size
- Keep detailed deletion log

---

## Success Criteria

### Metrics

- ✅ Reduced resource count by X%
- ✅ All remaining resources active (< 30 days)
- ✅ Zero production incidents from deletion
- ✅ Cost savings of $X/month
- ✅ Improved resource visibility

### Documentation

- ✅ Backup files stored securely
- ✅ Deletion log with rationale
- ✅ Updated IaC to prevent recreation
- ✅ Runbook for future cleanups
- ✅ Team awareness of process

---

## Adaptation Checklist (New Resource Type)

When applying this to a new resource type:

- [ ] Identify resource-specific naming patterns
- [ ] Determine environment identifiers to strip
- [ ] List key CloudWatch metrics to check
- [ ] Document what can/cannot be backed up
- [ ] Map resource dependencies
- [ ] Define "active" vs "inactive" criteria
- [ ] Create resource-specific backup script
- [ ] Create resource-specific restore script
- [ ] Test backup/restore in sandbox
- [ ] Document resource-specific considerations

---

## Tools & Technologies

**Required:**
- Python 3.7+
- boto3 (AWS SDK)
- AWS CLI configured
- pandas (CSV manipulation)
- fuzzywuzzy (fuzzy matching)

**Optional:**
- ControlMonkey (resource exports)
- jq (JSON processing)
- AWS CloudWatch Insights (advanced queries)

---

## Example: Applying to Lambda Functions

```bash
# 1. Export Lambda lists
aws lambda list-functions --profile dev-qa --query 'Functions[].FunctionName' > dev-qa-lambdas.json
aws lambda list-functions --profile prod --query 'Functions[].FunctionName' > prod-lambdas.json

# 2. Compare
python compare_lambdas.py dev-qa-lambdas.json prod-lambdas.json

# 3. Check activity
python check_lambda_activity.py lambda_candidates.csv --profile dev-qa

# 4. Backup
python backup_lambdas.py lambdas_to_delete.txt --profile dev-qa

# 5. Delete (dry-run first)
python delete_lambdas.py lambdas_to_delete.txt --profile dev-qa --dry-run
python delete_lambdas.py lambdas_to_delete.txt --profile dev-qa --execute
```

---

## References

- [SQS Cleanup Implementation](./README.md)
- [Workflow Guide](./WORKFLOW.md)
- [Backup/Restore Documentation](./BACKUP_RESTORE_README.md)
- [Activity Checking](./ACTIVITY_CHECK_README.md)

---

## Maintenance

**Review this skill:**
- After each resource cleanup
- When patterns change
- When new resource types are added
- Quarterly process review

**Update with:**
- Lessons learned
- New patterns discovered
- Tool improvements
- Team feedback
