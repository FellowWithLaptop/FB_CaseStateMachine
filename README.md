# FB_CaseStateMachine

![alt text](https://github.com/FellowWithLaptop/FB_CaseStateMachine/blob/fbd3ddd37be86def557bec65cdec209f48f872f0/Icon.png?raw=true)

A simple function block to make it easier and clearer to program a state machine in structured text (IEC 61131-3).  If you write your state machine with a CASE instruction and an enum, you might miss functions like an entry and exit step, timeout functions or logging possibilities.  This module should show you how you can implement all these things.  The example was created with TwinCAT and uses the TwinCAT Eventlogger. The basic concept is also possible in Codesys. 
