# Implementation
Application OPIDDaily is implemented as an ASP.NET Framework application using the ASP.NET MVC 5 project template provided by Visual Studio 2019
(Community Edition). It uses ASP.NET Identity 2.0 to define a  set of user roles. Each user role is associated with a separate MVC controller.
Controller inheritance is used to share editing functionality across user roles.

It may be useful to upgrade application OPIDDaily to use the more modern ASP.NET Core technology. It may also be useful to provide an alternative to the
free hosting service AppHarbor, which is currently used by OPIDDaily. See the AppHarbor section of the Infrastructure tab for details on this.

The graphical user interface of OPIDDaily is built using Bootstrap 3.0.0. Each user role is associated with its own layout file which defines a
Bootstrap navbar containing links to the OPIDDaily features available to users in the role. The ASP.NET Identity system ensures that a user in a
specified role cannot visit any pages outside of those allowed to users in that role. (See the section on Role Controllers.)

Because of its use of SignalR, application OPIDDaily will always require a server side component. See the section on
SignalR on this tab for a discussion of this.

## Users
OPIDDaily is a role-based system. Each registered user will be assigned a user role by the OPIDDaily administrator.
The role that a user is assigned will determine the OPIDDaily features available to the user. A user's assigned role will depend upon whether
the user volunteers at the front desk, is a volunteer interviewer or is a back office volunteer. In addition, there will be a manager's role
to be assigned to Operation ID managers who want to watch the client processing flow during a day of operation.

The OPIDDaily administrator will be in the role of Superadmin and will have access to features necessary for the maintenance of application OPIDDaily.
There will be only one Superadmin account, but the credentials for this account will be available to maintainers of the application.

## The SuperAdmin User
OPIDDaily defines a pre-registered SuperAdmin user who has privileges to

* create new roles
* invite new users to register in a pre-determined role
* add new agencies

The credentials for the SuperAdmin user are configured on file Startup.cs. File Startup.cs is where the Superadmin role is created. There is only a
single user, the SuperAdmin, with role of Superadmin.

## MVC Routing
Application OPIDDaily uses only the default routing rule supplied by the Visual Studio MVC 5 template. This default routing rule is found in
`.../App_Start/RouteConfig.cs`.

    routes.MapRoute(
      name: "Default",
      url: "{controller}/{action}/{id}",
      defaults: new { controller = "Users", action = "Index", id = UrlParameter.Optional }
    );

For the sake of simplicity, future development of application OPIDDaily should strive to keep this as the one and only routing rule.

## Role Controllers
There is an MVC controller defined for each role defined by the superadmin user. Each controller defined for a role inherits from SharedController to
implement shared functionality. The role controllers manage the views of application OPIDDaily. The implementation of each role controller
defines methods that are accessible through the menubar defined on the layout file for the role. For example, the FrontDeskController - which implements
the FrontDesk role - contains methods ExpressClient and ExistingClient (found on the SharedController) invoked from the menubar defined on file

    ~/Shared/_FrontDesk.cshtml.

This is the layout file for the FrontDesk role. Each view returned by the FrontDeskController includes this layout file, thereby ensuring that a
user in the role of FrontDesk will only invoke methods defined by the FrontDeskController.

Each other role controllers is implemented the same way: each has a defined layout file that is included in each view returned by the controller.
The layout file defines a menubar that specifies the methods that users in the role can invoke.

As protection against unauthorized access to methods of a role controller, use of each role controller is limited to users in the role associated with
the controller. For example, the FrontDeskController is protected by the annotation

    [Authorize(Roles = "FrontDesk")]

Access is then restricted by the functionality of ASP.NET Identity to authenticated users in role FrontDesk.

A quirk of role-based authorization of controllers occurs when a new user is registered in a role. The AccountController register method redirects
a newly registered user to the Index method of the UsersController. This method then in turn attempts to redirect the user to the Home method of
the appropriate controller, based on the newly registered users role. For example, a newly registered user in the role BackOffice will be redirected
to the Home method of the BackOfficeController. Here lies the rub. The BackOfficeController is decorated with the attribute

    [Authorize(Roles = "BackOffice")]

but the newly registered user, even though he/she is in role BackOffice as determined by the UsersController is blocked by the attribute! The
attribute causes the user to be redirected to the Login method of the AccountController. This may be by design and logging in with the same
credentials just supplied in the registration process works. But it would seem that the act of registering should cause the new user to be logged in
upon registration. When a new session is started, a user registered in role BackOffice is not blocked by the Authorize attribute even though the login
workflow is basically the same as the registration workflow. Go figure!

In any event, registration of new users for application OPIDDaily has been handled by the SuperAdmin user who therefore knows the password of
each user. But this is not a problem because users do not have access to private data. User accounts are simply established to separate users from
each other and to restrict their access to portions of the website.  

## The SharedController
Each role controller derives from SharedController. The SharedController implements the shared editor functionality available to the different roles.
Deriving role controllers from a shared controller is an extremely useful technique for building a role-based editor.

## The UsersController
The UsersController controls access to the ASP.NET Identity tables used to store registered users and the roles they are in. The method
UsersController.Index is the entry point for an authenticated user. The role an authenticated user is in determines the method the user will be
redirected to from this entry point.

## jqGrid
The entire implementation of application OPIDDaily is structured around instances of jqGrid appearing in MVC Views. Each jqGrid is initially populated
by a call to an MVC action made through the url property of the grid. For example, the clientsGrid on view FrontDesk/Clients.cshtml is initially
populated by the call

    "@Url.Action("GetClients", "FrontDesk")"

which is the value of the url argument to grid clientsGrid. (Method GetClients is found on the SharedController.) The clientsGrid is updated by
SingalR without requiring a page refresh! This enhances the interactivity of the OPIDDaily application.

Each instance of a jqGrid defines a *pager*, which defines the CRUD operations supported by the grid. Each CRUD operation is
implemented by an MVC action of the role controller associated with the grid.

There is a collection of [jqGrid Demos](http://trirand.com/blog/jqgrid/jqgrid.html) that was very helpful during the development of OPIDDaily.

## Dashboard
The jqGrid used to represent the dashboard of service requests supports filtered server side searching. Performing filtered searches on
the server side is necessary in order to preserve pagination, which will be needed if there are more than 25 service requests active
at any given time. The dashboard is sorted by ascending order of service request expiry. A service request rolls off the dashboard
once its expiry has passed.

The dashboard is made searchable via the configuration:

    jQuery("#dashboardGrid").jqGrid('filterToolbar', { searchOperators: true });

Each column which supports filtered searching (example Agency Name) includes the parameter

    search: true

in its column declaration. Filtered searching is implemented by the method SharedController/GetDashboard.

The parameter sps of type SearchParameters is populated by the POST that calls method GetDashboard. The
POST will include the search string as the value of the jqGrid column name, for example AgencyName or LastName.
Method GetDashboard calls Clients.GetDashboardClients which in turn calls GetFilteredDashboardClients in case
a filtered search has been requested. This is determined by the boolean variable _search. The value of _search
will be true if a filtered search has been requested and false otherwise. The value of _search is false on the
initial call to GetDashboard and on each call to refresh the dashboard.

The parameter setting

    loadonce : false

as part of the jqGrid declaration enables the server side filtering.

## Styling a jqGrid using jquery-ui
The default jquery ui theme that comes with a Visual Studio 2019 project is referenced by
adding the lines

   "~/Content/themes/base/jquery-ui.css",
   "~/Content/themes/base/jquery.ui.theme.css"

to the Content/css bundle on AppStart/BundleConfig.cs and adding the link

    <link href="@Url.Content("~/Content/themes/base/jquery.ui.all.css")" rel="stylesheet" />

to each file that contains a jqGrid.

The file jquery.ui.all.css contains styling for all the jquery ui components. But this is not
necessary for the OPIDDaily application, because the only jquery component contained on any
page is the jqGrid component. Instead of linking in jquery.ui.all.css by the above link, replace
this link by the link

    <link href="@Url.Content("~/Content/jquery.jqGrid/ui.jqgrid.css")" rel="stylesheet" />

on each page that uses a jqGrid. This links in file ui.jqgrid.css that comes with the jqGrid
download. This file has a problem with correctly displaying the caption of a jqGrid. A new
version was downloaded from the [jqDemos page](http://www.trirand.com/blog/jqgrid/themes/ui.jqgrid.css)
and copying the displayed stylesheet to the file copied.ui.jqgrid.css. When this is linked
in instead of the original ui.jqgrid.css the caption is displayed correctly. However, it still
uses the default theme referenced above.

To change the theme, go to the Download tab on jqueryui.com. Uncheck all components, since
only the jqGrid needs to be styled and this is done by ui.jqgrid.css which came with the
jqGrid download. At the bottom of the page, select a theme (the jqDemos page uses the Redmond
theme) and click the Download button to get a .zip file.

The .zip file includes files named jquery-ui.css and jquery-ui.theme.css. Rename them according
to the selected theme (example: redmond-jquery-ui.css and redmond-jquery-ui.theme.css) and
copy them to the project. The .zip file also contains an images folder which contains images
referenced by the theme. Rename the existing Content/themes/base/images folder to be old-images
and create a new images folder. Copy the images from the .zip file to the new images folder.  

Finally, on file AppStart/BundleConfig.cs, replace the lines providing the default theme (see
above) by the lines

     "~/Content/themes/base/redmond-jquery-ui.css",
     "~/Content/themes/base/redmond-jquery-ui.theme.css"

This will cause the styling on a page containing a jqGrid to be done according to the Redmond
theme.

## jquery Datatables
The Research Table is implemented as a server side datatable. It is dependent on 2 packages downloaded using the NuGet
package manager:

    jquery.datatables  v1.10.15
    datatables.mvc5    v0.1.0

The first of the 2 packages adds folders:

    ~/Content/DataTables
    ~/Scripts/DataTables

The packages should be loaded in the given order.

## NowServing
By convention, wherever the code makes use of a variable with the name nowServing, its value is the id of a client that has been selected from a jqGrid.
Understanding this convention greatly simplifies understanding the code base. For example,
when a row in the clientsGrid defined on FrontDeskClients.cshtml is selected, the JavaScript function that is the value of the onSelectRow property of
the grid is invoked. When the function is invoked, the value passed to its nowServing argument is the id associated with the client represented by the
selected row. The function posts to the server side method NowServing of the FrontDeskController (found on SharedController) via the code

    Url.Action("NowServing", "FrontDesk")

passing the JavaScript variable nowServing in the post as a result of the line

    postData: { nowServing: nowServing }

Method SharedController/NowServing has an optional argument called nowServing. MVC data binding will cause this variable to be bound to the JavaScript
variable in the post.

## Households
Table **Clients** is a list of clients and their dependents. Most clients represent only themselves but some have dependents. A client together with any
dependents he or she may have is referred to as a *household*. The head of a household is a top level member of the table **Clients**. The HH field of
each top level client C is set to 0, that is C.HH = 0. If client D is a dependent of client C then the HH field of client D points back to client C through
the setting D.HH = C.Id. In fact, dependent D belongs to a subgrid of the jqGrid used to
render the **Clients** table. Dependent D is added to the subgrid whose parent entry is the entry for client C. If client C is has dependents, then
C.HeadOfHousehold will be set to true and the row rendering client C in the jqGrid will indicate that client C is the head of a household.

## Messaging and Color Coding
In the same-day-service model of Operation ID, clients would occasionally return checks that had been issued to them. A check was generally returned for
one of two reasons: either the check had expierd or it was issued for the wrong amount. Either of these reasons for needing to return a check may occur
under the remote operations model of Operation ID. To support this a primitive messaging system has been developed to allow the Back Office at Operation
ID to communicated with a case manager via in application text messages.

If a check visible in a client's check history needs to be returned, the client's case manager may initiate a text message exchange with the Back office
by expandng the plus sign (+) next to the check to open up a message subgrid. Clicking the add button (+) on this subgrid will allow the case manager to
create a message to send to the Back Office. When the message is saved, it will appear in red in the subgrid and the client's row will appear in red
in the clients grid. The red row in the case manager's grid indicates that he/she is waiting for a reply. The green row in the Back Office dashboard
indicates that a message has been received.

When the Back Office highlights a client's green row on the dashboard and then selects Service Request from the menubar this will cause the client's check
history  to appear. A check on this history that is highlighted in green indicates that it has a message waiting. To see the message, the Back Office will
expand the plus sign (+) to open up a subgrid of messages. This will expose a message sent by a case manager.

The Back Office may reply to the message by clicking the add button (+) on the subgrid. The entered message will appear in red in the
subgrid upon saving and the client's row in the dashboard will be highlighted in red. This signifies that the Back Office has replied to the
message sent by the case manager and is now waiting for a reply, if any is needed.

There is one additional form of messaging available only to the Back Office. This messaging causes a client's row to turn blue on both the Back Office
dashboard and a case manager's clients grid when the stage of a client turns to either CheckedIn of BackOffice. This let's the case manager know that
action is being taken at Operation ID.

## The AgencyId data field in table **AspNetUsers**
Application OPIDDaily is used by users at Operation ID and by users representing agencies that refer clients to
Operation ID. The AgencyId field in table **AspNetUsrs** will be set to 0 for Operation ID users (for example
the TicketMaster user) and will be set to the AgencyId of their referring agency for other users. The AgencyId
data field in the **Clients** table is set in the same way. So, the AgencyId of a client entered into the **Clients**
table at Operation ID on a day of operation will be set to 0. The AgencyId of a client entered by a user representing
a referring agency will be set to the AgencyId of the referring agency. Clients are tagged in this way to allow
Operation ID users to see all clients and to allow a users from a referring agency to see only the clients referred
by that agency.

## Date Management
Dates are treated differently at AppHarbor from the way they are treated on the desktop. In both environments, a date is
stored as the midnight hour. For example, March 26, 2020 is stored as 2020-03-26:00:00:00:000 in the desktop database as
well as in the AppHarbor database. But if this date is stored in a ClientViewModel object and reported as the value of a jqGrid
column using the date formatter, then the desktop reports it as 03/26/2020 whereas AppHarbor reports it as 03/25/2020. It is not
clear why this happens, but a way to make both environments report 03/26/2020 is to offset the value stored in the view model by
12 hours to noon (2020-03-26:12:00:00:000). This is for done several DateTime data fields in Clients.ClientEntityToClientViewModel.
(A 1 hour offset was tried first, but it did not work.) The same 12 hour offsetting was done with the Date field in a VisitViewModel.
This can be found on file DAL/Visits.cs.

When a jqGrid row containing a date is edited the date posted to the server is the midnight time, not the offset to noon time.
When the row is retrieved in a ClientViewModel, the time will again be offset by 12 hour to noon and will thus display as intended
in the jqGrid.

## The ServiceDate and Expiry data fields in table **Clients**
When the TicketMaster adds a new client to the **Clients* table, the date the client is entered is recorded in the
ServiceDate data field of table **Clients**. The same date is recorded in the Expiry data field. When a Case Manager
user enters a new client into the **Clients** table, the ServiceDate data field is set to the date the client is
entered, but the Expiry data field is set at least 30 days into the future. This expiry date will be printed
on the Case Manager Voucher given to a client by his/her Case Manager. Application OPIDDaily will preload the
Dashboard table seen by the TicketMaster with any clients whose Expiry date has not yet past. This will allow
a client with a Case Manager Voucher to be tracked on the day he/she appears at Operation ID. It is
the client's responsibility to appear at Operation ID on or before the expiry date printed on their voucher.  

## SessionHelper
The SessionHelper class found on fie DAL/SessionHelper.cs is the key method for managing state in application OPIDDaily. This class is
used to store key value pairs in the sesssion context private to each authenticated user.

Managing the value of NowServing is a key use of the SessionHelper. Instead of having methods called GetNowServing and SetNowServing, the
SharedController uses polymorphism to define two methods called NowServing with different signatures. The NowServing method with zero arguments invokes
method SessionHelper.Get and the NowServing method with optional argument nowServing invokes method SessionHelper.Set. These two NowServing methods are
invoked by many methods on SharedController to implement editing functionality private to an authenticated user.

## Service Tickets
A primary goal of application OPIDDaily is the production of *Service Tickets*. A service ticket is a single piece of paper that shows the services
requested by a given client together with the documents the client is supplying in support of his/her service request. In addition, a service ticket
provides a history of service requests from previous visits by the client, if any.

Service Tickets are produced by the TicketMaster at the front desk for same-day-service clients. In practice, the TicketMaster only creates a Service
Ticket for a client with a history of previous visits. This Service Ticket is attached to the referral letter for a same-day-service client and the
client is picked up by an interviewer. The Service Ticket contains a section of supporting documents which an interviewer can use as a worksheet
while interviewing a client. The supporting documents section is not filled out by the front desk admin. The interviewer will assist the client in
filling out paperwork in support of service(s) sought.

It has been suggested that a client's Service Ticket be forwarded to a back office admin before work on a client's paperwork begins. By passing
the Service Ticket to a back office admin, a client's service voucher(s) can be cut and recorded in Apricot while the client is busy with paperwork.
In effect a client's **BackOffice** stage can proceed in parallel with his/her **Interviewing** stage once the Service Ticket has been produced.

A Service Ticket is specified by a Case Manager before production of a Case Manager Voucher. See the Remote Operation ID section of the Background tab
for information about this. A Service Ticket for Back Office use is printed by an Operation ID employee monitoring the remote service requests that
appear on the Back Office service request dashboard display. See the Remote Operation ID section of the Background tab for information about this.

## SignalR
Application OPIDDaily uses version 2.4.1 of Microsoft's SignalR framework for real-time communications. SignalR is installed using the NuGet package
manager. SignalR is a push-technology used by OPIDDaily to update displays without requiring a page refresh.

The SignalR connection hub is defined on file DAL/DailyHub.cs. Invoking a method of DailyHub in the server side code causes a push-notification to
go out to all clients registered to the hub. For example, when a front desk admin adds a new client to the jqGrid managed by the FrontDeskController
(view Clients.cshtml), the method SharedController.AddClient will invoke method DailyHub.Refresh. This method executes

    hubContext.Clients.All.refreshPage();

which sends a push notification to all registered clients of the hub. The registered clients include all other users logged in as front desk
admins. They will each see their view Clients.cshtml update without the need to refresh that view. In this way all grids are kept in synch with the grid
to which the client was added.

SignalR is also used as the means to update a progress bar display used when processing external Excel files containing check data.
The code for doing so is found in this excellent [Code Project article](https://www.codeproject.com/articles/1124691/signalr-progress-bar-simple-example-sending-live-d). Application OPIDDaily makes use of this code. On the client side, the progress bar is included as a partial view by the statement

     @Html.Partial("_ModalProgressBar")

See for example file ~/Views/BackOffice/Merge.cshtml. On the server side the progress bar is updated by executing a call like

     DailyHub.SendProgress("Merge in progress...", i, checkCount);

found on file DAL/Merger.cs.

SignalR is also used by both the special users **Client1** and **Client2** to refresh the tablet computer displays used by these users. (See the section
Managing Users on the Database tab.)

## ExcelDataReader
Application OPIDDaily uses version 3.60 of the ExcelDataReader package downloaded using the NuGet package manager. The package is used to read
externally generated Excel data files containing check data that is imported into OPIDDaily. The code that makes use of the package is found in
files Utils/ExcelData.cs and Utils/MyExcelDataReader.cs
