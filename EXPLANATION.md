# Bug Explanation

## What was the bug?

The `Client.request()` method failed to refresh the OAuth2 token when `oauth2_token` was a dictionary instead of an `OAuth2Token` instance. The test `test_api_request_refreshes_when_token_is_dict` expected the Authorization header to be set after refresh, but it was missing.

## Why did it happen?

The conditional logic on line 27-29 only checked two conditions:
1. If `oauth2_token` is falsy (None or empty)
2. If it's an `OAuth2Token` instance AND expired

When `oauth2_token` was a non-empty dict, condition 1 failed (dicts are truthy), and condition 2 failed (not an OAuth2Token instance). The token never refreshed, so no Authorization header was set.

## Why does your fix solve it?

The fix adds an explicit check: `isinstance(self.oauth2_token, dict)`. Now the refresh triggers when the token is:
- Missing (None)
- A dict (legacy format)
- An expired OAuth2Token instance

This ensures dict tokens are always refreshed to proper OAuth2Token objects.

## One realistic case / edge case your tests still don't cover

An expired OAuth2Token with `expires_at` exactly equal to the current timestamp. The `expired` property uses `>=`, so a token expiring at the exact current second would be considered expired, but there's no test verifying this boundary condition works correctly in the refresh logic.
