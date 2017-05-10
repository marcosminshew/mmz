# Bullhorn REST API
This document covers the elements of Bullhorn's REST API that are used by the web application. Authentication, the format of requests, and the endpoints used are discussed. Comprehensive documentation of the API can be found at [developer.bullhorn.com](http://developer.bullhorn.com/sites/default/files/BullhornRESTAPI_0_2.pdf). Further recommended reading: [Getting started with the Bullhorn REST API](http://developer.bullhorn.com/articles/getting_started).

## Authentication
Establishing an authorized session with the RESTful service is a multi-step process that involves:
1. Obtaining an OAUTH *authorization_code*
2. Using the *authorization_code* to obtain an *access_token* (and a *refresh_token*)
3. Using the *access_token* to obtain a *BhRestToken*

In order to start the process a username, password, Client Id, & Client Secret are required. Look to ./src/api/ats-router-constants.js for the values used by the app.
The *authorization_code* can be returned manually by executing:

`curl -v "https://auth.bullhornstaffing.com/oauth/authorize?action=Login&response_type=code&username={username}&password={password]&client_id={Client Id}"`

The payload returned will contain a **Location** value which is a URL that contains a **code** parameter. This parameter's value is the *authorization_code*. See ./src/api/ats-router.js for the application's method of obtaining an *authorization_code*.

Next execute the following to receive an *access_token* and a *refresh_token*:

`POST https://auth.bullhornstaffing.com/oauth/token?grant_type=authorization_code&code={authorization_code}&client_id={Client Id}&client_secret={Client Secret}`

Now use the *access_token* to obtain a *BhRestToken*:

`POST https://rest.bullhornstaffing.com/rest-services/login?version=*&access_token={access_token}&ttl=20`

On subsequent requests to rest-services you'll need to send the value returned as an HTTP header labeled *BhRestToken*. In the same payload a *restUrl* value is returned. This URL includes the *corporate_code* and is used as the prefix to all subsequent requests:

`restUrl = https://rest9.bullhornstaffing.com/rest-services/{corporate_code}/`

Your *access_token* will expire once the time-to-live (ttl) has passed and the server will return a 401 error. Use the *refresh_token* to obtain new tokens:

`POST https://auth.bullhornstaffing.com/oauth/token?grant_type=refresh_token&refresh_token={refresh_token}&client_id={Client Id}&client_secret={Client Secret}}`

Once you have the new *access_token* make a subsequent /login request to obtain a new *BhRestToken*.

## Making GET requests
A **GET** request is comprised of several parts:
- The *restUrl*
- The **query** or **search** endpoint
- The Entity name
- The search string
- The list of fields to return
and then optionally you can specify the sort order, the start, and count values (for pagination).

A **query** request uses a SQL **where** clause. An example could be:

`{restUrl}query/JobOrder?where=isOpen=true and isPublic=1&fields=id,title,address,categories&count=200&orderBy=id`

A **search** request uses a Lucene search in a **query** parameter:

`{restUrl}search/Candidate?query=isDeleted:false&fields=id,firstName,lastName,category,categories&count=200&sort=id`

For more on Lucene searches [go here](https://lucene.apache.org/core/3_6_2/queryparsersyntax.html).

### search vs. query
Most endpoints support both **query** and **search** endpoints. Candidate supports only **search**.

The payload returned by **search** contains a "total" property which is the count of the records returned. The **query** payload does not contain the total unless you add the parameter `showTotalMatched=true`.

Lucene searches are less flexible than SQL where clauses.

To sort the result set of a **query** request add an **orderBy** parameter to the URL. To sort the result set of a **search** request add a **sort** parameter.

## POST & PUT requests
**POST** is used (unconventionally) to update records and **PUT** is used to create new records. **POST** and **PUT** requests contain only:
- The *restUrl*
- The **entity** endpoint
- The Entity's name
- For **POST** the record's "id" value.

An example **PUT** to _create_ a record:

`PUT {restUrl}entity/Candidate`

An example **POST** to _update_ a record (also used for **DELETE**):

`POST {restUrl}entity/Candidate/39922`

## Endpoints Used
### JobOrder
The list of available positions is returned from JobOrder. In order for a particular position to be available it must have:
- isOpen=true
- isPublic=1
- isDeleted=false

isDeleted must be checked because Bullhorn does not allow JobOrder records to be "hard" deleted but rather sets isDeleted to true (a so-called "soft" delete). Since the UI displays the total number of records found use the **search** endpoint:

`GET {restUrl}search/JobOrder?query=isOpen:true AND isPublic:1 AND isDeleted:false&fields=id,title&count=200&sort=-id`

Note: `sort=-id` puts the list in reverse-creation order.

The `/join-us/available-positions` UI requires the JobOrder's "title" and "address(city,state)". The job detail UI also displays the "description". Filters are available for "categories", "onSite", and "customText12" which is used for 'Job Level'. Going forward new JobOrder records should be certain to provide values for all of these fields.

### Candidate
A Candidate record is created for each application submitted. The same user submitting two applications will create duplicate Candidate records. Some custom fields are used to store the data.

|Custom field|Value Stored|
| --- | --- |
|customInt1|Qualifaction Year (PQE)|
|customText2|Other Bar Admissions|
|customText7|VISA sponsorship required?|
|customText13|Admitted to the Bar?|
|customText17|LinkedIn URL|

Several of the Candidate.Address() fields are required in the Bullhorn UI. To facilitate modification of records using that interface the required Address fields get default values, "--". A default value is also provided for "source" (shown below). A sample payload for creating a Candidate (PUT):

    {
        "firstName": "firstName",
        "lastName": "lastName",
        "name": "firstName" + " " + "lastName",
        "email": "email@address.com",
        "phone": "(nnn) nnn-nnnn",
        "customText17": "a Linkedin url",
        "category": {"id": nnnnnn},
        "customInt1": nnnn,
        "customText13": "yes or no",
        "customText2": "comma,separated,string",
        "customText7": "yes or no",
        "source": "Direct - Axiom website",
        "address": {"address1": "--", "city": "--", "state": "--", "zip": "--"}
    }
The payload returned upon successful completion of the PUT will contain an "id" for the newly created record.

### CandidateEducation
One or more CandidateEducation records can be submitted by the application UI. Only one custom field is used.

|Custom field|Value Stored|
| --- | --- |
|customInt1|Year of Graduation|

You must supply the Candidate "id" value returned from the previous step. A sample payload for creating a CandidateEducation:

    {
        "candidate": {"id": nnnnnn},
        "school": "school",
        "major": "major",
        "degree": "degree",
        "gpa": n,
        "customInt1": nnnn,
        "comments": "comments"
    }

### CandidateWorkHistory
Similarly, one or more CandidateWorkHistory records can be submitted. No custom fields are used. Date values are entered as Unix timestamps in milliseconds. Again you need the Candidate "id". A sample payload:

    {
        "candidate": {"id": nnnnnn},
        "title": "title",
        "companyName": "companyName",
        "startDate": nnnnnnnnnnnnn,
        "endDate": nnnnnnnnnnnnn,
        "comments": "comments"
    }

### JobSubmission
Create a JobSubmission to complete the application process. You need both the Candidate id and the JobOrder "id". Set "dateWebResponse" to the current timestamp. "status" and "source" get default values as shown in the sample below. Setting "status" to "New Lead" will cause the JobSubmission to be treated as a "Job Web Response" in the Bullhorn UI which must be promoted to a "Submission" by a recruiter.

    {
        "candidate": {"id": nnnnnn},
        "dateWebResponse": nnnnnnnnnnnnn,
        "jobOrder": {"id": nnnnnn},
        "status": "New Lead",
        "source": "axiomlaw.com web portal"
    }