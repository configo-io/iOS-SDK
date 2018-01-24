# Change Log
All changes to the SDK will be documented in this file.
This project follows the [Semantic Versioning](http://semver.org) style.

## [1.10.1] - 2017-01-24
### Fix
- Edge case where data is not updated to SDK consumer

## [1.10.0] - 2017-01-16
### Rename
- `-presentPopupWithName:callback:` to `-presentCampaignWithName:callback:`
- block type `CFGPopupCallback` to `CFGCampaignCallback`

### Fix
- Cordova relative url open now uses `window.location = <href>`


## [1.9.12] - 2017-12-28
### Fix
- View / Element data collection for better view binding


## [1.9.11] - 2017-12-25
### Fix
- Feature disabled for UITableViewCell user interaction
- Campaign callbacks are called with a URL like argument describing the action

### Change
- View binding comparison algorithm


## [1.9.10] - 2017-12-20
### Add
- Public setters logs

### Fix
- Some campaigns are not clickable at times.
- Fix campaign capping time between when more than session view capping is more than 1.

## [1.9.9] - 2017-12-18
### Add
- Public errors constants file "CFGErrors.h"

### Fix
- Campaign triggers now work when feature is of type "disabled" and toggled off.

## [1.9.8] - 2017-12-14
### Added
- Feature type: hidden/disabled support
- Tooltip support
- Survey support

### Fix
- Sometimes a campaign was not launched after previous camapaign was presented

### Move
- `typedef` declarations are now in their own file
