# lucee-cheats-and-tricks
Just a bunch of snippets

## Get Server Web Context Information
Sometimes you don't know where is the web-context placed. Throw this snippet somewhere into your code and run it!
```JavaScript
<cfscript>
	/**
	* returns a struct with the server/web context information that is bound to this template.
	*/
	public struct function getServerWebContextInfoAsStruct(){
	
		//get pageContext/CFMLFactoryConfig of actual template
		local.pageContext=getpagecontext();
		local.pageCFMLFactory=local.pageContext.getCFMLFactory();
		local.pageCFMLFactoryConfig=local.pageCFMLFactory.getConfig();
		
		//get the Servlets configuration and initial Parameters (e.g. set in Tomcats conf/web.xml)
		local.servletConfig = getpagecontext().getServletConfig();
		local.servletInitParamNames = servletConfig.getInitParameterNames();
		
		// populate struct with gathered information
		local.info={
				"context-label" : getpagecontext().getCFMLFactory().getLabel(),
				"configFileLocation" : pageCFMLFactoryConfig.getConfigFile(),
				"ServletInitParameters": [:]
				};
		
		// if available, iterate enum of InitParamNames and get the values
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

## Get a struct of all available two-letter language codes and their target language names 
Lucee uses the underlying Java locale from java.util.Locale for lsFunctions. As long as the language is supported in Java, you can use it's locale. 
This script returns the 2-letter locales and the language names in their respective language (different from the getDisplayName() function).

```JavaScript
<cfscript>
/**
 * returns as struct of all available 2-letter codes of the underlying java.util with the referring Language DisplayName (target language)
 */
public struct function getAvailableJavaLocalesAsStruct(){

    // Get Locale List
    local.JavaLocale = CreateObject("java", "java.util.Locale");
    local.availableJavaLocalesArray=JavaLocale.getAvailableLocales();
    // initialize an ordered struct with shorthand [:]
    local.availableJavaLocalesStruct =[:];
    cfloop( array= "#availableJavaLocalesArray#" item="itemLocale" ){
        if( len( itemLocale.toLanguageTag() ) == 2 ){
            local.displayNameTargetLanguage=itemLocale.info();
            local.availableJavaLocalesStruct[itemLocale.toLanguageTag()] = UcFirst( local.displayNameTargetLanguage["display"]["language"] );
            }	 
    }
    
    return local.availableJavaLocalesStruct;

}
writeDump(var="#getAvailableJavaLocalesAsStruct()#");
</cfscript>
```
