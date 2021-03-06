# account-service
Used by the AOS development team for verifying the Advantage Online Shopping web site and app, and especially the service account
# This suite of tests is used by R&D for verifying that AOS is working. There are actually 4 different test suites:
- Mobile Android
- Mobile iOS
- NET
- Web
## Suite contents
Each suite has:
- an application model
- Java class that extends
- many local methods that are used to decompose complex tests into  simpler routines
UnitTestClassBase, and contains 6-12 individual tests.

The project is rather old. The LeanFT version is specified on line 5 of the pom file as 14.3.0-SNAPSHOT. 
In NimbusClient the only versions that are currently supported are 14.52.0, 14.53.0, and when the year 2020 NimbusClient comes out, 15.0.0. You will have to modify the pom file to your desired version. Also note that Nimbus Client does not use the "-SNAPSHOT" string.
## Fixing application models
Prior to v14.52, Application Models are not automatically built after cloning a project. 
(V14.52 and later versions fix this via additional entries in the pom file).

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
Note that the commands were run from within the git repository within the leanft directory, so relative paths were used both for the location of the .tsrx file and location of the outputDir.
# Discussion of general contents
### Use of configuration values
Each suite has "hard coded" configuration information that you must change in order to run the suite. Examples include:
- details of the cell phone to use from UFT Mobile
- IP of the IOS web site
- and many others
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
So in this way, "dynamic" values be be used from Jenkins, but running  from IntelliJ will use the hard coded, appURL2 value.
### Verifications and Logging
Almost every test has several verifications: some negative, some positive.

Print is used extensively in order to document what is happening.

The execution time of every test is calculated and printed.
## Web - AdvantageWebTest
### addMainUserIfNotExists
1. Checks if someone is currently logged in; logs out if so
2. Creates a new unique username by appending a random number at the end of configured string
3. Creates the user
4. Verifies creation
### contactSupportTest
Very straightforward. Just uses the same hard coded values every time.
### createNewAccount
This calls the same code as used in addMainUserIfNotExists (above), step 3.
### createNewAccountNegative
This calls the same code as above, with a flag to signify that the create account should fail. It causes a failure by not entering an email address, and verifies that the creation failed.
### logOutTest
Just a simple test
### orderServiceTest
This test is an end-to-end purchase of an item:
1. Add an item to the card
2. Checkout (which requires logging in)
### There are 8 tests to verify that all pay methods work
These test also purchase different items, while cycling through different categories and items. There are also negative tests.
### verifySearchUsingURL
Verifies that searching for categories works.
### deleteAccountTest
Calls the usual createNewAccount method, but with a hard coded username of "deleteUser". Then deletes this new account and verifies the deletion. In this way, this test uses the same account name for every execution.
### payButtonRegExTest
This test shows how regular expressions can be used within a LeanFT test to create verification steps using a regular expression. It does it on the Checkout button, which has a dynamic total value.
## Web - AdvantageSRFTest
This is a small subset of the Web tests above. Only 6 of them. Since SRF is currently depricated, they should not be run.
## Mobile - androidTests
Of course these tests are designed to run on android phones.
### AddNewUserAndCheckInitials
This tests assumes that a user is currently logged in. It then:
```
if (correctUser) then
    SignOut
    CreateNew user
else
    if (isSignedIn) then
        Signout
    end if
end if
```
### CreateExistingUserTest
This is a negative test.
```
if (isSignedIn) then 
    SignOut
end if
Go through the process of trying to create a new user, but user a username that already exists.
Verify that account creation failed.
```
### NegativeLogin
Another negative test.

Try to sign in with credentials that are known to be bad, and verify that login fails.
### UpdateCartTest
This is an end to end shopping test, similar to what is in the web test.

1. Open app
2. Login if not logged in
3. Select Tablets tile
4. Select a Tablet
5. Add +1 to quantity
6. Click on Add to Cart
7. Click on Cart
8. swipe to Edit
9. Change amount and color
10.  click on add to cart
11. Open cart menu and validate that the cart was updated correctly (the old color is not there)
12. Click checkout
13. Select SafePay as Payment method
14. Click Pay Now
15. Verify receipt 
### PayMasterCreditTest
This is essentially the same as the previous test, but payment method is MasterCard.
### PayButtonRegExTest
This is essentially the same as the web test of the same name.
### ChangePasswordTest
This is a straightforward test, doing what the name sames. However, in order to be able to run multiple times, it then changes the password back to the original password.
## IOSTests
## These tests are essentially the same Android or web test.
### AddNewUserTest
### InvalidLoginTest
### LogOutTest
### OutOfStockTest
### CreateExistingUserTest
##These test are unique to IOS
### OutOfStockTest
A negative test, verifying that shopping cannot be completed if the item is out of stock.
### UpdateCartTest
This is a pretty standard shopping test. The one new thing it does is interact with the color object to filter.
### priceRangeElement
Standard shopping, but filter by price range.
### PurchaseHugeQuantityTest
A negative test. Verify that if the number of elements are larger than a limit within the application, that a specific warning message appears.
## NET
There are only two tests in this suite, but it shows the use of WPF objects and API tests, instead of web objects. It is written against the Administration console. The console is installed on NimbusClient. It is also available for download from the AOS help page.
### addUserTest
1. Add a new user
2. Verify that the user was created via an API test
3. Delete the user so this test can be run many times
### searchItemAndChangePropertyTest
Very straightforward test.
