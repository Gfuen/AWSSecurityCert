# AWS Security Notes

# Domain 6: Management and Security Governance

## AWS Accounts - Basics

* AWS Account is a container for identities(user) and Resources
```
-AWS Accounts require a unique email address, payment method. and account name
-Email address can be used for multiple accounts
-AWS Root user(* of this account) is created unique identity that can only access that specific AWS account
-AWS Root user has all permissions and cant be restricted
-Account payment method set by default when setting up account
-AWS Accounts can contain the impact of admin errors or exploits. Use separate accounts for sperate things(TEST, DEV, PROD) or teams or products or clients.
-By default access to an AWS Account & resources is denied except for the Account Root User
-IAM users can have 2 access keys
```

## AWS Organizations
```
-Gain discounts according to more accounts in AWS Organizations for certain services. Consolidation of reservations and volume discounts
-Consolidated Billing/ Payer Account is directed to Management account instead of directly accounts in AWS organization. Combines bills into 1 bill.
-Need to create OrganizationAccountAccessRole when creating an account in AWS organization
-By default OrganizationAccountAccessRole is created when inviting AWS account into AWS organization
-Management/Master account is the account used to create the organization and manage accounts in it
-Root account is the container at the top of AWS organization hierarchy which contains accounts and containers
-OU Organization Units can contain accounts or other nested Organization Units
-Best Architecture is log into 1 AWS account that controls IAM and then switch into specific AWS account using OrganizationAccountAccessRole
-Can use Switch Role to go into other accounts from right hand side within AWS Organization
-When switching role need Acount ID, Role Name, and Display name to use Role in another AWS member account
-Need Valid UNIQUE email address when creating AWS account in AWS Organization
```

## Service Control Policies (SCP)
```
-JSON Policy document can be attached to AWS Organization/ AWS Organization Units/ AWS Organization Member accounts. Service Control Policies are inherited down the Organizational tree.
-AWS Management account cant be restriced with Service Control Policies in AWS Organization
-SCPs are account permission boundaries. They limit what the account (including account root user) can do
-SCPs don't grant any permissions. They just define a limit.
-AWS applies default FullAWSAccess Policy to everything. Means SCPs have no effect on accounts by default in an AWS Organization
-Types of Architecture for SCPs:
    -Allow list: Keep FullAWSAccess Policy and create S3Deny Policy or any policy required to restrict (Lower Admin Overhead)
    -Deny List: Remove FullAWSAccess Policy and create AllowS3 Policy or any policy required for allowing permissions
-Effective permissions in an account are those that overlap in AWS identity Policies and applicable SCPs
```

## AWS Control Tower
```
-AWS Control Tower orchestrates other AWS services to provide this functionality
-Quick and easy setup of multi-account environment
-Organizations, IAM Identity Center, CloudFormation, Config, and more
-Landing Zone - multi-account environment
-... SSO/ID Federation, Centralised Logging & Auditing
-Guard Rails - Detect/Mandate rules/standards across all acounts
-Account Factory - Automates and Standardises new account creation
-Dashboard - single page oversight of the entire environment
-By default creates 2 Organization Units; Foundational OU and Custom OU
-By default creates 2 AWS accounts in Foundational OU; Audit Account and Log Archive Account
-By default creates Account Factory creates Provisioned AWS accounts in Custom OU
```

### AWS Control Tower - Landing Zone
```
-Well Architected multi-account environment - Home Region
-...built with AWS Organizations, AWS Config, CloudFormation
-Security/Foundational OU - Log Archive & Audit Accounts (CloudTrail & Config Logs)
-Sandbodx OU - Test/less rigid security
-Can create other OU's and Accounts
-IAM Identity Center (AWS SS0) - SSO, multiple-accounts, ID Federation
-Monitoring and Notifications - CloudWatch and SNS
-End User account provisioning via Service Catalog
```

### AWS Control Tower - Guard Rails
```
-Guardrails are rules - multi-account governance
-3 types; Mandatory, Strongly Recommended, or Elective
-Preventive - Stop you doing things (AWS ORG SCP)
    -..enforced or not enabled
    -..ie allow or deny regions or disallow bucket policy changes
-Detective - compliance checks (AWS CONFIG Rules)
    -clear, in violation, or not enabled
    -..detect CloudTrail enabled or EC2 Public IPv4
```

### AWS Control Tower - Account Factory
```
-Automated Account Provisioning
-Done by cloud admins or end users (with appropriate permissions)
-Guardrails - automatically added when new accounts created
-Account admin given to a named user (IAM Identity Center)
-Account & network standard configuration
-Accounts can be closed or repurposed
-Can be fully integrated with a businesses SDLC
```

## AWS Config
```
-Primary job; Record configuration changes over time on resources
-Auditing of changes, compliance with standards
-Does not prevent changes happening ... no protection
-Regional Service ... supports cross-region and account aggregation
-Changes can generate SNS notifications and near-realtime events via EventBridge & Lambda
-Config Rules, AWS Managed or Custom rules, evaluate resources against a defined standard (Compliant vs Non-Compliant)
-Config Rules can invoke Lambda functions or SSM to change configurations
```

## AWS Service Catalog
```
-Document or Database created by an IT Team 
-Organized collection of products
-Offered by the IT Team
-Key Product Information: Product Owner, Cost, Requirements, Support Information, Dependencies
-Defines approval of provisioning from IT and Customer side
-Manage costs and scale service delivery
-Self-service Portal for 'end users'
-..launch predefined (by admin) products
-End user permissions can be controlled
-Admins can define those products
-..and the permissions reqiured to launch them
-Build products into portfolios
```

## AWS Resource Access Manager (RAM)
```
-Share AWS resources between AWS Acounts
-Products need to support RAM
-Shared with Principals (Accounts, OU's or ORG)
-Shared resources can be accessed natively
-No charge only the service cost
-Substantial changes to traditional AWS architectures
-AWS rotate which physical facilities are used for Availability Zones. My us-east-1a might not be the same as your us-east-1a
-Coordinating resouces between accounts becomes difficult
-AZ IDs
    -use1-az1 is always use1-az1
-Owner account creates a share, provides a name
-Owner retains full ownership
-Defines the principal (AWS account, OU, Org) with whom to share
-If participant is inside an ORG with sharing enabled its accepted automatically
-For non ORG accounts, or sharing with AWS Organizations is not enabled - you need to accept an invite
-VPCs can be created which provide shared infrastructure services to other AWS accounts
-VPC Owners create and manage the VPC & Subnets and can share to participants
-Participants can provision services into the shared subnets, read and reference network objects but not modify or delete them
-VPC Owners cannot delete or modify resources created by participant VPCs
-Participants cannot view or modify resources created by other participants but resources can communicate via L3 networking
```

## AWS Trusted Advisor
```
-ACCOUNT level - no agents to install
-5 checks provided; Cost Optimization, Performance, Security, Fault Tolerance and Service Limits
-7 Core checks with basic & developer support plans
    -S3 Bucket Permissions - NOT OBJECTS
    -Security Groups - Specific Ports Unrestricted
    -IAM use
    -MFA on Root Account
    -EBS Public Snapshots
    -RDS Public Snapshots
    -50 service limit checks
-Anything beyond requires Business or Enterprise support plan
    -115 Further checks + Core Checks
    -Access via the AWS Support API
    -CloudWatch Integration - react to changes
- Severity Results; Green Okay, Need to Investigate, Need to take Action
```

## CloudFormation Physical & Logical Resources
```
-CloudFormation Template - YAML or JSON
-.. contains logical resources - the "WHAT"
-Templates are used to create Stacks
-.. 1 stack, 100 stacks, 20.. stacks in each region
-..stacks create physical resources from the logical
-if a stacks template is changed - physical resources are changed
-if a stack is deleted, normally, the physical resources are deleted
-A stack.. creates, updates, & deletes physical resources based on logical resources in the template

```

## CloudFormation Template & Pseudo Parameters
```
-Template Parameters accept input - console/CLI/API
-..when a stack is created or updated
-Can be referenced from within Logical Resources
-..influence physical resources and/or configuration
-Can be configured with Defaults, AllowedValues, Min and Max length & AllowedPatterns, NoEcho & Type
-Pseudo parameters that can be injected into the template and stack by AWS such as AWS region us-east-1 based on the environment
-Best practice is to limit explicit inputs
```

## CloudFormation Intrinsic Functions
```
-Ref & Fn::GetAtt
-Fn::Join & Fn::Split
    - Value: !Join ['', ['http://', !GetAtt Instance.DNSName]]
-Fn::GetAzs & Fn::Select
    -Availability Zone: !Select [0, !GetAZs '']
-Conditions (Fn:: IF, And, Equals, Not & OR)
-Fn::Base64 & Fn::Sub
-Fn::Cidr
-Later .. Fn::ImportValue, Fn::FindInMap, Fn::Transform
-Using !Ref on template or pseudo parameters returns their value. When used with logical resources - the physical ID is usually returned
    -!GetAtt LogicalResource.Attribute or !Ref LatestAmiId
```

## CloudFormation Mappings
```
-Templates can contain a Mappings object
-..which can contain many mappings
-..which map keys to values, allowing lookup
-Can have 1 key, or Top & Second Level
-Mappings use the !FindInMap intrinsic function
-Common use ... retrieve AMI for given region & architecture
-Improve Template Portability
```

## CloudFormation Outputs
```
-Templates can have an optional Outputs section
-Values can be declared in this section..
-..visible as outputs when using the CLI
-..visible as outputs in the console UI
-..accessible from a parent stack when using nesting
-..Can be exported, allowing cross-stack references
```

## CloudFormation Conditions
```
-Created in the optional 'Conditions' section of a template
-Conditions are evaluated to TRUE or FALSE
-..processed BEFORE resources are created
-Use the other intrinsic functions AND, EQUALS, IF, NOT, OR
-..associated with logical resources to control if they are created or not
```

## CloudFormation DependsOn
```
-CloudFormation tries to be efficient
-..does things in parallel (create, update, & delete)
-..tries to determine a dependency order (VPC => SUBNET => EC2)
-..references or functions create these
-DependsOn lets you explicitly define these
-..If Resources B and C depend on A
-..both wait for A to complete before starting
-An Elastic IP requires an IGW attached to a VPC in order to work .. but there is no dependency in the template-implicit or explicit
-DependsOn creates an explicit dependency. WPEIP will be created AFTER InternetGatewayAttachment is complete & deleted BEFORE its deleted
```

## CloudFormation Wait Conditions & cfn-signal
```
Provisioning Process
-Logical Resources in the template
-...Used to create a Stack
-creates physical resources in AWS
-..Logical Resource CREATE_COMPLETE = ALL OK?
TEMPLATE => STACK => INSTANCE

CloudFormation Signal
-Configure CloudFormation to hold (more on this soon..)
-Wait for 'X' number of success signals
-Wait for Timeout H:M:S for those signals (12 hour max)
-if success signals recieved .. CREATE-COMPLETE
-if failure signal receieved .. creation fails
-if timeout is reached .. creation fails
-.. CreationPolicy or WaitCondition

CloudFormation CreationPolicy
-CreationPolicy applies signal requirement (3) and timeout (15M)
-Tied to specific resource

CloudFormation WaitCondition
-WaitCondition can depend on other resources. Other resources can depend on the WaitCondition
-Generates a PreSigned URL for resource signals - JSON passed back in signal response
```

## CloudFormation Nested Stack's
```
-Resources in a single stack share a lifecycle
-Stack resource limits (500) which can complicate large deployments
-Can't easily reuse resources e.g. A VPC
-Can't easily reference other stacks. Isolated by default.

CFN Nested Stacks
-Root stack - stack that gets created first. You create it through console or API
-Parent stack - parent of any stacks which it immediately creates. refer to that has its own nested stack.
-While templates can be reused in other stacks
-Use when the stacks from part of one solution - lifecycle linked
-Overcome the 500 Resource Limit of one stack
-Modular templates ... code reuse
-Make the installation process easier..
-..nested stacks created by the root stack
-Use only when everything is lifecycle linked
```

## CloudFormation Cross-Stack References
```
-CFN stacks are designed to be isolated and self-contained
-Outputs are normally not visible from other stacks
-Nested stacks can reference them ...
-Outputs can be exported ... making them visible from other stacks
-Exports must have a unique name in the region
-Fn:ImportValue can be used instead of Ref
-Export is used to export the output of a stack ... to a unique name in that region of that account
-Template is not a stack, template is used to create one or more stacks
-MEANT for Service-Orientated & different lifecycles & STACK Reuse
```

## CloudFormation Deletion Policy
```
-if you delete a logical resource from a template
-.. by default, the physical resource is deleted
-This can cause data loss..
-With deletion policy, you can define on each resource ..
-..Delete (Default), Retain or (if supported) Snapshot
-EBS Volume, ElastiCache, Neptune, RDS, Redshift
-Snapshots continue on past Stack lifetime - you have to clean up ($$)
-ONLY APPLIES TO DELETE .. NOT REPLACE
-SNAPSHOT feature is not available on all AWS Resource Types
```

## CloudFormation Stack Roles
```
-When you create a stack - CFN creates physical resources
-CFN uses the permissions of the logged in identity
-Which means you need permissions for AWS
-CFN can assume a role to gain the permissions
-This lets you implement role separation
-The identity creating the stack..doesnt need resource permissions - only PassRole
-IDEAL for identities that need to provision in CloudFormation that they wouldnt otherwise need to 
```

## CloudFormation Change Sets
```
-Template => Stack => Physical Resources (CREATE)
-Stack(Delete) => (Delete) Physical Resources
-v2 Template => Existing Stack => Resources Change
-No Interruption, Some Interruption, Replacement
-Change Sets let you preview changes (A Change Set)
-..multiple different versions (lots of change sets)
-Chosen changes can be applied by executing the change set
```

## CloudFormation (CFN) Custom Resources
```
-Custom Resources let CFN i8ntegrate with anything it doesnt yet, or doesnt natively support
-Passes data to something, gets data back from something
```

# Domain 4: Identity and Access Management

## IAM Identity Policies
```
-Explicit DENY => Explicit ALLOW => Default DENY
-DENY ALLOW DENY
-Inline Policies
    -Special Allows for Denys
-Managed Policies 
    -Reusable
    -Customer Managed or AWS Managed
    -Should be used for set operational rights
```

## IAM Users and ARNs
```
-IAM Users are an identity used for anything requiring long-term AWS access e.g. Humans, Applications, or Service Accounts
-99% of the time IAM type to use is IAM user
-Principal => IAM Login => Authenticated Identity
-Amazon Resource Name (ARN) - Uniquely identify resources within any AWS accounts
-5,000 IAM Users per account (possible exam question)
-IAM User can be a member of 10 groups
-This has systems design impacts..
-Internet-scale applications
-Large orgs & org merges
-IAM Roles & Identity Federation fix this
```


## IAM Groups
```
-IAM groups are containers for Users
-IAM groups have no credentials and cant login into them
-IAM groups are made easier IAM user management (teams, projects, departments)
-IAM Users can be a member of more than 1 IAM group
-IAM groups can have Inline and Managed policies attached
-No LIMIT on IAM users in a IAM group
-No Nesting in IAM groups
-Limit of 300 groups but can be increased with a support ticket
-IAM Groups are not a true identity. They cant be referenced as a Principal in a policy
-Resource policies cant reference IAM groups
```

## IAM Roles
```
-IAM Roles are assumed .. you become that role
-Ideal scenario is if multiple or unknown principals need to assume an AWS identity
-2 types of policies; Trust Policy and Permissions Policy
-Trust Policy - which identities can assume the role
-Permissions Policy - what the role can do
-IAM roles can be referenced in Resource Policies
-Secure Token Service sts:AssumeRole give assume role permission
```

## IAM When to use IAM Roles
```
-Lambda Execution Role ideal for IAM role
-Break Glass - Emergency ideal for IAM role
-IAM role ideal when you want to reuse existing identities in AWS
-External accounts be used in AWS directly
-OnPrem Active Directory users assumign role into AWS account
-Mobile Application (ride sharing app) => Web Identities (Facebook, Google, etc..) => Role 
    -Web Identity Federation
    -No AWS Credentials on the app
    -Uses Existing customer logins
    -Scales to 100,000,000's of accounts
-Role to switch between accounts in AWS Organization
```

## Service Linked Roles
```
-IAM role linked to a specific AWS service
-Predefined by a service
-..providing permissions that a service needs to interact with other AWS services on your behalf
-Service might create/delete the role..
-..or allow you to during the setup or within IAM
-You cant delete the role until its no longer required
-Role Separation using PassRole permissions
```

## Security Token Service (STS)
```
-Generates temporary credentials (sts:AssumeRole*)
-Similar to access keys
-..they expire and dont belong to the identity
-Limited Access
-Used to access AWS resources
-Requested by an Identity (AWS or External)
-TRUST policy - who can assume the role
-AccessKeyID - Unique ID of the credentials
-Exipiration - Date and Time of credential expiration
-SecretAccessKey - Used to sign requests
-SessionToken - Unique token which must be passed with all requests
```

## EC2 Instance Roles & Profile
```
-EC2 Instance roles and Instance Profiles are how applications running on an EC2 instance can be given permissions 
to access AWS resources on your behalf
-InstanceProfile is the wrapper for EC2 role that is attached to instance
-Temp Credentials delivered via instance meta-data
-iam/security-credentials/role-name
-Automatically rotated - Always valid
-Should always be used rather than adding access keys into instance
-CLI tools will use ROLE credentials automatically
```

## IAM Policy Variables
```
-IAM Policy Variables are placeholders when you dont know the exact value of a resource or condition key when writing the policy
-Example: "arn:aws:iam:account-id:user/${aws:username}" - Replace username with Bob or Julie username
```

## IAM Policy Interpretation 
```
-NotAction - matches anything other than actions listed
-StringNotEquals - matches any string not listed
-"aws:RequestedRegion" - resolve to region of resource trying to interact with
-"Condition": {"StringLike": {"s3:prefix": ["${aws:PrincipalTag/team}/*"]}} - Matches Condition for Team name with "team" principal tag value
```

## IAM Policy Evaluation Logic
```
-Organization SCPs
-Resource Policies
-IAM Identity Boundaries
-Session Policies
-Identity Policies
-..what about different accounts?
-Mult-Account access - allow = allow (Access must be allowed from both accounts for 1 identity to the resource)
```

## IAM Permission Boundaries and Delegation
```
-Only IDENTITY permissions are impacted by boundaries - any resource policies are applied in full
-Boundaries can be applied to IAM Users or IAM Roles
-Permissions boundaries don't GRANT any access on their own. They define maximum permissions an identity can receive
-Permissions that are allocated to identities OUTSIDE of a permissions boundary have no effect
-Can attach both an identity policy and permission boundary at the end of aws console of IAM Identity creation
```

## External ID .. Confused Deputy
```
IAM Role with External ID
-DevOps Inc is a deputy .. acting on Catagrams behalf Catagram TRUSTS them
-A confused deputy .. doggogram fools DevOpsInc into applying its instructions to Catagram instead of doggogram
-Role ARNs arent sensitive - assume they are known
-DevOps Inc creates a UNIQUE (across all customers) External ID
-Catagram role is configured with External ID - only allows assumption if provided
-When doggogram falsely provides catagram role they are given a different external ID
-Policy ex:
    -"Condition": {"StringEquals": {"sts:ExternalId": "1337"}}
-External IDs are an effective way of solving confused deputy attacks
```

## Directory Service - Microsoft AD
```
-Built using Microsoft Active Directory 2012 R2
-Managed using Standard Active Directory tools
-Supports Group Policy and single sign-n (SSO)
-Supports Schema extension - MS AD Aware Apps
-Sharepoint, SQL, Distributed File System (DFS)
-Two sizes - Standard (30,000) & Enterprise (500,000)
-Used for AD Authentication/Authorization of products and services within AWS
-Highly-Available by default (2AZ+)
-Includes monitoring, recovery, replication, snapshots and maintenance - configurable;managed by AWS
-Supports one-way and two-way external and forest trusts with on-premises active directory
-Directory in AWS - can operate through a network link failure to any connected on-premises systems
-Supports RADIUS-based MFA
-Best choice for more than 5,000 users and if you need trust relationships between AWS and your on-premises directories
-Microsoft AD is the only directory service type which supports AD Native schema extensions. These are required by some AD applications.
-Identity Federation with other cloud platforms & applications such as Azure AD or Microsoft 365
```

## Identity Federation
```
-Identity Federation is a process where external identities are used instead of services maintaining their own identities. 
-Better experience for customers and less admin overhead for services
-Identity Providers can use enterprise standards such as SAML or can be Web Identity Providers such as Google, Facebook, or Twitter
-The Service Provider trusts the Identity Provider generated tokens can be used instead of application logins
-When to use IDF
    -Reduce the number of logins for customers to manage
    -Reduce barrier to entry
    -More secure - logins managed externally
    -Less admin overhead
    -Scalable - beyond any AWS limits (5,000 IAM)
    -Operate with existing single source of truth enterprise ID stores
```

## Amazon Cognito
```
-Authenticatio, Authorization, and user management for web/mobile apps
-USER POOLS - Sign-in and get a JSON WEB Token (JWT)
-User directory management and profiles, sign-up & sign-in (customisable web UI), MFA and other security features
-By using user pools & social sign-in your application can standardise on a single "token" (User Pool tokens)
-IDENTITY POOLS - Allow you to offer access to Temporary AWS Credentials
-Unauthenticated Identities - Guest Users
-Federated Identities - SWAP - Google, Facebook, Twitter, SAML 2.0 & User Pool for short term AWS Credentials to access AWS Resources
-Cognito assumes an IAM role defined in Identity Pool and returns temporary AWS credentials
```

## SAML Federation
```
-SAML 2.0 (Security Assertion Markup Language)
-Open standard used by many IDP's (e.g. MS ADFS)
-Indirectly use on-premises IDs with AWS (Console & CLI)
-Enterprise Identity Provider ... SAML 2.0 Compatible
-Existing identity maangement team
-Single source of truth ... more than 5,000 users
-Uses IAM Roles & AWS Temporary Credentials (12 hour validity)
-Application initiated process
-SAML assertion is like a token issued by Identity Provider to be used in combination with final request with STS:AssumeRole
-Bidirectional Trust between Identity Provider (Saml compliant) and SAML/SSO endpoint
```

## AWS Single Sign-On (SSO)
```
-Manage SSO Access - AWS Accounts and External Applications
-Flexible Identity Source
-AWS SSO - Built-In Identity Store
-AWS Managed Microsoft AD
-On-premises Microsoft AD (Two way trust or AD Connector)
-External Identity Provider - SAML 2.0
-Preferred by AWS vs traditional 'workforce' identity federation
-AWS SSO Integrates with a range of workplace Identity Sources ranging from built-in AWS SSO identities through to self-managed on-premises Active directory
-SSO is the AWS Preferred workplace identity solution - it handles SSO & permissions for AWS accounts and external applications such as slack and dropbox
```

## AWS PreSigned URLs
```
-PreSigned URLs can be used for GET or PUT operations
-PreSigned URLs are used in the context of the AWS identity that generated them when someone else uses them
-PreSigned URLs are time limited
-PreSigned URLs are encoded with authentication information inside
-You can create a URL for an object you have no access to
-When using the URL, the permissions match the identity which generated it..
-Access denied could mean the generating ID never had access .. or doesnt now
-Dont generate with a role .. URL stops working when temporary credentials expire ..
```

## S3 Security
```
-S3 is private by default
-S3 Bucket Policy
    -A form of resource policy
    -Like identity policies, but attached to a bucket
    -Resource perspective permissions
    -ALLOW/DENY same or DIFFERENT accounts
    -ALLOW/DENY Anonymous principals
-Access Control List (ACLs)
    -ACLs on objects and bucket
    -A subresource
    -Legacy
    -Inflexible & Simple permissions
    -ACL Permissions; READ, WRITE, READ_ACP, WRITE_ACP, FULL_CONTROL
-When to choose which policies
-Identity: Controlling different resources
-Identity: You have a preference for IAM
-Identity: Same account
-Bucket: Just controlling S3
-Bucket: Anonymous or Cross-Account
-ACLSs: NEVER - unless you must
```

## S3 Object Lock
```
-Object Lock enabled on 'new' buckets* (Support req for existing)
-Write-Once-Read-Many (WORM) - No delete, No overwrite
-Requires versioning - individual versions are locked
1 - Retention Period
2 - Legal Hold
-Both, One or the other, or none
-A Bucket can have default Object Lock Settings
-S3 Object Lock - Retention 
    -Specify DAYS & YEARS - A Retention Period
    -COMPLIANCE mode - Cant be adjusted, deleted, overwritten
    -..even by the account root user
    -..until retention expires
    -GOVERANCE mode - special permissions can be granted allowing lock settings to be adjusted
    -s3:BypassGoveranceRetention..
    -..x-amz-bypass-goverance-retention:true (console default)
-S3 Object Lock - Legal Hold
    -Set on an object version - ON or OFF
    -..no retention
    -NO Deletes or Changes until removed
    -s3:PutObjectLegalHold is required to add or remove
    -Prevent accidental deletion of critical object versions
```

## S3 Versioning & MFA
```
-S3 bucket can not be disabled once enabled (Exam question)
-Versioning lets you store multiple versions of objects within a bucket.
-Operations which would modify objects generate a new version
-Objects arent deleted - Object deletion markers are put in place to hide objects
-Object versioning cannot be switched off - only suspended
-Object versioning - Space is consumed by ALL versions
-You are billed for ALL versions
-Only way to 0 costs - is to delete the bucket
-MFA Delete - Enabled by versioning configuration
-MFA Delete - MFA is required to change bucket versioning state
-MFA Delete - MFA is required to delete versions
-Serial number (MFA) + Code passed with API Calls
```

## EC2 Instance Metadata
```
-EC2 Service provides data to Instances
-Accessible inside ALL instances
-http://169.254.169.254
-http://169.254.169.254/latest/meta-data
-Environment
-Networking
-Authentication
-User-Data
-NOT AUTHENTICATED or ENCRYPTED
-Ex: curl http://169.254.169.254/latest/meta-data/public-hostname
    -LUKE
```

# Domain 1: Threat Detection and Incident Response

## AWS Abuse Notice, UAP, & Pen Testing
```
-AWS Acceptable Use Policy AUP - What you CAN and CANT do with a service (AWS)
-No Illegal or Fraudulent Activity
-Dont violate the rights of tohers
-Dont use to threaten, incite, promote or actively encourage violence, terrorism, or serious harm
-No content or activity that promotes child sexual exploitation or abuse
-No usage which violates security, integrity or availability of any user, network, computer, or
communication system, application, or network or computing device
-No usage for distribution, publishing, sending or facilitating sending of mass email, promotions, advertising or other solicitations (spam)
-Notification via email or AWS Health Dashboard
-..AWS Trust & Safety Team (They DONT provide support)
-Something bad is happening
-These are the details
-Fix it, confirm its fixed
-..what you have done to prevent it happening again?
-DO THIS .. or your service/account could be suspended
-Form - Report AWS Abuse (anyone can use this)
-Enter details of abuse, date/time, contact info
-AWS can identify abuse internally...
-Limits on port scanning, penetration testing & DDOS testing/simulations
-..doing this outside the rules = abuse
-..8 services dont require pre-approval
```

## Amazon GuardDuty (Threat Detection)
```
-Continous security monitoring service
-Analyses supported Data Sources
-..plus AI/ML, plus threat intelligence feeds
-Identifies unexpected and unauthorized activity
-..notify or event-driven protection/remediation
-Supports multiple accounts (MASTER and MEMBER)
```

## AWS Security Hub
```
-Single location for security management and remediation
-Regional service .. full compliance enable in all regions
-Console UI and Security Hub API
-NOT RETROACTIVE - wroks from enable point onwards...
-Readiness scores/findings against security standards
-CIS AWS Foundations, PCI DSS, AWS Foundational Security Best Practices
-Automated remediation using EventBridge
-Administrator & Member Accounts (independent of ORG mgmt/member)
-Aggregation Region & Linked Regions
-Aggregates data from Partner & AWS services (Config, Macie, Inspector, GuardDuty, IAM, Firewall Manager)
-AWS Security Findings Format (ASFF)
```

## Amazon Detective
```
-Security Investigation service
-Integrates with other AWS services .. time-based events
-e.g. login attempts, API calls (CloudTrail), network traffic (VPC Flow Logs) and guard duty findings
-Interactive exploration and tracing through data
-Identify abnormal events, security issues, explore related resources
-Tailored Visualization - is this normal or abnormal for X or Y?
-Administration and Member account arhitecture
-Administrator account invites member accounts..
-Member accounts contribute data..
-ORG Management delegates an account as Detective Administrator account for the ORG
-..can add any ORG acccount .. option for automatic add
-..can also invite accounts outside of the ORG
```

## Revoking IAM Role Temporary Security Credentials
```
-Permissions policy updated with a 'AWSRevokeOlderSessions' inline DENY for any sessions older than NOW
-DENY assumptions before NOW
-Bad Actor cant directly perform an sts:AssumeRole so remains DENIED
-Cant manually invalidate temporary credentials
```

# Domain 3: Infrastructure Security

## Public and Private AWS Services
```
-Public or Private Service - NETWORKING PERSPECTIVE
-VPCs are isolated unless configured otherwise
-3 Network zones
    -"Public Internet" Zone
    -"AWS Public" Zone
    -"AWS Private" Zone
-ON-PREMSIS can access VPCs only if configured via VPN or Direct Connect
-Private Services (EC2) can given a public IP - 1:1 translated by the IGW
```

## Custom VPCs
```
-Regional Service - All AZs in the region
-Isolated network
-Nothing IN or OUT without explicit configuration
-Flexible configuration - simple or multi-tier
-Hybrid Networking - other cloud & on-premises
-Default or Dedicated Tenacy!
-IPv4 Private CIDR Blocks & Public IPs
-1 Primary Private IPv4 CIDR Block
-..Min /28 (16 IP) Max /16 (65,536 IP)
-Optional secondary IPv4 Blocks
-Optional single assigned IPv6 /56 CIDR Block

-DNS in a VPC
    -Provided by R53
    -VPC 'Base IP +2' Address
    -enableDnsHostnames - gives instances DNS Names
    -enableDnsSupport - enables DNS resolution in VPC
```

## VPC Subnets
```
-AZ Resilient
-A subnetwork of a VPC - within a particular AZ
-1 Subnet => 1 AZ, 1 AZ => 0+ Subnets
-IPv4 CIDR is a subset of the VPC CIDR
-Cannot overlap with other subnets
-Optional IPv6 CIDR (/64 subset of the /56 VPC - space for 256)
-Subnets can communicate by default with other subnets in the VPC
-Subnet IP Addressing
    -Reserved IP addresses (5 in total)
    -10.16.16.0/20 (10.16.16.0 => 10.16.31.255)
    -Network Address (10.16.16.0)
    -'Network +1' (10.16.16.1) - VPC Router
    -'Network +2' (10.16.16.2) - Reserved (DNS*)
    -'Network +3' (10.16.16.3) - Reserved Future Use
    -Broadcast Address 10.16.31.255 (Last IP in subnet)
```

## DHCP 
```
-DHCP - Dynamic Host Configuration Protocol
-..auto configuration for network resources
-..starts with L2 broadcast to get info from DHCP server
-IP address, Subnet Mask & Default Gateway
-DNS Servers & Domain Name
-NTP Servers, NetBios Name Servers & Node type
-AmazonProvidedDNS & Custom DNS Domain
-DHCP Option Sets cannot be edited - once created they're immutable
-DHCP Option Sets can be associated with 0 or more VPCs
-Each VPC can have MAX "1" Option Set attached
-Associating a new option set is immediate - but changes require a DHCP Renew which takes time
-AWS supply public and private DNS name. To use custom domains you also need to use custom DNS servers.
```

## VPC Router Deep Dive
```
-Virtual Router within a VPC
-Highly Available - across all AZs in that region
-Scalable - no performance management required
-Routes traffic between subnets
--...FROM EXTERNAL networks INTO the VPC
--...FROM the VPC into EXTERNAL networks
-Interface in every subnet ... "subnet+1" addresss (default GW via DHCP Option Set)
-Controlled using route tables
-Every subnet has a VPC Router Interface
-Routing Tables control routing decisions
-VPC router directs on-premises traffic at gateway
-Every VPC is created with a Main route table (RT)
-..default for every subnet in the VPC
-Custom route tables can be created and associated with subnets in the VPC - removing the Main RT
-Subnets are associated with ONE RT (Main or Custom)
-Route tables contain routes - most specific routes first
-RT's can be associated with gateways (more on this later)
-Every subnet needs at least 1 RT
```

## Stateful  vs Stateless Firewalls
```
-TCP is bi-directional
-Stateless Firewalls remember the response ephemeral ports .. its not the well known APP port
-Stateful firewall is intelligent enough to identify the REQUEST and RESPONSE components of a connection as being related
    -ALLOWING the REQUEST, means the RESPONSE is automatically allowed
    -Stateful means lower admin overhead
```

## Network Access Control Lists (NACL)
```
-Acts as traditional firewall for controlling traffic around VPCs
-Associated with subnets
-Connections made WIHIN a subnet are not affected by NACLs
-NACLs contain rules grouped into INBOUND or OUTBOUND
-NACLs are stateless. Dont know if request is INBOUND or OUTBOUND
-Rules are processed in order. Lowest rule number first.
-These rule pairs (app port & ephemeral ports) are need on each NACL for each communication type which occurs 1. WITHIN a VPC 2. TO a VPC 3. FROM a VPC
-VPC is created with a Default NACL
-Custom NACLs can be created for a specific VPC and are initially associated with NO subnets
-Custom NACls come with 1 DENY INBOUND and 1 OUTBOUND DENY
-Only impacts data cross subnet boundary
-NACLs cannot be assigned to AWS resources - only subnets
-Use together with Security Groups to add explicit DENY (Bad IPs/Nets)
-A NACL can be associated with MANY subnts
```

## Security Groups
```
-Stateful - detect response traffic automatically
-Allowed (IN or OUT) request = allowed response
-NO EXPLICIT DENY .. only ALLOW or implicit DENY
-..cant block specific bad actors
-Supports IP/CIDR .. and logical resources
-.. including other security groups and ITSELF
-Attached to ENIs not instances (even if the UI shows it this way)
-An SG reference aplies to anything which has the SG attached
-Within an SG .. an "SG source" is the same as "anything with the SG attached"
-Using a "self reference" means "anything with this SG attached"
-This scales with ADDS and REMOVES from the SG
-IP Changes are automatically handled
```

## Internet Gateway IPv4 and IPv6
```
-"Can be" created and attached to AWS VPCs
-.. 1:1 Relationship .. 1 IGW per VPC, 1 VPC per IGW
-Architecturally at the VPC & AWS Public Zone Border
-.. used to access AWS Public Services and Public Internet
-Highly available by default & scales by default
-Works with IPv4 and IPv6 - IN and OUT
-.. (IPv4) 1:1 Static NAT - Private IPv4 <=> Public IPv4
-.. (IPv6) Routers IPv6 packets IN and OUT of a VPC
-Flows from VPC to Public space services never leave the AWS Network
-IGW - Public Subnet
    -1. Create IGW
    -2. Attach IGW to VPC
    -3. Create Custom Route Table
    -4. Associate Route Table to Subnet
    -5. Default Routes => IGW
    -6. Subnet allocate Public IPv4
-IGW - Static NAT
```

## Egress-Only Internet Gateway
```
-With IPv4 addresses are private and public
-NAT allows private IPs to access public networks
-..without allowing externally initiated connections (IN)
-With IPv6 all IPs are public
-Internet Gateway (IPv6) allows all IPs IN and OUT
-Egress-Only is outbound-only for IPv6
-Egress-Only Gateway is HA by default across all AZs in the region - scales as required
```

## Bastion Hosts
```
-A server which runs at a network edge
-..hardened to withstand attack from public networks
-Acts as a gatekeeper between two zones (public => Private)
-Jumpbox is a generic term .. server between zones
-A Bastion is a subset of Jumpbox .. hardened
-Ingress control - logging, authentication & security
```

## Port Forwarding
```
-Forwards an external port to an internal port
-The External exposed port can be different than the internal port
```

## Nat Instance
```
-NAT Software running on an EC2 instance
-.. End of Life (NAT AMI is Amazon Linux, 2018.03 - Not Recommended)
-.. non specialised
-Speed based on size and type of EC2 instance
-Disable source/destination check!
-Security Groups on the NAT instance & NACL on the subnet
-Can also be BASTION Host & Can be configured for port forwarding
-..self-managed
-Example: NAT Instance is an EC2 Instance .. ONE AZ // NO RESILIENCE
-Scripts can be used to implement HA
-..RT changes based on failures
-Size/Performance fixed - can manage costs
-Combine NAT/BASTION/Port Forwards
-SG and NACL
-Flow logs for monitoring
-Works across DX, VPN, Peering Connections (Not Ideal)
```

## NAT Gateway
```
-Runs from a public subnet & uses elastic IP (Static IP)
-AZ Resilient Service (HA in that AZ) - hardware failure
-For region resilience - NATGW in each AZ ...
-..RT in for each AZ with that NATGW as target
-..Separate failure domains (service consumption IN AN AZ)
-Managed, scales to 45 Gbps, $ Duration & Data Volume
-5 Gbps by default .. scales to 45 Gbps
-Want more? ... split resources into multiple subnets
-... and use multiple NATGW's
-Elastic IP cannot be changed on a NATGW
-.. if you need to change it ... Provision a New NATGW
-NATGW cannot be used across VPC Peers, Site-to-Site VPN connections, or AWS Direct Connect
-55,000 simultaneous connections to each unique destination
-.. ~900/s to a single destination (55,000/minute)
-ErrorPortAllocation (.. uses 1024-65535)
-For S3/DynamoDB - use gateway endpoint (0 cost)
-NACL on the subnets ... SG on the private Instances
-... NO Security Groups ON NAT Gateway
```

## IPVSEC VPN Fundamentals
```
-IPSEC is a group of protocols
-It sets up secure tunnels across insecure networks
-.. between two peers (local and remote)
-Provides Authentication ..
-.. and encryption
- Data inside tunnels is encrypted - secure connection over an insecure network
-Remember - Symmetric Encryption is fast
-... but it's a challenge to exchange keys securely
-Asymmetric Encryption is slow..
-.. but you can easily exchange public keys
-IPSEC has 2 main phases:
    -IKE Phase 1 (Slow & Heavy)
        -Authenticate - PreShared key (password) / Certificate
        -Using Asymmetric encryption to agree on, and create a shared symmetric key ..
        -IKE SA Created (phase 1 tunnel)
    -IKE Phase 2 (Fast & Agile)
        -Uses the keys agreed in phase 1
        -Agree encryption method, and keys used for bulk data transfer
        -Create IPSEC SA .. Phase 2 tunnel (architecturally running over Phase 1)
-Policy-based VPNs
-.. rule sets match traffic => a pair of SAs
-.. different rules/security settings
-Route-based VPNs
-.. target matching (prefix)
-.. matches a single pair of SAs
```

## Virtual Private Gateway
```
-Gateway Object between AWS VPCs and non AWS Networks
-.. e.g. on-premises & other clouds
-Attached to max 1 VPC at a time ...
-... can be detached and reattached
-... it maintains its connections (migration) from previous and current VPC connection
-DX (Private VIF), DX Gateway & Site-2-Site VPN (public IPs)
-Private ASN - Defaults to 64512
-HA be default - multiple AZs (VPNs multiple endpoints)
-Any VPC Route tables which enable route propagation will have any "VGW learned" routes added automatically
-Dynamic VPNs can act as backups for Direct Connect. DX learned Dynamic routes are preferred vs Dynamic VPN
-Dynamic Site-2-Site VPNs created between multiple customer sites and a shared Virtual Private Gateway
-VPC can route to all business sites. Each business site can route to the VPC and other business sites on the same VGW
-Each site advertises a unique CIDR range to the VGW
-Advertisements are sent to other CGWs using the same VGW
```

## AWS Site-to-Site VPN
```
-A logical connection between a VPC and on-premises network encrypted using IPSec, running over the public internet*
-Full HA - if you design and implement it correctly
-Quick to provision ... less than an hour
-Virtual Private Gateway (VGW)
-Customer Gateway (CGW)
-VPN Connection between the VGW and CGW
-Static VPN 
    -Routes for remote site added to route tables as static routes
    -Networks for remote side statically configured on the VPN connection
    -No load balancing and multi connection failover
-Dynamic VPN
    -Multiple VPN Connections provide HA and traffic distribution
    -Routes added automatically (Route propagation) to RTs or added statically as static routes
-VPN Considerations
    -Speed Limitations ~ 1.25Gbps
    -Latency Considerations - inconsistent, public internet
    -Cost - AWS hourly cost, GB out cost, data cap (on premises)
    -Speed of setup - hours ... all software configuration
    -Can be used as a backup for Direct Connect (DX)

```

## Client VPN
```
-Managed implementation of "OpenVPN"
-.. any client with OpenVN software is supported
-Connect to a Client VPN endpoint
-... associated to 1 VPC
-.. 1+ Target Networks (high availability)
-.. billed based on network associations
-By default all traffic is routed via ClientVPN
-Split Tunnel means the ClientVPN routes are "Added" to existing ones
    -This allows efficient access to local networks, or the public internet - avoiding the VPN tunnel
-Split tunnel is NOT the default.. it must be enabled else all data goes via tunnel
```

## VPC Endpoints (Gateway)
```
-Provide private access to S3 and DynamoDB
-Prefix List added to route table => Gateway Endpoint
-Highly Available (HA) across all AZs in a region by default
-Endpoint policy is used to control what it can access
-Regional ... cant access cross-region services
-Prevent Leaky Buckets - S3 Buckets can be set to private only by allowing access ONLY from a gateway endpoint
-Public addressing isnt required for S3 DynamoDB access
-Gateway Endpoints are not accessible outside the VPC
-Endpoint Policy controls what the Gateway Endpoint can be used for
```

## VPC Endpoints (Interface)
```
-Provide private access to AWS Public Services
-historically ... anything NOT S3 and DDB ... but S3 is now supported
-Added to specific subnets - an ENI - not HA
-For HA .. add one endpoint, to one subnet, per AZ used in the VPC
-Network access controlled via Security Groups
-Endpoint Policies - restrict what can be done with the endpoint
-TCP and IPv4 ONLY (Exam Question Possible)
-Uses PrivateLink
-Endpoint provides a NEW service endpoint DNS
-e.g. vpce-123-xyz.sns.us-east-1.vpce.amazonaws.com
-Endpoint Regional DNS
-Endpoint Zonal DNS
-Applications can optionally use these, or ...
-PrivateDNS overrides the default DNS for services
-Private DNS associates a private R53 hosted zone to the VPC changing the default service DNS to resolve to the interface endpoint ip
```

## VPC Endpoint Policies
```
-vpce allow access to an entire service in a region
-VPC Endpoint Policy just controls restrictions for identities
-Some services support endpoint policies
-Limits access via that endpoint only
-Contains a principal
-Can contain conditions
-Commonly used to limit what private VPCs can access
-And endpoint policy limits what resources or actions the endpoint will allow
-An explicit deny contained in a bucket policy applies if the access is NOT via a specific vpc endpoint
```

## Advanced VPC DNS & DNS Endpoints
```
-DNS via the VPC .2 Address 10.16.0.2 (A4L)
-.2 is reserved in every subnet
-Now called the Route53 Resolver
-Provides R53 Public & Associated Private Zones
-Only accessible from within a VPC
-Hybrid network integration is problematic - IN and OUT
-Queries for records which arent in R53 public or private hosted zones are forwarded out to public DNS
-AWS instances use the R53 Resolver. Historically this had no ability to forward queries onto on-premises resolvers.
-A4L On-premises clients and resolver have no connectivity to the R53 resolver in AWS. They cant resolve any AWS private zones.
-DNS boundary between the two environments imposed by the lack of communication between DNS infrastructure
-VPC configured to provide the DNS Forwarder via DHCP. Requests arrive and are forwarded to the R53 resolver by default - or selectively to on-premises as required
-On-premises resolver can forward on any queries for non local zones to the DNS Forwarder in AWS - which communciates with the R53 Resolver
-VPC interfaces (ENIs) - Accessible over VPN or DX
-Inbound and Outbound
-Inbound = On-premises can forward to the R53 Resolver
-Outbound = Conditional Forwarders, R53 to On-premises
-Rules control what requires are forwarded
-corp.animals4life.org => On-premises DNS Nameservers
-On-premises DNS is configured to forward queries to the R53 Inbound interfaces (ENI's in VPC Subnets)
    -Connectivity over VPN or Direct Connect
-Outbound endpoints forward queries based on defined rules to other DNS servers
```

## VPC Peering
```
-Direct encrypted network link between 2 VPCs
-Works same/cross-region and same/cross-account
-(optional) Public Hostnames resolve to private IPs
-Same region SG's can reference peer SGs
-VPC Peering does NOT support transitive peering
-Routing Configuration is needed, SGs & NACLs can filter
-Route tables at both sides of the peering connection are needed, directing traffic flow for the remote CIDR at the peer gateway object
-VPC Peering connections cannot be created where there is overlap in the VPC CIDRs - ideally NEVER use the same address ranges in multiple VPCs
-Communication is encrypted and transits over the AWS global network when using cross-region peering connections
```

## EBS Encryption Architecture
```
-Ciphertext stored at rest
-Accounts can be set to encrypt by default - default KMS Key
-Otherwise choose a KMS Key to use
-Each volume uses 1 unique DEK
-Snapshots & future volumes use the same DEK
-Cant change a volume to NOT be encrypted
-OS isnt aware of the encryption .. no performance loss
```

## EBS Volume Secure Wipes
```
-Volumes are allocated in a sanitised state - raw unformatted block storage
-When deleted, data remains. With encrypted volumes data is encrypted at resut, unaccessible outside original account.
-Disks sanitised by EBS before reuse
```

## S3 Access Points
```
-Simplify managing access to S3 Buckets/Objects
-Rather than 1 bucket w/ 1 Bucket Policy ...
-... create many access points
-... each with different policies
-... each with diffferent network access controls
-Each access point has its own endpoint address
-Created via Console or aws s3control create-access-point --name secretcats --acount-id 123456789012 --bucket catpics
-Resource (Bucket) Policy applied to the Bucket Large and difficult to manage
-Each access point has a unique DNS address for network access
-Access point policies control permissions for access via the Access point & is functionally equivalent to a bucket policy.
-Access Point policy can restrict identities to certain prefix(s), tags or actions based on need
-Access Points can be configured for access via a VPC - requires a VPC endpoint. Access via this route can be enforced by endpoint policies
```

## CloudFront Architecture
```
-Origin - the source location of your content
-S3 Origin or Custom Origin
-Distribution - the 'configuration' unit of CloudFront
-Edge Location - local cache of your data
-Regional Edge Cache - Larger version of an edge location. Provides another layer of caching.
-Upload direct to origins. NO CACHING
-Origins are used by behaviors as content sources
-A distribution can have many behaviors which are configured with a path pattern. If requests match that pattern - that behavior is used .. otherwise the default
```

## AWS Certificate Manager (ACM)
```
-HTTP - Simple and Insecure
-HTTPS - SSL/TLS Layer of Encryption added to HTTP
-Data is encrypted in-transit
-Certificates prove identity
-Chain of trust - Signed by a trusted authority
-ACM lets you run a public or private Certificate Authority (CA)
-Private CA - Applications need to trust your private CA
-Public CA - Browsers trust a list of providers, which can trust other providers
-ACM can generate or import Certificates
-If generated ... it can automatically renew
-If imported .. you are responsible for renewal
-Certificates can be deployed out to supported services
-Supported AWS Services ONLY (e.g. CloudFront and ALBs .. NOT EC2)
-ACM is a regional service
-Certs cannot leave the region they are generated or imported in
-To use a cert with an ALB in ap-southeast-2 you need a cert in ACM in ap-southeast-2
-Global Services such as CloudFront operate as though within 'us-east-1'
-For CloudFront think of the distribution in us-east-1. The cert is deployed by ACM to the distribution - then the distribution sends to the edge locations
```

## CloudFront and SSL
```
-CloudFront Default Domain Name (CNAME)
-https://abcd.cloudfront.net/
-SSL Supported by default ... *.cloudfront.net cert
-Alternate Domain Names (CNAMES) e.g. cdn.catagram.....
-Verify Ownership (optionally HTTPS) using a matching certificate
-Generate or import in ACM .. in us-east-1 (Exam question)
-HTTP or HTTPS, HTTP => HTTPS, HTTPS Only
-Two SSL Connections: Viewer => CloudFront and CloudFront => Origin
-... Both need valid public certificates (and intermediate certs)
-Historically
    -..every SSL enabled site needed its own IP
    -encryption starts at the TCP connection ...
    -Host headers happens after that - Layer 7 // Application
    -SNI is a TLS extension, allowing a host to be included
    -Resulting in many SSL Certs/Hosts using a shared IP
    -Old browsers dont support SNI ... CF charges extra for dedicated IP
-Certificate issued by a trusted certificate authority (CA) such as Comodo, DigiCert, or Symantec, or ACM (us-east-1) MATCHES THE DNS NAME
-Only public trusted issued certificates are able to be applied to CloudFront distributions
-S3 Origins handle certificates natively
```

## CloudFront Security OAI and Custom Origins
```
-An OAI is a type of identity
-It can be associated with CloudFront distributions and buckets
-CloudFront 'becomes' that OAI
-That OAI can be used in S3 Bucket Policies
-DENY all BUT one or more OAI's
-Once OAI is associated with the distribution accesses are FROM the OAI
-Edge Locations via OAI allowed
-Custom Origins can securely require Custom Header/Firewall around Custom Origin
```

## CloudFront Geo Restrictions
```
-CloudFront Geo Restriction
-3rd Party Geolocation
-CF - Whitelist or Blacklist - COUNTRY ONLY
-CF - GeoIP Database 99.8%+ Accurate
-CF - Applies to the entire distribution
-3rdParty - Completely customisable ...
-Return Object or 403 (Forbidden) depending on Geo Restriction on clients location
-3rd Party Geolocation requires a compute instance, a private distribution and the generation of signed URLs or Cookies - but can restrict based on almost anything (licensing, user login status, user profile fields and much more)
```

## CloudFront Private Behaviors Signed URLs & Cookies
```
-Public - Open Access to objects
-Private ... requests require Signed Cookie or URL
-1 Behaviour - Whole Distribution PUBLIC or PRIVATE
-Multiple Behaviours - each is PUBLIC or PRIVATE
-OLD - A CloudFront Key is created by an Account Root User
-OLD - THE ACCOUNT is added as a TRUSTED SIGNER ..
-NEW - TRUSTED KEY GROUP(S) added ..
-CloudFront Signed URLs vs Cookies
  -SignedURLs provides access to one object
  -Historically RTMP distributions couldnt use cookies
  -Use URLs if your client doesnt support cookies
  -Cookies provides access to groups of objects
  -Use for groups of files/all files of a type - e.g. all cat gifs
  -Or if maintaining application URLs is important
```

## CloudFront - Field Level Encryption
```
-Client <=> Origin can be encrypted using HTTPS
-Information is processed at the web server as HTTP
-... i.e plaintext
-Field Level encryption happens at the edge ...
-Happens separately from the HTTPS tunnel
-A private key is needed to decrypt individual fields
-The sensitive data is encrypted at the edge and remains separately controlled through its lifecycle
```

## Lambda@Edge
```
-You can run lightweight Lambda at edge locations
-Adjust data between the Viewer & Origin
-Currently supports Node.js and Python
-Run in the AWS Public Space (Not VPC)
-Layers are not supported
-Different Limits vs Normal Lambda Functions
-Example Use Cases
    -A/B Testing - Viewer Request
    -Migration Between S3 Origins - Origin Request
    -Different Objects Based on Device - Origin Request
    -Content by Country - Origin Request
```

## DDOS 101
```
-Attacks designed to overload websites
-Compete against 'legitimate connections'
-Distributed - hard to block individual IPs/Ranges
-Types of Attacks
  -Application Layer - HTTP Flood
  -Protocol Attack - SYN Flood
  -Volumetric - DNS Application
-... often involve large armies of compromised machines (botnets)
-Legitimate users of the application are prevented from accessing the website because they have to compete for access with the attack. The performance of the application is reduced to failure levels.
```

## AWS Shield
```
-AWS Shield Standard & Advanced - DDOS Protection
-Shield Standard is FREE, Shield Advanced has a cost
-Network Volumetric Attacks (L3) - Saturate Capacity
-Network Protocol Attacks (L4) - TCP SYN Flood
-... leave connections open, prevent new ones
-... L4 can also have a volumetric component
-Application Layer Attacks (L7) - e.g. web request floods
-... query.php?search=all_the_cat_images_ever
-AWS Shield - Standard
    -Free for AWS customers
    -... protection at the perimeter
    -... region/VPC or the AWS edge
    -Common Network (L3) or Transport (L4) Layer Attacks
    -Best protection using R53, CloudFront, AWS Global Accelerator
-AWS Shield - Advanced
    -$3k per month (per ORG), 1 year lock-in + data (OUT) /M
    -Protects CF, R53, Global Accelerator, Anything associated with EIPs (i.e EC2), ALBs, CLBs, NLBs
    -Not automatic - must be explicitly enabled in Shield Advanced or AWS Firewall Manager Shild Advanced Policy (Exam Question)
    -Cost protection (i.e EC2 scaling) for unmitigated attacks
    -Proactive Engagement & AWS Shield Response Team (SRT)
    -WAF Integration - includes basic AWS WAF fees for web ACLs, rules, and web requests
    -Application Layer (L7) DDOS protection (uses WAF)
    -Real time visibility of DDOS events and attacks
    -Health-based detection - application specific health checks, used by proactive engagement team
    -Protection groups
```

## AWS Network Firewall
```
-Firewall - VPC, Subnets & Firewall Policy
-Firewall Policy - 1:M Relationship - Fireall policy attached to one or more firewalls - defines filtering behavior
-Rule Group - collection of rules
-... either stateless or stateful
-... processing order & default action
-Rules - Traffic is evaluated against rules, pass, drop, forward
-Rule Groups
    -Rule groups contain rules ...
    -... customer created or AWS managed
    -Defined upfront as stateful or stateless (type)
    -Capacity ... limit on the processing requirements for the group (all the rules in the group)
-Stateless
    -Inspects each packet in isolation - isnt state aware
    -Request and Response are different
    -INBOUND Request & OUTBOUND Request are different
    -5 tuple - Protocol, SRC/DST CIDR, SRC/DST Port
    -Actions: Pass, Drop, Forward or Custom
    -Processes in order ... then stops
    -(default = lowest number = high priority)
-Stateful
    -Uses suricata rules engine
    -Default = Pass, order = pass, drop, alert
    -Rules = suricata format or built in standard for simple/domain list
    -Simple = 5-Tuple w/ state (~ Similar to Security Groups)
    -Domain list = domain names & protocol types
    -... wildcards, headers(http), SNI(https)
    -IPS Rules - Suricata compatible
```

## Implementing DNSSEC with Route53
```
-ZSK Creation and Rotation handled by R53 Internally
-Create Alarms for DNSSECInternalFailure and DNSSECKeySigningKeysNeedingAction
-DNSSEC Validation can be enabled for VPCs - Invalid results on DNSSEC enabled zones wont be returned
```

# Domain 2: Security Logging and Monitoring

## CloudWatch
```
-Architecture Concepts
    -Ingestion, Storage, and Management of Metrics
    -Public Service - public space endpoints
    -... AWS Service Integration - management plane
    -... Agent integration ... e.g. EC2-richer metrics
    -... On-Premises integration via Agent/API (custom metrics)
    -... Application integration via API/Agent (custom metrics)
    -View data via console UI, CLI, API, dashboards & anomaly detection
    -Alarms ... react to metrics, can be used to notify or perform actions
-Data
    -Namespace = container for metrics e.g. AWS/EC2 & AWS/Lambda
    -Datapoint = Timestamp, Value, (optional) unit of measure
    -Metric ... time ordered set of data points
    -... CPUUtilization, NetworkIn, DiskWriteBytes - (EC2)
    -Every metric has a MetricName (CPUUtilization) and a Namespace (AWS/EC2)
    -Dimension ... name/value pair
    -e.g. CPUUtilization Name=InstanceId, Value=i-111111111 (cat)
    -e.g. CPUUtilization Name=InstanceId, Value=i-222222222 (dog)
    -AutoScalingGroupName, ImageId, InstanceId, InstanceType
    -Resolution ... Standard (60s granularity) ... High (1s)
    -Retention ... sub 60s Retained for 3 hours
    -60s (1 minute) retained for 15 days
    -300s (5 minutes) retained for 63 days
    -3600s (1 hour) retained for 455 days
    -As data ages, its aggregated and stored for longer with less resolution
    -Statistic ... aggregation over a period (e.g. Min, Max, Sum, Average)
    -Percentile - e.g. p95 & P97.5
-Alarms
    -Alarm - watches a metric over a time period
    -... ALARM or OK
    -... value of metric vs threshold ... over time
    -... one or more actions
    -Alarm Resolution
-Every datapoint has a timestamp, value and optionally a unit of measure
-Certain AWS Metrics allow aggregation across dimensions, for example CPUUtilization by instance type - NOT ALL and NOT CUSTOM
```

## CloudWatch Logs Architecture
```
-CloudWatch Logs - Ingestion
    -Public Service - Store, Monitor, Access logging data
    -AWS, On-Premises, IOT or any application
    -CWAgent - system or custom application
    -VPC Flow logs
    -CloudTrail ... Account events and AWS API Calls
    -Elastic Beanstalk, ECS Container Logs, API GW ... Lambda execution logs
    -Route53 - Log DNS Requests
-CloudWatch Logs
    -Log groups - grouping of log streams
    -Log stream - collection of log events of data from an AWS resource taken from 1 source
-CloudWatch Logs - Subscriptions
    -LogGroup => Subscription Filter => Kinesis Data Firehose (near real time data) => S3 Bucket
    -LogGroup => Subscription Filter => AWS Managed Lambda or Custom Lambda for realtime data to be delivered to somewhere => Destination
-CloudWatch Logs - Summary
    -For any log management scenarios default to CloudWatch Logs
    -On-Premsises AND AWS
    -Export to S3 ... CreateExportTask - 12 hours
    -Near Realtime or persist logs - Kinesis Firehose
    -Firehose for any 'firehose destinations'
    -Realtime - Lambda or Kinesis Data stream (KCL consumers)
    -Elasticsearch - AWS Managed Lambda
    -Metric Filter ... scan log data, generate a CloudWatch Metric
```

## CloudWatch Events and EventBridge
```
-If X happens, or at Y time(s) ... do Z
-EventBridge is ... CloudWatch Events v2 (*)
-A default Event bus for the account
-... In CloudWatch Events this is the only bus (implicit)
-EventBridge can have additional event busses
-Rules match incoming events ... (or schedules)
-Route the events to 1+ Targets ... e.g. Lambda 
```

## S3 Events
```
-Notification generated when events occur in a bucket
-... can be delivered to SNS, SQS and Lambda Fucntions
-Object Created (Put, Post, Copy, CompleteMultiPartUpload)
-Object Delete (*, Delete, DeleteMarkerCreated)
-Object Restore (Post(Initiated), Completed)
-Replication (OperationMissedThreshold, OperationReplicatedAfterThreshold, OperationNotTracked, OperationFailedReplication)
-EventBridge is an alternative and supports more types of events and more services
```

## SNS Architecture
```
-Public AWS Service - network connectivity with Public Endpoint
-Coordinates the sending and delivery of messages
-Messages are <= 256KB payloads
-SNS Topics are the base entity of SNS - permissions and configuration
-A publisher sends messages to a TOPIC
-TOPICS have Subscribers which receive messages
-e.g. ... HTTP(S), Email(-JSON), SQS, Mobile Push, SMS Messages & Lambda
-SNS used across AWS for notifications - e.g. CloudWatch & CloudFormation
-Delivery Status - (including HTTP, Lambda, SQS)
-Delivery Retries - Reliable Delivery
-HA and Scalable (Region)
-Server Side Encryption (SSE)
-Cross-Account via TOPIC Policy
```

## Amazon Inspector
```
-Scans EC2 instances & the instance OS
-... also containers
-... Vulnerabilities and deviations against best practice
-Length ... 15 mins, 1 hour, 8/12 hours or 1 day
-Provides a report of findings ordered by priority
-Network Assessment (Agentless)
-Network & Host Assessment (Agent)
-Rules packages determine what is checked
-Network Reachability (no agent required)
-... Agent can provide additional OS visibility
-Check reachability end to end. EC2, ALB, DX, ELB, ENI, IGW, ACLs, RT's, SG's, Subnets, VPCs, VGWs & VPC Peering
-RecognizedPortWithListener, RecognizedPortNoListener, RecognizedPortNoAgent
-UnrecognizedPortWithListener
-Packages (... Host Assessments, Agent Required)
-Common vulnerabilities and exposures (CVE)
-Center for Internet Security (CIS) Benchmarks
-Security best practices for Amazon Inspector
```

## AWS Trusted Advisor
```
-Account level - no agents to install, it just works
-Cost Ootimization, Performance, Security, Fault Tolerance and Service Limits
-7 Core checks with basic & developer support plans
-Anything beyond requires Business or Enterprise plans
-Free 7 Core checks (basic or developer support)
    -S3 Bucket Permissions - NOT Objects
    -Security Groups - Specific Ports Unrestricted
    -IAM use
    -MFA on Root Account
    -EBS Public Snapshots
    -RDS Public Snapshots
    -50 service limit checks
-Business and Enterprise Support ...
    -Core 7 checks, PLUS ...
    -115 Further checks (14 cost, 17 security, 24 fault tolerant, 10 performance and 50 service limit)
    -Access via the AWS Support API
    -CloudWatch integration - react to changes
-3 different levels of severity
```

## VPC Flow Logs
```
-Capture metadata (NOT CONTENTS)
-Attached to a VPC - ALL ENIs in that VPC
-.. Subnet - ALL ENIs in that Subnet
-.. ENIs directly
-Flow Logs are NOT realtime (exam question)
-Log Destinations ... S3 or CloudWatch Logs
-... or Athena for querying in S3 ...
-Flow logs capture metadata from the capture point down
-Flow logs can capture ACCEPTED, REJECTED or ALL metadata
-Flow log data
    -ICMP = 1, TCP = 6, UDP = 17
-To and FROM 169.254.169.254, 169.254.169.123, DHCP, Amazon DNS Server & Amazon Windows license not recorded
```

## Application (Layer 7) Firewalls
```
-Layer 7 Firwalls are aware of the Layer 7 protocol i.e HTTP
-L7 FW can identify normal or abnormal rquests Protocol specific attacks
-Encryption terminated on the L7 Firewall
-New Encrypted L7 Connection between FW and backend
-Data at L7 can be inspected and blocked, replaced, or tagged (adult, spam, off topic)
-Able to identify, block & adjust specific applications e.g. Facebook
-L7 FW keeps all L3, L4 & L5 features but can react to L7 elements (DNS, RATE, Content, Headers etc)
```

## Web Application Firewall (WAF), WEBACLs, Rule Groups and Rules
```
-Rules within Rule Groups
-Rule Groups within WEBACL
-Architecture is based on creating a feedback loop between logs generated by WAF/Applications or ecosystem data to update WEBACL
-WEBACL is the main unit of configuration within WAF
-WEBACL Default Action (ALLOW or BLOCK) - Non matching
-Resource Type - CloudFront or Regional Service 
-... ALB, API GW, AppSync ... pick region
-Add Rule Groups or Rules .. processed in order
-Web ACL Capacity Units (WCU) - Default 1500
-..can be increased via support ticket
-WEBACL's are associated with resources (this can take time)
-... adjusting a WEBACL takes less time than associating one
-Rule Groups
    -Rule Groups contain rules
    -They don't have default actions ... that's defined when groups or rules are added to WEBACLs
    -Managed (AWS or Marketplace), Yours, Service Owned (i.e Shield & Firewall Manager)
    -Rule groups can be referenced by multiple WEBACL
    -Have a WCU capacity (defined upfront, max 1500*)
-WAF Rules
    -Type, Statement, Action
    -Type: Regular or Rate-based
    -Statement: (WHAT to match) or (COUNT ALL) or (WHAT & COUNT)
    -... origin country, IP, label, header, cookies, query parameter, URI path, query string, body (first 8192 bytes ONLY), HTTP Method
    -Single, AND, OR, NOT
    -Action: Allow*, Block, Count, Captcha ... Custom Response (x-amzn-waf-), Label
    -... labels can be referenced later in the same WEBACL ... multi-stage flows
    -ALLOW & BLOCK stop processing, Count/Captcha actions continue
-WEBACL - Monthly ($5/month*) (remember these can be reused)
-RULE on WEBACL - Monthly ($1/month*)
-REQUESTS per WEBACL - Monthly ($0.60/1 million*)
-Intelligent Threat Mitigation
-Bot Control - ($10/month) & ($1/1 million requests*)
-Captcha - ($0.40/1,000 challenge attempts*)
-Fraud Control/Account Takeover ($10/month* & $1/1,000 logina attempts*)
-Marketplace Rule Groups - Extra costs
```

## CloudTrail Architecture
```
-Logs API calls/activities as a CloudTrail Event
-90 days stored by default in Event History
-Enabled by default - no cost for 90 day history
-To customise the service ... create 1 or more Trails
-Management Events and Data Events
-Regional service
-Logs are generated based on Region or Global Service
-Enabled by default ... but 90 days .. no S3
-Trails are how you configure S3 and CWLogs
-Management events only by default
-IAM, STS, CloudFront => Global Service Events (logged to us-east-1)
-NOT REALTIME - There is a delay
```

## CloudTrail Log File Integrity
```
-CloudTrail delivers log files periodically into an S3 Bucket
-Enabling Log Validation directs CloudTrail to CREATE and SIGN digest files ... each for an hour of log delivery
-These are stored in the same bucket ... but in a different folder from the logs ... granular security
-Digest files contain HASH of each log file in the past hour
-If there is no log file matching a hash in a digest ... that log has been deleted or edited
-Missing digests can be identified if signatures dont match
-CloudTrail signs each digest file with its private key
-Enabled on a trail
-Digests have hashes of log files for 1 hour period
-Digests are signed with a CloudTrail private key
-... unique per region
-Delivered into a different folder vs logs (same bucket)
-CLI(validate-logs) to validate ... not the console UI
```

## AWS Athena
```
-Serverless Interactive Querying Service
-Ad-hoc queries on data - pay only data consumed
-Schema-on-read - table-like translation
-Original data never changed - remains on S3
-Schema translated data => relational-like when read
-Output can be sent to other services
-Supports standard formats of structured, semi-structured and unstructured data. Source data is stored on S3
-Athena can directly read many AWS data formats such as CloudTrail, ELB Logs and Flow Logs
-Data on S3 is FIXED
-"Tables" are defined in advance in a data catalog and data is projected through when read. It allows SQL-like queries on data without transforming source data
-Output can be sent to visualization tools
-Billed based on data consumed during query
-Queries where loading/transformation isnt desired
-Occassional/Ad-hoc queries on data in S3
-Serverless querying scenarios - cost conscious
-Querying AWS logs - VPC Flow Logs, CloudTrail, ELB logs, cost reports etc ...
-AWS Glue Data Catalog & Web Server Logs
-w/ Athena Federated Query ... other data sources
```

## Amazon Macie 
```
-Data Security and Data Privacy Service
-Discover, Monitor and Protect Data ... stored in S3 Buckets
-Automated discovery of data i.e PII, PHI, Finance
-Managed Data Identifiers - Built-in - ML/Patterns
-Custom Data Identifiers - Proprietary - Regex Based
-Integrates - With Security Hub & 'finding events' to EventBridge
-Centrally manage ... either via AWS ORG or one Macie account Inviting
-Findings can be delivered as an event to EventBridge and then to other AWS services
-Amazon Macie - Identifiers
    -Managed data identifiers - maintained by AWS
    -... growing list of common sensitive data types
    -... Credentials, finance, Health, personal identifiers
    -Custom data identifiers - created by you
    -Regex - defines a 'pattern' to match in data e.g. [A-Z]-\d{8}
    -Keywords - optional sequences that need to be in proximity to regex match
    -Maximum Match Distance - how close keywords are to regex pattern
    -Ignore Words - if regex match contains ignore words, it's ignored
-Amazon Macie - Findings
    -Policy Findings or Sensitive data findings
    -Ex: Policy:IAMUser/S3BlockPublicAccessDisabled
    -Ex: Policy:IAMUser/S3BucketEncryptionDisabled
    -Ex: Policy:IAMUser/S3BucketPublic
    -Ex: Policy:IAMUser/S3BucketSharedExternally
    -Ex: SensitiveData:S3Object/Credentials
    -Ex: SensitiveData:S3Object/CustomIdentifier
    -Ex: SensitiveData:S3Object/Financial
    -Ex: SensitiveData:S3Object/Multiple
    -Ex: SensitiveData:S3Object/Personal
```

## AWS Glue 101
```
-Serverless ETL (Extract, Transform & Load)
-... vs datapipeline (which can do ETL) and uses servers (EMR)
-Moves and transforms data between source and destination
-Crawls data sources and generates the AWS Glue Data catalog
-Data Source: Stores: S3, RDS, JDBC Compatible & DynamoDB
-Data Source: Streams: Kinesis Data Stream & Apache Kafka
-Data Targets: S4, RDS, JDBC Databases
-AWS Glue - Data Catalog
    -Persistent metadata about data sources in region
    -One catalog per region per account
    -Avoids data silos ...
    -Amazon Athena, Redshift Spectrum, EMR & AWS Lake Formation all use Data Catalog
    -... configure crawlers for data sources
-Crawlers connect to data stores, determine schema and create metadata in the data catalog
-Glue jobs can be initiated manually or via events using EventBridge
```

## AWS Artifact
```
-Self-service Portal
-Access to AWS Compliance Reports
-... and review/access agreements with AWS
-ISO, PCI, FedRAMP, SOC
-... used within formal due-dilligence or audit processes
-Generally you can share these withy your customers
-On-demand ... !!!
```

# Domain 5 Data Protection

## Hardware Security Module (HSM)
```
-Separate device that is isolated from main infrastucture
-Data is sent to the HSM for Encryption or Decryption. Requests are made to generate data encryption keys or to sign data.
-All operations happen on the HSM
-Keys are stored securely inside a secure enclave ... keys never leave and operations are performed on the HSM
-HSMs are tamper proof & hardened against physical or logical attacks
-Accessed via tightly controlled industry standard APIs
-Role separation ... HSM admins can update & maintain but dont always have full acess
```

## AWS Key Management Service (KMS) 101
```
-Regional & Public Service
-Create, Store and Manage Keys
-Symmetric and Asymmetric keys
-Cryptographic operations (encrypt, decrypt & ...)
-Keys never leave KMS - Provides FIPS 140-2 (L2) (Level 2 Compliance)
-KMS Keys
-KMS Keys are logical - ID, date, policy, desc & state
-... backed by physical key material
-Generated or Imported
-KMS Keys can be used for up to 4KB of data
-KMS and KMS Keys
-Data Encryption Keys (DEKs)
    -GenerateDataKey - works on > 4KB
    -Plaintext Version
    -Ciphertext Version
    -Encrypt data using plaintext key
    -Discard
    -Store encrypted key with data
-KMS Keys are isolated to a region & never leave ...
-... Multi-region keys discussed (if required) in a different video
-... AWS Owned & Customer Owned
-... w/ Customer owned ... AWS Managed or Customer Managed KEYS
-Customer managed keys are more configurable
-KMS Keys support rotation
-Backing Key (and previous backing keys)
-Aliases
-Key Policies (Resource)
-Every KEY has one!
-Key Policies + IAM Policies
-Key Policies + Grants
```

## CloudHSM
```
-With KMS ... AWS Manage ... Shared but separated
-True "Single Tenant" Hardware Security Module (HSM)
-AWS provisioned ... fully customer managed
-Fully FIPS 140-2 Level 3 (KMS is L2 Overall, some L3 Complaint)
-Industry Standard APIs - PKCS#11, Java Cryptography Extensions (JCE), Microsoft CryptoNG (CNG) libraries
-KMS can use CloudHSM as a custom key store, CloudHSM integration with KMS
-HSMs keep keys and policies in sync when nodes are added or removed
-HSMs operate in an AWS managed HSM VPC Interfaces are added to customer VPC
-CloudHSM Use Cases
    -No Native AWS Integration ... e.g. no S3 SSE
    -Offload the SSL/TLS Processing for Web Servers
    -Enable Transparent Data Encryption (TDE) for Oracle Databases
    -Protect the Private Keys for an Issuing Certificate Authority (CA)
```

## S3 Object Encryption CSE/SSE
```
-Buckets arent cencrypted ... objects are ...
-Client-Side Encryption (S3 never sees plaintext)
-Server-Side Encryption
    -Server-Side Encryption with Customer-Provided Keys (SSE-C)
    -Server-Side Encryption with Amazon S3-Managed Keys (SSE-S3 No KEY Control No Role Separation) (this is the default AES-256)
    -Server-Side Encryption with KMS KEYS Stored in AWS Key Management Service (SSE-KMS KEY Rotation Control Role Separation)
```

## Envelope Encryption
```
-Encryption Flow
  -Plaintext DEK is used to encrypt the plaintext data. The Plaintext DEK is immediately discarded
  -Unique DEK per object. Always stored encrypted with the object.
-KEK = Asymmetric or Symmetric (w/KMS)
-DEK = Symmetric
-Decryption Flow
    -Request is made to "unwrap" or "decrypt" the DEK
    -If ALLOWED, plaintext DEK is returned from Key Management Service
    -Plaintext DEK decrypts cipher text and is then discarded
    -Decrypting with a symmetric key is faster. Less data (the DEK only) is transferred to KMS
-Asymmetric keys are flexible (public part is ... public)
-... but they're slow
-Symmetric keys are fast ... but difficult to securely move
-Envelope encryption can be best of both (if you use Asymmetric KEKs)
-Symmetric keys are used for encrypt/decrypt (speed)
-... secured with Asymmetric keys (normally) for flexibility
-In either case ... less data T0/FROM the key storage service (i.e KMS)
-Unique DEKs can be used per object
```

## S3 Bucket Keys
```
-S3 w/out Bucket Keys
    -Each Object "PUT" using sse-kms uses a unique DEK
    -Unique DEK, stored with the object
    -Each Data Encryption Key (DEK) is an API call to KMS
    -Using a single KMS key results in a scaling limit for PUTS per second per key
    -Calls to KMS have a cost & levels where throttling occurs - 5,500 or 10,000 or 50,000 p/s across regions
-S3 with Bucket Keys
    -Time Limited Bucket Key used to generate DEKs within S3
    -Bucket keys significantly reduces KMS API calls - reducing cost and increasing scalability
    -Not Retroactive, only effcets objects after enabled on bucket
-CloudTrail KMS events now show the bucket ....
-.. Not the Object
-Works with replication ... the object encryption is maintained
-... if replicating plaintext to a bucket using bucket keys the object is encrypted at the destination side (ETAG changes)
```

## AWS vs Customer Managed KMS Keys
```
-AWS Managed Keys
    -Created by AWS when you use encryption in a service
    -1 per service per region
    -Key naming is standardised aws/service i.e aws/s3
    -... usable by that service in that region ... only
    -Metadata readable by the creator AWS account
    -The Key policy cannot be changed ... aws managed
    -Rotated every about 365 days, cannot be changed or disabled
    -No cross-account usage (fixed key policy)
    -No monthly cost ... Has usage costs beyond free tier
-Customer Managed Keys
    -Created explicitly by customers ... NOT NAMED aws/servicename
    -... used by services which support customer managed keys
    -Full control of key policy, account trust can be removed
    -requiring all permissions to be ALLOWED/DENIED via the key policy
    -Role Separation - admin rights vs usage rights ... set per key
    -Can be used cross-accoutn via Key Policy changes
    -Rotation is default about 365 days can be disabled
    -Monthly cost ... plus usage costs beyond free tier
```

## Importing Key Material vs Generated Key Material
```
-Generated by KMS or Imported (BYOK) (Symmetric Keys Only)
-KMS Key = Logical ... Key Metadata + Key Material
-Regulation - you need to prove you generated it (regulations/standards)
-Same key material inside and outside of AWS
-To allow for immediate deletion and restore later
-... generated is 7-30 days & cant be restored
-Keep material outside of AWS for Disaster Recovery
-You can import and re-import key material 
-... you cannot change it (no auto rotation)
-... manual key rotation using Aliases
-Key Origin = AWS_KMS | EXTERNAL | AWS_CLOUDHSM
-... cant change the origin after its created
-No portability ... cant decrypt KMS ciphertext outside of AWS using the raw material
-"Deleting" an imported Key .. deletes the key material, NOT the key or metadata
-... AWS can restore key & metadata but you manage the material
```

## Asymmetric keys in KMS
```
-Key Pair (Private & Public) - Private never leaves KMS
-Public key can be downloaded via console or CLI ... kms:GetPublicKey
-RSA - Encrypt/Decrypt, Signing and Verification ... but not both
-Elliptic Curve (ECC) - Signing and Verification
-SM2 (China) - Encrypt/Decrypt, Signing and Verification ... not both
-KeySpec = ENCRYPT_DECRYPT | SIGN_VERIFY (cant be changed)
-AWS Services which use KMS use Symmetric NOT Asymmetric
```

## Digital Signing using KMS
```
-App A needs to sign some data (file)
-App B needs to know the file hasnt been altered
-App B needs to know the file comes from App A
-... only App A should be able to sign it
-Using KSM Asymmetric keys
-... able to be secured using identity policies and key policies
-File or Digest (HASH) submitted to KMS using the SIGN API
-... less than 4KB ... KMS handles end-to-end
-... otherwise you create a digest (HASH) and send that to KMS ... telling it via the MessageType parameter
-KMS uses the private key part to create a signature of the digest (HASH) ... this is signing
-Store/send file and signature somewhere App B can access
-App B takes the full and creates its own digest (HASH) of that file
-Send App B's digest (HASH), signature, algorithm and key used to KMS via the Verify API
-KMS uses the public key to verify the file (via the digest) matches the signature
-... file hasnt been changed & it was signed by App A
-Can be done without using KMS via the public key part ... which can be exported out of KMS
```

## Encryption SDK - Data Key Caching
```
-Client Side Encryption Library - Apache 2.0 license
-... C, .NET, AWS CLI, Java, Javascript & Python
-Handles data keys and (many AWS/On-Prem) wrapping keys
-Handles message formatting in a standard way
-... if your app uses encryption .. this will save you time
-By default ... unique data key for every encryption
-KMS rate limits ...
-Encryption SDK - Data Key Caching
    -GenerateDataKey limits (remember, 1 per encrypt)
    -50,000 (us-east-1, us-west-2, eu-west-1) shared
    -10,000 (us-east-2, ap-southeast-1 & 2, ap-northeast-1, eu-central-1, eu-west-1)
    -5,500 per second (shared)
    -Cryptographic Materials Cache (CMM)
    -Security threshold (max age(mandatory), max messages encrypted, max bytes encrypted)
```

## KMS Security Model & Key Policy
```
-Implicit TRUST S3 TRUSTS owning AWS account
-Explicit TRUST S3 TRUSTS owning AWS account *if key policy allows
-Assuming no explicit DENY statements .. access depends on if the account TRUST is present on the key policy or not
-Most services IMPLICITLY TRUST their AWS account
-That account (via identity policies) can trust identities
-Resource policies are how that trust can be granted to external AWS accounts/identities
-With KMS ... that TRUST is EXPLICIT via a key policy
-... if present, then identities policies can ALLOW permissions on KMS keys
-.. it can be removed
-.. meaning that identity policies will no longer be enough
-.. explicit ALLOWS will be required on the key as the ONLY way to ALLOW
-Conceptually split usage actions from admin actions .. role separation
```

## KMS Grants
```
-Grants ALLOW permissions on 1 KMS key
-Like identity and key policies .l.. only designed for temporary access
-.. and least permission access
-Generally used by AWS services which use KMS
-.. they create a grant on behalf of a user
-... with JUST the permissions required for that service
-Grants cant DENY .. they are retired once the task is completed
-Pause and think about how complex key policies could be ...
-Grantee Principal = identity who can use the grant
-... operations can be used, without specifying the grant
-... there can be a 5 minute propagation delay ... AccessDenied Error
-... Grant token can be generated and used before then
-Grants allow operations .. this can include CreateGrant!
-ValidationError if you try to do something a grant doesnt ALLOW
-Retire a grant = finished the task, grantee retires
-Revoke a grant = Done by a key administrator
```

## Multi-Region KMS Keys
```
-Single or multi-region is decided upfront, cant be adjusted later
-Multi-region keys start 'mrk-'
-Can be imported or generated Symmetric or Asymmetric
-Custom key stores are not supported
-Repliacs are full keys(replicated) they work if the primary fails
-Deleting primary also deletes replicas ... change primary if required
-Shared properties are replicated. KEY ID, Material, Origin, Spec, Usage, Rotation
-When using key rotation, new material is not used until fully replicated
-Decreases latency
-Disaster Recovery (DR)
-Active-Active Applications w/ global HA/FT
-Digital Signing
```

## CloudHSM vs KMS
```
-KMS - AWS Product Integration
-KMS - Standard AWS APIs & Access Methods
-KMS - FIPS validated HSMs .. you're ok with shared
-KMS - FIPS 140-2 LEVEL "2" 
-CloudHSM - Standard APIs (PKCS#11, JCE, CryptoNG)
-CloudHSM - Dedicated HSMs, Self Managed
-CloudHSM - FIPS 140-2 LEVEL "3"
```

## KMS Custom Key Stores
```
-Keys which can be interacted with via KMS & KMS APIs
-All encryption operations are done on CloudHSM (via KMS)
-FIPS 140-2 Level "3" HSMs
-Secondary, Independent Audit Path
-No shared HSMs are used
-Benefits of CloudHSM, used via familiar KMS APIs & access methods
-Currently ONLY supports symmetric keys
-No multi-region key support
-Doesnt support automatic key rotation
-Each CloudHSM cluster supports one custom key store
-"kmsuser" is created on the CloudHSM cluster
-Same AWS ACCOUNT & REGION (KMS & CloudHSM)
```

## AWS Secrets Manager
```
-It does share functionality with Parameter Store
-Designed for secrets (... passwords, API KEYS ..)
-Usable via Console, CLI, API or SDKs (integration)
-Supports automatic rotation .. this uses lambda
-Directly integrates with some AWS products (.. RDS)
```

## RDS Encryption & IAM Authentication
```
-SSL/TLS (in transit) is available for RDS, can be mandatory
-RDS supports EBS volume encryption - KMS
-Handled by HOST/EBS
-AWS or Customer Managed CMK generates data keys
-Data keys used for encryption operations
-Storage, Logs, Snapshots & replicas are encrypted
-... encrypted cant be removed
-RDS MSSQL and RDS Oracle support TDE
-... Transparent Data Encryption
-Encryption handled within the DB engine
-RDS Oracle supports integration with CloudHSM
-Much stronger key controls (even from AWS)
-Data is encrypted and decrypted by the host
-Snapshots of encrypted RDS instances are also encrypted using the same key
-RDS IAM Authentication
    -RDS Local DB Account configured to use AWS Authentication Token
    -Policy attached to Users or Roles maps that IAM identity onto the local RDS user
    -Authorization is controlled by the DB Engine. Permissions are assigned to the local DB User. IAM is NOT used to authorise, only for authentication
```

## DynamoDB Encryption
```
-DDB Encryption client used to encrypt data prior to sending to DynamoDB AWS never see plaintext
-Data between DDB endpint and DDB table - plaintext or client encrypted data
-A table HAS to be encrypted at rest, unencrypted is NOT an option
-GSIs, Streams, Global Tables & Backups are all encrypted
-DynamoDB Encryption - Considerations
    -You cannot use a customer managed key with DAX
    -LSI/GSI use the same key as the table
    -Backups use the same key, restores require THAT key
    -... restored tables can use different encryption settings
    -Customer maanged keys work with transactions BUT a temp copy uses AWS owned key
    -Data in streams has a 24-hour lifetime even when a key is disabled
    -If a customer managed key is disabled or inaccessible for 7 days, a table is backed up & archieved. You'll need the key to restore it!
```

## KMS encryption context
```
-Traditional Encryption
    -.. IN Plaintext & Key & Algorithm, OUT Ciphertext
-Traditional Decryption
    -.. IN Ciphertext & Key & Algorithm, OUT Plaintext
-Provides confidentiality....
-Encryption Context = Additional Authenticated Data (AAD)
-Non Secret key value pairs you provide when encrypting
-The same key value pairs MUST be provided when decrypting 
-It ensures the integrity of data ...
-Generally you provide a known part of data .. e.g. email/id
-... with the encryption/decryption process
```

## Elastic Load Balancer Architecture
```
-Configured to run in 2+ AZs. 1+ Nodes are place dinto a subnet in each AZ and scale with load.
-Each ELB is configured with an (A) record DNS name. This resolves to the ELB nodes.
-Internet-facing Nodes have Public IPs
-Internal only have Private IPs
-Load Balancers (Nodes) are configured with listeners which accept traffic on a port & protocol and communicate with targets on a port and protocol
-Internet-facing LB nodes can access public & private EC2 instances
-8+ free IPs per subnet and a /27 or larger subnet to allow for scale
-Internal load balancers can be used to allow scaling between application tiers
-LBs allow each tier to scale independently
-Each Node gets 1005 / Number of Nodes
-ELB is a DNS A Record pointing at 1+ Nodes per AZ
-Nodes (in one subnet per AZ) can scale
-Internet-facing means nodes have public IPv4 IPs
-Internal is private only IPs
-EC2 doesnt need to be public to work with a LB
-Listener Configuration controls WHAT the LB does
```

## Application Load Balancing (ALB) vs Network Load Balancing (NLB)
```
-1 SSL PER CLB
-CLBs dont scale ... every unique HTTPS name requires an individual CLB because SNI isnt supported
-1 SSL PER RULE
-V2 Load Balancers support rules and target groups. Host based rules using SNI and an ALB allows consolidation
-Application Load Balancer (ALB)
    -Layer 7 Load balancer ... listens on HTTP and/or HTTPS
    -No other Layer 7 protocols (SMTP, SSH, Gaming, ...)
    -... and NO TCP/UDP/TLS Listeners
    -L7 content type, cookies, custom headers, user location, and app behavior
    -HTTP HTTPS always terminated on the ALB - no unbroken SSL (security teams!) (exam question)
    -... a new connection is made to the application
    -ALBs MUST have SSL certs if HTTPS is used
    -ALBs are slower than NLB ... more levels of the network stack to process
    -Health checks evaluate application health .. layer 7
-Application Load Balancer Rules
    -Rules direct connections which arrive at a listener
    -Processed in priority order
    -Default rule = catchall
    -Rule Conditions: host-header, http-header, http-request-method, path-pattern, query-string & source-ip
    -Actions: forward, redirect, fixed-response, authenticate-oidc & authenticate-cognito
-Network Load Balancer 
    -Layer 4 Load balancer ... TCP, TLS, UDP, TCP_UDP
    -No visibility or understanding of HTTP or HTTPS
    -No headers, no cookies, no session stickiness
    -Really really really fast (25% of ALB latency)
    -.. SMTP, SSH, Game Servers, financial apps (not http/s)
    -Health checks JUST check ICMP/TCP Handshake ... Not app aware
    -NLB's can have static IPs - useful for whitelisting
    -Forward TCP to instances .. unbroken encryption (exam)
    -Used with private link to provide services to other VPCs (exam)
```

## ELB SSL Offload and Session Stickiness
```
-SSL Offload
-Bridging 
    -Listener is configured for HTTPS. Connection is terminated on the ELB & needs a certificate for the domain.
    -ELB initiates a new SSL connection to backend instances. Instances need SSL certificates and the compute required for cyprtographic operations
-Pass-Through
    -Listener is configured for TCP. No encryption or decryption happens on the NLB. Connection is passed to backend instance.
    -Each instance needs to have the appropriate SSL cert installed. With this arhitecture there is no certificate exposure to AWS ... all self-managed and secured.
-Offload
    -Listener is configured for HTTPS. Connections are terminated and then backend connections use HTTP
    -ELB to instance connections use HTTP - no certificate or cryptographic requirements
-Connection stickiness
    -With no stickiness connections are distributed across all in-service backend instances. Unless application handles user state this could cause user logoffs shopping cart losses
    -Stickiness generates a cookie which locks the device to a single backend instance for a duration
```

## Load Balancer Security Policies
```
-Security Policy = Set of Ciphers & Protocols (Which are OK to use, on listener)
-Protocol ensures secure client => server comms (can use many Ciphers)
-Client and Server present Ciphers/Protocols ... best supported one is picked
-You control policy between Client => LB
-AWS chosen one is used LB => Targets ... ELBSecurityPolicy-2016-08
-Newer Policies = more secure, less compatible (default ELBSecurityPolicy-2016-08)
-ELBSecurityPolicy-FS if you need forward secrecy
```
