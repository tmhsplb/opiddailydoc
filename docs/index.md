# Background
Operation ID is a ministry that exists to serve the need of individuals who need to obtain identification documents to support their application for
a variety of services from housing to job application. The ministry is volunteer driven and has been in place for over thirty years. During this time
thousands of individuals have received help to support their identification needs. Help is provided in the form of vouchers that can be used to pay
for services rendered by either the Texas Department of Public Safety for a Texas ID or by the Bureau of Vital Statistics of Texas or another state
to acquire a certified copy of a birth certificate.  Over the years of Operation ID's existence various paper driven pipelines of client processing in
providing identification services have been used. The currently existing pipeline consists of four stages which are executed sequentially.

* **Screening**
* **Check In**
* **Interviewing**
* **Back Office Voucher Writing**

In the **Screening** stage, a client is greeted at the front desk by a volunteer who checks that the client's referral letter is valid and that the
documents the client possesses support the services they are seeking. The referral letter is provided by the agency representing the client to Operation
ID and this agency performs the first level of screening. If this first level of screening is done carefully, the **Screening** stage at Operation ID
can proceed very quickly, allowing the client to proceed to the Check In stage. The referral letter must specify the services that a client is seeking
at Operation ID.  

In the **Check In** stage (also performed by a volunteer at the front desk) the client is entered into the Operation ID database if he/she is
not already in the database. The database is maintained by the Apricot database driven application software provided by the vendor
[Social Solutions](https://www.socialsolutions.com/). Apricot is used to capture a client's demographic data together with a history of a client's
previous visits to Operation ID. The demographic data includes a client's name, date of birth and age (a calculated field based on date of birth). The
demographic data also includes a link to a household table, if a client is part of a
household being processed together.

Apricot stores basic demographic data in a table it refers to as the Client Tracking table and stores client visits in a related table called the
Client Tracking Document Folder. (In terms of database technology, the Client Tracking Document Folder is related to the Client Tracking table by
a foreign key.) Operation ID makes use of the Client Tracking Document Folder to record services provided to a client during previous visits. Whenever
a service is provided to a client (example: writing a check for a birth certificate), the service is recorded by date together with the number of
any issued check. At a date after the date of service, the disposition of an issued check is recorded. Most often this disposition will be either
Cleared, indicating that the check has been used by the client or Voided, indicating that the check is no longer valid.

During the **Check In** stage at the front desk, a client's history of previous visits (if any) is manually copied onto a small slip of paper which is
then stapled to the client's referral letter. Previous visits may make a client ineligible for a service being requested. Operation ID enforces
a twice-in-a-lifetime policy for birth certificate or ID services. If a client's history of previous visits reveals that they have twice
previously been issued checks for a birth certificate and both checks have been used, then the client is not eligible for another check for a
birth certificate. Similarly, if a client has twice previously been issued checks for an ID (either a Texas ID or a Texas Driver's License) and both
checks have been used, then they are not eligible for another check for an ID. If a client is ineligible for a requested service, then they are so
informed at the front desk. A client ineligible for a requested service may appeal their ineligibility and may, at the discretion of Operation ID
management, be granted an exception.

Most clients are either first time clients (no previous visit history) or clients who have not yet exhausted eligibility for a requested service. In
either case, such a client is ready for the **Interviewing** stage. A client ready to be interviewed may have to wait for a (volunteer) interviewer to
become available. Clients are processed on a first-come-first-served basis and frequently have to wait for an available interviewer. When an interviewer
picks up a waiting client, the client's requested services are reviewed and paperwork in support of these services is completed. If the client is
representing themselves alone, the paperwork generation may go fairly quickly. If the client is requesting an out-of-state birth certificate, more time
is required, because the paperwork required to acquire a certified birth certificate varies from state to state. If the client represents a household,
then paperwork for each member of the household is required and this may slow down the interviewing process considerably. In any event, under the
current client processing pipeline, all paperwork required is completed during the interviewing process before the back office can generate any voucher
for a client.

When paperwork filled out during the **Interviewing** stage is sent to the back office, the **Back Office Voucher Writing** stage may begin. This stage
may not  begin immediately upon the delivery of the completed paperwork for a given client, because a backlog of previous clients may need to be
processed first. When a client's paperwork is eventually processed by the back office, what remains to be done is straightforward. A voucher for each
service requested by a client is first generated by use of Quickbooks (operated by a volunteer). Then the voucher number of each generated voucher is
recorded in the Apricot database (by another volunteer) in the client's visit history. This record keeping step is what enables enforcement of the
twice-in-a-lifetime service policy mentioned above. After vouchers have been generated and recorded, they are delivered to the client and the pipeline of
client processing is completed.

What has been described above may be referred to as the **same-day-service model** of client processing. An alternative model, Remote Operation ID, will
be described in a section below. Remote Operation ID will be described as a virtual platform providing the same client services without the need for a
client to appear at Operation ID.

## The OPIDDaily Website
The OPIDDaily website was designed to partially automate the pipeline of client processing at Operation ID by allowing the back office production of
vouchers to proceed in parallel with the interviewing of clients to generate paperwork supporting their needs. OPIDDaily takes advantage of the fact
that a client's voucher needs are already known during the **Screening** and **Check In** stages. This allows the front desk volunteers to send the back
office volunteers information which allows the **Back Office Voucher Writing** stage to begin, possibly even before the **Interviewing** Stage has
started.

In support of parallel processing using OPIDDaily a **Service Ticket** describing a client's previous visits (if any) together with the services the
client is seeking is printed on a printer at the front desk by the front desk check in volunteer. It has been suggested that in straightforward
cases this Service Ticket be sent directly to the back office so that back office volunteers could begin processing the service requested on
the Service Ticket in advance of the **Interviewing** stage being completed. In effect, the Interviewing stage and the **Back Office Voucher Writing**
stage would be run in parallel. This has the potential of delivering substantial time savings over the currently existing sequential pipeline. The
**Screening** and **Check In** stages would require a small amount of additional time in order to generate the Service Ticket for a client, but this
extra time would be more than compensated for by the ability to run the **Interviewing** and **Back Office Voucher Writing** stages in parallel.

In practice, a Service Ticket is not issued for each client appearing at the front desk on a day of service. Instead, in order to keep the line moving,
a Service Ticket is only generated for clients with a history of previous visits. Clients with no previous visits are quickly moved through the
front desk after their name and date of birth have been recorded in the Apricot database. The Service Ticket generated for a client with a visit
history is attached to the client's referral letter rather than being sent to the back office. This does not take advantage of the parallel
processing mentioned above, but it has nevertheless proved to be valuable. A client's referral letter must contain the services the client
is seeking. A Service Ticket for a client with previous history summarizes the requested services in a tabular format.

## Remote Operation ID
In March 2020  Operation ID was shutdown due to the coronavirus pandemic. At the time of this writing (April 2020) Operation ID is still shut down
and it is not known when it will reopen. When the shutdown began, work was begun on a simple extension of OPIDDaily that would allow for providing
identification services remotely. Remote operation replaces the same-day-service model of identification services that have historically been provided
by Operation ID by a virtual platform that allows case managers to submit service requests to Operation ID through the OPIDDaily website. The service
request replaces the referral letter used in the same-day-service model described above.

Under the same-day-service model, there is only a single logged in user, the **TicketMaster**. The TicketMaster is a volunteer who sits
at the front desk and uses OPIDDaily to generate a Service Ticket for each client with a history of previous visits. In the remote service model,
case managers who work at Operation ID partner agencies login to the OPIDDaily website to submit service requests on behalf of their clients. The last
step of generating a service request for a client is the printing and signing of the **Case Manager Voucher**. The Case Manager Voucher is a
piece of paper describing the service request. It includes a disclaimer that makes the client aware that requested services may be denied
because of previous visits to  Operation ID for the same services. The Case Manager Voucher must be signed by both the client and the case manager.
The voucher will be retained by the case manager, who will later redeem it for checks covering the requested services.

A service request submitted by a case manager appears on a dashboard monitored by an Operation ID employee. The dashboard is simply a list of
service requests placed by case managers working at different agencies. When a new service request appears on the dashboard, its arrival is first
acknowledged by checking a checkbox on the request. This acknowledgment
will be seen by the case manager who placed the request, informing him/her that Operation ID is aware of the request. The service request will
next be used to generate a Service Ticket which will be printed and added to a collection of outstanding Service Tickets. When Service Tickets
representing enough clients have been accumulated, the status of each represented client will be changed to BackOffice, indicating that the
service requests have been passed to the back office at Operation ID for processing. The respective case managers will see this change in
status for their clients.

Processing a Service Ticket in the back office is exactly the same process that takes place in the same-day-service model. It involves using Quickbooks to
cut checks for services specified by a Service Ticket and recording the check numbers in the corresponding client database entry in the Apricot database.
This processing will be handled by an Operation ID volunteer, as is done in the same-day-service model.

Once the check(s) have been cut for a client, the Operation ID employee monitoring the dashboard changes the client's status to Done indicating
to the client's case manager that processing has completed and checks may be picked up at the Operation ID office. The checks will be released
to a case manager as redemption of the corresponding Case Manager Voucher.

The process of remote operation described above involves only the case manager and Operation ID personnel. A client works with his/her case manager
to generate a service request and then waits for the checks to be picked up at Operation ID by the case manager. Unlike the same-day-service
model, the client never appears at Operation ID. This will be especially important as long as the same-day-service operation remains shut down.

The Remote Operation ID process clearly requires more time to complete than the same-day-service model. As a practical matter, a Case Manager Voucher
will indicate a 30 day period of validity. This is actually a time limit on how long Operation ID is given to respond to the corresponding service request
once it appears on the OPIDDaily dashboard. The dashboard list will be sorted in chronological order of service request expiration dates. A service request will
automatically roll off the dashboard once its expiration date has passed.

## Service History
A Service Ticket lists not only the services a client is requesting but also the services the client has received in prior visits (if any). Listing
previous services rendered is made possible by including a table of previously issued checks in the OPIDDaily database. Each record in the table of
checks contains a check number, the check purpose, the name of the client to whom the check was issued and the disposition of the check, whether it was
Cleared by the bank, Voided by the bank or one of several dispositions private to Operation ID. A check record does not include who the payee of the
check is or the amount of the check. The check is referenced to the Apricot database simply by its unique number, not by the name of the client to whom
the check was given.

Periodically a pair of back office reports is processed by OPIDDaily to update its table of checks with the bank declared disposition of checks. These
reports are generated by Quickbooks. One report lists check numbers of Cleared checks and the other report lists check numbers of Voided checks. This
updating of the OPIDDaily table of checks implies that check numbers are recorded in the OPIDDaily database before their dispositions are known. This is
indeed the case. To effect this, a report is periodically exported from the Apricot database and processed by OPIDDaily to import issued check into the
OPIDDaily table of checks.

By consulting the table of checks in the OPIDDaily database, it is possible to produce a service history for a client. A client with a service
history is referred to as an Existing Client and a client with no previous service history is referred to as an Express Client. The term
express client was chosen to imply that such a client's service request would be both easier and faster to process in the historic same-day-service
model. As explained above, in practice a Service Ticket for an Express Client is not generated. In the remote operation model, a Service Ticket
will be generated at Operation ID for both Existing Clients and Express Clients. Since there will be no referral letter under this model, the
Service Ticket will be the only means of specifying requested services.

Frequently a client will return to Operation ID requesting service before the disposition of a check previously issued to the client is known. In such a case,
a back office volunteer will need to consult a Quickbooks ledger to determine the check's disposition. If the disposition can be determined,
it may affect the service request by the client. In the same-day-service model, a client may need to be informed the he/she is not eligible for a requested
service, because they have exhausted their twice-in-a-lifetime eligibility. This is sometimes a difficult conversation, especially if the client has endured
a long line only to learn they are not eligible for the service they are requesting. In the remote operation model, Operation ID will convey ineligibility through
the OPIDDaily website to a client's case manager. The case manager will then need to remind the client of the disclaimer about potential service denial that the client was made aware
of when they signed their Case Manager Voucher.

## This Document
OPIDDaily is available as a password protected website to registered users. This document describes the design and implementation of OPIDDaily.
The Infrastructure tab describes how project OPIDDaily is maintained on a desktop host and how it is deployed to the .NET hosting service AppHarbor.
The Database tab describes how the OPIDDaily database is managed on the desktop and at AppHarbor. The Implementation tab provides some details
concerning the implementation of OPIDDaily.
