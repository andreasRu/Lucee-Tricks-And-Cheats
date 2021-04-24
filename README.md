# lucee-cheats-and-tricks
Just a bunch of snippets

## Get Server Web Context Information
Sometimes you don't know where is the web-context placed. Throw this snippet somewhere into your code and run it!
```
<cfscript>
	/**
	* returns a struct with the server/web context information that is bound to this template.
	*/
	public struct function getServerWebContextInfoAsStruct(){
	
		//get pageContext of actual template
		local.pageContext=getpagecontext();
		local.pageCFMLFactory=local.pageContext.getCFMLFactory();
		local.pageCFMLFactoryConfig=local.pageCFMLFactory.getConfig();
		local.servletConfig = getpagecontext().getServletConfig();
		local.servletInitParamNames = servletConfig.getInitParameterNames();
			
		local.info={
					"label" : getpagecontext().getCFMLFactory().getLabel(),
					"configFileLocation" : pageCFMLFactoryConfig.getConfigFile(),
					"ServletInitParameters": [:]
					};
		
		if( StructIsEmpty(  local.servletInitParamNames ) ){
			
			structInsert( 
							local.info["ServletInitParameters"] , 
							"info", 
							"No initial parameters have been submitted to this servlet engine. Very probably the servlet engines default setting is being used."
			);
	
		} else {
			
			cfloop( collection="#servletInitParamNames#" item="item" ){
				
				structInsert( local.info["ServletInitParameters"] , item, local.servletConfig.getInitParameter( item.toString() ) );
			};
			
		};
	
		return local.info;
	}
	
writedump(var="#getServerWebContextInfoAsStruct()#");		
</cfscript>
```
