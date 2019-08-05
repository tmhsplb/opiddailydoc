# Infrastructure
The infrastructure of project OPID Daily refers to the tools and technologies used to develop OPID Daily, exclusive of the implementation itself. This
section will be useful to a developer wanting to maintain and further develop OPID Daily.  It describes both the desktop development environment and the
AppHarbor deployment environment for application OPID Daily.

## Hosting Environments
There are 3 hosting environments for OPID Daily: desktop, staging and production. They differ in the database connection string used by each.
The connection string is configured as the value of variable SQLSERVER_CONNECTION_STRING in the `<appSettings>` section of
Web.config. The static value configured there is used by the desktop environment. The static value is overwritten by injection (at AppHarbor)
when Eref is deployed to create either a staging or production release. The transformation files Web.Staging.config and Web.Release.config
play a role in these deployments. The staging deployment at AppHarbor has its Environment variable set to Staging to force Web.Staging.config
to be used upon deployment. This is done in the Settings section of the deployed application. The production deployment at AppHarbor has its
Environment variable set to Release by default. This causes Web.Release.config to be used upon deployment.
 
## Visual Studio Project
The Visual Studio 2019 (Community Edition) project representing application OPID Daily was developed using an ASP.NET Identity 2.0 sample project 
developed by Syed Shanu as a starting point. The project is described in the 
[excellent CodeProject article ASP.NET MVC Security and Creating User Role](https://www.codeproject.com/Articles/1075134/ASP-NET-MVC-Security-And-Creating-User-Role).

The sample project uses the Visual Studio MVC5 project template and makes use of Katana OWIN middleware for user authentication. The use of Katana is 
built into the ASP.NET Identity 2.0 provider used by the project template, as is explained in the CodeProject article.

On the Properties page of the Visual Studio project remember to select Local IIS as the server and click the Create Virtual Directory button to set

     http://localhost/OpidDaily
     
as the Project Url. These two actions create an application called OpidDaily under the Default Web Site in IIS and enable project OpidDaily to be run
in a desktop version of IIS under this Url. Without this, the desktop IIS cannot be used to host the application. See the section on configuring IIS 
below.

Visual Studio incluse a SQL Server, but it is more convenient to have SQL Server Management Studio (SSMS) available. This requires a (lengthy) 
download. The version of SSMS used for the development of the OPIDDaily application is v18.0. Note that SQLEXPRESS is a separate download from SSMS; it 
does not come bundled with SSMS.

## SQL Server Express
The desktop version of OPID Daily makes use of a SQL Server Express to store information about clients. 
 
The SQL Server Express database for OPidDaily was created by executing the SQL query

    create database OPIDDailyDB  
  
executed inside of SSMS. With this database selected in SSMS, there are two SQL queries that need to be executed to enable IIS
to talk to SQL Server Express. The first query is

       CREATE USER [NT AUTHORITY\NETWORK SERVICE]
       FOR LOGIN [NT AUTHORITY\NETWORK SERVICE]
       WITH DEFAULT_SCHEMA = dbo;

This query creates the database user NT AUTHORITY\NETWORK SERVICE. The second query is

      EXEC sp_addrolemember 'db_owner', 'NT AUTHORITY\NETWORK SERVICE'
      
This query grants user NT AUTHORITY\NETWORK SERVICE the necessary permissions to communicate with IIS. These same two queries also need to be executed
in the AppHarbor database to prepare it to communicate with IIS. See below for information about the AppHarbor deployment of OpidDaily.

It is also necessary to change the application pool identity of application OpidDaily running under IIS to NETWORKSERVICE. See the section on configuring 
IIS.


## Entity Framework Code First
An application based on Entity Framework Code First may have multiple data contexts referencing a single database, as is the case for application
OPIDDaily. In application OPIDDaily a data context is reserved for database migrations used by the ASP.NET Identity subsystem.  

Supporting multiple data contexts was enabled by some manual scaffolding in the code base. In the case of the OPIDDaily application this scaffolding 
consisted  of creating a project folder called DataContexts with two subfolders: IdentityMigrations and OPIDDailyMigrations. Also, a new folder project 
called Entities was added to the OPIDDaily Visual Studio Solution to contain the classes defining the entities used by the solution.

Before the code was run for the first time, the PowerShell command

    PM> Enable-Migrations -ContextTypeName OPIDDaily.DataContexts.IdentityDb -MigrationsDirectory DataContexts\IdentityMigrations
    
was executed. This created the two files

    DataContexts\IdentityMigrations\Configuration.cs
    DataContexts\IdentityMigrations\IdentityDB.cs
    
which initiaized the IdentityDB data context. It is worth taking a look at these two files. In particular, the file IdentityDB.cs was edited to point to 
the application connection string through Config.ConnectionString. 
    
Executing the above PowerShell command allows the ASP.NET Identity system to automatically update the OPIDDailyDB with the ASP.NET Identity tables 
the first time the program is run. Running the program for the first time automaticlly also created the migration

    DataContexts\IdentityMigrations\201906051504117_InitialCreate.cs
    
which specifes the code used to create the ASP.NET Identity tables. It is worth taking a look at this file.

The first time the program was run, the Superadmin user, sa, was created in database OPIDDailyDB. This was done by method
Startup.Configuration on  the toplevel file OPIDDaily\Startup.cs. This file specifies the user sa as the first user in role SuperAdmin (created
on the file). It also points to the toplevel file Config.cs through the reference Config.SuperadminPassword, where the password of user sa is configured.

This project used as its starting point the [excellent CodeProject article ASP.NET MVC Security and Creating User Role](https://www.codeproject.com/Articles/1075134/ASP-NET-MVC-Security-And-Creating-User-Role)
It was necessary to move the class ApplicationDbContext from file Models\IdentityModels.cs on the sample project to file DataContexts/IdentityDb.cs 
to make things work.

The PowerShell command

    PM> Enable-Migrations -ContextTypeName OPIDDaily.DataContexts.OPIDDailyDB -MigrationsDirectory DataContexts\OPIDDailyMigrations
    
was executed to initialize the OPIDDaily data context. This created the two files

    DataContexts\OPIDDailyMigrations\Configuration.cs
    DataContexts\OPIDDailyMigrations\OPIDDailyDB.cs

It is worth studying these two files.

The first entity added to the OPIDDaily project was the class Entities\Client.cs. This entity was connected to the OPIDDDaily data context by the 
inclusion of the declaration

    public DbSet<Client> Clients { get; set; }
    
on file DataContexts\OpidDailyDB.cs.

Running the PowerShell command

    PM> add-migration -ConfigurationTypeName OPIDDaily.DataContexts.OPIDDailyMigrations.Configuration "Clients"
    
caused the migration 

    201906152111225_Clients.cs
    
to be added to DataContexts\OPIDDailyMigrations.

Running the PowerShell command

    PM> update-database -ConfigurationTypeName OPIDDaily.DataContexts.OPIDDailyMigrations.Configuration

then caused table Clients to be created in database OPIDDailyDB by executing the Up method of the above migration. Notice that the Up method refers
to two database columns, ReferralDate and AppearanceDate, which are not in the currently deployed version of table Clients. The data migration

    201907122132153_RemoveToDates.cs
    
was used to remove these columns when it was realized they would not be needed.

The second entity to be added to project OPIDDaily was the class Entities\Visit.cs. This entity was connected to the OPIDDaily data context by the
inclusion of the declaration

    public DbSet<Visit> Visits { get; set; }
    
on file DataContexts\OpidDailyDB.cs.

Since table Visits was intended to be related to table Clients in the OPIDDailyDB by a foreign key, the declaration

    public ICollection<Visit> Visits { get; set; }
    
was added to class Entities\Client.cs. Entity Framework Code First automatically detected this when the "History" migration (described next)
was created.

Running the PowerShell command

    PM> add-migration -ConfigurationTypeName OPIDDaily.DataContexts.OPIDDailyMigrations.Configuration "History"
    
then caused the migration 

    2019071220006570_History.cs
    
to be added to DataContexts\OPIDDailyMigrations. Studying the Up method of this migration, it is seen that the new table Visits to
be created will have a foreign key relationship to table Clients.

Running the PowerShell command

    PM> update-database -ConfigurationTypeName OPIDDaily.DataContexts.OPIDDailyMigrations.Configuration

then caused table Visits to be created in database OPIDDailyDB by executing the Up method of the above migration. As desired, table Visits
has a foreign key relationship to table Clients. To see this, use SSMS to study the columns of table Visits.

Each additional database change requires a pair of commands: an add-migration command followed by an update-database command. 
Executing an add-migration command creates a .cs file in the folder associated with the ConfigurationTypeName. Study this .cs file before executing the
update-database command. If the database changes indicated in the .cs file are not correct, simply delete the .cs file before running the 
update-database command and then try again.

## Configuring IIS
Development of the Eref application was performed under IIS on the localhost machine. This was done so that the development environment would match the deployment environment at AppHarbor as closely as possible.

The localhost application server, Internet Information Services (IIS), was not pre-installed on the localhost; however, it is part of the operating system that can easily be activated. To activate IIS, go to the Programs section of the Control Panel and turn on the IIS feature:

    Programs > Programs and Features > Turn Windows features on or off > Internet Information Services
  
After checking this box, expand it by clicking the plus sign (+) next to it and go to the section

    World Wide Web Services > Application Development Features
      
In this section, check the checkboxes for

        ASP
        ASP.NET 3.5
        ASP.NET 4.6
        
if they are not already checked. This will cause additional Application Pools to be made available to IIS. 

The Eref application is installed as an application under the Default Web Site in IIS as described in the section describing the Visual Studio Project.
The Basic Settings dialog box for application Eref (accessible from the Actions pane of IIS), will give the physical path to the folder containing the 
source code as

    C:\Projects\Eref\Eref
  
The folder above this (`C:\Projects\Eref`) is the folder containing the project solution file, Eref.sln. Do not change it!
Application Eref must be configured to use the application pool .NET v4.5. in the Basic Settings dialog box. (This application pool became 
available by enabling the features described above.)
 
Finally, change the application identity of the selected application pool (.NET v4.5) to NetworkService. To do this, highlight Application Pools on the
IIS Connections panel. This will cause the available application pools to appear in the IIS body panel. Highlight the .NET v4.5 application pool and
then select Set Application Pool Defaults… to display a dialog box that will enable you to change the identity of the application pool. The dialog box
is reachable either from the context menu of the highlighted application pool (under right mouse click) or from the Actions panel on the right of the
IIS display.

The dialog box contains a section labeled Process Model which contains an entry labeled Identity. Selecting the Identity entry adds an ellipsis next to
the bold ApplicationPoolIdentity. Selecting the ellipsis brings up a dialog box with the pre-selected radio button Built-in account. Select
NetworkService from the dropdown menu associated with this radio button. After approving this selection, the Identity column of the application pool
.NET v4.5 will show NetworkService. See the xection on SQL Server Express for how to establish user NetworkService.


## Git for Windows
Visual Studio 2015 (Community Edition) comes with built-in support for GitHub. A new project can be added to Git source control on the desktop by simply
selecting `Add to Source Control` from the context menu of the Solution file in the Solution Explorer. Once a project is under Git source control it can 
be added to a remote GitHub repository by using tools available through Visual Studio. However, a technique preferred by many developers is to use [Git
for Windows](https://git-for-windows.github.io/). Git for Windows provides a BASH shell interface to GitHub which uses the same set of commands
available at GitHub itself. Git for Windows integrates with Windows Explorer to allow a BASH shell to be opened on a project that has been added to a
desktop Git repository. Simply point Windows Explorer at the folder containing the project solution file and select `Git BASH Here` from the context
menu of the folder to open a Git for Windows BASH shell. Then execute Git commands from this shell window. Git for Windows also offers Git GUI, a
graphical version of most Git command line functions. To open Git GUI simply select `Git GUI Here` from Windows Explorer.

## GitHub
A Main Street Ministries (MSM) account has been established at github.com with credentials:

    User name: msmapricot
    Email: apricot@msmhouston.org
    Password: <secret>

Git for Windows is used to create a remote to save to the MSM account. The remote is created in the Git BASH shell by opening the shell on the folder 
which contains the Eref.sln file (folder `C:/Projects/Eref`) and issuing the command

    git remote add origin https://github.com/msmapricot/eref.git
    
Creating this remote only needs to be done once, because Git for Windows stores the remote in Visual Studio. To see this, go to the Team Explorer tab of
the Visual Studio Solution Explorer and select Settings from the Welcome to GitHub for Visual Studio menu. Then select Repository Settings and find the
remote called origin under the Remotes section.

## AppHarbor
AppHarbor (appharbor.com) is a Platform as a Service Provider which uses Amazon Web Services infrastructure for hosting applications and Git as a 
versioning tool. When an application is defined at AppHarbor, a Git repository is created to manage versions of the application's deployment.
The Eref application is defined as an application at AppHarbor to create the production repository of the desktop application. The staging version
of the desktop application is defined by a repository called SEref. The Eref repository at AppHarbor is maintained at the Catmaran paid subscription
level and the SEref repository is maintained at the free Canoe subscription level.

A Main Street Ministries (MSM) account has been created at AppHarbor for deployment of the application. The credentials for this account are:

    User name: msmapricot
    Email: apricot@msmhouston.org
    Password: <secret>

Only user msmapricot can deploy directly to an application in the MSM account. Any other user needing to deploy to an application in the MSM account 
must be declared a collaborator on this application. A collaborator is a user associated with a different account established at AppHarbor. For example, 
my personal account at AppHarbor uses the user name tmhsplb. The user name tmhsplb has been added as a collaborator on the Eref application, thereby
allowing me to deploy to this application.

The remote configured for Eref in Visual Studio is:

    https://msmapricot@appharbor.com/eref.git
    
This remote is configured from a Windows Git BASH shell by the command

    git remote add eref https://msmapricot@appharbor.com/eref.git
    
After the remote is configured in the Git BASH shell, issuing the command

    git push eref master
    
will deploy the master branch of Eref to AppHarbor as application Eref.

If you reset your password at AppHarbor, the 'git push' command will no longer work from the Git BASH shell. You need to have git prompt you for your
new password. To do this on a Windows 10 machine, go to

   Control Panel > User Accounts > Credential Manager > Windows Credentials
   
and remove the AppHarbor entry under Generic Credentials. The next time you push, you will be prompted for your repository password.

An application such as SEref deployed using the free Canoe subscription level at AppHarbor has limitations that make it unsuitable for production
use. Under the Canoe subscription, the IIS application pool of application SEref has a 20 minute timeout, which forces SEref to spin up its resources
again after each 20 minutes of idle time.

The URL of the Canoe version is

    https://eref.apphb.com
    
The free Yocto version of SQL Server is used as an add-on to the Eref deployment. The Yocto version is free but has a limit of 20MB of storage space,
which is adequate for development purposes.  

The remote configured for SEref in Visual Studio is:

    https://msmapricot@appharbor.com/seref.git
    
This remote is configured from a Windows Git BASH shell by the command

    git remote add seref https://msmapricot@appharbor.com/seref.git
    
After the remote is configured in the Git BASH shell, issuing the command

    git push seref master
    
will deploy the master branch of Eref to AppHarbor as application SEref.

The production version of application Eref was created by changing the subsription of application Eref from Canoe to Catamaran. The URL of the 
production version is still https://eref.apphb.com.
    
It is possible to use AppHarbor to generate a custom domain name for an application, but this has not been done for the Eref application. The SQL 
Server of the production version of Eref was upgraded from the free Yocto version to the paid uses the Nano version which has a 10GB storage limit. 
The staging application SEref uses the free Canoe service and the free Canoe version of SQL Server.

On June 6, 2019 I ran into a problem when I first pushed my OPIDDaily solution from my laptop to its GitHub repository and tried to pull the resulting
solution onto my desktop computer. When I tried to run the solution from my desktop it complained about missing part of the path /bin/roslyn/csc.exe.
I found a fix that worked at StackOverflow

     https://stackoverflow.com/questions/32780315/could-not-find-a-part-of-the-path-bin-roslyn-csc-exe
     
There were many proposed fixes, but the one that looked easiest to try was: unload project OPIDDaily and then reload it. This was the first fix
I tried and it worked!

## jqGrid
Almost every page of the Eref application features a grid produced by the jQuery jqGrid component. It was installed into the Eref project by using
the Package Manager command:

    PM> Install-Package Trirand.jqGrid -Version 4.6.0
    
There is a collection of [jqGrid Demos](http://trirand.com/blog/jqgrid/jqgrid.html) that was very helpful during the development of Eref.

## log4net
Application logging is handled by Version 2.0.8 of log4net by the Apache Software Foundation. This package was installed using the Visual Studio NuGet
package manager. The application log for project Eref is maintained as a database table as described in 
[this article](https://logging.apache.org/log4net/release/config-examples.html) describing the AdoNetAppender for log4net. The article includes a script
for creating table Log (renamed AppLog in application Eref). The script must be executed as a query in SSMS to create table AppLog in the database.

The application log is configured by the connection string named ErefConnectionString on Web.config. The value of this connection string is overwritten
when the application is deployed to AppHarbor. See the Connection String section of the Database tab.

## ELMAH
Unhandled application errors are caught by ELMAH. Version 2.1.2 of Elamh.Mvc was installed in project Eref by using the Visual Studio NuGet package
manager. By default, the ELMAH log can only be viewed on the server that hosts the application in which ELMAH is installed. To make the ELMAH log
visible to a client remotely running the application, add

    <elmah>
       <security allowRemoteAccess="1" /> 
    </elmah>
    
to the `<configuration>` section of file Web.config.

To see ELMAH in action, modify the URL in the browser address bar to, for example,

    eref.apphb.com/Admin/Foo
    
This will generate an unhandled error because the MVC routing system will not be able to resolve the URL. Then go to

    eref.apphb.com/elmah.axd
    
to see that this error has been caught by ELMAH.

Installation of the Elmah.Mvc package adds the necessary DLL's and makes the necessary changes to Web.config to configure ELMAH for use. By default
ELMAH will write to a database table called ELMAH_Error. The DDL Script definition of this table is found in a 
[separate download](https://elmah.github.io/downloads/). Download the DDL Script for MS SQL Server from the referenced web page. The script is a .SQL
file which may be executed as a query inside SSMS to create table ELMAH_Error.

The ELMAH log is configured by the connection string named ErefConnectionString on Web.config. The value of this connection string is overwritten
when the application is deployed to AppHarbor. See the Connection String section of the Database tab.

The `<sytem.web>` section of Web.config must configure

    <httpHandlers>
      <add verb="POST,GET,HEAD" path="elmah.axd" type="Elmah.ErrorLogPageFactory, Elmah" />
    </httpHandlers>
    
 and the `<system.webServer>` section must configure

     <handlers>
       <add name="Elmah" verb="POST,GET,HEAD" path="elmah.axd" type="Elmah.ErrorLogPageFactory, Elmah" />
     </handlers>
    
in order for ELMAH to log both on the local IIS and on the remote server at AppHarbor. It is also necessary to set the connection string alias
as described in the Connection String section of the Database tab.

## Google Maps
The referral letter generated by Eref includes a static Google Map showing the location of Main Street Ministries. A new copy of this map is generated
each time a referral letter is requested. The map is generated using the Google Maps static map API. The key for this API is stored in the
`<appSettings>` section of Web.config under the name GmapKey.

## MkDocs
This document was created using MkDocs as was the [MkDocs website](http://www.mkdocs.org/) itself. MkDocs was installed following the guide
on [this page](https://www.sitepoint.com/building-product-documentation-mkdocs/). This guide is useful for setting up the environment; however,
the syntax for the file mkdocs.yml has changed from that described in the guide. The new syntax can be found at in the User Guide section of 
[this document](https://www.mkdocs.org/user-guide/writing-your-docs/#configure-pages-and-navigation).

An MkDocs document is a static website and can hosted by
any service that supports static sites. This MkDocs document is hosted by [GitHub Pages](https://pages.github.com/). The [Brackets](http://brackets.io/)
open source text editor was used to develop the document on the desktop. An MkDocs document uses HTML Markdown for a desktop development version of a
document. GitHub provides a [cheatsheet for Markdown syntax](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).

MkDocs provides a built-in preview server. To start this server, open a BASH Shell on the folder containing the mkdoc.yml file of the project
and execute

    mkdocs serve

Then go to

    http://127.0.0.1:8000
    
in a desktop browser. Pages can be edited and saved while in preview mode. The changes will be reflected in the browser document.

When it is time to publish a version of a document, in a Git BASH shell opened on the folder containing the mkdocs.yml file, issue the command

    mkdocs build

to expand the Markdown version of the document into an HTML version into the /site folder. Then open the GIT GUI on the folder containing
the mkdocs.yml file and use the GUI to create a new GIT repository on the local disk.

Next create a repository to hold the documentation at GitHub. Click on the Settings tab for the newly created repository and scroll down to
the GitHub Pages section. Then select the master branch source and click on the Save button.

After this define a remote called origin for the document:

    git remote add origin https://github.com/msmapricot/erefdoc

This command references the pre-created GitHub repository erefdoc. The remote only  needs to be defined once. It will be remembered by the 
Git BASH shell. 

In the shell issue the following commands:

    git add -A

    git commit -a -m 'Comment on new version of eref document'
   
    git push origin master
   
This will push the master branch of the document to the repository identified by the remote called origin. To view the published document go to:

    https://msmapricot.github.io/erefdoc/site
   
It may take several minutes before the changes are available.
   
   