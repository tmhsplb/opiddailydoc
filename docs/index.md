# Background
Operation ID is a ministry that exists to serve the need of individuals who need to obtain identification documents to support their application for
a variety of services from housing to job application. The ministry is volunteer driven and has been in place for over thirty years. During this time
thousands of individuals have received help to support their identification needs. Help is provided in the form of vouchers that can be used to pay
for services rendered by either the Texas Department of Public Safety for a Texas ID or by the Bureau of Vital Statistics of Texas or another state
to acquire a certified copy of a birth certificate.  Over the years of Operation ID's existence various paper driven pipelines of client processing in
providing identification services have been used.  The currently existing pipeline consists of four stages which are executed sequentially.

* **Screening**
* **Check In**
* **Interviewing**
* **Back Office Voucher Writing**

In the **Screening** stage, a client is greeted at the front desk by a volunteer who checks that the client's referral letter is valid and that the
documents the client possesses support the services they are seeking. The referral letter is provided by the agency representing the client to Operation
ID and this agency performs the first level of screening. If this first level of screening is done carefully, the **Screening** stage at Operation ID
can proceed very quickly, allowing the client to proceed to the Check In stage.

In the **Check In** stage (also performed by a volunteer at the front desk) the client is entered into the Operation ID database maintained by the
Apricot database driven application software provided by the vendor [Social Solutions](https://www.socialsolutions.com/). Apricot is used to capture a
client's demographic data together with a history of a client's previous visits to Operation ID. The demographic data includes a client's name, date of birth and age (a calculated field based on date of birth). The demographic data also includes a link to a household table, if a client is part of a
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

# OPIDDaily
The OPIDDaily application was designed to partially automate the pipeline of client processing at Operation ID by allowing the back office production of
vouchers to proceed in parallel with the interviewing of clients to generate paperwork supporting their needs. OPIDDaily takes advantage of the fact
that a client's voucher needs are already known during the **Screening** and **Check In** stages. This allows the front desk volunteers to send the back
office volunteers information which allows the **Back Office Voucher Writing** stage to begin, possibly even before the **Interviewing** Stage has
started.

Using OPIDDaily, a Service Ticket describing a client's previous visits (if any) together with the services the client is seeking is
printed on a printer in the back office. The back office volunteers can begin processing the Service Ticket document in advance of the **Interviewing**
stage being completed. In effect, the Interviewing stage and the **Back Office Voucher Writing** stage can be run in parallel. This has the potential of
delivering substantial time savings over the currently existing sequential pipeline. The **Screening** and **Check In** stages will require a small
amount of additional time in order to generate the Service Ticket document for a client, but this extra time will be more than compensated for by the
ability to run the **Interviewing** and **Back Office Voucher Writing** stages in parallel.

OPIDDaily will be available as a password protected website to registered users. This document describes the design and implementation of OPIDDaily.
The Infrastructure tab describes how project OPIDDaily is maintained on a desktop host and how it is deployed to the .NET hosting service AppHarbor.
The Database tab describes how the OPIDDaily database is managed on the desktop and at AppHarbor. The Implementation tab provides some details
concerning the implementation of OPIDDaily.
