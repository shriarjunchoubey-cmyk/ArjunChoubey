# README - ARA-ADMIN #

This repository deals with the admin functionality for the ARA applications. It also contains the interface for ingesting PLM data.

This application is deployed to two different slots in Azure. One deals with the admin functionality, the other is the endpoint for the PLM data ingest.

The admin application allows users to view, upload and update data used in the other ARA applications. Users can view and update data through tables in the UI, and make bulk uploads of certain data with Excel sheets.

The .Net code can be edited in Visual Studio 2017. Open the Ara.Admin.sln file on your local machine.

The Angular code can be edited in VSCode.

## Code Structure ##

The code is split into separate projects, each of which has it's own responsibility. These are outlined below.

### Ara.Admin.Client ###

This is the UI of the application, written in Angular with the [PrimeNg](https://www.primefaces.org/primeng/#/) component library. 

Most of the screens in the UI are data tables. They share common code for this table, which can be found in \\Ara.Admin.Client\src\app\components\table

To run the UI tests, open a command line in the Ara.Admin.Client folder, and run `npm run test`. To update the [snapshots](https://jestjs.io/docs/en/snapshot-testing), run `npm run test -- -u`.

To run the stories generated with [Storybook](https://storybook.js.org/basics/guide-angular/), open a command line in the Ara.Admin.Client folder, and run `npm run storybook`.


Local running of the Angular application can be found below.

### Ara.Admin.Common ###

This is an ASP.Net project. It contains common code shared between the rest of the .Net projects in this repository.

### Ara.Admin.Repository ###

This contains the repository classes for interacting with the data entities in the database. The entity models can be found in the Ara-database repository.

There is a separate repository class for each of the admin screens, and a few for the PLM import in a separate folder (PlmImportRepositories). Each class has it's own interface. As most of the repository classes do similar things (get paged, filtered & sorted data from the database), there are a couple of base classes that a lot of them inherit from - BaseRepositoryForPagedAndSortedQueries and BaseRepositoryForPagedAndSortedQueriesUsingDerivedProperties. Usage of these base classes is best discovered by inspecting classes that already use them.

### Ara.Admin.Server ###

This contains the API controller classes for the admin API. The controller methods in this project are called by the UI to retrieve and update data. 

As for the repository classes, there is a separate controller class for each admin screen.

[Swagger](https://swagger.io/) is used to display details of the endpoints. This is the screen that's navigated to by default when the project is run.

The controllers are authenticated with the Identity Server, which can be found in the Ara.Identity repository.

Local running of the API can be found below.

### Ara.Admin.Service ###

This contains the service layer classes for both the admin and PLM interface functionality. These serve as an extra layer of abstraction between the controller classes and repository classes, and contain a lot of the business logic for uploading and retrieving data. 

As before, there is a separate service class and interface for each admin screen. 

This project also contains the logic for parsing the XML received from PLM into data entities, which are passed to the repository layer.

### Ara.Interface.Server ###

This contains a single controller, which houses the endpoint for PLM data ingest. Master data in XML format is sent to this endpoint on a schedule from PLM, via a Mulesoft intermediary layer. 

This interface is secured with SSL, and requires a client certificate for authentication. The certificate is then checked against the details in the appsettings. To change the certificate details for local development, alter the values in appsettings.Development.json

### Tests ###

This folder contains multiple test projects for the .Net code. To run these, open the solution file as above in Visual Studio, and click on Tests -> Run -> All tests.

## Running locally ##

To run the admin project locally, do the following:

1. Ensure you have a local database set up and running. Instructions for this can be found in the main SMG.
2. Check out the latest version of the develop branch.
3. Open the .Net solution in Visual Studio. Change the start-up project to be Ara.Admin.Server
4. Change the 'ConnectionString' parameter in appsettings.Development.json to be "Data Source=.\\SQLEXPRESS; Initial Catalog=RegulatoryAra;Persist Security Info=True;Integrated Security=True;MultipleActiveResultSets=True;". This will point it at your local database. 
5. Remove the KeyVault settings, these take precedence over DatabaseSettings.ConnectionString
6. Open and run the Ara.Identity project. Details of this can be found in the README in that repository.
7. Change the 'Authority' parameter in appsettings.Development.json to be "http://localhost:64443"
8. Open the Angular solution in VSCode, and edit the endPoints.json file. Change the AraAdmin provider to be "http://localhost:53593"
9. Run the .Net Ara.Admin.Server project through Visual Studio
10. Run the Angular app by opening a command line in the Ara.Admin.Client folder, and running `npm start`
11. Navigate to localhost:4200 in a browser, and (hopefully) voilà!

To run the PLM interface project locally, change the startup project in Visual Studio to be Ara.Interface.Server and run the project. The endpoint can be hit by using Postman, pointed at 'http://localhost:50749/api/MasterPlmData'. To disable the certificate validation locally, comment out lines 60 - 63 in the PlmInterfaceController, but make sure you don't check this change in!

If you need to run against the database in the Dev Azure environment:-
1. You do not need to specify a DatabaseSettings.ConnectionString.
2. The KeyVault appsettings.Development.json must contain:
```
  "KeyVault": {
    "DatabaseConnectionStringKey": "DevDatabaseConnectionString",
    "KeyVaultUrl": "https://bieno-fs-d-57294-appk-01.vault.azure.net/"
  },
```
3. In Visual Studio you must be logged into your Unilever account, this Authenticates you with KeyVault. If you have previously been logged in to another account (eg Tessella), you may need to log out and log back in using your Unilever account.
4. You must have been granted access to the instance of KeyVault that you are trying to connect to.
