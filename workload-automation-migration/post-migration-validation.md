# Post-Migration Validation Guide for IBM Workload Automation

## Overview

This document provides comprehensive validation procedures to ensure the successful migration from IBM Workload Automation SaaS to a self-managed 10.1 environment. All validation steps must be completed and documented before declaring the migration successful.

---

## Validation Phases

1. **Immediate Post-Cutover Validation** (Hours 0-12)
2. **Short-Term Validation** (Days 1-7)
3. **Medium-Term Validation** (Weeks 2-4)
4. **Long-Term Validation** (Months 2-3)

---

## Phase 1: Immediate Post-Cutover Validation (Hours 0-12)

### 1.1 System Health Checks

#### Master Domain Manager (MDM) Validation
```bash
# Check MDM status
conman "status @"

# Verify MDM processes
ps -ef | grep TWS

# Check MDM logs for errors
tail -f $TWS_HOME/stdlist/logs/JobManager.log

# Verify database connectivity
conman "showcpus"
```

**Validation Criteria:**
- [ ] MDM status shows "ACTIVE"
- [ ] All required processes running
- [ ] No critical errors in logs
- [ ] Database connection successful
- [ ] License valid and not expired

**Expected Results:**
- MDM operational
- All services started
- Database responsive
- No error messages

#### Dynamic Workload Console (DWC) Validation
```bash
# Check DWC service status
systemctl status DWC

# Verify web server
curl -k https://localhost:9443/console

# Check DWC logs
tail -f $DWC_HOME/logs/console.log
```

**Validation Criteria:**
- [ ] DWC service running
- [ ] Web interface accessible
- [ ] Login page loads correctly
- [ ] No errors in logs
- [ ] SSL certificate valid

**Expected Results:**
- DWC accessible via browser
- Login successful
- Dashboard loads
- No JavaScript errors

#### Agent Validation
```bash
# List all agents
conman "showagents"

# Check agent status
conman "showagents;status"

# Test agent connectivity
conman "link @#AGENT_NAME"
```

**Validation Criteria:**
- [ ] All agents listed
- [ ] All agents show "LINKED" status
- [ ] Agent communication successful
- [ ] No connection errors

**Expected Results:**
- All agents connected
- Communication bidirectional
- No timeout errors

### 1.2 Data Integrity Validation

#### Object Count Verification
```sql
-- Count job definitions
SELECT COUNT(*) FROM JOB_DEFINITIONS 
WHERE FOLDER_PATH LIKE '/MIGRATED_WORKLOADS%';

-- Count job streams
SELECT COUNT(*) FROM JOB_STREAMS 
WHERE FOLDER_PATH LIKE '/MIGRATED_WORKLOADS%';

-- Count calendars
SELECT COUNT(*) FROM CALENDARS 
WHERE FOLDER_PATH LIKE '/MIGRATED_WORKLOADS%';

-- Count variables
SELECT COUNT(*) FROM VARIABLES 
WHERE FOLDER_PATH LIKE '/MIGRATED_WORKLOADS%';
```

**Validation Criteria:**
- [ ] Job definition count matches source
- [ ] Job stream count matches source
- [ ] Calendar count matches source
- [ ] Variable count matches source
- [ ] Resource count matches source
- [ ] Prompt count matches source

**Expected Results:**
- 100% of objects migrated
- No missing definitions
- Counts match export manifest

#### Dependency Validation
```bash
# Check job dependencies
conman "sj /MIGRATED_WORKLOADS/@;deps"

# Verify predecessor/successor relationships
conman "sj /MIGRATED_WORKLOADS/CRITICAL_JOB;showdeps"
```

**Validation Criteria:**
- [ ] All dependencies preserved
- [ ] Predecessor relationships correct
- [ ] Successor relationships correct
- [ ] Conditional dependencies intact
- [ ] Cross-folder dependencies working (if any)

**Expected Results:**
- All dependencies functional
- No broken links
- Dependency chains complete

### 1.3 Security Validation

#### Access Control Verification
```bash
# Verify folder permissions
conman "showsec /MIGRATED_WORKLOADS"

# Test user access
# Login as different user roles and verify access
```

**Validation Criteria:**
- [ ] Migration admin group has full control
- [ ] Migration operator group has modify rights
- [ ] Migration viewer group has read-only access
- [ ] Production operators cannot access migration folder
- [ ] Audit group has read access

**Expected Results:**
- ACLs correctly applied
- Isolation maintained
- No unauthorized access

#### Authentication Testing
```bash
# Test LDAP authentication
# Attempt login with various user accounts

# Verify MFA (if enabled)
# Test MFA for admin accounts

# Check audit logs
tail -f $TWS_HOME/stdlist/logs/audit.log
```

**Validation Criteria:**
- [ ] LDAP authentication working
- [ ] User credentials validated
- [ ] MFA functioning (if enabled)
- [ ] Failed login attempts logged
- [ ] Successful logins logged

**Expected Results:**
- All users can authenticate
- MFA prompts appear
- Audit trail complete

### 1.4 Critical Job Execution

#### First Job Execution Test
```bash
# Submit a test job
conman "sbj /MIGRATED_WORKLOADS/TEST_JOB"

# Monitor job execution
conman "sj /MIGRATED_WORKLOADS/TEST_JOB;st"

# Check job output
conman "sj /MIGRATED_WORKLOADS/TEST_JOB;stdlist"
```

**Validation Criteria:**
- [ ] Job submitted successfully
- [ ] Job executed on correct agent
- [ ] Job completed successfully
- [ ] Job output captured
- [ ] Job logs available

**Expected Results:**
- Job status: SUCC
- Execution time reasonable
- Output as expected
- No errors

#### Critical Job Stream Validation
```bash
# List critical job streams
conman "sj /MIGRATED_WORKLOADS/CRITICAL_*"

# Verify scheduling
conman "sj /MIGRATED_WORKLOADS/CRITICAL_STREAM;showsched"

# Check job stream status
conman "sj /MIGRATED_WORKLOADS/CRITICAL_STREAM;st"
```

**Validation Criteria:**
- [ ] All critical job streams identified
- [ ] Schedules correctly configured
- [ ] Dependencies working
- [ ] Jobs executing in correct order
- [ ] No job failures

**Expected Results:**
- Critical jobs running
- Schedules active
- No delays or failures

### 1.5 Integration Validation

#### Database Connections
```bash
# Test database job
conman "sbj /MIGRATED_WORKLOADS/DB_TEST_JOB"

# Verify connection
# Check job logs for database connectivity
```

**Validation Criteria:**
- [ ] Database connections successful
- [ ] SQL queries executing
- [ ] Data retrieval working
- [ ] No connection timeouts

**Expected Results:**
- Database jobs complete
- Data accurate
- Performance acceptable

#### File Transfer Validation
```bash
# Test file transfer job
conman "sbj /MIGRATED_WORKLOADS/FTP_TEST_JOB"

# Verify file transfer
# Check destination for transferred files
```

**Validation Criteria:**
- [ ] File transfers successful
- [ ] Files transferred completely
- [ ] File permissions correct
- [ ] No transfer errors

**Expected Results:**
- Files at destination
- File integrity verified
- Transfer logs clean

#### Email Notification Testing
```bash
# Trigger notification job
conman "sbj /MIGRATED_WORKLOADS/NOTIFICATION_TEST"

# Verify email received
# Check email content and formatting
```

**Validation Criteria:**
- [ ] Email notifications sent
- [ ] Recipients correct
- [ ] Content accurate
- [ ] Formatting correct

**Expected Results:**
- Emails delivered
- Content as expected
- No delivery failures

---

## Phase 2: Short-Term Validation (Days 1-7)

### 2.1 Daily Operations Validation

#### Day 1 Validation
**Focus: Stability and Basic Functionality**

- [ ] All scheduled jobs executed
- [ ] No unexpected failures
- [ ] Performance within acceptable range
- [ ] No security incidents
- [ ] User access working
- [ ] Monitoring alerts appropriate
- [ ] Backup completed successfully

**Daily Checklist:**
```bash
# Morning checks
conman "sj @;st" | grep -i "FAIL\|ABEND\|ERROR"
conman "showagents;status" | grep -v "LINKED"

# Review overnight jobs
conman "sj @;st;from=yesterday"

# Check system resources
df -h
free -m
top -b -n 1 | head -20

# Review logs
grep -i "error\|critical\|fatal" $TWS_HOME/stdlist/logs/*.log
```

#### Day 2-3 Validation
**Focus: Pattern Analysis**

- [ ] Job execution patterns normal
- [ ] Resource utilization stable
- [ ] No recurring errors
- [ ] Performance trends acceptable
- [ ] User feedback positive
- [ ] Integration points stable

**Metrics to Track:**
- Average job duration
- Job success rate
- System resource usage
- User login frequency
- Error frequency

#### Day 4-7 Validation
**Focus: Comprehensive Testing**

- [ ] All job types tested
- [ ] Peak load handling verified
- [ ] Failover tested (if HA)
- [ ] Backup/restore tested
- [ ] Disaster recovery procedures validated
- [ ] User acceptance testing completed

### 2.2 Performance Validation

#### Job Execution Performance
```sql
-- Compare execution times
SELECT 
    job_name,
    AVG(execution_time) as avg_time,
    MAX(execution_time) as max_time,
    MIN(execution_time) as min_time
FROM job_history
WHERE execution_date >= CURRENT_DATE - 7
GROUP BY job_name
ORDER BY avg_time DESC;
```

**Validation Criteria:**
- [ ] Average execution time ≤ SaaS baseline
- [ ] No significant performance degradation
- [ ] Peak load handled successfully
- [ ] Resource utilization optimal

**Performance Benchmarks:**
| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| Avg Job Duration | ≤ SaaS + 10% | | |
| Job Success Rate | ≥ 99% | | |
| System CPU | ≤ 70% | | |
| System Memory | ≤ 80% | | |
| Database Response | ≤ 100ms | | |

#### System Performance
```bash
# CPU utilization
sar -u 1 10

# Memory utilization
vmstat 1 10

# Disk I/O
iostat -x 1 10

# Network throughput
sar -n DEV 1 10
```

**Validation Criteria:**
- [ ] CPU utilization < 70% average
- [ ] Memory utilization < 80%
- [ ] Disk I/O not saturated
- [ ] Network latency acceptable
- [ ] No resource bottlenecks

### 2.3 User Acceptance Testing

#### Business User Validation
**Test Scenarios:**

1. **Job Submission**
   - [ ] Users can submit jobs
   - [ ] Job parameters work correctly
   - [ ] Job output accessible

2. **Job Monitoring**
   - [ ] Users can view job status
   - [ ] Job logs accessible
   - [ ] Alerts received appropriately

3. **Reporting**
   - [ ] Reports generate correctly
   - [ ] Data accurate
   - [ ] Export functionality works

4. **User Interface**
   - [ ] DWC responsive
   - [ ] Navigation intuitive
   - [ ] Search functionality works

**User Feedback Form:**
| Area | Rating (1-5) | Comments |
|------|--------------|----------|
| Ease of Use | | |
| Performance | | |
| Functionality | | |
| Reliability | | |
| Overall Satisfaction | | |

**Acceptance Criteria:**
- [ ] Average rating ≥ 4.0
- [ ] No critical issues reported
- [ ] All required functionality available
- [ ] User training adequate

### 2.4 Integration Testing

#### End-to-End Workflow Testing
**Test Critical Business Workflows:**

1. **Financial Close Process**
   - [ ] All jobs execute in sequence
   - [ ] Data transfers complete
   - [ ] Reports generated
   - [ ] Notifications sent
   - [ ] Timing meets requirements

2. **Data Warehouse Load**
   - [ ] Extract jobs complete
   - [ ] Transform jobs execute
   - [ ] Load jobs successful
   - [ ] Data validation passes
   - [ ] Performance acceptable

3. **Batch Processing**
   - [ ] Batch jobs submit correctly
   - [ ] Processing completes on time
   - [ ] Output files generated
   - [ ] Downstream systems updated

**Integration Points:**
- [ ] ERP systems
- [ ] Database systems
- [ ] File transfer systems
- [ ] Email systems
- [ ] Monitoring systems
- [ ] Ticketing systems

---

## Phase 3: Medium-Term Validation (Weeks 2-4)

### 3.1 Stability Assessment

#### Week 2 Validation
**Focus: Sustained Operations**

- [ ] 7 days of stable operation
- [ ] No major incidents
- [ ] Performance consistent
- [ ] User satisfaction maintained
- [ ] All SLAs met
- [ ] Backup/recovery validated

**Metrics Review:**
```sql
-- Weekly job statistics
SELECT 
    DATE(execution_date) as date,
    COUNT(*) as total_jobs,
    SUM(CASE WHEN status = 'SUCC' THEN 1 ELSE 0 END) as successful,
    SUM(CASE WHEN status = 'FAIL' THEN 1 ELSE 0 END) as failed,
    AVG(execution_time) as avg_duration
FROM job_history
WHERE execution_date >= CURRENT_DATE - 14
GROUP BY DATE(execution_date)
ORDER BY date;
```

#### Week 3-4 Validation
**Focus: Optimization and Tuning**

- [ ] Performance optimization completed
- [ ] Resource allocation tuned
- [ ] Monitoring thresholds adjusted
- [ ] User feedback incorporated
- [ ] Documentation updated
- [ ] Training gaps addressed

### 3.2 Capacity Validation

#### Resource Capacity Assessment
```bash
# Analyze resource trends
sar -A -f /var/log/sa/sa* | grep -A 5 "Average"

# Database capacity
SELECT 
    tablespace_name,
    ROUND(used_space/1024/1024, 2) as used_mb,
    ROUND(free_space/1024/1024, 2) as free_mb,
    ROUND((used_space/total_space)*100, 2) as pct_used
FROM dba_tablespace_usage;

# Disk space trends
df -h | awk '{print $1, $5}' | grep -v "Use%"
```

**Validation Criteria:**
- [ ] Sufficient capacity for growth
- [ ] No resource constraints
- [ ] Scalability validated
- [ ] Capacity planning updated

**Capacity Metrics:**
| Resource | Current | Projected (6mo) | Action Required |
|----------|---------|-----------------|-----------------|
| CPU | | | |
| Memory | | | |
| Disk Space | | | |
| Database | | | |
| Network | | | |

### 3.3 Disaster Recovery Validation

#### DR Test Execution
```bash
# Simulate failure scenario
# Document recovery steps
# Measure recovery time
# Validate data integrity
```

**DR Test Scenarios:**

1. **MDM Failure**
   - [ ] Failover to backup MDM
   - [ ] Jobs continue execution
   - [ ] No data loss
   - [ ] Recovery time < RTO

2. **Database Failure**
   - [ ] Database restored from backup
   - [ ] Data integrity verified
   - [ ] Services resumed
   - [ ] Recovery time < RTO

3. **Agent Failure**
   - [ ] Jobs rerouted to backup agent
   - [ ] No job failures
   - [ ] Automatic recovery

4. **Complete Site Failure**
   - [ ] DR site activated
   - [ ] All services restored
   - [ ] Business continuity maintained

**DR Validation Criteria:**
- [ ] RTO met (< 4 hours)
- [ ] RPO met (< 15 minutes)
- [ ] All critical systems recovered
- [ ] Data integrity verified
- [ ] Business operations resumed

---

## Phase 4: Long-Term Validation (Months 2-3)

### 4.1 Consolidation Readiness

#### Pre-Consolidation Assessment
**Evaluate readiness to merge folders:**

- [ ] 30+ days of stable operation
- [ ] All issues resolved
- [ ] Performance optimized
- [ ] User acceptance achieved
- [ ] Security validated
- [ ] Compliance verified

**Consolidation Criteria:**
| Criterion | Target | Actual | Ready? |
|-----------|--------|--------|--------|
| Uptime | > 99.5% | | |
| Job Success Rate | > 99% | | |
| User Satisfaction | > 4.0/5 | | |
| Incident Count | < 5/month | | |
| SLA Compliance | 100% | | |

### 4.2 Folder Consolidation Validation

#### Post-Consolidation Checks
```bash
# Verify merged folder structure
conman "showfolders"

# Check security after merge
conman "showsec /PRODUCTION"

# Validate all jobs accessible
conman "sj @;st"

# Test cross-folder dependencies
conman "sj @;deps"
```

**Validation Criteria:**
- [ ] Folder structure optimized
- [ ] Security policies unified
- [ ] All jobs accessible
- [ ] Dependencies intact
- [ ] No functionality lost

### 4.3 SaaS Decommissioning Validation

#### Pre-Decommissioning Checklist
- [ ] All data migrated and validated
- [ ] All integrations cutover
- [ ] All users migrated
- [ ] No dependencies on SaaS
- [ ] Backup of SaaS data retained
- [ ] Decommissioning approved

#### Post-Decommissioning Validation
- [ ] SaaS environment shut down
- [ ] No impact on operations
- [ ] All functionality available
- [ ] Users adapted to new environment
- [ ] Documentation updated

---

## Validation Reporting

### Daily Status Report Template

```
IBM Workload Automation Migration - Daily Status Report
Date: [Date]
Report Period: [Time Range]

EXECUTIVE SUMMARY:
- Overall Status: [Green/Yellow/Red]
- Jobs Executed: [Count]
- Success Rate: [Percentage]
- Critical Issues: [Count]

SYSTEM HEALTH:
- MDM Status: [Status]
- DWC Status: [Status]
- Agent Status: [X/Y Linked]
- Database Status: [Status]

JOB EXECUTION:
- Total Jobs: [Count]
- Successful: [Count]
- Failed: [Count]
- Running: [Count]
- Waiting: [Count]

PERFORMANCE METRICS:
- Avg Job Duration: [Time]
- System CPU: [Percentage]
- System Memory: [Percentage]
- Database Response: [ms]

ISSUES AND RESOLUTIONS:
1. [Issue Description] - [Status] - [Owner]
2. [Issue Description] - [Status] - [Owner]

UPCOMING ACTIVITIES:
- [Activity 1]
- [Activity 2]

RISKS AND CONCERNS:
- [Risk 1]
- [Risk 2]

Prepared by: [Name]
Next Report: [Date/Time]
```

### Weekly Summary Report Template

```
IBM Workload Automation Migration - Weekly Summary
Week Ending: [Date]

MIGRATION STATUS:
- Phase: [Current Phase]
- Completion: [Percentage]
- On Schedule: [Yes/No]

KEY METRICS:
- Total Jobs Executed: [Count]
- Overall Success Rate: [Percentage]
- Average Job Duration: [Time]
- System Uptime: [Percentage]
- User Satisfaction: [Score]

ACCOMPLISHMENTS:
- [Accomplishment 1]
- [Accomplishment 2]
- [Accomplishment 3]

ISSUES RESOLVED:
- [Issue 1] - [Resolution]
- [Issue 2] - [Resolution]

OPEN ISSUES:
- [Issue 1] - [Priority] - [Owner] - [ETA]
- [Issue 2] - [Priority] - [Owner] - [ETA]

RISKS:
- [Risk 1] - [Mitigation]
- [Risk 2] - [Mitigation]

NEXT WEEK FOCUS:
- [Activity 1]
- [Activity 2]
- [Activity 3]

RECOMMENDATIONS:
- [Recommendation 1]
- [Recommendation 2]

Prepared by: [Name]
Distribution: [List]
```

---

## Success Criteria Summary

### Technical Success Criteria
- [x] 100% of objects migrated
- [x] Zero data loss
- [x] All critical jobs executing
- [x] Performance meets/exceeds baseline
- [x] Security controls operational
- [x] Monitoring functional
- [x] Backup/recovery validated
- [x] High availability working (if applicable)
- [x] All integrations functional
- [x] Documentation complete

### Business Success Criteria
- [x] No SLA violations
- [x] User satisfaction > 80%
- [x] Cutover within planned window
- [x] All business processes operational
- [x] Support tickets within normal range
- [x] Stakeholder approval obtained
- [x] Budget within limits
- [x] Timeline met

### Operational Success Criteria
- [x] Team trained and competent
- [x] Procedures documented
- [x] Runbooks validated
- [x] Support model established
- [x] Change management operational
- [x] Incident response tested
- [x] Continuous improvement plan in place

---

## Final Sign-Off

### Validation Completion Certificate

```
IBM WORKLOAD AUTOMATION MIGRATION
POST-MIGRATION VALIDATION CERTIFICATE

This certifies that the migration from IBM Workload Automation SaaS 
to self-managed IBM Workload Automation 10.1 has been completed and 
validated according to the established criteria.

Validation Period: [Start Date] to [End Date]

VALIDATION RESULTS:
✓ All technical criteria met
✓ All business criteria met
✓ All operational criteria met
✓ All stakeholder requirements satisfied

APPROVALS:

Technical Lead: _________________ Date: _______
Signature

Business Owner: _________________ Date: _______
Signature

Security Officer: _________________ Date: _______
Signature

Project Manager: _________________ Date: _______
Signature

Executive Sponsor: _________________ Date: _______
Signature

MIGRATION STATUS: SUCCESSFUL

Date of Completion: [Date]
```

---

## Appendix

### A. Validation Scripts

#### System Health Check Script
```bash
#!/bin/bash
# system_health_check.sh

echo "=== IBM Workload Automation Health Check ==="
echo "Date: $(date)"
echo ""

echo "1. MDM Status:"
conman "status @"
echo ""

echo "2. Agent Status:"
conman "showagents;status"
echo ""

echo "3. Recent Job Failures:"
conman "sj @;st" | grep -i "FAIL\|ABEND"
echo ""

echo "4. System Resources:"
echo "CPU:"
top -b -n 1 | head -5
echo ""
echo "Memory:"
free -h
echo ""
echo "Disk:"
df -h
echo ""

echo "5. Recent Errors in Logs:"
grep -i "error\|critical" $TWS_HOME/stdlist/logs/*.log | tail -20
echo ""

echo "=== Health Check Complete ==="
```

#### Job Statistics Script
```bash
#!/bin/bash
# job_statistics.sh

echo "=== Job Statistics Report ==="
echo "Date: $(date)"
echo ""

# Today's jobs
echo "Today's Job Summary:"
conman "sj @;st;from=today" | awk '
    /SUCC/ {succ++}
    /FAIL/ {fail++}
    /ABEND/ {abend++}
    /EXEC/ {exec++}
    END {
        print "Successful: " succ
        print "Failed: " fail
        print "Abended: " abend
        print "Executing: " exec
        total = succ + fail + abend + exec
        print "Total: " total
        if (total > 0) {
            print "Success Rate: " (succ/total)*100 "%"
        }
    }
'
echo ""

echo "=== Report Complete ==="
```

### B. Validation Checklists

#### Daily Validation Checklist
```
Date: ___________  Validator: ___________

Morning Checks:
[ ] MDM operational
[ ] DWC accessible
[ ] All agents linked
[ ] No critical errors in logs
[ ] Overnight jobs completed
[ ] Backup successful

Midday Checks:
[ ] Job execution on schedule
[ ] No performance issues
[ ] User access working
[ ] No security alerts

Evening Checks:
[ ] All scheduled jobs completed
[ ] System resources normal
[ ] No open critical incidents
[ ] Tomorrow's schedule loaded

Issues Identified:
_________________________________
_________________________________

Actions Taken:
_________________________________
_________________________________

Sign-off: ___________
```

### C. Contact Information

**Escalation Matrix:**

| Level | Role | Contact | Response Time |
|-------|------|---------|---------------|
| L1 | Operations Team | [Contact] | 15 minutes |
| L2 | Technical Lead | [Contact] | 30 minutes |
| L3 | Project Manager | [Contact] | 1 hour |
| L4 | Executive Sponsor | [Contact] | 2 hours |

**Vendor Support:**
- IBM Support: [Number]
- Database Support: [Number]
- Network Support: [Number]

---

**Document Version:** 1.0  
**Last Updated:** [Date]  
**Next Review:** Post-Migration  
**Document Owner:** Project Manager