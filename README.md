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
public struct function getAvailableLanguageJavaLocalesAsStruct(){
  // Get Locale List
    local.JavaLocale = CreateObject("java", "java.util.Locale");
    local.availableJavaLocalesArray=JavaLocale.getAvailableLocales();
    // initialize an ordered struct with shorthand [:]
    local.availableJavaLocalesStruct =[:];
    cfloop( array= "#availableJavaLocalesArray#" item="itemLocale" index="i"){
        if( len( itemLocale.toLanguageTag() ) == 2  ){
            local.displayNameTargetLanguage=itemLocale.info();
            local.availableJavaLocalesStruct[itemLocale.toLanguageTag()] = {
            "targetLanguage": UcFirst( local.displayNameTargetLanguage["display"]["language"] ),
            "displayName": UcFirst(  itemLocale["displayName"] ),
            "ISO639-2": local.displayNameTargetLanguage["iso"]["Language"],
            "ISO639-1": local.displayNameTargetLanguage["Language"]
            }
        }	 
    }
    
    return local.availableJavaLocalesStruct;

}
writeDump(var="#getAvailableLanguageJavaLocalesAsStruct()#");
</cfscript>
```

## How to connect to a DB with Maria JDBC Driver and CreateObject and execute a query:
As example: Download the OSGI compliant Bundle from https://mvnrepository.com/artifact/org.mariadb.jdbc/mariadb-java-client/2.6.1 to commandbox server instance \lucee-x.x.x.x\WEB-INF\lib:

Sample code testDSN.cfm
```ini
<cfscript>
    classLoader = createObject("java", "org.mariadb.jdbc.Driver");
    driverManager = createObject("java","java.sql.DriverManager");
    connector = driverManager.getConnection("jdbc:mariadb://localhost:3306/test?user=root&password=mypassword");
    connectionString = connector.createStatement();
    result = connectionString.ExecuteQuery("SHOW TABLES FROM TEST;");
    
    writeDump( "#result#" );
    writeDump( "#classLoader#" );
</cfscript>
```

## Memberfunctions & cfloops/for-loops

### cfloop vs listMap (with arrow function () => {...}
```ini
<cfscript>
myTransportations="bicycle,bus,foot,car,train,airplane";
// two expressions, lexical scoping
finallist="";
cfloop( list="#myTransportations#",  item="element", index="index") {
	    finallist = finallist.listAppend(
	   	"#index#:" & element.listLast(",").uCFirst();
	   	)
  }

writedump(var="#[finallist]#");
</cfscript>
```

vs.

```ini
<cfscript>
myTransportations="bicycle,bus,foot,car,train,airplane";
// one expression with closure function
finallist=listMap( myTransportations, ( element, index, list) => {
	            return "#index#:" & element.listLast(",").uCFirst();
	        }
	    );
writedump(var="#[finallist]#");
</cfscript>
```

Further example:

```ini
<cfscript>
myTransportationSequence="car,bicycle,bus,foot,car,train,airplane,bus,foot";
speedSequence = listMap( 
			myTransportationSequence, 
			( element, index, list) => {
			
				switch(element){
				    case "bicycle":  return "#index#:" & "slow"
				    case "bus":  return "#index#:" & "normal"
				    case "foot":  return "#index#:" & "very slow"
				    case "car": return "#index#:" & "fast"
				    case "train": return "#index#:" & "very fast"
				    case "airplane": return "#index#:" & "ultra fast"
				    default: return "don't know the transportation speed"; 
				}
				
			}
	    );
writedump(var="#[speedSequence]#");
</cfscript>
```

```ini
<cfscript>
initialAmount=1000;
transferedQuarterList="120,140,123,90";
listOfTransactions="";
interestPerQuarter=0.12;
amount = listReduce( 
		transferedQuarterList, 
		( acc, element ) => {
			
		    var newAmount = acc + ( acc + element ) * interestPerQuarter ;
		    var calculationString = " #acc# + " &  ( acc + element ) & " * " & interestPerQuarter & " = " & acc + ( acc + element ) * interestPerQuarter ;
		    
		    listOfTransactions = listOfTransactions.listAppend(

		    		calculationString

		    );
		    
		    return newAmount;

		}

	    , initialAmount
	    );
writedump(var="#[initialAmount, transferedQuarterList,listOfTransactions,amount]#");
</cfscript>
```
