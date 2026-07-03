---
title: Zoho Creator
---

## How to build an application
1. Write a [of Work](Scope) for the project
2. Design & build the database
3. Build record summaries & pages
4. Optimize form appearance
5. Take application backup
6. Transfer application to customer account

## Extended manual
### Fetching records from a CRM custom view using API v2
Before, you could use zoho.crm.searchRecords(), but this turned out to be inefficient since it can't be indexed.

This new method is a little cumbersome to get used to, mainly due to the lack of good documentation, but here are some basic instructions.

1. create your custom view in CRM with the criteria you want
2. use zoho.crm.getRecords(), and pass the custom view ID in the "additional parameters" list.

example function:


recordsWithEmail1 = zoho.crm.getRecords("Leads",1,200,{"cvid":"1150000000000000888"});

### Searching Records by Multiple Criteria

// get Active Employees
    activeEmployees  =  Employees  Active;
    info "Active Employees Count: " + (activeEmployees.count());
    // of those, get only those that belong to the appropriate Region
    activeEmployeesInRegion = List();
    qualifiedEmployeesCount = 0;
    qualified = List();
    for each e in activeEmployees
    {
        // info e;
        if (e.Community  ==  region)
        {
            qualifiedEmployeesCount = (qualifiedEmployeesCount  +  1);
            info e.Name + " " + e.Community;
            m = map();
            m.put("Community", e.Community);
            m.put("Name", e.Name);
            qualified.add(m);
            //activeEmployeesInRegion.add(e);
        }
    }
    info "Qualified Employees: " + qualifiedEmployeesCount;
    info activeEmployeesInRegion;
    info "Qualified: " + qualified;

### Report URL Criteria using "OR"

{report URL}?{Field_Name}={A},{B},{C},{D}

### Prepopulating CRM fields
When you want to fill out a CRM Lookup field via script, you have to update both the field deluge name, and the field ID. example:

    oppDetails = zoho.crm.getRecordById("Potentials", input.Opportunity_ID);
    // fetch Contact & fill out lookup field
    contactID = (oppDetails.get("CONTACTID")).toLong();
    contactRecord = zoho.crm.getRecordById("Contacts", contactID);
    input.Contact_ID = contactID;
    contactName = ((contactRecord.get("First Name")) + " ") + contactRecord.get("Last Name");
    input.Contact = contactName;


### Random Numbers
Zoho Creator doesn't have a built-in random function, but you can make use of the random.org API to generate random numbers as needed.


list getRandomNumbers(int min, int max, int count)
{
    // this function written by Ian Melchior during his own valuable personal time. If you find it in your application somewhere, you have him to thank. Truth be told, he enjoyed making it.
    // Documentation here: https://api.random.org/json-rpc/1/basic
    apiKey = "getYourOwnApiKeyFromRandom.org";
    url = "https://api.random.org/json-rpc/1/invoke";
    randomRequest = Map();
    randomRequest.put("jsonrpc","2.0");
    randomRequest.put("method","generateIntegers");
    requestParameters = Map();
    requestParameters.put("apiKey",apiKey);
    requestParameters.put("n",count);
    requestParameters.put("min",min);
    requestParameters.put("max",max);
    requestParameters.put("replacement",false);
    // rP2 = { "apiKey" : apiKey, "n" : 3, ("min") : 1, ("max") : 100, "replacement" : false };
    randomRequest.put("params",requestParameters);
    randomRequest.put("id","4");
    parameterString = randomRequest.toString();
    // edit parameterString
    parameterString = parameterString.replaceAll("\"{","{",true);
    parameterString = parameterString.replaceAll("}\"","}");
    parameterString = parameterString.replaceAll("\\\"","\"");
    //
    requestRandoms = postUrl(url,parameterString).toMap();
    // get random numbers and add to a list
    parseRequest = requestRandoms.get("result");
    //parseRequest = parseRequest.get("random").toList();
    //info requestRandoms;
    //info parseRequest;
    randomNumbers = parseRequest.getJSON("random").getJSON("data").remove("").remove("").toList();
    //info randomNumbers.get(0);
    return randomNumbers;
}


### Send Mail
Normally, Zoho Creator will only allow you to send mail in On Success functions. However, it is possible to send a template On Field Update using the following syntax:
#### Attach a file from a File Upload field to the email
insert the following line in the sendmail function after the Message parameter:




Attachments : file:input.File_upload


#### Attach Record Summary Template as PDF or Inline


sendmail

[
    from:zoho.adminuserid
    to:zoho.adminuserid,"yourname@domain.com"
    subject:"Subject of the email"
    message:"Your message" 
    content type:HTML
    attachments :template:Record_Summary_Name as PDF
]


To insert the template inline, simply modify the last line as follows:

Attachments :template:Record_Summary_Name as Inline

Attachments can't be added to a sendmail task in a custom function to my knowledge: you are likely to get this error:

 In user defined actions,form link name and record id are mandatory for template attachment


What you can do instead is build a report of the entries in that form, then add a custom button column that triggers the sendmail function for that record.

#### Search CRM for Record with Empty Field
Not sure yet, it's not in the manual(https://www.zoho.com/creator/help/script/search-records-in-crm.html). Ticket 16128423 is open for this issue.

### Format DateTime for CRM
If you're trying to add or update a datetime value in CRM, the way Creator formats it by default may not be the same as the format CRM requires. To get around this:

time = now.addDay(1);
formattedTime = time.toString("yyyy-MM-dd") + "T" + time.toString("HH:mm:ss") + "-00:00";


### Formula field documentation
As of July 12, 2017, there is no official documentation of the AND and OR functions for formula field. According to a chat with Zoho Creator support staff, you can use '||' for OR and '&&' for AND.
#### Formula for calculating Age based on Birth Year
 if(Date_of_Birth.addYear(zoho.currentdate.getYear() - input.Date_of_Birth.getYear()) <= zoho.currentdate,zoho.currentdate.getYear() - input.Date_of_Birth.getYear(),zoho.currentdate.getYear() - input.Date_of_Birth.getYear() -1)

### Checklist for making a picklist field into it's own table
1. Create new table, with formula field as the first field

### How to handle phone numbers
Phone numbers are strings, and definitely not integers or anything else you might confusedly imagine.
