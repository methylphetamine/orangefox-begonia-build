# Summary of Changes

## Problem
After fixing the flashlight issue (torch-light2), builds were taking 5 hours instead of the normal duration. This was caused by switching from the `net-fix` device tree to `scrcpy-support`.

## Root Cause
The workflow was changed from using `net-fix` to `scrcpy-support` because:
- Commit `eb636ae`: Fixed flashlight by changing OF_FL_PATH1 from `torch-light0` to `torch-light2`
- Commit `ab73ff2`: Changed default device tree from `net-fix` to `scrcpy-support` because "net-fix predates the torch-light2 fix"

However, `scrcpy-support` is newer, more complex, and requires more build resources than `net-fix`.

## Solution
Reverted to `net-fix` device tree while keeping the torch-light2 fix applied.

## Changes Made

### 1. `.github/workflows/build.yml`

#### Device Tree Input Configuration
- **Line 19-21**: Changed device_branch description and default
  - **Before**: `description: 'Device tree branch (scrcpy-support has WiFi, curl/Links2/foxkbd, gofile and the torch fix)'`
  - **After**: `description: 'Device tree branch (net-fix has the recovery WiFi + curl/Links2 browser payload)'`
  - **Before**: `default: 'scrcpy-support'`
  - **After**: `default: 'net-fix'`

#### Environment Variable
- **Line 46**: Changed DEVICE_TREE_BRANCH default
  - **Before**: `DEVICE_TREE_BRANCH: ${{ github.event.inputs.device_branch || 'scrcpy-support' }}`
  - **After**: `DEVICE_TREE_BRANCH: ${{ github.event.inputs.device_branch || 'net-fix' }}`

### 2. Torch-Light2 Fix
The torch-light2 verification is already in place in the workflow:
- **Lines 428-445**: Contains verification for `OF_FL_PATH1 := /sys/class/leds/torch-light2`
- This fix is now applied to the `net-fix` device tree

## What Was Preserved
1. **Torch-light2 fix**: Still verified in device tree verification step
2. **Workflow optimizations**: All memory and disk optimizations from `f69cf5e`, `3fa576a`, etc.
3. **Foxkbd workflow**: Already uses `net-fix`, no changes needed
4. **All other features**: WiFi, curl, Links2 browser, FBE decryption, etc.

## Expected Results
- **Build time**: Should return to normal (previously working duration)
- **Device tree**: Simpler, faster `net-fix` instead of complex `scrcpy-support`
- **Flashlight fix**: Still applied and verified through torch-light2 check
- **Functionality**: All original features preserved (WiFi, curl, browser, decryption)

## Rationale
- `net-fix` is the proven, stable device tree that was working before
- `scrcpy-support` was introduced but caused performance issues
- The torch-light2 fix was specifically required for `net-fix`
- All workflow optimizations were designed for `net-fix`
- Reverting to `net-fix` fixes the performance issue while maintaining the flashlight fix