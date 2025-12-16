.. _leap_architecture:

==========================
Leap Service: Architecture
==========================

Data Protection Overview
========================

Introduction
-------------

D-Wave takes the protection of customer information seriously. We implement industry-accepted
controls and technology and combine enterprise-grade security features with comprehensive
audits of our applications, systems, and networks to ensure your data is protected.

At a high-level, D-Wave's Leap service includes the following security measures:

Data Center and Network Security:

*   Virtual private cloud (VPC), subnets, security groups, and access control lists (ACLs)
    separate network traffic for different classes of applications.
*   “Defense-in-depth” approach with checkpoints at multiple layers.

Application Security:
*   User sign-in requires multifactor authentication and compliance with password policies.
*   Infrastructure and applications undergo vulnerability and external penetration testing.
*   Edge firewall policies protect against common web application vulnerabilities.
*   Operating systems and infrastructure are hardened and patched regularly.
*   Biweekly application updates ensure that any vulnerabilities are addressed quickly.

Data Security:

*   Data communication between customers and the Leap service is encrypted in transit.
*   Database and database backups are encrypted at rest.
*   Database backups are periodic and automated.
*   We ensure that third-party processors meet the security, operational, and reliability
    standards consistent with our customer expectations.

Security Policies:

*   D-Wave has defined security processes to control access to customer data.

Application Monitoring:

*   The Leap service is continuously logged and monitored by an automated system.
*   The incident response team is available to respond to events 24x7.

Compliance:

*   The Leap service received its initial SOC 2 Type 2 attestation in December 2023, and 
    continues to renew this attestation on an annual basis.

    The SOC 2 Type 2 report attests to the service's industry-standard security in terms
    of customer information, data protection, risk management, and other governance and
    operational controls.


Application Overview
====================

Customers access the Leap service through two primary interfaces: the Leap user interface
(Leap UI) and programmatically through the solver API (SAPI) web service. After sign-in,
customers have access to the interactive demos, application examples (resources), and use
their personalized dashboard to view problem and solver details. Customers also have
access to a limited set of self-administration features to change their settings or manage
access tokens for SAPI.

The SAPI hosts a webservice that allows users to send problems to the solvers and retrieve
solutions via JSON-encoded HTTPS. D-Wave's open-source Ocean software developer kit
(SDK), which is available for users to download from GitHub, simplifies and manages this
process. API access is authorized through a randomly generated user access token.

Architecture
------------

Figure 1 is a high-level diagram of the application architecture. The
frontend components and the classical compute portion of the hybrid solvers are hosted on
cloud resources in Amazon Web Services (AWS) datacenters, while the quantum proexssing units
(QPU) and related software components are hosted in D-Wave, partner datacenters.
D-Wave-controlled customer information is stored a central database, referred to as user data
storage (RDS) in the diagram below

Figure 1. Simplified application architecture showing persistent and temporary datastores.

The problem data that QPU and hybrid solvers process is stored in a separate service, the
problem storage service (DSS), using a combination of in-memory cache (Amazon
MemCache), cloud storage (Amazon S3), and a problem database (Amazon RDS). QPU
problem data is sent to the backend for processing by the QPUs and associated compute
hardware.

Application Security
--------------------

Customers access the Leap UI through a supported web browser, Google's Chrome and
Apple's Safari browsers, using the HTTPS protocol and a minimum of TLS 1.2 or greater.
For API access, customers can download the Ocean client to simplify communications with
the APIs. Again, HTTPS is required, and TLS 1.2 is the minimum supported cryptographic
protocol between the client and server.

Authentication
--------------

Customers are authenticated in the Leap service using two factors. The first factor is the customer's
username, currently the sign-up email, and a customer-selected password. The second factor
implements an account verification step using a time sensitive passcode sent to the customer's
email account.

Password Policy
***************

The Leap service's password policy requires that passwords be at least eight characters in
length and include a combination of four types of characters: symbols, upper-case letters,
lower-case letters, and numerals.

Second Factor
*************

The second factor in the authentication process is email-based. The Leap service sends the
user an email, as specified during signup, with a time-sensitive verification code to enter at
the login verification page. This code is a one-time use, randomly generated token that
expires after 15 minutes. This second factor is triggered on first login or when a significant
client change is detected by the server, including but not limited to a browser version or
type change, IP address mismatch, or OS version change. This feature cannot be by-passed
or disabled in the Leap service.

Account Lockout Policy
**********************

Additional safeguards are in place to prevent brute force attacks against unauthorized
access to a customer's account, which is done using an account lockout feature. If an
unauthorized user unsuccessfully attempts to sign in to the Leap service four times in a row,
the customer's account will be locked for a period of 10 minutes, after which the customer
may attempt to sign in again.

Account Change Notifications
----------------------------

Customers are notified through email if an account profile change has occurred. Specifically,
this will happen if a new API token is generated, the account password or other personal
details are changed. To help ensure that these notifications, as well as the second-factor
emails, are sent to the correct address, customers are not able to change their email address.
They must contact support for this change.

User Access Control
-------------------

The Leap service implements role-based access controls with three privilege levels: user,
program manager, and administrator. The user privilege level limits users to modifying only
their account details, including contact information, default project setting, and password
reset functionality. Users with program manager-level privileges have access to the account
details and quota management for the projects that they manage, typically for a single
organization. A program manager also has view-only access to the usage statistics for the
projects that they manage.

Administrator Access and Controls
*********************************

Users with administrator access are limited to D-Wave support personnel only. Administrators
can create, read, update, and delete any user or problem information in the system database.
As an additional level of security, administrators are only able to log into the system from an
allowed IP address.

Sessions Security
-----------------

The Leap service uses several session controls to further enhance client-side security. The
server implements a session timeout to limit the ability for session reuse. Cross-site request
forgery (CSRF) tokens are used to reduce the ability for a malicious site to submit a request
on behalf of a valid user. The Leap service also limits cross-origin requests by implementing
a cross-origin resource sharing (CORS) header to server responses.

Password Security
-----------------

Server side, passwords are encrypted in the Leap service using the Django framework for
password management, implemented in the PBKDF2 algorithm with an SHA256 hash, as
recommended by NIST.

API Security
------------

The Leap service uses a randomly generated 40-character API token to authorize access to
the SAPI. The API token is sent in the encrypted payload of the HTTPS request. The
customer is responsible for securing the token, which should be kept private and not saved
in customer code or other checked-in files.

Summary of token functionality and properties:

*   Authorize client sessions with the API that sends and receives problem data (problem 
    data contains no usernames/passwords).
*   API token is in the following format: [project code]-[random 40 character code].
    40-character code is a UTF-8 random string generated using HMAC-SHA1.
*   Cannot be used to gain access to Web UI or any customer information.
*   The customer is responsible for rotating the token as there is no automated expiration.


Platform Security
=================

Leap Quantum Cloud Service
--------------------------

D-Wave successfully completed an initial SOC 2 Type 2 audit in 2023, and continues to
renew this attestation annually. SOC 2 Type 2 reports can be shared with current or
potential customers upon execution of a nondisclosure agreement.

Third-Party Processors
----------------------

All third-party processors and data centers used by the Leap service have SOC 2 attestation
reports and/or have achieved ISO 270001 certification. Each service is vetted using a
standard evaluation process and vendor risk assessment to ensure they meet our
requirements for security and operational standards.

Customer Data Management  
========================

D-Wave follows industry-accepted practices for the security of customer data, using well-
known third-party services to deliver our products. This section describes where customer
data is stored, the transport mechanisms, and the steps we take to secure this data.

Transmission
------------

All data transmitted between a client and the Leap UI or the APIs is implemented using
HTTPS and a minimum TLS encryption version of 1.2. Additionally, data communication
between the Leap service's components and third-party processors is encrypted using a
minimum of TLS version 1.2.

Storage
-------

Customer data is securely stored and accessible only to authorized D-Wave customer
support and administration personnel for the purpose of diagnosing customer issues,
improving the service, and monitoring system operation. Personal data is stored in a
relational database that is protected from inadvertent access using network and identity
access management (IAM) controls within the AWS environment. 

Customer Information 
--------------------

With respect to customer information, only the third-party processors and data centers listed
in table 1 are used in the processing or storage of customer information. This information is
accessible by a limited group of authorized D-Wave customer service and administration
personnel. The following table details what customer information is collected, which third-
party processors have access to this information, the underlying data center providers, and
the region where the data resides.

Additional Information on Third-Party Services and Datacenters
--------------------------------------------------------------

Leap uses third-party infrastructure to provide a full-service solution for our customers.
Certain customer information, some of it personal identifiable information (PII), is contained
within these systems. This section is updated regularly to provide relevant information on
where this customer information is hosted and how it is used by D-Wave.

Amazon Web Services (AWS)
-------------------------

Amazon's AWS infrastructure is used to host the Leap service; however, customer
information stored in Amazon's datacenters is only accessible to D-Wave-authorized
personnel.

Customer information is stored within an RDS database instance running in an isolated
network security group. Access is restricted to authorized D-Wave components also running
in the same Amazon VPC. The database is prohibited from routing to the internet and
backups are automatically generated, encrypted, and saved to AWS cloud storage (S3).

Customer information stored:

*   Username, email, organization, corporate information.

Security:

*   Certified to a number of international standards, including ISO 27001/17/18 and SOC
    1/2/3. See https://aws.amazon.com/compliance/programs/ for details.


Zendesk (Community and Support Feature)
---------------------------------------

To manage support inquiries in the Leap service, Zendesk is used to implement the
community support content and self-help system. Customers can enter support enquiries
directly in the UI, through the community pages, or through a dedicated email
(support@dwavesys.com). Placing a post in the community requires an active Leap account;
however, read-only access to the community does not require an account, so posts in the
community should be considered public information.

Customer information stored:

*   Customer name, contact details, and support inquiry details. Note that these details are
    stored internally and do not appear on the community pages, instead a user alias is
    shown.

Security:

*   Complies with SOC 2 and ISO 27001/18. See https://www.zendesk.com/product/zendesk-security/.

Salesforce
----------

Salesforce is a customer relationship management (CRM) platform used to manage customer
contracts and lead information for follow-up with customers. D-Wave implements role-
based access to this data for sales, marketing, and system administration purposes.

Customer information stored:

*   Customer name, contact details, phone number, industry, and other information
    collected at sign-up or upgrade events.

Security:

*   Transport Layer Security (TLS) version 1.2 or higher for browser encryption.
*   Token-based authentication between the Leap service and Salesforce for added security.
*   Certified to a number of international standards, including ISO/IEC 27001:2005, SAS 70
    Type II. See https://compliance.salesforce.com/en.

Marketo
-------

Marketo, by Adobe, is a marketing automation platform used by D-Wave to collect customer
insights for marketing purposes. Customers can opt out of receiving marketing or sales-
related information during the account creation process, but this does not limit the
information stored within Marketo. D-Wave implements role-based access to this data for
sales, marketing, and system administration purposes.

Customer information stored:

*   Customer name, contact details, phone number, industry information, site related
    activity.

Security:

*   Token-based authentication between the Leap service and Marketo for added security.
*   Complies with SOC 2 Type 2. See https://www.marketo.com/company/trust/security/.

Problem Data
============

This section describes the problem data flows from the client application through to the
Leap service and the internal handling and storage of this data.

Problem Formulations  
--------------------

Problems for the QPU and hybrid solvers are formulated as mathematical models that take the
following form:

Minimize an objective:


Subject to constraints:



Different solvers support different variations of the model above, but all models consist of
coefficients pulled or derived from data and variables that represent the quantities that are being
solved for. Since raw data is only used to build the model and is not directly submitted to the Leap
service, the business problem and data is obfuscated. For example, an inventory optimization
problem may need to consider existing stock at different locations when deciding whether more
needs to be procured. The coefficients may be constructed by summing stock at all locations or
they may be pulled from each location. Either way, the original data and problem is obfuscated
because the model is uploaded into the Leap service without context for the data or problem.

To ensure business or sensitive data is not inadvertently included in the model, users should not
use sensitive or personal information for variable names, constraint labels, or problem labels.

The solvers return samples or sets of solutions. Each sample consists of a value for each variable,
the energy associated with the sample, and information about constraint violation and feasibility
for constrained quadratic models (CQM) and nonlinear models (NL).

Problem Data Storage 
--------------------

Problem data is securely stored and available only to authorized D-Wave customer support
and operations staff for the purpose of diagnosing customer issues and monitoring system
operation.

Frontend (Leap Quantum Cloud Service Infrastructure On AWS)
***********************************************************

For all solver types, hybrid or QPU, the Leap service stores the problem data in the
following locations:

*   Amazon ElastiCache (Redis database): cache for problem data. Includes no customer-
    identifiable information. The cached problem data expires after 10 hours.
*   Amazon RDS (MySQL database): stores problem data up to a 1000-problem limit per
    customer, then the problem data is discarded.
*   Amazon S3 storage: problem data is customer accessible for up to 30 days, at which
    point it is deep archived. The deep-archived problem data does not contain PII.

Backend (On-Premises Hosted Infrastructure)
*******************************************

For problems sent to the QPU, the data is cached in the following location:

*   Redis database: temporarily stores the JSON-encoded problem to be solved on the
    quantum processor. Once the QPU has solved the problem, the results, solution
    energies, timing information, and qubit states are temporarily stored here until the
    frontend fetches the results. This data does not include customer-identifiable
    information. Problem data is automatically deleted from the Redis database after two
    hours.

Problem Data Transmission
-------------------------

QPU Solvers
***********

QPU problem data is transmitted between the frontend, an AWS datacenter located in
Oregon (US-WEST2) or the EU (EU-Central) region, and a backend located in one of our
regional hosting facilities in Burnaby, Canada, Los Angeles and Huntsville, USA, or Juelich,
Germany. See ADDCrOSSREF section for more details. Data in-transit between the frontend and the
backend is transmitted over a dedicated private network connection (AWS Direct Connect),
or if the link fails, over an encrypted IPsec tunnel:

*   Primary connection: dedicated private connection using AWS Direct Connect over a
    private address space.
*   Failover connection: IPsec tunnel over the internet in case of primary connection failure.
*   No customer identification information is transmitted over the WAN.
*   The approximate transmission time per problem between the frontend and backend is
    10 milliseconds.


Hybrid Solvers
**************

Leap's hybrid solvers execute on classical compute at AWS, known as a hybrid worker.
Because the hybrid worker runs solely on the frontend, the customer problem data does not
transit to the backend where the QPUs are hosted. The hybrid solver, however, does create
QPU sub-problems that are used to update the hybrid algorithm. This generated QPU
problem has no direct mapping to the original customer problem data.

Operational Security Controls
=============================

Network Architecture
--------------------

As previously discussed, the Leap service, deployed at https://cloud.dwavesys.com, uses
Amazon Web Services (AWS), and on-premises infrastructure at D-Wave or a partner
datacenters to host the service. The platform is designed and managed in alignment with
standard industry practices. The diagram in Figure 2 is a high-level schematic of the
platform.

Within AWS, the Leap service is deployed inside a virtual private cloud (VPC) with a
private network address space in the US-WEST-2 and EU-CENTRAL regions, as shown in
figure 2. Each region includes a local problem store that ensures QPU access is performant
for users in these regions. The problem data persists in the region in which it was submitted; 
however, there are no controls in place to ensure this data will remain in the region. Problem 
data can be transferred out of the local region by administrators, for troubleshooting
purposes, or by users through the Leap UI.

Key architectural points are as follows:

*   The VPC is a collection of compartmentalized security groups and network ACLs to
    implement and enforce the “principle of least privilege.”
*   The internet-facing interfaces of the platform are the web security groups. All traffic
    between web-clients and these groups is routed through the WAF and application load
    balancer (ALB) and encrypted using SSL (TLS v1.2 minimum); all other network access
    to internal components is blocked.
*   The data tier is isolated in a separate network security group with no outbound or
    inbound internet access. Only authorized servers within the VPC can access this
    database.
*   The VPC is designed to securely host the frontend software components that
    communicate with the backend (on-premises) services and solver and is isolated from
    D-Wave's enterprise infrastructure.

In addition to AWS resources, the D-Wave datacenter hosts the quantum computing
systems, processing nodes, refrigeration, and control electronics. As mentioned previously,
the quantum computing systems do not receive any personal customer information;
however, problem data is transmitted from the AWS environment to the datacenter. This
transmission path uses an AWS Direct Connect (AWS-DC) point-to-point private connection
with a backup encrypted IP-SEC tunnel, routed over the internet, as a failover connection if
the AWS-DC path fails.

Figure TBD

Application Security Testing and Vulnerability Management
---------------------------------------------------------

At least once per quarter, D-Wave performs an application vulnerability assessment of the
Leap service to ensure that any known vulnerabilities are detected and addressed. D-Wave
also works with independent security consultants to perform penetration testing of the Leap
service at least annually.

Any vulnerability detected is tracked within our management system and remediated
according to its severity, risk, and impact. Additionally, D-Wave subscribes to vulnerability
notices from our service providers (e.g., AWS) and updates/patches operating systems and
other services according to the patching policy.

Data Protection, Recovery, and Destruction
******************************************

D-Wave maintains a separate production environment from our development and testing
environments, to help ensure that customer information is not inadvertently used outside of
the production environment. Customer information is encrypted at rest using AES
symmetric key encryption. Access keys are not shared between the production environment
and the development or testing environments and are rotated using a defined management
process.

Customer information is backed up automatically on a regular interval and encrypted using
the same encryption method as the active database. All backups are stored within the same
national boundary as the active database.

After a customer's Leap service agreement expires, the customer's account
information is deleted manually using a defined process that deletes the account
information and anonymizes any Leap service forum posts. Anonymized customer data is
used to preserve any statistically relevant metrics for monitoring and reporting.

Insider Access
**************

D-Wave has policies and procedures in place to ensure that customer information is secure
from unauthorized insider access. All employee agreements contain confidentiality
provisions, and reference and background checks are carried out for critical and security-
related roles. New employees are only provided access to resources once an employee
record is created in the HR system and removed as soon as possible after an employee
leaves the company. There is also a well-defined onboarding process which includes
training in IT and building security practices. This is reinforced with regular phishing
campaigns and training.

Access to resources that host customer data is limited to those employees who need to
support customers, the application, or infrastructure. These employees are only able to
access the system from within D-Wave's network, either by physical presence or via remote
access through the corporate VPN. A valid username and password, along with a second
factor, are required to connect to the corporate VPN. Additionally, a single sign-on system is
used to authenticate users who need to access console applications hosting Leap service
infrastructure.

Only company-owned and managed IT equipment, including portable media, may be
connected to the corporate network or Leap service resources. Company-owned equipment
that that is allowed on these networks is deployed with a fully managed mobile device
management (MDM) solution which includes an automated threat detection and removal
system to protect against viruses, malware, ransomware, and other common threats, remote 
device management tools, and a compliance module to push device polices such as
encryption and patch management.

Change Management
*****************

D-Wave follows a defined process to ensure that security and customer impacting changes
to the Leap service are appropriately managed before implementation. Feature requests and
changes are tracked and managed through a combination of a requirements planning
process, a sprint-based development cycle, and using tracking and automation tools to
enforce the process. Once changes are approved and scheduled for development and
targeted for release, the changes are deployed to our test and staging environments, where
the team validates the changes and executes regression test suites.

Upon completion of the testing cycle, and approval by the team, the release is deployed to
the Leap service's production environment. Currently, the deployment cadence for the
regular development cycle is every two weeks.

In addition to the regular development cycle, off-cycle deployments can be done in response
to well-defined conditions, typically after identification of a critical security vulnerability.  
These changes are managed using the same process as regular changes, but are limited in
scope, thus allowing for a faster deployment cycle.


Monitoring and Incident Management
**********************************

Several methods are used by D-Wave’s operations and support teams to monitor the Leap
service for service failures, unauthorized access, application performance, and security
events. Application logs contain access information on each request, and this information is
collected and indexed within an elastic search cluster. If an abnormal condition is detected,
the system alerts our 24x7 on-call personnel and the event is automatically logged in the
response management system.

Users can monitor the status of the Leap service at https://status.dwavesys.com. Any active
and planned maintenance activities are displayed at the same location.