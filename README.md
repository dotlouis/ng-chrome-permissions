An AngularJS factory to manage (request, remove, check) permissions more easily

# Introduction

The way Chrome handles permissions is not flexible (all or nothing). This module allows to have finer control over which permission is granted and which is denied.

Besides, the module makes uses of [bluebird's Promises](https://github.com/petkaantonov/bluebird) for you to consume instead of usual callbacks.

Basically ng-chrome-permissions is a convenient wrapper of the *chrome.permissions* api.

## Installation

*This repo is not yet registered in the bower registry but you can still install it like so:*

1. `bower install virtual-dev/ng-chrome-permissions`
2. Add the script tag to your html `<script src="ng-chrome-permissions.js"></script>`
3. Add the module to your dependencies `angular.module('yourModule', ['ngChromePermissions'])`

## API

- [`.contains(Object permissions)`](#containsobject-permissions---promise)
- [`.request(Object permissions)`](#requestobject-permissions---promise)
- [`.remove(Object permissions)`](#removeobject-permissions---promise)

The permissions Object must be constructed like the original [chrome.permission](https://developer.chrome.com/extensions/permissions#type-Permissions) one *(for consistency)*:
```
{
    permissions: Array<String>,
    origins: Array<String>
}
```
with `permissions` and `origins` properties being optionals.

The resulting object of these methods' Promise is constructed this way:
```
{
    granted:{
        permissions: Array<String>,
        origins: Array<String>
    },
    denied: {
        permissions: Array<String>,
        origins: Array<String>
    }
}
```

#### Required and Optional permissions

The module handles required and optional permissions so you don't need to worry about checking which permission is required and which one is optional. Just pass-in permissions and you'll get the list of granted/denied permissions.

#### Use requested APIs with Promises

Once you have requested the permissions you need. The module takes care of [promisifying](https://github.com/petkaantonov/bluebird/blob/master/API.md#promisification) the corresponding chrome API. Which means you can do something like:
```Javascript
ChromePermissions.request({permissions:['bookmarks']})
.then(function(){
    return chrome.bookmarks.getRecentAsync(5);
})
.then(function(bookmarks){
    // do something with bookmarks
});
```
*Note: The Async suffix of .getRecentAsync() comes from promisification. see [bluebird's API](https://github.com/petkaantonov/bluebird/blob/master/API.md#promisification) for more info*


#### `.contains(Object permissions)` -> `Promise`

Checks each permissions/origins one by one.
Takes an Object of permissions and/or origins.
Returns a promise which always resolves.
The resulting object contains the granted and/or denied permissions.

**Example:**
```Javascript
    ChromePermissions.contains({
        permissions: ['bookmarks','downloads'],
        origins: ['https://google.com']
    }).then(function(permissions){
            // depending on extension's manifest, you may have
            // something like:
            // {
            //     granted: {
            //         permissions: ['downloads'],
            //         origins: ['https://google.com']
            //     },
            //     denied:{
            //         permissions: ['bookmarks']
            //     }
            // }
    })
```

#### `.request(Object permissions)` -> `Promise`

Checks each permissions one by one, then asks the user for the one which are not already granted.
Requesting permissions prompts the user once for all requested permissions.
Takes an Object of permissions and/or origins.
Returns a promise which resolves if the permissions have been granted, and rejects if permissions have been denied.
The resulting object contains the granted and/or denied permissions.

**Example:**
```Javascript
    ChromePermissions.request({
        permissions: ['management','browsingData']
    }).then(function(permissions){
            // depending on extension's manifest, you may have
            // something like:
            // {
            //     denied:{
            //         permissions: ['management','browsingData']
            //     }
            // }
    })
```

#### `.remove(Object permissions)` -> `Promise`

Revokes given permissions. Skips the required ones.
Takes an Object of permissions and/or origins.
Returns a promise which always (normally) resolves.
*If not, please fill an issue with the permission Object you provided as argument, the resulting object and the manifest permissions.*
The resulting object contains the granted and/or denied permissions.

You can think of it this way:

The returned denied permissions are the one that have been successfully removed,
the granted ones are the one that have not been removed because they are required. You can see a list of optional permissions [here](https://developer.chrome.com/extensions/permissions)

*Note: after removing an optional permission, requesting it again won't prompt the user. See this [issue](https://code.google.com/p/chromium/issues/detail?id=122578)*

**Example:**
```Javascript
    ChromePermissions.remove({
        permissions: ['downloads','history']
    }).then(function(permissions){
            // permissions =
            // {
            //     denied:{
            //         permissions: ['history']
            //     },
            //     granted:{
            //         permissions: ['downloads']
            //     }
            // }
            // the download permission is not denied
            // because it's not an optional permission
            // and therefore hasn't been removed
    })
```
