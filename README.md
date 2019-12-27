# account-service
Used by AOS development team for verifying the Advantage Online Shopping web site and app, and especially the service account

# This suite of tests is used by R&D for verifying that AOS is working. There are actually 4 different test suites:

- Mobile Adroid
- Mobile IOS
- .NET
- Web

## Suite contents
Each suite has:

- an application model
- Java class that extends UnitTestClassBase, and contains 6-12 individual tests.

The project is rather old. The Leanft version is specified on line 5 of the pom file as 14.3.0-SNAPSHOT. 
In NimbusClient the only versions that are currently supported are 14.52.0, 14.53.0, and when the year 2020 NimbusClient comes out, 15.0.0.

## Fixing application models

Also, prior to v14.52, Application Models are not automatically built after cloning a project. 
(V14.52 and later versions fix this via additional entries in the pom file.)

To build the application models in this project, from a command prompt, run these 4 commands (These were run from git bash on windows, which is insensitive to '/' or '\\')
```
demo@NimbusClient MINGW64 ~/account-service/leanft (master)
$ java -jar  "C:\Program Files (x86)\Micro Focus\Unified Functional Testing\Tools\AppModelCodeGenerator\Java\appmodel-code-generator.jar" src/Mobile/AdvantageAndroidApp.tsrx -package Mobile -outputDir appmodels

demo@NimbusClient MINGW64 ~/account-service/leanft (master)
$ java -jar  "C:\Program Files (x86)\Micro Focus\Unified Functional Testing\Tools\AppModelCodeGenerator\Java\appmodel-code-generator.jar" src/Mobile/AdvantageIOSApp.tsrx -package Mobile -outputDir appmodels

demo@NimbusClient MINGW64 ~/account-service/leanft (master)
$ java -jar  "C:\Program Files (x86)\Micro Focus\Unified Functional Testing\Tools\AppModelCodeGenerator\Java\appmodel-code-generator.jar" src/NET/DotNetAppModel.tsrx -package NET -outputDir appmodels

demo@NimbusClient MINGW64 ~/account-service/leanft (master)
$ java -jar  "C:\Program Files (x86)\Micro Focus\Unified Functional Testing\Tools\AppModelCodeGenerator\Java\appmodel-code-generator.jar" src/Web/AdvantageStagingAppModel.tsrx -package Web -outputDir appmodels
```

Note that the commands were run from within the git repository within the leanft directory, so relative paths were used both the location of the .tsrx file and location of the outputDir.

# Discussion of general contents

Each suite has "hard coded" configuration information that you must change in order to run the suite. Examples include:

- details of the cell phone to use from UFT Mobile
- IP of the IOS web site

The tests are implemented with many "internal methods" that are used to decompose the tests into small, maintainable sections. 

The values are controlled with code segments like this:
```
// In the class definition block, there are variables/code like
    public static String appURL = System.getProperty("url", "defaultvalue");
    public static String appURL2 = "16.60.158.84";

// The System.getProperty is used to read values when the tests are run from within Jenkins. 
// Jenkins is configured to run with maven
// Values are passed in with the -D flag, such as -Durl=http://www.advantageonlineshopping.com

// The full line is:
Executing Maven:  -B -f C:\JenkinsSlave\workspace\demoapp_folder\demoapp_tests_leanft_maven\pom.xml clean test -P leanft-test -Dtest=Web.AdvantageWebTest#purchaseMasterCreditLaptopTest -Durl=http://www.advantageonlineshopping.com -Dbrowser_type=Chrome -Denv_type=Local -Dapplication_path=C:\Debug\AdvantageShopAdministrator.exe

// Then, in the code run before each test
   @Before
    public void setUp() throws Exception {
        startTimeCurrentTest = System.currentTimeMillis();
        printCaptionTest(curTestName.getMethodName());
        initBeforeTest();
    }

// And, in the initBeforeTest(), there is code like:
    if (appURL.equals("defaultvalue"))
                appURL = appURL2;

    // Now use the URL
    browserNavigate(appURL);
```

So in this way, "dynamic" values be be used from jenkins, but running directly from IntelliJ will use the hard coded, appURL2 value.


## Web

### addMainUserIfNotExists

1. Checks if someone is currently logged in; logs out if so
2. Creates a new unique username by appending a random number at the end of the unique string
3. Creates the user

