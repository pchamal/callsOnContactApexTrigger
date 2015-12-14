## SFDC Trigger for counting calls on a Contact Object

I love the [The law of diminishing returns] (http://www.investopedia.com/terms/l/lawofdiminishingmarginalreturn.asp). 

Calling me an irrational dork but it definitely ranks in my top 3.14 Economic laws.

![alt text](http://vignette2.wikia.nocookie.net/economics/images/d/dd/Marginal_Utility.JPG/revision/latest?cb=20060726191218 "Diminishing Returns Graph")

Anyways, you can literally apply this law over and over again when you are doing SaaS sales. One pretty obvious application of this is around cold calling on your contacts. 

After a certain number of calls, you should really just give that contact a rest and move on to the next one. Opportunity cost is everything when you are trying to be a efficient and highly productive sales org. 

So, our org is working on implementing a field that will count all outbound calls on any account and increment the numeric value of field by one (integer++ FTW!!). Having a set field that aggregates this data will allow us to run some pretty awesome reports. 

One awesome outcome would be to only call on contacts where the number of dials on that contact is < 11 (11 seems to be our orgs cut off point. If you haven't gotten a demo in 11 calls, then you should probably find a different contact or let that contact(and Account) rest. 

You could populate your Inside Sales power dialer with untouched contacts (or minimally touched contacts). 

Since you can't execute this through workflow rules, I wrote an Apex Trigger.  

Below is the apex trigger I wrote which essentially counts all tasks that are 'completed' AND task type = 'call' and increments a number field (Number_of_Dials_on_Contact). You could add other criteria to only count calls that quarter or in the last "N" months or so. You can check out how to manipulate your SOQL queries [here] (https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql_select_dateformats.htm). 

```Java
trigger totalCalls on Task (after insert) {
set<String> whoIDs = new Set<String>();
for (Task t: Trigger.new) {
        whoIds.add(t.WhoId);
    }
    List <Contact> cons = [SELECT Id, Number_of_Dials_on_Contact__c FROM Contact WHERE Id =: whoIds];
    Map <String, Task> TaskMap = new Map<String, Task> (); 
    
    for (Task t : Trigger.new) {
    List<Task> tasks = [SELECT Id,Type,Status FROM Task WHERE (whoId = : t.whoId) AND (ActivityDate=LAST_N_MONTHS:2)];
            
    integer numDials = 0;
    
    for (Task a: tasks) {
    if (a.Status.equals('Completed') && a.Type.equals('Call')) {
    
    numDials++;
                }
            }
    
    for (Contact c: cons) {
    if (c.Id == t.whoId) {
     c.Number_of_Dials_on_Contact__c = numDials;
                }
            }
        }
    
       update cons;
   }
 ```
 