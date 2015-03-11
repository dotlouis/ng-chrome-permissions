An AngularJS factory to manage (request, remove, check) permissions more easily

# Introduction

The way Chrome handles permissions is not flexible (all or nothing). This module allow to have finer control on which permissions is granted and which is denied.

Besides, it uses callbacks and we all know Promises are here to stay ([and for a reason](http://spion.github.io/posts/why-i-am-switching-to-promises.html)). So this module makes uses of [bluebird's Promise](https://github.com/petkaantonov/bluebird) for you to consume.

Basically ng-chrome-permissions is a convenient wrapper of the chrome.permissions api.

## Installation

*This repo is not yet registered in the bower registry but you can still install it like so:*

1. `bower install virtual-dev/ng-chrome-permissions`
2. Add the script tag to your html `<script src="ng-chrome-permissions.js"></script>`
3. Add the module to your dependencies `angular.module('yourModule', ['ngChromePermissions'])`

## API

- [`.contains(Object permissions)`](#containsobject-permissions---promise)
- [`.request(Object permissions)`](#requestobject-permissions---promise)
- [`.remove(Object permissions)`](#removeobject-permissions---promise)

The permissions Object must be constructed like the original [chrome.permission](https://developer.chrome.com/extensions/permissions) one *(for consistency)*:
```
{
    permissions: Array<String>,
    origins: Array<String>
}
```
with `permissions` and `origins` properties being optionals.

#### Required and Optional permissions

The module handles required and optional permissions so you don't need to worry about checking which permission is required and which one is optional. Just pass-in permissions and you'll get the list of granted/denied permissions.

#### Use requested APIs with Promises

Once you have requested the permissions you need. The module takes care of [promisifying](https://github.com/petkaantonov/bluebird/blob/master/API.md#promisification) the corresponding chrome API. Which mean you can do something like:
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

Check each permissions/origins one by one.
Takes an Object of permissions and/or origins.
Returns a promise which always resolves.
The resulting object contains the granted and/or denied permissions.
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

Check each permissions one by one, then ask the user for the one which are not already granted.
Requesting permissions prompts the user once for all requested permissions.
Takes an Object of permissions and/or origins.
Returns a promise which resolves if the permissions have been granted, and rejects if permissions have been denied.
The resulting object contains the granted and/or denied permissions.
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

Revoke given permissions. Skip the required ones.
Takes an Object of permissions and/or origins.
Returns a promise which always (normally) resolves.
If not, please fill an issue with the permission Object you gave in (arg), the result and the manifest permissions.
The resulting object contains the removed permissions/origins.
```JSON
{
    permissions: Array<String>,
    origins: Array<String>
}
```
*Note: after removing an optional permission, requesting it again won't prompt the user. See this [issue](https://code.google.com/p/chromium/issues/detail?id=122578)*

**Example:**
```Javascript
    ChromePermissions.remove({
        permissions: ['downloads','browsingData']
    }).then(function(permissions){
            // depending on extension's manifest, you may have
            // something like:
            // {
            //     permissions: ['browsingData']
            // }
            //
            // the download permission is not in the returned
            // object because it's not an optional permission
            // and therefore hasn't been removed
    })
```
