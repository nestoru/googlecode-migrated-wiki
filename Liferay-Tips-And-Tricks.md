I will be documenting here those bits I find using a lot in Liferay code. See also [[Customizing Liferay|Customizing Liferay]].

 1. Show an error message:
 {{{
<div class="portlet-msg-error"><%=error %></div>
 }}}
 1. Force pdf documents to be downloaded (remove pdf from being opened directly by the browser (changing content-disposition=attachment)
 {{{
#cd ~/liferay/ext/ext-impl
#vi src/portal-ext.properties
mime.types.content.disposition.inline=flv,swf,wmv
#ant deploy-properties
 }}}