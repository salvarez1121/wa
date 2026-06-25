# Security and Isolation Requirements for Workload Automation Migration

## Executive Summary

This document defines the security architecture and isolation requirements for migrating IBM Workload Automation from SaaS to a self-managed 10.1 environment. The approach emphasizes defense-in-depth, least privilege access, and complete isolation of migrated workloads during the validation phase.

## Security Objectives

1. **Data Protection**: Ensure confidentiality, integrity, and availability of scheduling data
2. **Access Control**: Implement role-based access with least privilege principles
3. **Isolation**: Maintain complete separation between existing and migrated workloads
4. **Audit Compliance**: Enable comprehensive logging and monitoring
5. **Secure Communication**: Encrypt all data in transit and at rest
6. **Incident Response**: Establish procedures for security event handling

## Folder Isolation Strategy

### Folder Structure Design

```
/TWS (Root)
├── /PRODUCTION (Existing Workloads)
│   ├── /FINANCE
│   ├── /HR
│   ├── /OPERATIONS
│   └── /IT_SERVICES
│
└── /MIGRATED_WORKLOADS (Isolated Migration Folder)
    ├── /FINANCE_MIGRATED
    ├── /HR_MIGRATED
    ├── /OPERATIONS_MIGRATED
    └── /IT_SERVICES_MIGRATED
```

### Isolation Principles

1. **Physical Separation**: Distinct folder hierarchy
2. **Security Boundaries**: Separate ACLs for each domain
3. **Resource Isolation**: Dedicated agent pools where possible
4. **Network Segmentation**: Separate security groups if needed
5. **Audit Separation**: Distinct audit trails

## Access Control Lists (ACLs)

### ACL Design for Isolated Folders

#### Production Folder ACLs (Existing)

```
Folder: /PRODUCTION
├── Owner: PROD_ADMIN_GROUP
├── Full Control: PROD_ADMIN_GROUP
├── Modify: PROD_OPERATOR_GROUP
├── Read: PROD_VIEWER_GROUP
└── Deny: MIGRATED_ADMIN_GROUP (explicit deny during migration)

Security Domain: PRODUCTION_DOMAIN
```

#### Migrated Folder ACLs (New)

```
Folder: /MIGRATED_WORKLOADS
├── Owner: MIGRATION_ADMIN_GROUP
├── Full Control: MIGRATION_ADMIN_GROUP
├── Modify: MIGRATION_OPERATOR_GROUP
├── Read: MIGRATION_VIEWER_GROUP
├── Read: AUDIT_GROUP (for compliance)
└── Deny: PROD_OPERATOR_GROUP (prevent cross-contamination)

Security Domain: MIGRATION_DOMAIN
```

### Role Definitions

#### Migration Administrator Role
**Group**: `MIGRATION_ADMIN_GROUP`

**Permissions**:
- Full control over /MIGRATED_WORKLOADS folder
- Create, modify, delete job definitions
- Manage calendars and variables
- Configure security settings within migration domain
- View audit logs
- Execute emergency procedures

**Members**:
- Migration project lead
- Senior workload automation administrators
- Database administrators (read-only)

#### Migration Operator Role
**Group**: `MIGRATION_OPERATOR_GROUP`

**Permissions**:
- Modify job streams in /MIGRATED_WORKLOADS
- Submit and monitor jobs
- View job logs and output
- Modify job variables
- Cannot modify security settings
- Cannot delete job definitions

**Members**:
- Application support teams
- Operations staff
- Business analysts (testing phase)

#### Migration Viewer Role
**Group**: `MIGRATION_VIEWER_GROUP`

**Permissions**:
- Read-only access to /MIGRATED_WORKLOADS
- View job definitions and schedules
- View job execution history
- View reports
- Cannot modify any objects

**Members**:
- Business users
- Auditors
- Management
- Support staff

#### Audit Role
**Group**: `AUDIT_GROUP`

**Permissions**:
- Read-only access to all folders
- Access to all audit logs
- Access to security reports
- Cannot modify any objects
- Cannot execute jobs

**Members**:
- Compliance officers
- Security team
- Internal audit

## Security Configuration

### Authentication

#### LDAP/Active Directory Integration

```yaml
Authentication Configuration:
  Primary Method: LDAP/Active Directory
  Secondary Method: Local accounts (emergency only)
  
LDAP Settings:
  Server: ldaps://ldap.company.com:636
  Base DN: dc=company,dc=com
  User Search Base: ou=users,dc=company,dc=com
  Group Search Base: ou=groups,dc=company,dc=com
  
  Bind DN: cn=tws_service,ou=service_accounts,dc=company,dc=com
  SSL/TLS: Required
  Certificate Validation: Enabled
  
  User Attribute Mapping:
    Username: sAMAccountName
    Email: mail
    Full Name: displayName
    Groups: memberOf
```

#### Multi-Factor Authentication (MFA)

```yaml
MFA Configuration:
  Enabled: true
  Required For:
    - Administrative access
    - Remote access
    - Production folder access
  
  Methods:
    - TOTP (Time-based One-Time Password)
    - Push notifications
    - SMS (backup only)
  
  Exemptions:
    - Service accounts (with enhanced monitoring)
    - Emergency break-glass accounts (logged)
```

### Authorization Model

#### Role-Based Access Control (RBAC)

```
┌─────────────────────────────────────────────────────────┐
│              Authorization Hierarchy                     │
└─────────────────────────────────────────────────────────┘

Level 1: System Administrators
├── Full system access
├── Security configuration
├── User management
└── Emergency procedures

Level 2: Domain Administrators
├── Full access within assigned domain
├── User management within domain
├── Security configuration within domain
└── Cannot access other domains

Level 3: Operators
├── Job execution and monitoring
├── Variable modification
├── Log viewing
└── Limited to assigned folders

Level 4: Viewers
├── Read-only access
├── Report generation
└── No modification rights
```

### Network Security

#### Firewall Rules

```
Master Domain Manager (MDM):
  Inbound:
    - Port 31116 (TWS communication) - From agents only
    - Port 31131 (DWC communication) - From DWC server only
    - Port 22 (SSH) - From jump host only
    - Port 443 (HTTPS) - From DWC only
  
  Outbound:
    - Port 31116 - To agents
    - Port 1521/5432/50000 - To database
    - Port 389/636 - To LDAP
    - Port 25 - To mail server

Dynamic Workload Console (DWC):
  Inbound:
    - Port 443 (HTTPS) - From corporate network
    - Port 22 (SSH) - From jump host only
  
  Outbound:
    - Port 31131 - To MDM
    - Port 389/636 - To LDAP

Agents:
  Inbound:
    - Port 31116 - From MDM only
    - Port 22 (SSH) - From jump host only
  
  Outbound:
    - Port 31116 - To MDM
    - Application-specific ports
```

#### Network Segmentation

```
┌─────────────────────────────────────────────────────────┐
│              Network Zones                               │
└─────────────────────────────────────────────────────────┘

Management Zone (10.1.0.0/24):
├── Jump hosts
├── Monitoring servers
└── Administrative workstations

Control Zone (10.1.1.0/24):
├── Master Domain Manager (Primary)
├── Master Domain Manager (Backup)
├── Dynamic Workload Console
└── Database servers

Execution Zone (10.1.2.0/24):
├── Agent Pool 1 (Production)
├── Agent Pool 2 (Migrated)
└── Agent Pool 3 (Extended)

DMZ Zone (10.1.3.0/24):
├── External-facing agents
└── API gateways
```

### Encryption

#### Data in Transit

```yaml
TLS Configuration:
  Minimum Version: TLS 1.2
  Preferred Version: TLS 1.3
  
  Cipher Suites (Allowed):
    - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
    - TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
  
  Certificate Requirements:
    - 2048-bit RSA minimum
    - SHA-256 signature algorithm
    - Valid from trusted CA
    - Subject Alternative Names configured
    - Certificate expiry monitoring enabled
  
  Components Using TLS:
    - MDM to DWC communication
    - DWC to user browsers
    - MDM to agents
    - Database connections
    - LDAP connections
```

#### Data at Rest

```yaml
Database Encryption:
  Method: Transparent Data Encryption (TDE)
  Algorithm: AES-256
  Key Management: Hardware Security Module (HSM)
  Key Rotation: Every 90 days
  
File System Encryption:
  Method: LUKS (Linux) / BitLocker (Windows)
  Algorithm: AES-256-XTS
  Key Storage: TPM or HSM
  
Backup Encryption:
  Method: AES-256-GCM
  Key Management: Separate from production keys
  Verification: Encrypted backup restore testing
```

## Security Policies

### Password Policy

```yaml
Password Requirements:
  Minimum Length: 14 characters
  Complexity:
    - Uppercase letters: Required
    - Lowercase letters: Required
    - Numbers: Required
    - Special characters: Required
  
  History: 12 previous passwords
  Maximum Age: 90 days
  Minimum Age: 1 day
  Lockout Threshold: 5 failed attempts
  Lockout Duration: 30 minutes
  
Service Accounts:
  Password Length: 32 characters
  Rotation: Every 180 days
  Stored in: Password vault
  Access: Restricted to authorized personnel
```

### Session Management

```yaml
Session Configuration:
  Idle Timeout: 15 minutes
  Absolute Timeout: 8 hours
  Concurrent Sessions: 1 per user (configurable)
  
  Session Security:
    - Secure cookie flag: Enabled
    - HttpOnly flag: Enabled
    - SameSite: Strict
    - Session token rotation: On privilege escalation
```

### Audit Logging

```yaml
Audit Configuration:
  Log Level: INFO (minimum)
  
  Events to Log:
    Authentication:
      - Login attempts (success/failure)
      - Logout events
      - Password changes
      - MFA events
    
    Authorization:
      - Permission changes
      - Role assignments
      - ACL modifications
      - Access denials
    
    Operations:
      - Job submissions
      - Job modifications
      - Job deletions
      - Schedule changes
      - Variable modifications
      - Calendar updates
    
    Administrative:
      - Configuration changes
      - User management
      - Security policy changes
      - System maintenance
  
  Log Retention:
    - Online: 90 days
    - Archive: 7 years
    - Compliance logs: 10 years
  
  Log Protection:
    - Write-once storage
    - Integrity verification
    - Encrypted transmission to SIEM
    - Access restricted to audit group
```

## Isolation Implementation Steps

### Phase 1: Pre-Migration Security Setup

#### Step 1: Create Security Groups

```bash
# Create LDAP groups for migration
# Execute via LDAP admin tools

Groups to Create:
- MIGRATION_ADMIN_GROUP
- MIGRATION_OPERATOR_GROUP
- MIGRATION_VIEWER_GROUP
- AUDIT_GROUP

Group Membership:
- Assign appropriate users to each group
- Document group membership
- Obtain approval from security team
```

#### Step 2: Configure Folder Structure

```bash
# Create isolated folder structure in TWS
# Execute via composer CLI or DWC

composer create folder /MIGRATED_WORKLOADS \
  -owner MIGRATION_ADMIN_GROUP \
  -description "Isolated folder for SaaS migration"

# Create sub-folders
composer create folder /MIGRATED_WORKLOADS/FINANCE_MIGRATED
composer create folder /MIGRATED_WORKLOADS/HR_MIGRATED
composer create folder /MIGRATED_WORKLOADS/OPERATIONS_MIGRATED
composer create folder /MIGRATED_WORKLOADS/IT_SERVICES_MIGRATED
```

#### Step 3: Apply ACLs

```bash
# Set ACLs on migration folder
# Execute via security configuration tools

# Grant full control to migration admins
composer grant /MIGRATED_WORKLOADS \
  -group MIGRATION_ADMIN_GROUP \
  -permission FULL_CONTROL

# Grant modify to migration operators
composer grant /MIGRATED_WORKLOADS \
  -group MIGRATION_OPERATOR_GROUP \
  -permission MODIFY

# Grant read to migration viewers
composer grant /MIGRATED_WORKLOADS \
  -group MIGRATION_VIEWER_GROUP \
  -permission READ

# Grant read to audit group
composer grant /MIGRATED_WORKLOADS \
  -group AUDIT_GROUP \
  -permission READ

# Explicit deny to production operators
composer deny /MIGRATED_WORKLOADS \
  -group PROD_OPERATOR_GROUP \
  -permission ALL
```

### Phase 2: Import Security Configuration

#### Step 4: Map SaaS Users to Self-Managed

```yaml
User Mapping Strategy:
  
  SaaS User → Self-Managed User:
    - Match by email address
    - Validate against LDAP
    - Create missing users (with approval)
    - Document mapping in spreadsheet
  
  Role Mapping:
    SaaS Admin → MIGRATION_ADMIN_GROUP
    SaaS Operator → MIGRATION_OPERATOR_GROUP
    SaaS Viewer → MIGRATION_VIEWER_GROUP
  
  Orphaned Users:
    - Identify users not in LDAP
    - Contact user management
    - Create service accounts if needed
    - Document exceptions
```

#### Step 5: Import Security Definitions

```bash
# Import security definitions from SaaS export
# Modify to use new security groups

# Pre-process export file
sed -i 's/SAAS_ADMIN_GROUP/MIGRATION_ADMIN_GROUP/g' export.xml
sed -i 's/SAAS_OPERATOR_GROUP/MIGRATION_OPERATOR_GROUP/g' export.xml

# Import with security mapping
composer import export.xml \
  -folder /MIGRATED_WORKLOADS \
  -security-mapping security_map.xml \
  -validate-only

# Review validation results
# Fix any issues
# Execute actual import
composer import export.xml \
  -folder /MIGRATED_WORKLOADS \
  -security-mapping security_map.xml
```

### Phase 3: Validation and Testing

#### Step 6: Security Testing

```yaml
Security Test Cases:

1. Access Control Testing:
   - Verify migration admins can access migration folder
   - Verify production operators cannot access migration folder
   - Verify audit group has read-only access
   - Test explicit deny rules

2. Authentication Testing:
   - Test LDAP authentication
   - Test MFA for admin accounts
   - Test session timeout
   - Test concurrent session limits

3. Authorization Testing:
   - Test role-based permissions
   - Test folder-level security
   - Test object-level security
   - Test cross-folder access restrictions

4. Audit Testing:
   - Verify all security events are logged
   - Test log integrity
   - Verify SIEM integration
   - Test log retention policies

5. Encryption Testing:
   - Verify TLS for all connections
   - Test certificate validation
   - Verify database encryption
   - Test backup encryption
```

## Security Monitoring

### Real-Time Monitoring

```yaml
Security Monitoring Dashboard:

Metrics to Monitor:
  - Failed login attempts
  - Unauthorized access attempts
  - Privilege escalation events
  - Configuration changes
  - Unusual job execution patterns
  - Network anomalies
  - Certificate expiration warnings

Alerting Thresholds:
  Critical:
    - 5+ failed logins in 5 minutes
    - Unauthorized access to production folder
    - Security configuration changes
    - Disabled audit logging
  
  Warning:
    - 3+ failed logins in 10 minutes
    - Certificate expiring in 30 days
    - Unusual job execution times
    - High privilege account usage

Alert Destinations:
  - Security Operations Center (SOC)
  - Email to security team
  - SMS for critical alerts
  - Ticketing system integration
```

### Security Reporting

```yaml
Regular Security Reports:

Daily Reports:
  - Failed authentication summary
  - Access violations
  - Configuration changes
  - High-privilege account activity

Weekly Reports:
  - Security event trends
  - User access review
  - Certificate status
  - Compliance metrics

Monthly Reports:
  - Comprehensive security posture
  - Risk assessment
  - Compliance status
  - Recommendations for improvement

Quarterly Reports:
  - Security audit results
  - Penetration test findings
  - Vulnerability assessment
  - Strategic security initiatives
```

## Compliance Requirements

### Regulatory Compliance

```yaml
Compliance Frameworks:

SOX (Sarbanes-Oxley):
  - Segregation of duties
  - Change management controls
  - Audit trail requirements
  - Access control documentation

PCI-DSS (if applicable):
  - Network segmentation
  - Encryption requirements
  - Access control measures
  - Logging and monitoring

GDPR (if applicable):
  - Data protection measures
  - Access controls
  - Audit logging
  - Data retention policies

HIPAA (if applicable):
  - Access controls
  - Encryption
  - Audit trails
  - Incident response

Industry-Specific:
  - Document specific requirements
  - Implement controls
  - Regular compliance audits
  - Remediation tracking
```

### Audit Requirements

```yaml
Audit Preparation:

Documentation Required:
  - Security architecture diagram
  - Access control matrix
  - User access list
  - Security policies
  - Incident response procedures
  - Change management records
  - Security testing results
  - Compliance certifications

Audit Evidence:
  - Authentication logs
  - Authorization logs
  - Configuration change logs
  - Security event logs
  - Access review records
  - Training records
  - Incident reports

Audit Frequency:
  - Internal: Quarterly
  - External: Annually
  - Compliance: As required
  - Penetration testing: Annually
```

## Incident Response

### Security Incident Procedures

```yaml
Incident Response Plan:

Phase 1: Detection and Analysis
  - Monitor security alerts
  - Analyze suspicious activity
  - Determine incident severity
  - Document initial findings

Phase 2: Containment
  - Isolate affected systems
  - Disable compromised accounts
  - Block malicious traffic
  - Preserve evidence

Phase 3: Eradication
  - Remove malicious code
  - Patch vulnerabilities
  - Reset compromised credentials
  - Verify system integrity

Phase 4: Recovery
  - Restore from clean backups
  - Verify system functionality
  - Monitor for reinfection
  - Gradual service restoration

Phase 5: Post-Incident
  - Root cause analysis
  - Document lessons learned
  - Update security controls
  - Conduct training

Incident Severity Levels:

Critical (P1):
  - Data breach
  - System compromise
  - Ransomware attack
  - Response Time: Immediate

High (P2):
  - Unauthorized access
  - Malware detection
  - DDoS attack
  - Response Time: 1 hour

Medium (P3):
  - Policy violations
  - Suspicious activity
  - Failed security controls
  - Response Time: 4 hours

Low (P4):
  - Minor policy violations
  - Security warnings
  - Non-critical vulnerabilities
  - Response Time: 24 hours
```

## Security Best Practices

### Operational Security

1. **Least Privilege**: Grant minimum necessary permissions
2. **Separation of Duties**: No single person has complete control
3. **Regular Reviews**: Quarterly access reviews
4. **Secure Defaults**: Deny by default, allow by exception
5. **Defense in Depth**: Multiple layers of security controls
6. **Continuous Monitoring**: Real-time security monitoring
7. **Patch Management**: Regular security updates
8. **Secure Configuration**: Hardened system configurations
9. **Backup Security**: Encrypted, tested backups
10. **Incident Preparedness**: Regular drills and training

### Migration-Specific Security

1. **Isolated Testing**: Complete separation during validation
2. **Gradual Exposure**: Phased approach to consolidation
3. **Rollback Capability**: Maintain ability to revert
4. **Enhanced Monitoring**: Increased vigilance during migration
5. **Change Control**: Strict change management procedures
6. **Documentation**: Comprehensive security documentation
7. **Training**: Security awareness for migration team
8. **Third-Party Security**: Secure coordination with IBM Support
9. **Data Validation**: Verify integrity of migrated data
10. **Post-Migration Audit**: Comprehensive security review

## Appendix

### Security Checklist

```
Pre-Migration Security:
[ ] LDAP/AD integration configured
[ ] Security groups created
[ ] MFA enabled for admin accounts
[ ] Folder structure created
[ ] ACLs applied
[ ] Firewall rules configured
[ ] TLS certificates installed
[ ] Audit logging enabled
[ ] SIEM integration configured
[ ] Security monitoring dashboard configured

During Migration:
[ ] User mapping completed
[ ] Security definitions imported
[ ] Access controls validated
[ ] Encryption verified
[ ] Audit logs reviewed
[ ] Security testing completed
[ ] Incident response team on standby
[ ] Enhanced monitoring active

Post-Migration:
[ ] Security audit completed
[ ] Access reviews conducted
[ ] Documentation updated
[ ] Training completed
[ ] Compliance verification
[ ] Penetration testing scheduled
[ ] Lessons learned documented
[ ] Security improvements implemented
```

### Contact Information

**Security Team:**
- Security Manager: [Name/Contact]
- Security Engineer: [Name/Contact]
- Compliance Officer: [Name/Contact]
- Incident Response: [24/7 Contact]

**Emergency Contacts:**
- Security Hotline: [Number]
- SOC: [Number]
- IBM Support: [Number]

---

**Document Version:** 1.0  
**Classification:** Confidential  
**Last Updated:** [Date]  
**Next Review:** [Date]  
**Document Owner:** Chief Information Security Officer