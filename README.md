# 📂 Nextcloud WebDav file picker

![pages](https://github.com/eneiluj/nextcloud-webdav-filepicker/workflows/build-pages/badge.svg?branch=master&event=push)

The Nextcloud WebDav file picker is a web component to access Nextcloud files.

It requires the [WebAppPassword](https://apps.nextcloud.com/apps/webapppassword) to be installed on the target Nextcloud server.

## Demo

* [Example of Vue application using the component](https://eneiluj.github.io/nextcloud-webdav-filepicker/examples/with-vue.html)
* [Example of simple script using the file picker wrapper](https://eneiluj.github.io/nextcloud-webdav-filepicker/examples/without-vue.html)

## Intro

### Features

This file picker is able to

* select multiple files and:
	* get their path
	* download them as [Files](https://developer.mozilla.org/en-US/docs/Web/API/File)
	* generate WebDav download links
	* help you to easily generate Nextcloud share links
* select a target directory and:
	* get its path
	* generate a WebDav upload link
* upload local files

### Authentication

🔒 Supported authentication methods are:

* Provide a login and:
	* a classic password
	* an app password
	* an OAuth bearer token
* Let the file picker authenticate on its own with the web login flow

⚠ A login is still required if you want to use an OAuth token.

### How to include it

There are two ways to include this file picker in your web application:

* A Vue.js component
* A wrapper script

The file picker can optionally show buttons to open it an perform actions. You can also use your own custom elements (anywhere in your application) to trigger the file picker actions.

## The wrapper

Once you've imported `filePickerWrapper.js` you can call the `window.createFilePicker()` function
to mount the file picker somewhere in your web page. This function returns the component to let you interact with it later.

Parameters of `createFilePicker(mountPoint, url, login, password, accessToken, color, multipleDownload, multipleUpload, enableGetFilesPath, enableGetFilesLink, enableDownloadFiles, enableGetSaveFilePath, enableGetUploadFileLink, enableUploadFiles)` function:

* mountPoint (String): the ID of the element in which the file picker is mounted
* url (string, mandatory): the Nextcloud base URL
* login (string): the user name
* password (string): the user password, an app password or an OAuth access token
* accessToken (string): an OAuth token (use this parameter if you absolutely want to use HTTP Authorization header to authenticate. Using the OAuth token as a password is recommended)
* color (hex color string): the main file picker color (default: Nextcloud blue, `#0082c9`)
* multipleDownload (boolean): let the user select multiple files in the file picker (default: `true`)
* multipleUpload (boolean): let the user select multiple local files to upload (default: `true`)
* enableGetFilesPath (boolean): show the "Get files path" button (default: `true`)
* enableGetFilesLink (boolean): show the "Get files link" button (default: `true`)
* enableDownloadFiles (boolean): show the "Get files link" button (default: `true`)
* enableGetSaveFilePath (boolean): show the "Get files link" button (default: `true`)
* enableGetUploadFileLink (boolean): show the "Get files link" button (default: `true`)
* enableUploadFiles (boolean): show the "Upload files" button (default: `true`)

Set login and password/accessToken to `null` to let the file picker authenticate through the web login flow and get an app password by itself.

### Example

Here is a minimal example getting files paths and displaying them in the console:

``` html
<!-- The component will be mounted there -->
<div id="mount_point"></div>
<!-- Import the wrapper -->
<script src="filePickerWrapper.js"></script>
<script>
	// when the page is loaded
	document.addEventListener('DOMContentLoaded', (event) => {
		// create and mount the component
		const filepicker = window.createFilePicker('mount_point', 'https://my.nextcloud.org')

		// listen to events coming from the component
		document.addEventListener('get-files-path', (e) => {
			console.debug('received "get-files-path" event')
			console.debug(e.detail)
			e.detail.forEach((path) => {
				console.debug(path)
			})
		})
	})
</script>
```

### Events

Here are the events emitted by the component and the data they provide:

* `get-files-path`: an array of file paths
* `files-downloaded`: `successFiles` and `errorFiles`, arrays of [Files](https://developer.mozilla.org/en-US/docs/Web/API/File)
* `get-files-link`:
	* `webdavLinks` an array of Webdav download links
	* `pathList` list of selected paths
	* `ocsUrl` OCS API URL to [create Nextcloud share links](#create-nextcloud-share-links)
* `get-save-file-path`: the path of the selected target directory
* `upload-path-link-generated`: a WebDav upload link
* `files-uploaded`: `successFiles` and `errorFiles`, arrays of uploaded [Files](https://developer.mozilla.org/en-US/docs/Web/API/File)

## The Vue component

Get it from NPM (⚠ **not yet available**):
```
npm install nextcloud-webdav-filepicker
```

And use it:
```
import NcFilePicker from 'nextcloud-webdav-filepicker'
```

### Example

Here is a minimal example of a Vue.js app using the file picker:
``` vue
<template>
	<div id="example-app">
		<h2>My amazing app</h2>
		<NcFilePicker
			:nc-url="ncUrl"
			@get-files-path="onGetFilesPath">
		</NcFilePicker>
	</div>
</template>

<script>
import NcFilePicker from 'nextcloud-webdav-filepicker'

export default {
	name: 'ExampleApp',

	components: { NcFilePicker },

	props: [],

	data() {
		return {
			ncUrl: 'https://my.nextcloud.org',
		}
	},

	computed: {
	},

	methods: {
		onGetFilesPath(pathList) {
			console.debug('files were selected')
			console.debug(pathList)
		},
	},
}
</script>
```

### Props

``` javascript
/* === reactive props === */
// Nextcloud base URL
ncUrl: {
	type: String,
	required: true,
},
// Nextcloud user name
ncLogin: {
	type: String,
	default: '',
},
// Nextcloud user password/app password/OAuth access token
ncPassword: {
	type: String,
	default: '',
},
// OAuth access token if you absolutely want to use Bearer Authorization header (if not, using a token as a password works fine)
ncAccessToken: {
	type: String,
	default: '',
},
/* === props to control the fp component from the parent one === */
// file picker mode to determine what is done when the picker is opened
pickerMode: {
	type: String,
	default: '',
},
// prop to open the file picker if you don't want to use the buttons
pickerIsOpen: {
	type: Boolean,
	default: false,
},
/* === options === */
// enable multiple selection in all download modes
multipleDownload: {
	type: Boolean,
	default: true,
},
// enable multiple local files selection when uploading
multipleUpload: {
	type: Boolean,
	default: true,
},
// file picker title
getTitle: {
	type: String,
	default: null,
},
putTitle: {
	type: String,
	default: null,
},
// theming (reactive too)
themeColor: {
	type: String,
	default: '#0082c9',
	validator: (value) => {
		return value.match(/^#[0-9a-fA-F]{6}$/)
	},
},
/* === toggle buttons === */
// display the button to get files path
enableGetFilesPath: {
	type: Boolean,
	default: true,
},
// display the button to get files links
enableGetFilesLink: {
	type: Boolean,
	default: true,
},
// display the button to download files
enableDownloadFiles: {
	type: Boolean,
	default: true,
},
// display the button to get a save file path
enableGetSaveFilePath: {
	type: Boolean,
	default: true,
},
// display the button to get webdav upload link
enableGetUploadFileLink: {
	type: Boolean,
	default: true,
},
// display the button to upload local files
enableUploadFiles: {
	type: Boolean,
	default: true,
},
```

### Slots

There is a slot for each button that triggers a file picker action:

* get-files-path
* get-files-link
* download-files
* get-save-file-path
* get-upload-fileLink
* open-file-input

The click event is catched by the file picker component, no need to listen to it. So you can put whatever you want in those slots, a click anywhere in the slot will open the file picker just like if the default button was clicked.

## Extra

### Create Nextcloud share links

As long as CORS headers can't be changed to allow extra origins (like it's done with WebDav endpoints in WebAppPassword), the browser can't create new share links.
You can still do it anywhere else, on the server side of your web application for example. The `get-files-link` event provides a share link template and the OCS URL to create such share links. The OCS API endpoint looks like `https://my.nextcloud.org/ocs/v2.php/apps/files_sharing/api/v1/shares`.

Here is an example of link creation:

```
curl -H "OCS-APIRequest: true" -u login:token -X POST -d "path=/path/to/file&shareType=3" "$OCS_URL"
```

This will create and return a share link (shareType=3) with default permissions. The share token can be found in `ocs.data.token` of the JSON response. Then just place the token in the share link template. For example, if `get-files-link` gave you `https://my.nextcloud.org/index.php/s/TOKEN` as share link template and the token of the link you created is `wHx2BteGayciKiA`, then the share link is `https://my.nextcloud.org/index.php/s/wHx2BteGayciKiA`.