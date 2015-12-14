## SFDC Trigger for counting calls on a Contact Object

I love the [The law of diminishing returns] (http://www.investopedia.com/terms/l/lawofdiminishingmarginalreturn.asp). Go ahead, call me an irrational dork but it definitely ranks in my top 3.14 Economic laws.

![alt text](http://vignette2.wikia.nocookie.net/economics/images/d/dd/Marginal_Utility.JPG/revision/latest?cb=20060726191218 "Diminishing Returns Graph")

Anyways, you can literally apply this law over and over again when you are doing SaaS sales. One pretty obvious application of this is around cold calling on your contacts. 

After a certain number of calls, you should really just give that contact a rest and move on to the next one. Opportunity cost is everything when you are trying to be a efficient and highly productive sales org. 

So, our org is working on implementing a field that will count all outbound calls on any contact and increment the numeric value of field by one (integer++ FTW!!). Having a set field that aggregates this data will allow us to run some pretty awesome reports. 

One awesome outcome would be to only call on contacts where the number of dials on that contact is < 11 (11 seems to be our orgs cut off point. If you haven't gotten a demo in 11 calls, then you should probably find a different contact or let that contact (and Account) rest. 

You could populate your Inside Sales power dialer with untouched contacts (or minimally touched contacts). Or you could apply this formula to emails outreaches and see how many emails are having the best amount of success.

Since you can't really derive these insights through workflow/validation rules (salesforce point & click processes), I wrote an Apex Trigger to solve the problem.  

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
    List<Task> tasks = [SELECT Id,Type,Status FROM Task WHERE (whoId = : t.whoId) AND (ActivityDate=LAST_N_MONTHS:2)]; //LAST_N_MONTHS will allow us to only account for activity that occurred in the last 60 days. 
            
    integer numDials = 0;
    
    for (Task a: tasks) {
    if (a.Status.equals('Completed') && a.Type.equals('Call')) { 
    
    numDials++;
                } //the heart of the code, this takes task "a" which falls in the criteria of Status=completed (task must be completed) AND type=call. 
            }
    
    for (Contact c: cons) {
    if (c.Id == t.whoId) {
     c.Number_of_Dials_on_Contact__c = numDials; //takes the integer value from numDials and puts it in the appropriate field, in this case Number_of_Dials_on_Contact
                }
            }
        }
    
       update cons;
   }
 ```
 
####Let me know if you have any questions about this. 
Reach me at [p.hpukar@gmail.com](p.hpukar@gmail.com)
