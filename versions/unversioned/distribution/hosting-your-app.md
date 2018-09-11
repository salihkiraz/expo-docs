---
title: Hosting An App On Your Servers
---
Normally, when over-the-air (OTA) updates are enabled, your app will fetch JS bundles and assets from Expo’s CDN. However, there will be situations when you will want to host your JS bundles and assets on your own servers. For example, OTA updates are unusable in countries that have blocked Expo’s CDN providers. In other countries, users experience high latency when fetching from Expo’s CDNs. In these cases, you can host your app on your own servers to better suit your usecase. 

For simplicity, the rest of this article will refer to hosting an app for the android platform, but you could swap out android for ios at any point and everything would still be true.

## Export app

First, you’ll need to export all the static files of your app so they can be served from your CDN. To do this, run `exp export` in your project directory and it will output all your app’s static files to a directory named `dist`.  Your output directory should look something like this now:
```
.
├── android-index.json
├── ios-index.json
├── assets
│   └── 1eccbc4c41d49fd81840aef3eaabe862
└── bundles
      ├── android-01ee6e3ab3e8c16a4d926c91808d5320.js
      └── ios-ee8206cc754d3f7aa9123b7f909d94ea.js
```

## Build standalone app

In order to configure your standalone binary to pull OTA updates from your server, you’ll need to pass the appropriate URL to the hosted `index.json` in the `exp build` command.

For iOS builds, run the following commands from terminal:
`exp build:ios --public-url <path-to-ios-index.json>`  
Where the public-url parameter will be something like `https://quinlanj.github.io/self-host/ios-index.json`

For Android builds, run the following commands from terminal:
`exp build:android --public-url <path-to-android-index.json>`
Where the public-url parameter will be something like `https://quinlanj.github.io/self-host/android-index.json`


## Loading QR Code/URL in Development

You can also load a self hosted app as a QR code/URL into the Expo mobile client for development purposes. 

### QR code:
The URI you’ll use to convert to QR code, will be deeplinked using the `exps/exp` protocol. Both `exps` and `exp` deeplink into the mobile app and performs a request using HTTPS and HTTP respectively. You can create your own QR code using an online QR code generator from the input URI.

#### Here’s an example of how you’d do this with a remote server:

URI: `exps://quinlanj.github.io/self-host/android-index.json`

QR code: Generate the URI from a website like https://www.qr-code-generator.com/

#### Here’s an example of how you’d do this from localhost:

Run `exp export` in dev mode and then start a simple HTTP server in your output directory:

```
# export static app files
exp export --dev

# cd into your output directory
cd dist

# run a simple http server from output directory
python -m SimpleHTTPServer 8000
```

URI: `exp://localhost:8000/android-index.json`

QR code: Generate a QR code using your URI from a website like https://www.qr-code-generator.com/

### URL
If you are loading in your app into the expo client by passing in a URL string, you will need to pass in an URL pointing to your json file.

Here is an example URL from a remote server:
`https://quinlanj.github.io/self-host/android-index.json`

Here is an example URL from localhost:
`http://localhost:8000/android-index.json`

## Advanced Topics
### Debugging
When we bundle your app, minification is always enabled. In order to see the original source code of your app for debugging purposes, you can generate source maps. Here is an example workflow:

1. Run `exp export --dump-sourcemap`. This will also export your bundle sourcemaps in the `bundles` directory.
2. A `debug.html` file will also be created at the root of your output directory.
3. In Chrome, open up `debug.html` and navigate to the `Source` tab. In the left tab there should be a resource explorer with a red folder containing the reconstructed source code from your bundle. 

[![Debugging Source Code](./host-your-app-debug.png)](/_images/host-your-app-debug.png)

### Multimanifests
#### TODO(quin) this was released after the XDL release. Put this section in unversioned.

As new SDK versions are released, you may want to serve multiple versions of your app from your server endpoint. For example, if you first released your app with SDK 29 and later upgraded to SDK 30, you'd want users with your old standalone binary to receive the SDK 29 version, and those with the new standalone binary to receive the SDK 30 version.  

In order to do this, you can run `exp export:merge` to merge previously exported apps into a single multiversion app which you can serve from your servers.

Here is what the workflow would look like:

1. Release your app with SDK 29
2. Run `exp export --output-dir sdk29`. This exports the sdk 29 version of the app in the `sdk29` directory. 
3. Release your app with SDK 30
4. Run `exp export --output-dir sdk30`. This exports the sdk 30 version of the app in the `sdk30` directory. 
5. Merge into a single multiversion app by running `exp export:merge --source-dir sdk29 --source-dir sdk30`. This dumps a multiversion app into the `dist` output directory. The asset and bundle folders contain everything that the source directories had. The `index.json` files are an array of the the indexes found in the source directories.


### Asset Hosting
By default, all assets are hosted from an `assets` path resolving from your `public-url` (ie) `https://quinlanj.github.io/self-host/assets`.  You can override this behavior in the `assetUrlOverride` field of your `android-index.json`. All relative URL's will be resolved from the `public-url`.

### Special fields
Most of the fields in the `index.json` files are the same as in `app.json`. Here are some fields that are notable in `index.json`:

In index.json:
- `revisionId, commitTime, publishedTime`: These fields are generated by `exp export` and used to determine whether or not an OTA update should occurr. 
- `bundleUrl`: This points to the path where the app's bundles are hosted. They are also used to determined whether or not an OTA update should occurr. 
- `slug`: This should not be changed. Your app is namespaced by `slug`, and changing this field will result in undefined behavior in the Expo SDK components (ie) FileSytem
- `assetUrlOverride`: The path which assets are hosted from. It is by default `./assets`, which resolves with the base publicUrl that is initially passed in. 