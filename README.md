
# FB_CaseStateMachine

![alt text](https://github.com/FellowWithLaptop/FB_CaseStateMachine/blob/fbd3ddd37be86def557bec65cdec209f48f872f0/Icon.png?raw=true)

A simple function block to make it easier and clearer to program a state machine in structured text (IEC 61131-3).  If you write your state machine with a CASE instruction and an enum, you might miss functions like an entry and exit step, timeout functions or logging possibilities.  This module should show you how you can implement all these things.  The example was created with TwinCAT and uses the TwinCAT Eventlogger. The basic concept is also possible in Codesys. 
``` pascal
PROGRAM MAIN
VAR
    // state machine
    eState : E_MainStateMachine;
    (*The FB_Init method is used to pass the name of the state machine,
    the event entry and the variable of the state machine*)
    
    fbState : FB_CaseStateMachine('MainStateMachine', TC_Events.TcGeneralAdsEventClass.Pending, eState);

    //some transitions
    bInitDone : BOOL;
    bDoStuff : BOOL;
    timertrans1 : TON;
    nCount : INT;
    i : INT;

    //Error
    bError : BOOL;  
END_VAR
```
``` pascal
(*the case statement is in a repeat loop. If the state is to be changed in the same cycle,
the repeat loop executes the case statement a second time. Be careful when using this function*)
REPEAT

    (*Instead of using an initiger or enum with the Case statement,
      the CyclicCase statement is executed. This is used to pass the step name.*)
    CASE fbState.CyclicCase(TO_STRING(eState)) OF
        E_MainStateMachine.Init:
            
            (*As with a traditional state machine, the state variable can be written to directly*)
            IF bInitDone THEN
                eState := E_MainStateMachine.step0;
                bInitDone := FALSE;
            END_IF
            IF bDoStuff THEN
                eState := E_MainStateMachine.step4;
                bDoStuff := FALSE;
            END_IF
    
        E_MainStateMachine.step0:
            (*Many operations do not have to be performed cyclically.
              The EntryStep and Exit Step properties can be read for this purpose.*)
            IF fbState.EntryStep THEN
                timertrans1.IN := TRUE;
                timertrans1.PT := T#5S;
            END_IF
    
            timertrans1();
            (*Instead of writing an If statement and writing directly to the state variable,
              the TransChangeStep method can be used. When the transition becomes true, the state is changed.*)
              
            fbState.TransChangeState(E_MainStateMachine.step1, timertrans1.Q);
            
            fbState.TransChangeState(E_MainStateMachine.error, bError);
    
            IF fbState.ExitStep THEN
                timertrans1(IN := FALSE);
            END_IF
    
        E_MainStateMachine.step1:
   
            IF fbState.EntryStep THEN //entry step
               nCount := 0;
            END_IF
    
            //do Stuff
            FOR i := 0 TO 10 DO
                nCount := nCount + 1;
            END_FOR
            
            (*Instead of writing an If statement and writing the state variable directly,
            the ChangeStep method can be used to exit the state directly*)
            fbState.ChangeState(E_MainStateMachine.step2);
    
            IF fbState.ExitStep THEN //exit step
                nCount := 0;
            END_IF 
    
        E_MainStateMachine.step2:
        
            (*Some steps are just for organization or preparation.
              You can leave these steps and switch to another one in the same cycle.
              For this purpose you can use the mehode TransChangeStateAndRepeat.*)
            fbState.TransChangeStateAndRepeat(E_MainStateMachine.step3, TRUE);
           
        E_MainStateMachine.step3:
               
            (*do things and repeat the case statement in the same cycle*)
            IF bDoStuff THEN
                fbState.ChangeStateAndRepeat(E_MainStateMachine.step4);
                bDoStuff := FALSE;
            END_IF
            (*Some steps must be executed an exact time.
              Instead of creating TON function blocks you can use the property t.*)
            fbState.TransChangeState(E_MainStateMachine.Init, fbState.t >= T#5S);
            
        E_MainStateMachine.step4:
        
            (*get things done and return to the previous step*)
            fbState.TransChangeState(fbState.PreviousStep, fbState.t >= T#3S);    
        
        E_MainStateMachine.error:
    
            fbState.TransChangeState(E_MainStateMachine.Init, NOT bError);
    END_CASE
    
 (*This checks whether the case statement needs to be executed again.
   Unfortunately there is an error in the compiler which makes it necessary to add an "And True".
   Otherwise no Brakepoints are possible.*)
      
UNTIL fbState.NotToBeRepeated AND TRUE END_REPEAT
```

While executing the code, the current state and the execution time of the previous step is logged.
Note that step 2 and 3 were executed in one cycle and therefore the execution time is less than the cycle time.

![alt text](https://github.com/FellowWithLaptop/FB_CaseStateMachine/blob/4897f6d4fc89761bb30042eedbef254c7fd7f920/LoggedEvents.png?raw=true)

