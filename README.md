# Lucee-Tricks-And-Cheats
Just a bunch of snippets

## Get Server Web Context Information
Sometimes you're not sure where the web/server context is, e.g. when you've moved the context out of the document root. Throw this snippet somewhere into your code and run it!
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
			"servletInitParameters": [:]
			};

	// if available, iterate enum of InitParamNames and get the values

	cfloop( collection="#servletInitParamNames#" item="item" ){

		structInsert( local.info["ServletInitParameters"] , item, local.servletConfig.getInitParameter( item.toString() ) );
	};


	return local.info;
}
	
writedump(var="#getServerWebContextInfoAsStruct()#");		
</cfscript>
```

## Get a struct of all available two-letter language codes and their target language names of the underlying Java.util.Locale Class
Lucee uses the underlying Java locale from Java.util.Locale for ls-Functions. As long as the language is supported in Java, the locale can also be used in Lucee CFML.
Some Java version just don't support all Java Locales (e.g. Java 8 doesn't support the Locale "Filipino ( Philippines )", but AdoptOpenJDK 11.0.4 does. 
This script returns all availabe 2-letter language codes and their language names in their respective language (different from the getDisplayName() function which returns the name in the OS language ).

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
    cfloop( array= "#availableJavaLocalesArray#" item="itemLocale" index="i"){
        if( len( itemLocale.toLanguageTag() ) == 2 ){
            local.displayNameTargetLanguage=itemLocale.info();
            local.availableJavaLocalesStruct[itemLocale.toLanguageTag()] = {
            "language": UcFirst( local.displayNameTargetLanguage["display"]["language"] ),
            "displayName": itemLocale["displayName"],
            "codeISO-3166": itemLocale["Code (ISO-3166)"]
            }
        }	 
    }
    
    return local.availableJavaLocalesStruct;

}
writeDump(var="#getAvailableJavaLocalesAsStruct()#");
</cfscript>
```
