Multiple file uploading cannot be accomplished without the help of browser plugins. I tried using swfupload (Flash plugin + javascript) from within Liferay and here are some notes that hopefully will help others. I am using the plugin environments and all my tests have been performed from JSPPortlets.

 1. Include a Servlet in your plugin WAR as you would in normal servlet development.
 1. Return errors like:
```
out.println("ERROR: Error Description");
```
 1. Return the rest of the messages as you want them to appear for example:
```
out.println("Completed");
```
 1. Include necessary resources:
```
<script type="text/javascript" src="<%=renderRequest.getContextPath()%>/swfupload/swfupload.js?v=1"></script> 
<script type="text/javascript" src="<%=renderRequest.getContextPath()%>/swfupload/swfupload.queue.js"></script>
<script type="text/javascript" src="<%=renderRequest.getContextPath()%>/swfupload/fileprogress.js?v=1"></script> 
<script type="text/javascript" src="<%=renderRequest.getContextPath()%>/swfupload/handlers.js?v=5"></script>
```
 1. Initialize the plugin
```
<script type="text/javascript">

var swfu;
var jsessionBits = ';jsessionid=' + '<%=session.getId() %>';
//var uploadUrl = '<portlet:actionURL/>';
//uploadUrl = uploadUrl.replace('?', jsessionBits + '?');

//Using a servlet as portlets will not allow changing content-type
uploadUrl = "<%=renderRequest.getContextPath()%>/multipleUpload" + jsessionBits;


window.onload = function() {
	//alert(uploadUrl);
	var settings = {
		flash_url : "<%=renderRequest.getContextPath()%>/swfupload/swfupload.swf",
		upload_url: "<%=renderRequest.getContextPath()%>/multipleUpload" + jsessionBits,
		file_size_limit : "<%=Constants.UPLOAD_MAX_SIZE %>",
		file_types : "**.**",
		file_types_description : "All Files",
		file_upload_limit : 10,
		file_queue_limit : 0,
		custom_settings : {
			progressTarget : "fsUploadProgress",
			cancelButtonId : "btnCancel"
		},
		debug: true,

		// Button settings
		button_image_url: "<%=renderRequest.getContextPath()%>/images/XPButtonBrowseText_61x22.png",
		button_width: "61",
		button_height: "22",
		button_placeholder_id: "spanButtonPlaceHolder",
		button_text_style: ".theFont { font-size: 16; }",
		button_text_left_padding: 12,
		button_text_top_padding: 3,
		
		// The event handler functions are defined in handlers.js
		file_queued_handler : fileQueued,
		file_queue_error_handler : fileQueueError,
		upload_start_handler : uploadStart,
		upload_progress_handler : uploadProgress,
		upload_error_handler : uploadError,
		upload_success_handler : myUploadSuccess,
		upload_complete_handler : myUploadComplete
	};

	swfu = new SWFUpload(settings);
 };
</script>
```
 1. The way flash works here is simple but perhaps not intuitive enough. Any non HTTP 200 response will be interpreted as an error, however the error description will not be available. On the other hand every 200 response will be interpreted as success and there is a description that can be parsed. So, declare your custom function to handle success or modify handler.js#uploadSuccess() to deal with both custom ERRORS and SUCCESS conditions. Server errors, well I already said they will show up just with the code:
```
function myUploadSuccess(file, server_data, receivedResponse) {
	try {
		var progress = new FileProgress(file, this.customSettings.progressTarget);
		if(server_data.contains("ERROR")){
			progress.setError();
			progress.setStatus(server_data);
			progress.toggleCancel(false);
		}else{
			progress.setComplete();
			progress.setStatus("Complete.");
			progress.toggleCancel(false);
		}
	} catch (ex) {
		this.debug(ex);
	}
}
```
 1. Include the below method to get the files queued (multiple files upload with a single 'upload' click)
```
function myUploadComplete(file) {
	if (this.getStats().files_queued > 0) {
		this.startUpload();
	}
}
```
 1. Use the Portlet session from the portlet to store information to be passed to the servlet (For example in JSPPortlet#doDispatch()):
```
renderRequest.getPortletSession().setAttribute(WebKeys.THEME_DISPLAY, themeDisplay, PortletSession.APPLICATION_SCOPE);
try {
			serviceContext = ServiceContextFactory.getInstance(
					DLFileEntry.class.getName(), renderRequest);
		} catch (Exception e) {
			
			e.printStackTrace();
		}
		renderRequest.getPortletSession().setAttribute("serviceContext", serviceContext, PortletSession.APPLICATION_SCOPE);
```
 1. Use the Servlet session to retrieve the information set from the portlet:
```
ThemeDisplay themeDisplay = (ThemeDisplay) req.getSession().getAttribute(WebKeys.THEME_DISPLAY);
ServiceContext = (ServiceContext) req.getSession().getAttribute("serviceContext");
```

## Some customizations

 1. To make the queue items not dissapear comment the below from fileprogress.js for setError() and setComplte() prototypes.
```
//oSelf.disappear();
```
 1. To be able to show how many items were and weren't uploaded from the queue change handlers.js. You will note I do not use the queue_complete_handler as it will never be fired after I used a custom upload_complete_handler.
```
var errorCount = 0;
...
if (serverData.indexOf("ERROR") == 0) {
			errorCount++;
...
if (this.getStats().files_queued > 0) {
		this.startUpload();
	}else{
		myQueueComplete(this.getStats());
	}
...
function myQueueComplete(stats) {
	var numFilesUploaded = stats.successful_uploads - errorCount;
	var numErrors = stats.upload_errors + errorCount;
	errorCount = 0;

	var status = document.getElementById("divStatus");
	status.innerHTML = '';
	if(numFilesUploaded > 0){
		status.innerHTML = numFilesUploaded + " file"
		+ (numFilesUploaded === 1 ? "" : "s") + " uploaded.";
	}
	if(numErrors > 0){
		status.innerHTML = numFilesUploaded + " file"
		+ (numFilesUploaded === 1 ? "" : "s") + " uploaded.";
		status.innerHTML += numErrors + " file"
		+ (numFilesUploaded === 1 ? "" : "s") + " could not be uploaded.";
	}
```
