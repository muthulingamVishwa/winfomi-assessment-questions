# Winfomi-Asssessment-Question-and-Answer 

    write a trigger on the Opportunity object in Salesforce. The trigger should update a custom field called “Total Opportunity Amount” on each Opportunity record. This field should reflect the sum of the Amount fields from all Opportunity records related to the same Account.
    Additionally, you are provided with a class OpportunityT that has a static method ClosedDate. This method updates the CloseDate field of Opportunity records based on the maximum CloseDate of all Opportunity records related to the same Account.

# Trigger Class Opportunity 
    

        trigger OpportunityTrigger on Opportunity (before insert, before update,after insert ,after update) {
    if(Trigger.isInsert){
        if(Trigger.isafter){
              opp.amount(trigger.new);
           
        }else if(trigger.isBefore){
            
             OpportunityT.ClosedDate(trigger.new,trigger.oldmap);
        }
    }
    else if(Trigger.isUpdate){
             if(Trigger.isafter){
              opp.amount(trigger.new);
             }else if(Trigger.isBefore){
             }
    }
  }

  # Apex Class


  public class opp {
    
    public static Boolean isTriggerExecuted = false;

    public static void amount(list<Opportunity> opplist){
        if(isTriggerExecuted){
            return;
        }
        isTriggerExecuted = True;
       
        
        set<id> AccountId = new set<id>();
       

        for(Opportunity opp: opplist){
            AccountId.add(opp.AccountId);
          
        }
        
         map<id,Decimal> updateopp = new map<id,Decimal>();
        
        for(AggregateResult Result:[Select AccountId,
                                    sum(Amount) Total
                                    From opportunity 
                                    where AccountId in :AccountId
                                    Group by AccountID]){
            
            updateopp.put((id)Result.get('AccountId'),(Decimal)Result.get('Total'));
        }
          list<Opportunity>  opptupdate = new list<Opportunity>();
        for(Opportunity opp: [Select id,
                              AccountID,
                              Total_Opportunity_Amount__c From opportunity
                              Where AccountId in :AccountID ]) {
            if(updateopp.containsKey(opp.AccountId)){
                opp.Total_Opportunity_Amount__c = updateopp.get(opp.AccountId);
                 opptupdate.add(opp);
            }
        }
            if(!opptupdate.isEmpty()){
                update opptupdate;
                   
    }  
        isTriggerExecuted = false;
}
         
}   

public class OpportunityT {
    
    public static void ClosedDate(list<Opportunity> newlist,map<id,Opportunity> oldmap){
        
        
         set<id> AccountId =new Set<id>();
        map<id,date> oppdate=new map<id,date>();
        For(opportunity opp: newlist){
            AccountId.add(opp.AccountId);  
        }
        
        for(AggregateResult  Ar:[Select max(CloseDate) close ,AccountId from Opportunity Where AccountId in : AccountId Group by AccountId]){
             oppdate.put((Id)Ar.get('AccountId'),(Date)Ar.get('close'));
            
        }
        
        
         For(opportunity opp: newlist ){
             if(oppdate.containskey(opp.AccountId)){
             opp.CloseDate=oppdate.get(opp.AccountId);
             }
           
        }
        
      
        }
        
        
    }
