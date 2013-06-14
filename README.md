Simple Ajax Uploader
============================

A Javascript plugin for cross-browser Ajax file uploading. Supports multiple file uploading with progress bars.

Live demo: http://www.lpology.com/code/ajaxuploader/

### Overview ###
Simple Ajax Uploader allows developers to easily add Ajax file upload functionality to web applications. It abstracts away standard tasks and browser compatibility issues while preserving wide latitude for custom use.

The project began as a rewrite of Andrew Valum's original Ajax Upload plugin. The goal of the project is make file uploading easy for developers and pleasant for users.

### Quick Start Example ###
Include SimpleAjaxUploader.js into your page, and initialize the uploader when the DOM is ready:

```javascript

var uploader = new ss.SimpleUpload({
	button: 'upload-btn', // HTML element used as upload button
	url: '/PathTo/UploadHandler', // URL of server-side upload handler
	name: 'uploadfile' // Parameter name of the uploaded file
});
```

### Features ###
* Uses XMLHttpRequest and HTML5 File API with fall back to iframe method for Internet Explorer 9 and older.
* Supports multiple, concurrent file uploads - even in non-HTML5 browsers.
* No external libraries required - use it with or without jQuery.
* Works in all major browsers: IE7+, Firefox 4+, Safari 4+, Chrome, and Opera.
* Fast and lightweight - only 5KB, minified and gzipped.
* Built-in support for implementing truly cross-browser progress bars.
* Use any HTML element as the upload button.
* Provides individual callback functions for XHR-supported browsers and for browsers that do not support XHR uploads.

### How to Use ###
There are two main ways to use the plugin:

<strong>1. Single file uploading</strong> - Only one upload allowed at a time. Progress bar is an element that is re-used for each upload.<br />
<strong>2. Multiple file uploading</strong> - Allow multiple, concurrent file uploads. Progress bars are created on the fly before each upload.

#### Method 1: Single file uploading (one file at a time)  ####

This method uses static, on-page elements for the progress bar.

```javascript
  var sizeBox = document.getElementById('sizeBox'), // HTML element already on page
      progressBar = document.getElementById('progressBar'); // HTML element already on page
  
	var uploader = new ss.SimpleUpload({
        button: 'uploadButton', // file upload button
        url: 'uploadHandler.php', // server side handler
        name: 'uploadfile', // upload parameter name        
        progressUrl: 'uploadProgress.php', // enables cross-browser progress support (more info below)
        responseType: 'json',
        allowedExtensions: ['jpg', 'jpeg', 'png', 'gif'],
        maxSize: 500, // kilobytes
        hoverClass: 'ui-state-hover',
        focusClass: 'ui-state-focus',
        disabledClass: 'ui-state-disabled',
        onSubmit: function(filename, extension) {
            this.setFileSizeBox(sizeBox); // designate this element as file size container
            this.setProgressBar(progressBar); // designate as progress bar
          },         
        onComplete: function(filename, response) {
            if (!response) {
                alert(filename + 'upload failed');
                return false;            
            }
            // do something with response...
          }
	});        
```        

Before each upload, our two elements are assigned specific roles using these two functions:<br />
`setProgressBar(elem)` - Designates an element as the progress bar for an upload.<br />
`setFileSizeBox(elem)` - Designates an element as the container in which the file size of an uploading file will be inserted.

#### Method 2: Multiple file uploads ####

Below is an example of how to implement multiple file uploading with progress bars. A new progress bar is created for each file upload within the `onSubmit()` callback function. The progress bar is then removed when the upload is completed. 
 
```javascript  
	var uploader = new ss.SimpleUpload({
        button: 'uploadButton',
        url: 'uploadHandler.php', // server side handler
        progressUrl: 'uploadProgress.php', // enables cross-browser progress support (more info below)
        responseType: 'json',
        name: 'uploadfile',
        multiple: true,
        allowedExtensions: ['jpg', 'jpeg', 'png', 'gif'], // for example, if we were uploading pics
        hoverClass: 'ui-state-hover',
        focusClass: 'ui-state-focus',
        disabledClass: 'ui-state-disabled',   
        onSubmit: function(filename, extension) {
            // Create the elements of our progress bar
            var progress = document.createElement('div'), // container for progress bar
                bar = document.createElement('div'), // actual progress bar
                fileSize = document.createElement('div'), // container for upload file size
                wrapper = document.createElement('div'), // container for this progress bar
                progressBox = document.getElementById('progressBox'); // on page container for progress bars
            
            // Assign each element its corresponding class
            progress.className = 'progress';
            bar.className = 'bar';            
            fileSize.className = 'size';
            wrapper.className = 'wrapper';
            
            // Assemble the progress bar and add it to the page
            progress.appendChild(bar); 
            wrapper.innerHTML = '<div class="name">'+filename+'</div>'; // filename is passed to onSubmit()
            wrapper.appendChild(fileSize);
            wrapper.appendChild(progress);                                       
            progressBox.appendChild(wrapper); // just an element on the page to hold the progress bars    
            
            // Assign roles to the elements of the progress bar
            this.setProgressBar(bar); // will serve as the actual progress bar
            this.setFileSizeBox(fileSize); // display file size beside progress bar
            this.setProgressContainer(wrapper); // designate the containing div to be removed after upload
          },
          
         // Do something after finishing the upload
         // Note that the progress bar will be automatically removed upon completion because everything 
         // is encased in the "wrapper", which was designated to be removed with setProgressContainer() 
        onComplete:	function(filename, response) {
            if (!response) {
              alert(filename + 'upload failed');
              return false;
            }
            // Stuff to do after finishing an upload...
          }
	});
```

When a file is uploaded, a progress bar is dynamically created, the elements of which are assigned specific roles using these three functions:

`setProgressBar(elem)` - Designates an element as the progress bar for an upload. 

`setFileSizeBox(elem)` - Designates an element as the container in which the file size of an uploading file will be inserted.

`setProgressContainer(elem)` - Designates an element that is to be removed from the DOM after the upload finishes. 

### Cross-browser Progress Support - How it Works ###

When the plugin detects browsers that support the HTML5 File API, the `progress` event is used. For other browsers (i.e., IE9 and below), the plugin will instead retrieve progress updates from the server, which are provided by uploadProgress.php (included in extras folder). This requires PHP with the APC extension installed and the `apc.rfc1867` option enabled (instructions below).

In both cases, everything is handled internally - feature detection, calculation, key handling, etc. To enable this behavior, just provide the URL for uploadProgress.php in the `progressUrl` option.

#### Installing the APC extension ####
```
sudo pecl install apc
```

Accept the default settings, and then create a configuration file:

```
sudo vi /etc/php.d/apc.ini
```

Add these two lines, and then save:

```
extension=apc.so
apc.rfc1867 = 1
```

Restart your web server for the changes to take effect. Note that if APC is already installed, you may still need to add `apc.rfc1867 = 1` to apc.ini, as it is not enabled by default.

### Using Uploader.php ###

<strong>Note:</strong> This PHP class is included only for convenience. <strong>It is not required to use PHP with Simple Ajax Uploader.</strong> The plugin is agnostic to server configuration, so use any language you prefer.

```php
<?php
require('Uploader.php');

$upload_dir = '/img_uploads/';
$valid_extensions = array('gif', 'png', 'jpeg', 'jpg');

$Upload = new FileUpload('uploadfile');
$result = $Upload->handleUpload($upload_dir, $valid_extensions);

if (!$result) {
	echo json_encode(array('success' => false, 'msg' => $Upload->getErrorMsg()));	
} else {
	echo json_encode(array('success' => true, 'file' => $Upload->getFileName()));
}
```

You can also save the uploaded file with a different name by setting the `newFileName` property:

```php
$Upload = new FileUpload('uploadfile');
$ext = $Upload->getExtension(); // Get the extension of the uploaded file
$Upload->newFileName = 'customFileName.'.$ext;
$result = $Upload->handleUpload($upload_dir, $valid_extensions);
```

To access the newly uploaded file, use the `getSavedFile()` method to get the file's path after the upload is completed:
```php
$Upload = new FileUpload('uploadfile');
$result = $Upload->handleUpload($upload_dir, $valid_extensions);

if ($result) {
  $path = $Upload->getSavedFile();
  $imgsize = getimagesize($path);
  // image resizing stuff...
}
```

### API Reference - General ###

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><strong>button</strong><br />Default: <code>''</code></td>
            <td>String, Element</td>
            <td>File upload button. <strong>Required.</strong></td>
        </tr>
        <tr>
            <td><strong>url</strong><br />Default: <code>''</code></td>
            <td>String</td>
            <td>Location of the server-side file upload handler. <strong>Required.</strong></td>
        </tr>		
        <tr>
            <td><strong>name</strong><br />Default: <code>''</code></td>
            <td>String</td>
            <td>Upload parameter name. <strong>Required.</strong></td>
        </tr>
        <tr>
            <td><strong>progressUrl</strong><br />Default: <code>''</code></td>
            <td>String</td>
            <td>Set to the location of uploadProgress.php (included) to enable cross-browser upload progress tracking (see example above).</td>
        </tr>	                         
        <tr>
            <td><strong>data</strong><br />Default: <code>{}</code></td>
            <td>Object</td>
            <td>Additional data to be sent to the server.</td>
        </tr>
       
        <tr>
            <td><strong>multipart</strong><br />Default: <code>false</code></td>
            <td>Boolean</td>
            <td>Set to <code>true</code> for all files to be uploaded using multipart form method instead of direct binary stream.</td>
        </tr>        
        <tr>
            <td><strong>autoSubmit</strong><br />Default: <code>true</code></td>
            <td>Boolean</td>
            <td>By default, uploads commence as soon as a file is selected. Set to <code>false</code> to delay the upload and trigger manually.</td>
        </tr>
        <tr>
            <td><strong>responseType</strong><br />Default: <code>text</code></td>
            <td>String</td>
            <td>The type of data you're expecting back from the server. Default is plain text. Additional option is 'json'.</td>
        </tr>		
        <tr>
            <td><strong>debug</strong><br />Default: <code>false</code></td>
            <td>Boolean</td>
            <td>Setting <code>debug</code> to <code>true</code> will log progress messages and server response in the console.</td>
        </tr>		
        <tr>
            <td><strong>hoverClass</strong><br />Default: <code>''</code></td>
            <td>String</td>
            <td>Class applied to upload button when mouse is hovered.</td>
        </tr>		
        <tr>
            <td><strong>focusClass</strong><br />Default: <code>''</code></td>
            <td>String</td>
            <td>Class applied to upload button when focused.</td>
        </tr>	
        <tr>
            <td><strong>disabledClass</strong><br />Default: <code>''</code></td>
            <td>String</td>
            <td>Class applied to button when disabled.</td>
        </tr>		
        <tr>
            <td><strong>onChange(filename, extension)</strong></td>
            <td>Function</td>
            <td>Function to be called when user selects a file. The function gets passed two arguments: a string containing the filename; a string containing the file extension.</td>
        </tr>
        <tr>
            <td><strong>onSubmit(filename, extension)</strong></td>
            <td>Function</td>
            <td>Function to be called before file is uploaded. The function gets passed two arguments: a string containing the filename; a string containing the file extension. Return false stops upload.</td>
        </tr>
        <tr>
            <td><strong>onProgress(pct)</strong></td>
            <td>Function</td>
            <td>Function to be called on the progress event for browsers that support XHR uploads. The function gets passed one argument: an integer representing the upload completion percentage.</td>
        </tr>
        <tr>
            <td><strong>onComplete(filename, response)</strong></td>
            <td>Function</td>
            <td>Function to be called when the upload is completed. The function gets passed two parameters: a string containing the filename; the data returned from the server, formatted according to the <code>responseType</code> setting. If <code>responseType</code> is 'json', the response will be evaluated as JSON and will return a Javascript object.</td>
        </tr>
        <tr>
            <td><strong>onError(filename, errorType, response)</strong></td>
            <td>Function</td>
            <td>Function to be called if an error occurs during upload. The function gets passsed three parameters: a string containing the filename; a string containing the error type; a string containing the server response, if any.</td>
        </tr>
        <tr>
            <td><strong>startXHR(filename, fileSize)</strong></td>
            <td>Function</td>
            <td>Function to be called only in browsers that support XHR uploads (non-IE). Executes after <code>onSubmit</code> but prior to upload start. The function gets passed two arguments: a string containing the filename; a number that is the file size in kilobytes. Return false stops upload.</td>
        </tr>
        <tr>
            <td><strong>endXHR(filename)</strong></td>
            <td>Function</td>
            <td>Function to be called only in browsers that support XHR uploads (non-IE). Executes after upload is completed but prior to <code>onComplete</code>. The function gets passed one argument: a string containing the filename.</td>
        </tr>
        <tr>
            <td><strong>startNonXHR(filename)</strong></td>
            <td>Function</td>
            <td>Function to be called only in browsers that do not support XHR uploads (Internet Explorer). Executes after <code>onSubmit</code> but prior to upload start. The function gets passed one argument: a string containing the filename. Return false stops upload.</td>
        </tr>
        <tr>
            <td><strong>endNonXHR(filename)</strong></td>
            <td>Function</td>
            <td>Function to be called only in browsers that do not support XHR uploads (Internet Explorer). Executes after upload is completed but prior to <code>onComplete</code>. The function gets passed one argument: a string containing the filename.</td>
        </tr>		
	</tbody>
</table>

### API Reference - Multi-file uploading ###

The three functions below -- <code>setProgressBar()</code>, <code>setFileSizeBox()</code>, and <code>setProgressContainer</code>, were written specifically to support multiple, concurrent progress bars. See above for instructions and examples.


<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
      <tr>
        <td><strong>multiple</strong><br />Default: <code>false</code></td>
        <td>Boolean</td>
        <td>Set to <code>true</code> to enable multiple, concurrent file uploads.</td>    
      </tr>    
      <tr>
        <td><strong>maxUploads</strong><br />Default: <code>3</code></td>
        <td>Integer</td>
        <td>Max number of simultaneous uploads.</td>    
      </tr>       
      <tr>
        <td><strong>setProgressBar(element)</strong></td>
        <td>Function</td>
        <td>Designates an element to serve as an upload progress bar. The CSS width percentage of the element will be updated as the upload progresses.</td>    
      </tr>
      <tr>
        <td><strong>setFileSizeBox(element)</strong></td>
        <td>Function</td>
        <td>Designates an element as the container in which the file size of an uploading file will be inserted.</td>    
      </tr>      
      <tr>
        <td><strong>setProgressContainer(element)</strong></td>
        <td>Function</td>
        <td>Designates an element to be removed from the DOM upon completion of an upload. Useful for cleaning up dynamic progress bars.</td>   
      </tr>       
    </tbody>  
</table>

### API Reference - User experience ###

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>allowedExtensions</strong><br />Default: <code>[]</code></td>
      <td>Array</td>
      <td>
      Only allow file uploading for these extensions (case insensitive). Ex: 
      <code>allowedExtensions: ['jpg', 'jpeg', 'png', 'gif']</code>
      </td>
    </tr> 
    <tr>
      <td><strong>maxSize</strong><br />Default: <code>false</code></td>
      <td>Integer</td>
      <td>Maximum allowed file size (in kilobytes). Only works in browsers that support HTML5 File API.
      </td>
     </tr>    
     
    <tr>
      <td><strong>messages</strong><br />Default: </td>
      <td>Object</td>
      <td>Provide a custom error message for display when a user selects the wrong file.
      </td>
     </tr>     
     
  </tbody>
</table> 


### API Reference - Cross-browser progress utilities ###

The following items are only applicable if the <code>progressUrl</code> option is set to the URL of uploadProgress.php (see above for setup instructions).

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><strong>checkProgressInterval</strong><br />Default: <code>50</code></td>
            <td>Integer</td>
            <td>Length of delay (in milliseconds) between completed progress update checks.</td>
        </tr>	
        <tr>
            <td><strong>keyParamName</strong><br />Default: <code>APC_UPLOAD_PROGRESS</code></td>
            <td>String</td>
            <td>The name specified in PHP configuration to activate APC upload progress. This is the value of <code>apc.rfc1867_name</code> in PHP runtime config. (PHP default value is "APC_UPLOAD_PROGRESS")
            <br /><a href="http://php.net/manual/en/apc.configuration.php#ini.apc.rfc1867-name">More info at php.net</a>
            </td>
        </tr>	        
        <tr>
            <td><strong>onUpdateFileSize(filesize)</strong></td>
            <td>Function</td>
            <td>This function serves the specific purpose of providing the upload file size in browsers that do not support the HTML5 File API. It is called after the first progress update. The function gets passed one argument: the size (in KB) of the uploaded file.</td>
        </tr>	          
    </tbody>
</table>

### Instance Methods ###
<table>
    <thead>
        <tr>
            <th>Name</th>
			<th>Parameters</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><strong>submit()</strong></td>
			<td><i>none</i></td>
            <td>Begins the file upload process. Note that if <code>autoSubmit</code> is set to <code>true</code> (the default value), there is no need to manually call <code>submit()</code>. The upload process will begin immediately after the user selects a file.</td>
        </tr>	
        <tr>
            <td><strong>setData(data)</strong></td>
			<td><code>data</code> (Object)</td>
            <td>Replaces the data to be sent to the server. Note that all previously set data is entirely removed and replaced.</td>
        </tr>
        <tr>
            <td><strong>disable()</strong></td>
			<td><i>none</i></td>
            <td>Disables upload functionality.</td>
        </tr>		
        <tr>
            <td><strong>enable()</strong></td>
			<td><i>none</i></td>
            <td>Restores upload functionality.</td>
        </tr>			
	</tbody>
</table>

### License ###
Released under the MIT license.