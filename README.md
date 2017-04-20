# Booter Blacklist - Monitoring use cases
Booters are Internet services that offers to their customers the ability to launch DDoS attacks on a target of their choice. The name refers to their ability to ‘boot’ the target off of the Internet. They are also referred to as ‘DDoS-for-hire’, ‘DDoS-as-a-Service’, and ‘network stressers’. Booters differ from previous methods of launching DDoS attacks due to two new characteristics: the ease with which any individual can now launch a powerful DDoS attack, and the involvement of a third party, the Booter owner/operator, who provides the infrastructure necessary to perform a DDoS attack as a ‘for-hire’ service available to anyone.

## Why to monitor the access to Booters?
It is known that the number of attacks performed by Booters is non-stop increasing. Then, you ask yourself: how many are they? and which users are accessing Booters and pottentially are launching attacks from my network? To answer these and other questions we developed the Booter blacklist and made available at http://booterblacklist.com. Our main intention is (1) to mitigate the access of Booter customers by notifying them about the implications of Booter usage, (2) create evidences to prossecute responsible for attacks and (3) mitigate the overall phenomenon. Please consider the scenario depicted bellow: 


In this scenario, an user of your network access a Booter and a couple of minutes later you observe a DDoS attack (against a third user of your network). Although (you would say that) "this is not likely to happen", we have found (at SURFnet) many cases that this was exactly the case.

We **DO NOT** recomend the use the Booter blacklist to **BLOCK** users but inseady notify them that they are trying to access a service that can potentially harm other services in the Internet (and of course monitor them). 

## Use case: SURFnet's user monitoring
![surfnet_use_case.ipynb](https://github.com/jjsantanna/booterblacklist_use_cases/blob/master/surfnet/surfnet_case_analysis.ipynb)
