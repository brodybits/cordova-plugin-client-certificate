# Cordova Client Certificate authentication support plugin

Plugin that uses a client certificate for authentication, with special `myp12` file association as described below.

This plugin uses iOS implementation and API based on: [`mwaylabs/cordova-plugin-client-certificate`](https://github.com/mwaylabs/cordova-plugin-client-certificate)

This plugin version is known to include code from the following other plugin versions:

- [`EbilPanda/cordova-plugin-client-certificate-ebilpanda`](https://github.com/EbilPanda/cordova-plugin-client-certificate-ebilpanda)
- [`addictic/cordova-plugin-client-certificate-addictic`](https://github.com/addictic/cordova-plugin-client-certificate-addictic)
- [`zxyang/cordova-plugin-clientcertificate`](https://github.com/zxyang/cordova-plugin-clientcertificate) (this version seems to have copied some version of the code from [`johannes-staehlin/cordova-client-cert-authentication`](https://github.com/johannes-staehlin/cordova-client-cert-authentication))
- [`mwaylabs/cordova-plugin-client-certificate`](https://github.com/mwaylabs/cordova-plugin-client-certificate)
- [`johannes-staehlin/cordova-client-cert-authentication`](https://github.com/johannes-staehlin/cordova-client-cert-authentication)

LICENSE: Apache 2.0, with some code for iOS under [Apple MIT License](https://spdx.org/licenses/AML.html)

<!-- ## About: -->

This plugin handles client certificate request on iOS and Android. On iOS this plugin version supports using client certificates from both file association and local file system, as documented below. On Android this plugin supports using client certificates from the key chain, using shared preferences from the preference manager (code from [`johannes-staehlin/cordova-client-cert-authentication`](https://github.com/johannes-staehlin/cordova-client-cert-authentication)).

Specific for iOS:
- This plugin version uses `config-file` elements to configure the app to handle open requests for files with the custom `myp12` extension, by attempting to a register p12 client certificate from such a file with no password, on iOS only.
- This plugin reads the certificate in any folder even in a cordova.file.dataDirectory (r/w folder). (So you can retrieve a cert file from an API and use it)

UPDATES NOT TESTED on Android:

- Android platform implementation was replaced with an implementation similar to johannes-staehlin/cordova-client-cert-authentication, which is not expected to work with JavaScript API at all.

## Usage

## Prerequisites

**Recommended:**

Install Cordova File API plugin (`cordova-plugin-file`), as described below:

    $ cordova plugin add cordova-plugin-file

**Optional:**

Add the following to `config.xml`:

```xml
<platform name="android">
	<preference name="AndroidPersistentFileLocation" value="Internal" />
</platform>
<platform name="ios">
	<preference name="iosPersistentFileLocation" value="Library" />
</platform>
```

## Usage

### Installing the plugin

Create a new Cordova Project

    $ cordova create hello com.example.helloapp Hello

Install the plugin, for example:

    $ cd hello
    $ cordova plugin add https://github.com/mwaylabs/cordova-plugin-client-certificate

Install File API plugin:

    $ cordova plugin add cordova-plugin-file

### Android keychain

Similar to [`johannes-staehlin/cordova-client-cert-authentication`](https://github.com/johannes-staehlin/cordova-client-cert-authentication), as documented in: [`johannes-staehlin/cordova-client-cert-authentication`](https://github.com/johannes-staehlin/cordova-client-cert-authentication)

### Use with file association

**for iOS ONLY:**

This plugin version associates itself with the `myp12` extension in plist on iOS only when it is added to a Cordova app, as specified in `plugin.xml`.

If the user tells another iOS application such as Mail to open a `myp12` file with a Cordova app that uses this plugin version, this plugin will use the certificate from the `myp12` file. No special JavaScript code is required for this file association to function.

### Sample program

**WARNING:** Client certificate should NEVER be included in the `www` folder or any other part of a published app.

#### Quick sample

Copy a client certificate into the `www` folder.

Edit `www/js/index.js` and add the following code inside `onDeviceReady`:

```js
    clientCertificate.registerAuthenticationCertificate("certfilePath/cert.p12", "s3cr37", success, failure);
```

#### With File API

- Copy of an embedded certificate in `www` (read only) to a directory accessible in read/write
- Retrieve the final path and execute the handshake SSL

Add the following code inside `onDeviceReady`:

```js
var certAutomate = 'mycert.p12';
var certFolder = 'certificates/';
var appDir = cordova.file.applicationDirectory + 'www/';
var datDir = cordova.file.dataDirectory;

window.resolveLocalFileSystemURL(datDir, function(DirectoryEntry){
	// Create certFolder if doesn't exists in a r/w location (dataDirectory)
	DirectoryEntry.getDirectory(certFolder, {create: true, exclusive: false}, copyCertificateAutomate, onFailure);
}, onFailure);

var copyCertificateAutomate = function(DirectoryEntry){
	window.resolveLocalFileSystemURL(appDir + certFolder + certAutomate, function(FileEntry){
		window.resolveLocalFileSystemURL(datDir + certFolder + certAutomate, certAuthenticate, function(){
			// Copy the file to the r/w folder if not exists
			FileEntry.copyTo(DirectoryEntry, certAutomate, onSuccess, onFailure);
		});
	}, certAuthenticate);
};

var certAuthenticate = function() {
	// Full path to the cert
	var p12path = datDir.substring(7) + certFolder + certAutomate;
	var p12pass = 'myPassword';

	// Updated API from mwaylabs/cordova-plugin-client-certificate
	clientCertificate.registerAuthenticationCertificate(p12path, p12pass, certificateRegistred, onFailure);
};

var certificateRegistred = function(message) {
	console.log(message);
	// launch your web service that requires certificate authentication here
};

var onSuccess = function(message){
	console.log('Success : ', message);
};

var onFailure = function(message){
	console.log('Error : ', message);
};
```

### Build and run

Install iOS and Android platform

    cordova platform add ios
    cordova platform add android

Run the code

    cordova run android
    cordova run ios

## TODO

- Test updated plugin version on Android and document any possible differences

## For future consideration

- Prompt the user for a p12 certificate password on iOS, if necessary. Native dialog prompt would be ideal; using JavaScript on this plugin or callback to the application code would also be possible.
- Combine with johannes-staehlin/cordova-client-cert-authentication, if possible
- Add Windows platform

## More Info

For more information on setting up Cordova see [the Cordova CLI documentation](https://cordova.apache.org/docs/en/latest/guide/cli/index.html#installing-the-cordova-cli)

For more info on plugins see the [Cordova Plugin Development Guide](https://cordova.apache.org/docs/en/latest/guide/hybrid/plugins/index.html)
