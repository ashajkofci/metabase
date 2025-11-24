---
title: Testing License Scenarios
---

# Testing License Scenarios

This guide explains how Metabase's license/plan system works and how to test different license scenarios during development.

## Understanding the License System

### License Tiers

Metabase has different license tiers that control which premium features are available:

- **Pro**: Enterprise features with higher thresholds (4+ users OR 201+ queries for activation)
- **Starter**: Basic premium features with lower thresholds (2+ users OR 101+ queries for activation)
- **OSS (Open Source)**: No premium features

### How License Distinction Works

The license tier is determined by the `plan-alias` field in the token validation response. The code checks if the plan-alias starts with specific strings:

- Plans starting with "pro" → Pro tier
- Plans starting with "starter" → Starter tier

**Location in code:** `src/metabase/analytics/stats.clj` in the `completed-activation-signals?` function.

### Token Types

Metabase supports two types of license tokens:

1. **Remote Tokens** (64 hex characters):
   - Validated against token-check.metabase.com
   - Real-time validation with the Metabase store
   - Example: `a1b2c3d4e5f6...` (64 chars)

2. **Airgap Tokens** (starts with "airgap_"):
   - JWT-based tokens validated locally
   - For air-gapped/offline environments
   - Example: `airgap_eyJhbGc...`

**Token validation code:** `src/metabase/premium_features/token_check.clj`

## Testing Different License Scenarios

### Using the Development Override

For testing purposes, you can override the license plan type without needing an actual license token by using the `MB_DEV_LICENSE_TYPE` environment variable.

#### Setting the Override

**On Unix/Linux/Mac:**
```bash
export MB_DEV_LICENSE_TYPE="pro-cloud"
java -jar metabase.jar
```

**On Windows PowerShell:**
```powershell
$env:MB_DEV_LICENSE_TYPE="pro-cloud"
java -jar metabase.jar
```

**With Docker:**
```bash
docker run -p 3000:3000 \
  -e MB_DEV_LICENSE_TYPE="pro-cloud" \
  metabase/metabase
```

#### Valid License Types

Common values you can use for testing:

- `pro-cloud` - Pro tier, cloud-hosted
- `pro-self-hosted` - Pro tier, self-hosted
- `starter` - Starter tier
- Any other string will be treated as a custom plan

### Testing Activation Signals

Different license tiers have different activation thresholds:

**Pro Tier:**
- Requires 4+ active users OR 201+ queries
- Test by creating users or running queries

**Starter Tier:**
- Requires 2+ active users OR 101+ queries
- Lower thresholds for smaller deployments

### Example Test Scenarios

#### Scenario 1: Test Pro Features
```bash
# Set license to Pro
export MB_DEV_LICENSE_TYPE="pro-cloud"

# Start Metabase
java -jar metabase.jar

# Pro features should now be available
# - Advanced permissions
# - Audit logging
# - SSO integrations
# etc.
```

#### Scenario 2: Test Starter Limitations
```bash
# Set license to Starter
export MB_DEV_LICENSE_TYPE="starter"

# Start Metabase
java -jar metabase.jar

# Some Pro features should be unavailable
# Lower activation thresholds apply
```

#### Scenario 3: Test OSS Mode
```bash
# Don't set MB_DEV_LICENSE_TYPE
# Don't set MB_PREMIUM_EMBEDDING_TOKEN

# Start Metabase
java -jar metabase.jar

# No premium features available
```

## Feature Flags

Each premium feature is controlled by a feature flag. Features are defined in:
- `src/metabase/premium_features/settings.clj`

The features are determined by the token validation response. When using `MB_DEV_LICENSE_TYPE`, you're only overriding the plan-alias, not the individual feature flags. For complete feature testing, you may need an actual token.

## Implementation Details

### Code Flow

1. **Token Setting**: Token is set via `MB_PREMIUM_EMBEDDING_TOKEN` environment variable
2. **Token Validation**: Token is validated against the Metabase store (or locally for airgap)
3. **Plan Determination**: `plan-alias` field from validation response determines the tier
4. **Override**: `MB_DEV_LICENSE_TYPE` can override the plan-alias for testing
5. **Feature Checks**: Code uses `has-feature?` to check if specific features are enabled

### Key Functions

- `plan-alias` - Returns the current plan (checks MB_DEV_LICENSE_TYPE first)
- `has-feature?` - Checks if a specific feature is enabled
- `completed-activation-signals?` - Determines if activation thresholds are met

## Important Notes

⚠️ **Development Only**: The `MB_DEV_LICENSE_TYPE` override is intended for development and testing only. Do not use in production.

⚠️ **Limited Scope**: Setting `MB_DEV_LICENSE_TYPE` only overrides the plan-alias. It doesn't grant access to premium features without a valid token. You'll still need a real token for most premium functionality.

⚠️ **Testing Real Features**: To test actual premium features (not just plan detection), you'll need a valid license token from the Metabase store.

## Related Documentation

- [Environment Variables](../configuring-metabase/environment-variables.md#mb_dev_license_type)
- [Activating Enterprise Edition](../installation-and-operation/activating-the-enterprise-edition.md)
- [Premium Embedding Token](../configuring-metabase/environment-variables.md#mb_premium_embedding_token)
