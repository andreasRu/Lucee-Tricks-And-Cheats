# Lucee-Tricks-And-Cheats
Just a bunch of snippets

## Access databases (mdb) on Windows

Let’s say you have a setup like this:

> - a database named database.accdb with a table named table1 placed at `pathToYourLucee\lucee-express-5.3.10.97\database.accdb`,
> - a webroot served at `pathToYourLucee\lucee-express-5.3.10.97\webapps\ROOT`

**Step 1:** Download Lucee Express Version 5.3.10.97 and unzip it.

**Step 2:** Download the following OSGI compliant dependencies to your `pathToYourLucee\lucee-express-5.3.10.97\lib`:
- [commons-lang3-3.12.0.jar](https://repo1.maven.org/maven2/org/apache/commons/commons-lang3/3.12.0/commons-lang3-3.12.0.jar)
- [ucanaccess-5.0.0.jar](https://repo1.maven.org/maven2/net/sf/ucanaccess/ucanaccess/5.0.0/ucanaccess-5.0.0.jar)
- [jackcess-4.0.4.jar](https://repo1.maven.org/maven2/com/healthmarketscience/jackcess/jackcess/4.0.4/jackcess-4.0.4.jar)
- [hsqldb-2.7.1.jar](https://repo1.maven.org/maven2/org/hsqldb/hsqldb/2.7.1/hsqldb-2.7.1.jar)
- [commons-logging-1.2.jar](https://repo1.maven.org/maven2/commons-logging/commons-logging/1.2/commons-logging-1.2.jar)

**Step 3:** Create an **Application.cfc** at `pathToYourLucee\lucee-express-5.3.10.97\webapps\ROOT\Application.cfc` with the following code:
```JavaScript
//Application.cfc
component {

	this.Name = "MSAccessExample";
    	this.dataBasePath=expandPath("../../") & "database.accdb";
    	this.datasources["msAccessDB"] = {
        	class: "net.ucanaccess.jdbc.UcanaccessDriver",
        	connectionString: "jdbc:ucanaccess:///" & this.dataBasePath
     };
}
```
**Step 4:** Create an **index.cfm** with the following code at `pathToYourLucee\lucee-express-5.3.10.97\webapps\ROOT\index.cfm` with the following code:
```HTML
<!--- index.cfm --->
<cfquery name="myquery" datasource="msAccessDB" >
	select * from table1;
</cfquery>
<cfdump var="#myquery#">
```
**Step 5:** Run the Lucee server from your Lucee Express Version and execute the index.cfm file.



## How to setup a dev environment for the Lucee Admin:
[https://github.com/lucee/debug/issues/1](https://github.com/lucee/debug/issues/1)

## Function to sign a Kraken API call with CFML. This function returns a base 64 string of the signed Kraken Api Call as specified in the [KrakenAPI docs](https://docs.kraken.com/rest/#section/Authentication/Headers-and-Signature). This is also my answer posted at [Stackoverflow](https://stackoverflow.com/questions/73609099/kraken-api-coldfusion/73649378#73649378).

```JavaScript
<cfscript>

     /**
     * Returns a base 64 string of the signed Kraken Api Call as specified in the docs https://docs.kraken.com/rest/#section/Authentication/Headers-and-Signature
     * in CFML
     * Author: Andreas Rüger 2022
     * License: MIT 
     */
     public string function getKrakenSignature( urlpath, postdata, nonce, secretAsBase64) localmode=true {

        // assign arguments to local variables
        urlpath= arguments.urlpath;
        nonce= arguments.nonce;
        postdata = arguments.postdata;
        secretAsBase64= arguments.secretAsBase64;

        // convert urlpath to a binary Hex representation 
        urlpathBinary= toBinary( toBase64( urlpath ));
        urlpathBinaryAsHex= BinaryEncode( urlpathBinary, "HEX");


        // convert secret to binary
        secretBinary= ToBinary(  arguments.secretAsBase64 );

        // concatenate nonce and postdata
        noncePostdata = nonce & postdata; 

        //get binary digest as Hex representation
        noncePostdataDigestBinaryAsHex= hash( noncePostdata, "SHA-256" );


        // concatenate urlPath binary (hex) and noncePostDara binary (hex) 
        messageBinaryAsHex= urlpathBinaryAsHex & noncePostdataDigestBinaryAsHex;

        // convert message hex representation to binary
        messageBinary= BinaryDecode( messageBinaryAsHex, "HEX");

        // sign the message with hmac function
        messageHmacDigestBinaryAsHex = hmac( messageBinary, secretBinary, "HMACSHA512");
        messageHmacDigestBinary=BinaryDecode( messageHmacDigestBinaryAsHex, "HEX");

        return binaryEncode( messageHmacDigestBinary, "base64" );

    }

encodedPayLoad="nonce=1616492376594&ordertype=limit&pair=XBTUSD&price=37500&type=buy&volume=1.25";
nonce="1616492376594";
api_sec = "kQH5HW/8p1uGOVjbgWA7FunAmGO8lsSUXNsu3eow76sz84Q18fWxnyRzBHCd3pd5nE9qa99HAZtuZuj6F1huXg==";
urlpath="/0/private/AddOrder";
signature = getKrakenSignature( urlpath, encodedPayLoad, nonce, api_sec);
writeoutput( signature );

</cfscript>
```


## Get Server Web Context Information From Lucee
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

## Retrieve MP3 Tag Data with Apache TIKA Java & CFML
Just download tika-app-2.6.0.jar from https://repo1.maven.org/maven2/org/apache/tika/tika-app/2.6.0/tika-app-2.6.0.jar
to your classpath ( drop it to your Tomcat lib directory /or lucee-server/bundle/ and run the following script

```JavaScript
<cfscript>
/**
*
*   Retrieve MP3 Data with TIKA Java & CFML: Download https://repo1.maven.org/maven2/org/apache/tika/tika-app/2.6.0/tika-app-2.6.0.jar
*   and drop it to your classpath/bundle or Tomcat Lib directory. 
*
**/
public struct function getMP3Info( required string filename ) localmode = true {
    
    result = {
        "error" = "",
        "info" = {}
    };
    
    if ( FileExists( filename ) ) {

        file = createObject( "java", "java.io.File" ).init( arguments.filename );
        fileInputStream = createObject( "java", "java.io.FileInputStream" ).init( file );
        bodyContentHandler = CreateObject( "java", "org.apache.tika.sax.BodyContentHandler", "../../lib/tika-app-2.6.0.jar" );
        metaData = CreateObject( "java", "org.apache.tika.metadata.Metadata", "../../lib/tika-app-2.6.0.jar" );
        pcontext = CreateObject( "java", "org.apache.tika.parser.ParseContext", "../../lib/tika-app-2.6.0.jar" );
        mp3Parser = CreateObject( "java", "org.apache.tika.parser.mp3.Mp3Parser", "../../lib/tika-app-2.6.0.jar" );

        try {
            mp3Parser.parse( fileInputStream, bodyContentHandler, metaData, pcontext );
            //dump( metaData.names() ); 
           
            durationInSec = parseNumber( metaData.get( "xmpDM:duration" ) );
            hours = int( durationInSec / 60 / 60 );
            restTimeSec = durationInSec - ( hours * 60 * 60 );
            minutes = int( restTimeSec / 60 ); 
            restTimeSec = restTimeSec - ( minutes * 60 );
            seconds = restTimeSec;
            result.info = {
                "artist" = metaData.get( "xmpDM:artist" ),
                "album" = metaData.get( "xmpDM:album" ),
                "trackNumber" = metaData.get( "xmpDM:trackNumber" ),
                "duration" = numberformat( hours, "00" ) 
                            & ":" & numberformat( minutes, "00" ) 
                            & ":" & numberformat( seconds, "00" ),
                "genre" = metaData.get( "xmpDM:genre"),
                "comment" = metaData.get( "xmpDM:logComment" )
                
            }
        } catch ( any error ) {
            result[ "error" ] = error;
        }
        fileInputStream.close();
    } else {
        result [ "error" ] = "File not found";
    }
    return result;
}

mp3File = expandPath( "../../" ) & "song.mp3";
dump( getMP3Info( mp3File ) );


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

### listReduce: Use always if you need to reuse a calculated value to recursively pass it again to the closure (through accumulator acc)

```ini
<cfscript>
initialAmount=1000;
transferedQuarterList="120,140,123,90";
listOfTransactions="";
interestPerQuarter=0.12;
cummulatedAmount = listReduce( 
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
writedump(var="#[initialAmount:initialAmount, transferedQuarterList:transferedQuarterList,listOfTransactions:listOfTransactions,cummulatedAmount:cummulatedAmount]#");
</cfscript>
```


### simple listReduce example to return single values of a querie column:
```JavaScript
<cfscript>
fruitsQuery = queryNew( 
    "id, fruit , price" , "numeric, varchar , numeric" , 
    {
        id:     [1,2,3,4,5,6,7,8,9], 
        fruit:  [ "Bananas" , "Kiwis", "Apples", "Oranges", "Peaches", "Bananas" , "Kiwis", "Apples", "Uchuvas" ], 
        price: [ 1.99 , 0.99 , 2.99, 3.99, 6.99, 2.99, 3.99, 6.99, 5.00 ] 
    }
);

distinctFruitsArray = valueList( fruitsQuery.fruit ).listReduce(
    ( acc, element ) => {
        if( not acc.contains( element) ){
                acc.append( element );
        }
        return acc;
    }
    
    , [] , ","
);

dump(distinctFruitsArray);

</cfscript>
```

### struct.reduce Example:
```JavaScript
<cfscript>
/**
 * @hint check if passed file extensiona and mimeType matches the allowed mapped file-extensions/mimeTypes combinations, usable for fileUploads/imageMagick verbose identify ;
 */
public boolean
function matchesAllowedMimeTypesAndFileExtensions(
	string fileExtension required,
	string mimeType required
) {


	variables.allowedFileExtensionsAndMimeTypes = {
		jpg: 'image/jpeg',
		jpeg: 'image/jpeg',
		png: 'image/png'
	}
	variables.isFileExtensionsAndMimeTypesAllowed = false;
	variables.fileExtension = lcase( arguments.fileExtension );
	variables.mimeType = lcase( arguments.mimeType );

	variables.allowedFileExtensionsAndMimeTypes.reduce( ( result, allowedExtension, allowedMimeType ) => {
		if (
			variables.fileExtension == arguments.allowedExtension &&
			variables.mimeType == arguments.allowedMimeType ) {
			variables.isFileExtensionsAndMimeTypesAllowed = true;
		}
		return result
	}, "" );

	return variables.isFileExtensionsAndMimeTypesAllowed

}

writedump( matchesAllowedMimeTypesAndFileExtensions("jpg","image/jpeg") );
writedump( matchesAllowedMimeTypesAndFileExtensions("jpg","text/html") );
</cfscript>
```


### JfreeChart: A collection of commands to directly invoke the jfreeChart java package shipped with Lucee as an alternative to cfml's cfchart.

```JavaScript
<cfscript>
    // create all objects 
    ObjChartFactory = CreateObject("java", "org.jfree.chart.ChartFactory");
    ObjChartOrient = CreateObject("java", "org.jfree.chart.plot.PlotOrientation");
    ObjChartUtil = CreateObject("java", "org.jfree.chart.ChartUtilities");
    ObjXYLineAndShapeRenderer = CreateObject("java", "org.jfree.chart.renderer.xy.XYLineAndShapeRenderer");
    ObjBasicStroke = CreateObject("java", "java.awt.BasicStroke");
    ObjChartColor = CreateObject("java", " org.jfree.chart.ChartColor");
    ObjRectangleInsets = CreateObject("java", "org.jfree.ui.RectangleInsets");
    ObjShapeUtilities = CreateObject("java", "org.jfree.util.ShapeUtilities");
    ObjXYSeriesCollection = CreateObject("java", "org.jfree.data.xy.XYSeriesCollection");
    ObjXYDataset= CreateObject("java", "org.jfree.data.xy.XYDataset");
    ObjXYSeries= CreateObject("java", "org.jfree.data.xy.XYSeries");
    ObjNumberAxis= CreateObject("java", "org.jfree.chart.axis.NumberAxis");
    ObjNumberTickUnit= CreateObject("java", "org.jfree.chart.axis.NumberTickUnit");
    ObjStandardXYItemLabelGenerator  = CreateObject("java", "org.jfree.chart.labels.StandardXYItemLabelGenerator ");
    ObjXYItemLabelGenerator = CreateObject("java", "org.jfree.chart.labels.XYItemLabelGenerator");
    ObjNumberFormat= CreateObject("java", "java.text.NumberFormat");
    ObjFont=CreateObject("java", "java.awt.Font");   

    // Initialize Series 
    XYSeries=ObjXYSeries.init("series1");

    // add data to series 
    XYSeries.add(1,0.9);
    XYSeries.add(2,0.902);
    XYSeries.add(3,0.903);
    XYSeries.add(4,0.906);
    XYSeries.add(5,0.904);


    // Initialize second series 
    XYSeries2=ObjXYSeries.init("series2");

    // add data to series 
    XYSeries2.add(1,0.906);
    XYSeries2.add(2,0.904);
    XYSeries2.add(3,0.900);
    XYSeries2.add(4,0.901);
    XYSeries2.add(5,0.904);

    // initialize XYSeriesCollection
    XYDataset=ObjXYSeriesCollection.init();

    //add both series to collection
    XYDataset.addSeries(XYSeries);
    XYDataset.addSeries(XYSeries2);

    // Set Chart as createXYLineChart 
    Chart = ObjChartFactory.createXYLineChart ("This is some Title", "Name 1", "Name 2", XYDataset, ObjChartOrient.VERTICAL, true, true, true);

    // Set Range of Range Axis (y) 
    Chart.getPlot().getRangeAxis().setRange(0.887, 0.908);

    // Set Range of Domain Axis (x) 
    Chart.getPlot().getDomainAxis().setRange(1, 5);

    // Force Domain Axis to show values as Integer 
    Chart.getPlot().getDomainAxis().setStandardTickUnits(ObjNumberAxis.createIntegerTickUnits());

    // Define Steps of Values of Domain Axis (x) 
    Chart.getPlot().getDomainAxis().setTickUnit(ObjNumberTickUnit.init(0.5));

    // Define Steps of Values of Range Axis (y) 
    Chart.getPlot().getRangeAxis().setTickUnit(ObjNumberTickUnit.init(0.005));




    // Remove Legend 
    Chart.removeLegend();

    // Init 
    barrenderer = ObjXYLineAndShapeRenderer.init();

    //define Color for each serie
    barrenderer.setSeriesPaint(0,ObjChartColor.RED);
    barrenderer.setSeriesPaint(1,ObjChartColor.LIGHT_GREEN);

    //define Stroke for each serie
    barbasicstroke1 = ObjBasicStroke.init(1);
    barbasicstroke2 = ObjBasicStroke.init(10);
    barrenderer.setSeriesStroke(0,barbasicstroke1);
    barrenderer.setSeriesStroke(1,barbasicstroke2);

    //define Type of Markers for each serie
    triangle = ObjShapeUtilities.createDownTriangle(5);
    diamond = ObjShapeUtilities.createDiamond(8);
    barrenderer.setSeriesShape(0, triangle);
    barrenderer.setSeriesShape(1, diamond);

    barrenderer.setShapesFilled(true);
    barrenderer.setShapesVisible(true);

    // Set color of LabelValues
    barrenderer.setBaseItemLabelPaint(ObjChartColor.WHITE);


    // Set format for LabelValues and limit showing digits for use in Label-Genertator
    NumberFormatY = ObjNumberFormat.getNumberInstance();
    NumberFormatY.setMaximumFractionDigits(2); 
    NumberFormatX = ObjNumberFormat.getNumberInstance();
    NumberFormatX.setMaximumFractionDigits(4); 

    // initialize XYItemLabelGenerator: it sets the content and the format of the shown data. ("Some text {datasetIndex}",format)
    tmpgenerator=ObjStandardXYItemLabelGenerator.init("Y={2} / X={1} ", NumberFormatY, NumberFormatX);



    // initialize font with ObjFont.init( FontFamily, Style( bitwise: 0=normal, 1=bold, 2=italic, 3=bold|italic), size )
    font=ObjFont.init("Verdana",1,12);

    //Set font for ItemLables
    barrenderer.setBaseItemLabelFont(font);

    //Generate ItemLables
    barrenderer.setBaseItemLabelGenerator(tmpgenerator);
    barrenderer.setBaseItemLabelsVisible( true );
    Chart.getPlot().setRenderer(barrenderer);



    // Set Margins (top, left, bottom, right)
    newrectangle=ObjRectangleInsets.init(20,0,0,50);
    Chart.getPlot().setAxisOffset(newrectangle);

    // Change Background Color 
    Chart.getPlot().setBackgroundPaint( ObjChartColor.BLUE );

    // Change GridlineColors for Domain Axis (x) 
    Chart.getPlot().setDomainGridlinePaint(ObjChartColor.WHITE);

    // Change GridlineColors for Range Axis (y) 
    AWTColor = CreateObject("java", "java.awt.Color");
    // Set a color with AWTColor(redPercentage, greenPercentage, bluePercentage) as FLOAT 
    // lime(0,255,0) or magenta/fuchsia(255,0,255) with 255 being 1:
    LIMECOLOR= AWTColor.init(0,1,0);
    MAGENTACOLOR= AWTColor.init(1,0,1);

    Chart.getPlot().setBackgroundPaint( MAGENTACOLOR );

    // Prepare for Output  
    ChartImage = Chart.createBufferedImage(500, 500);
    ImageFormat = createObject("java", "org.jfree.chart.encoders.ImageFormat");
    EncoderUtil = createObject("java", "org.jfree.chart.encoders.EncoderUtil");
    ChartImgInBytes = EncoderUtil.encode( ChartImage, ImageFormat.PNG);


</cfscript>

<!--- display in browser --->
<cfoutput>
<img src="data:image/*;base64,#toBase64( ChartImgInBytes )#" />
</cfoutput>

```
