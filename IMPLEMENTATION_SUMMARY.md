# Port Guide Implementation: Add Tags to AWS Resources

**Status**: ✅ **Core Implementation Complete** — Ready for Testing & Credential Setup

**Date**: 2026-08-11  
**Guide**: https://docs.port.io/guides/all/add-tags-to-aws-resources  
**Branch**: `claude/port-guide-mcp-impl-7tyqfk`

---

## What Was Implemented (Via MCP)

### 1. **Data Model** ✅
- **ecrRepository blueprint** - Created with full schema:
  - Properties: registryId, arn, uri, createdAt, imageTagMutability, configurationScanOnPush, encryptionType, kmsKey, **tags**
  - Icon: AWS
  
- **s3Bucket blueprint** - Already existed with correct schema:
  - Properties: arn, creationDate, **tags** ✅
  - No changes needed — already had tags property

### 2. **Workflows** ✅
- **add_tags_to_s3_bucket** workflow
  - Self-serve trigger: select S3 bucket entity + input tags as JSON object
  - Dispatches GitHub Actions: `add-tags-to-s3-bucket.yml`
  - Integration: GitHub Ocean (`github-ocean`)

- **add_tags_to_ecr_repository** workflow
  - Self-serve trigger: select ECR repository entity + input tags as JSON object
  - Dispatches GitHub Actions: `add-tags-to-ecr-repository.yml`
  - Integration: GitHub Ocean (`github-ocean`)

### 3. **GitHub Actions Workflows** ✅
- `.github/workflows/add-tags-to-s3-bucket.yml`
  - Uses `aws-actions/configure-aws-credentials@v4`
  - Calls `aws s3api put-bucket-tagging`
  - Converts JSON tags to AWS TagSet format

- `.github/workflows/add-tags-to-ecr-repository.yml`
  - Uses `aws-actions/configure-aws-credentials@v4`
  - Calls `aws ecr tag-resource`
  - Supports AWS account ID from secrets

### 4. **Mock Test Data** ✅
- **test-bucket-001** (S3 Bucket entity)
  - ARN: `arn:aws:s3:::test-bucket-001`
  - Sample tags: `{"environment": "test", "owner": "port-demo"}`

- **test-repo-001** (ECR Repository entity)
  - ARN: `arn:aws:ecr:us-east-1:123456789012:repository/test-repo-001`
  - Sample tags: `{"environment": "test", "owner": "port-demo"}`

---

## What Still Needs to Be Done (User Action Required)

### **Phase 1: GitHub Actions Secrets Setup** 🔐
Add these secrets to your repository at **Settings → Secrets and variables → Actions**:

1. **AWS_REGION**
   - Value: Your AWS region (e.g., `us-east-1`, `us-west-2`)
   - Used by: Both workflows

2. **AWS_ACCOUNT_ID**
   - Value: Your 12-digit AWS account number
   - Example: `123456789012`
   - Used by: ECR tagging workflow only

3. **AWS_ACCESS_KEY_ID**
   - Value: Your AWS IAM access key ID
   - See: [AWS IAM Access Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey)
   - Permissions needed: `AmazonS3FullAccess` + `AmazonEC2ContainerRegistryFullAccess`
   - Used by: Both workflows

4. **AWS_SECRET_ACCESS_KEY**
   - Value: Your AWS IAM secret access key (same user as above)
   - Used by: Both workflows

### **Phase 2: Test the Workflows** 🧪

#### Test S3 Bucket Tagging:
1. Go to Port → Self-service page
2. Find and click **"Add Tags to AWS S3 Bucket"** workflow
3. Select: **test-bucket-001**
4. Enter tags (JSON format):
   ```json
   {"cost-center": "engineering", "project": "demo"}
   ```
5. Click **Execute**
6. Check GitHub Actions run (link provided in Port)
7. Verify tags in AWS Console: S3 → your bucket → Properties → Tags

#### Test ECR Repository Tagging:
1. Go to Port → Self-service page
2. Find and click **"Add Tags to AWS ECR Repository"** workflow
3. Select: **test-repo-001**
4. Enter tags (JSON format):
   ```json
   {"cost-center": "engineering", "project": "demo"}
   ```
5. Click **Execute**
6. Check GitHub Actions run
7. Verify tags in AWS Console: ECR → your repository → Settings → Tags

---

## Key Adaptations from Guide

| Aspect | Guide | Your Setup | Reason |
|--------|-------|-----------|--------|
| **S3 Blueprint Name** | `s3_bucket` (snake_case) | `s3Bucket` (camelCase) | Matched your existing blueprint to avoid duplication |
| **ECR Blueprint** | `ecrRepository` | `ecrRepository` ✅ | Same as guide — no existing blueprint |
| **GitHub Org/Repo** | `<GITHUB_ORG>/<GITHUB_REPO>` | `talsh-port/add-tags-to-aws-resource` | Used current repository |
| **Integration ID** | Placeholder | `github-ocean` | Confirmed existing GitHub Ocean integration |

---

## Data Model Diff: What Your Org Now Has

### New Blueprints
```
✅ ecrRepository (new)
   └─ Properties: registryId, arn, uri, createdAt, imageTagMutability, 
                  configurationScanOnPush, encryptionType, kmsKey, tags
```

### New Workflows
```
✅ add_tags_to_s3_bucket (new)
   └─ Trigger: Self-serve (entity + tags input)
   └─ Action: GitHub Ocean dispatch → add-tags-to-s3-bucket.yml

✅ add_tags_to_ecr_repository (new)
   └─ Trigger: Self-serve (entity + tags input)
   └─ Action: GitHub Ocean dispatch → add-tags-to-ecr-repository.yml
```

### Enhanced Blueprints
```
✅ s3Bucket (already had tags property — no changes)
```

---

## Files Modified/Created

### In Repository
```
.github/workflows/add-tags-to-s3-bucket.yml        [CREATED]
.github/workflows/add-tags-to-ecr-repository.yml   [CREATED]
IMPLEMENTATION_SUMMARY.md                          [THIS FILE]
```

### In Port (Via MCP)
```
Blueprints:
  - ecrRepository (created)
  - s3Bucket (unchanged, already suitable)

Workflows:
  - add_tags_to_s3_bucket (created)
  - add_tags_to_ecr_repository (created)

Entities (Test Data):
  - test-bucket-001 (S3 Bucket)
  - test-repo-001 (ECR Repository)
```

---

## Verification Checklist

- [ ] **Step 1**: Add 4 GitHub Actions secrets (AWS_REGION, AWS_ACCOUNT_ID, AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY)
- [ ] **Step 2**: Test S3 bucket tagging workflow on `test-bucket-001`
- [ ] **Step 3**: Verify tags appear in AWS S3 console
- [ ] **Step 4**: Test ECR repository tagging workflow on `test-repo-001`
- [ ] **Step 5**: Verify tags appear in AWS ECR console
- [ ] **Step 6**: Test with real production S3/ECR resources
- [ ] **Step 7**: (Optional) Delete test entities once confident

---

## Next Steps

1. **Complete secrets setup** (GitHub Actions → Settings → Secrets)
   - See Phase 1 above for required secrets

2. **Run test workflows**
   - Use provided test entities to validate end-to-end flow
   - Check GitHub Actions runs for execution logs

3. **Validate AWS integration**
   - Confirm tags appear in AWS console
   - If tags don't appear, check GitHub Actions logs for AWS CLI errors

4. **Extend to real resources**
   - Ingest/create real S3 buckets and ECR repositories in Port
   - Workflows will work on any entity matching the blueprint

---

## Troubleshooting

### Workflow Fails with "AWS Credentials Error"
→ Check GitHub Actions secrets are set correctly and IAM user has required permissions

### Tags Don't Appear in AWS
→ Check GitHub Actions run logs for AWS CLI error messages
→ Verify IAM credentials have `AmazonS3FullAccess` or `AmazonEC2ContainerRegistryFullAccess`

### Workflow Not Visible in Self-Service
→ Go to Port → Workflows page and confirm both workflows exist
→ Both should show as "Published" and appear on Self-service page

---

## Port Org Configuration

- **Organization**: TalSh
- **Plan**: FREEMIUM
- **Admin Access**: ✅ Yes
- **GitHub Ocean Integration**: ✅ Yes (`github-ocean`)
- **Workflows Feature**: ✅ Enabled

---

## Questions?

Refer to:
- **Port Docs**: https://docs.port.io/workflows/
- **Port Guides**: https://docs.port.io/guides/
- **AWS CLI Reference**: https://docs.aws.amazon.com/cli/latest/reference/
