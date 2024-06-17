** Please be patient with any Instana Dashboards, and the UI in general we are experiencing some slowness so please wait for data to load or to be updated **

## Overview:
This alert triggers when an call on the application server is taking more than 1.5 seconds to respond. For these alert we need to figure out what specific calls are taking excessive time. Once we have figured out the expensive calls we need to see how we can improve these calls, internally or with the help of the customer. 

Possible impacted application servers(api server, call center, store1.0 or 2.0), we DO NOT need to monitor the admin server.

By reviewing the custom:podName section of the alert we can see which application server is impacted from the alert overview on slack. (High lighted by red underline) 

## Severity: 

This should be a severity 1 case until the issue gets mitigated.  If this alert is re-triggering i.e. 10+, then keep the severity as sev1
until we have a good understanding of the root cause and agreement from the client to lower the severity.

Instructions - Open a Severity 1 ticket and engage Support immediately for review 


## Error/Condition: 
![error condition](https://media.github.ibm.com/user/348473/files/8b588d0c-4739-4c8c-97d7-18027871fc28)

> Alert triggers when a call on the application server is taking more than 1.5 seconds to respond. 

Here we can see in the highlighted red box the impacted application servers. (** There will be a  custom:serverName: attribute which will indiciate the impacted the application server. **)

![](https://media.github.ibm.com/user/348473/files/a698aa22-9d01-4441-a757-383ed2b0e80a)

When looking at the filters we can see how we are querying the data further:

* call > latency is >= 1500 (This is showing us only call that are greater then 1500 ms) 

* Deployment Label > server-name does not equal admin server (Also here we can see we are excluding the admin server)

* Also here we can see Senthil has a added to remove oms-abasc-prod-1 as there is known issue for this particular client/env.
## Symptoms: 


> Describe the symptoms of the issue. 


## Related Alerts:


This issue can be related to DB lock wait/Long running query, high GC usage issues. If IV calls are impacted then this could be related to IV as well.


## Mitigation:


1. Start with opening the alert, and clicking on the + icon to get more information about the alerting conditions and why the alert has occurred. 

![](https://media.github.ibm.com/user/348473/files/649a6747-5ffe-4e61-9a71-7b2112c14725)

2. Once you have arrived to the following page click on Analyze calls 
![](https://media.github.ibm.com/user/348473/files/f80d5613-6336-4e73-ae84-e6398c6a5c0b)

> Now you can beginning to mitigate the issue once you have clicked on Analyze Calls

3. First we need to find the call that have the highest latency we can do this by using the table to filter calls that have the highest latency. 

* Here we can see the highest calls as we see latency is filter by highest to lowest (Arrow is facing down)

![](https://media.github.ibm.com/user/348473/files/d753067f-c6c5-4823-acc6-6a5907468fa3)

Here we can see on a general overview that the api server (from the application server is impacted) 

Before we look into the specific call we can see if there are serveral call that are causing an issue or just one specific call. 

* To do this we can add an additional group filter of the call name (We have added this filter highlighted by the red box)

![](https://media.github.ibm.com/user/348473/files/f63ad5c0-0839-43a6-bd7a-06a2f12d7898)

* Grouping the following calls allowed us to understand that there is actually only 1 call (Because we can see it say only 1 Groups) that is expensive. By using the group functionality we also have a average mean of this call taking about 35.5 s per call and there were 41 calls made.

* If there were multiple calls we would review them all and share information about all call with customer. 

When we expand the group for the specific call it self we see the following:
![](https://media.github.ibm.com/user/348473/files/6f7d6c01-9819-4493-bb18-400c0197e915)

Now if we can see the highest call is taking 36.257 ms, to review the call further we can click on the individual call. 

Reviewing the call further.

When reviewing the call further there are important aspects we need to look into. We can use the different sections of call to understand the issue further, first starting with general information of the call.

![](https://media.github.ibm.com/user/348473/files/082a7530-d79d-49a7-bc0a-3619b78b5236)

* Here call name is: smcfs/restapi/executeFlow/BKSCreateReservation
* Type of call: Post
* We can see there are 12 errors logged 
* Total duration is 36.26 s

We can review the timeline to see a high overview of all the call made and the timeline also let us know when error were logged. Here we can see all the errors logged at the very end. 

1)Calls Subsection
![](https://media.github.ibm.com/user/348473/files/dcd36498-334b-4367-8377-b9799a3fda1c)

This view is very important as we can see all the different calls taking place and we can easily see which subcall is taking the post time here we can see the following sub call is taking the most time. (We can click the specific call - indicated by the red box) and once we have clicked the specific call we can see details of that call is reflected in the green box section. 

Now we know the specifically in the:

smcfs/restapi/executeFlow/BKSCreateReservation

 →  /inventory/us-8ef5cba7/v1/availability/node/ (The following sub call is taking the most time) making this overall call expensive. 

Based on the green section we can also see this read is timing out..

2)Logs subsection
![](https://media.github.ibm.com/user/348473/files/34c970d1-b928-4196-915f-08e882e99ee2)
* In the log subsection we can see all the error that were logged in the process through out the timeline to get a better understanding as well. 

Here we can see a IVintegrationApi timeout: this is also related to the above IV timeout call we noticed above

SockTimeoutExceptions: this should explain the read timeout 

_Also here we can see the trimmed message to get the error stack of this issue as well: _
_Trimmed Message: [1697870016655]<?xml version="1.0" encoding="UTF-8"?> <Errors> <Error ErrorCode="java.net.SocketTimeoutException_ReadTimedOut" ErrorDescription="Failed to invoke IVIntegrationApi : Read timed out" ErrorRelatedMoreInfo=""> <Attribute Name="ErrorCode" Value="java.net.SocketTimeoutException_ReadTimedOut"/> <Attribute Name="ErrorDescription" Value="Failed to invoke IVIntegrationApi : Read timed out"/> <Error ErrorCode="com.yantra.yfs.japi.YFSException" ErrorDescription="" ErrorRelatedMoreInfo="&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot;?>&#xa;&lt;Errors>&#xa; &lt;Error ErrorCode=&quot;java.net.SocketTimeoutException_ReadTimedOut&quot;&#xa; ErrorDescription=&quot;Failed to invoke IVIntegrationApi : Read timed out&quot; ErrorRelatedMoreInfo=&quot;&quot;>&#xa; &lt;Attribute Name=&quot;ErrorCode&quot; Value=&quot;java.net.SocketTimeoutException_ReadTimedOut&quot;/>&#xa; &lt;Attribute Name=&quot;ErrorDescription&quot; Value=&quot;Failed to invoke IVIntegrationApi : Read timed out&quot;/>&#xa; &lt;Attribute Name=&quot;IVRequestMarkedRetrySafe&quot; Value=&quot;Y&quot;/>&#xa; &lt;Attribute Name=&quot;Transaction-Id&quot; Value=&quot;&quot;/>&#xa; &lt;Attribute Name=&quot;IVRequestURL&quot; Value=&quot;https://api.watsoncommerce.ibm.com/inventory/us-8ef5cba7/v1/availability/node/&quot;/>&#xa; &lt;Attribute _

Once we have review the following we can share the following information with customer or internal team to mitigate the issue further.

## Known Issues:


> if this has occurred in past for customer, link the Support cases here


## Troubleshooting:


> Steps for troubleshooting 

## Background 

> Any history on the issue to help with understanding.
