# KidVM
Virtual CPU, Assembly, Computer, Win32 Interpreter, Glue code/Trampolines etc. Experiments

~ 2013? ...

C++, Win32, ...


```C++
//VirtualCPU_Win32.cpp

//(...)
SystemVM* sys = systemVM;	
//(...)

void CreateSystem(){
	 char* pathSrc = pathToSource;	   
	 if (systemVM == NULL)
	 {
	  systemVM = new SystemVM();	  	   
		systemVM->cpu->iTrace = 1;
		systemVM->assembler->ReadInstructionsFromSource(pathSrc, "vm_instructionsT"); //from Enum
	 }
  
	char sA[100];
	char sB[20];
	char sC[20];
	char sD[50]; //path
	char sE[50];

	DWORD addrs[10];
	DWORD top = 0;

    //strcpy(sA, "vmin.txt");
    //memcpy(&RAM.cMapAddress[addr], sText, strlen(sText)+1); //include the 0

	DWORD stringAddrStart = 50000;
	strcpy(sA, "vmin.txt");	
	strcpy(sB, "wt");		
	strcpy(sC, "Written by VM...");	

	DWORD ptrStr = stringAddrStart;
	addrs[0] = stringAddrStart;
	ptrStr = AddStringToVM(systemVM->RAM, ptrStr, sA);
	
	addrs[1] = ptrStr;
	ptrStr = AddStringToVM(systemVM->RAM, ptrStr, sB);

	addrs[2] = ptrStr;
	ptrStr = AddStringToVM(systemVM->RAM, ptrStr, sC);
	 
  int subAddr[20] = {13000, 13050, 13100, 13150};
  int subAddrSize = 4;
    //JSR_VM_IMMED_ABS_U
  char source[] = "PUSH_VM_IMMED_I 14343 0\n CALL_VM_EXTERN 20 0\n CALL_VM_EXTERN 17 0\n POP_VM_R0_I 0 0\n JSR_VM_IMMED_ABS_U 13150 0 \n JSR_VM_IMMED_ABS_U 13000 0 \n JSR_VM_IMMED_ABS_U 13050 0 \n JSR_VM_IMMED_ABS_U 13100 0\n JSR_VM_IMMED_ABS_U 13150 0\n HALT_VM_CPU 0 0"; //JSR_VM_IMMED_ABS_U 13300 0 == FILE ... commented #2-12-2013JE_VM_IMMED 15 0\n  CALL_VM_EXTERN 21 0
   char subroutine[] = "PUSH_VM_IMMED_I 99999 0\n CALL_VM_EXTERN 20 0\n CALL_VM_EXTERN 17 0\n POP_VM_R0_I 0 0\n RET_VM 0 0"; //JE_VM_IMMED 15 0\n  CALL_VM_EXTERN 21 0
   char proc2[] = "PUSH_VM_IMMED_TWO_I 12345 67890\n ADD_VM_STACK_STACK_I 0 0 \n CALL_VM_EXTERN 20 0\n CALL_VM_EXTERN 17 0\n POP_VM_R0_I 0 0\n RET_VM 0 0"; //JE_VM_IMMED 15 0\n  CALL_VM_EXTERN 21 0
   char proc3[] = "PUSH_VM_IMMED_I 25 0 \n POP_VM_R0_I 0 0\n PUSH_VM_IMMED_I 32 0\n PUSH_VM_REG_I 0 0\n CMP_VM_STACK_STACK_I 0 0 \n JGT_VM_IMMED 18 0 \n ADD_VM_REG_IMMED_I 0 1\n PUSH_VM_R0_I 0 0\n CALL_VM_EXTERN 17 0 \n POP_VM_REG_I 5 0\n JMP_VM_IMMED -24 0\n RET_VM 0 0"; //JMP_VM_IMMED_I --> JMP_VM_IMMED//-24 OK == -27 CMP: push a, push b --> JGT == b > a! //15 -- 12  //POP_VM_R0_I 0 0 \n POP_VM_R0_I 0 0 
   char proc4[] = "PUSH_VM_IMMED_I 88888 0 \n CALL_VM_EXTERN 20 0\n CALL_VM_EXTERN 115 0\n CALL_VM_EXTERN 17 0\ CALL_VM_EXTERN 17 0\n POP_VM_REG_I 5 0\n CALL_VM_EXTERN 20 0\n CALL_VM_EXTERN 180 0 \nRET_VM 0 0"; //CMP: push a, push b --> JGT == b > a! //15 -- 21  //POP_VM_R0_I 0 0 \n POP_VM_R0_I 0 0 

   //char proc5[] = "PUSH_VM_IMMED_I 500 0 \n CALL_VM_EXTERN 20 0\n CALL_VM_EXTERN 115 0\n CALL_VM_EXTERN 17 0\ CALL_VM_EXTERN 17 0\n POP_VM_REG_I 5 0\n CALL_VM_EXTERN 20 0\n CALL_VM_EXTERN 180 0 \nRET_VM 0 0"; //CMP: push a, push b --> JGT == b > a! //15 -- 21  //POP_VM_R0_I 0 0 \n POP_VM_R0_I 0 0 
  //PROC 5
   
	DWORD addrBaseIn = 13300;
	//CPU.SetRAM(systemVM->RAM->dwMapAddress);	
	systemVM->cpu->SetPushBase(addrBaseIn);

	CpuVM* CPU = systemVM->cpu;
	CPU->PushInstruction(PUSH_VM_IMMED_TWO_U, addrs[1], addrs[0]); //"wt", "vmin.txt" 
	CPU->PushInstruction(CALL_VM_EXTERN, 201, 0);
	CPU->PushInstruction(POP_VM_R0_U, 0, 0);	 //Ptr to the FILE*  
	CPU->PushInstruction(PUSH_VM_R0_U, 0, 0);	 //Ptr to the FILE* //no need to push?, already there, 7.4.2021
	CPU->PushInstruction(CALL_VM_EXTERN, 19, 0);  //printf %u -- ptr to the file	
	CPU->PushInstruction(POP_VM_REG_I, 2, 0);	 //return value, written elements? to reg 1
	//CPU->PushInstruction(POP_VM_REG_U, 2, 0);	 //Ptr to the FILE*
	CPU->PushInstruction(PUSH_VM_R0_U, 0, 0);	 //Ptr to the FILE*
	CPU->PushInstruction(PUSH_VM_IMMED_U, addrs[2], 0); //Ptr to the string	
	CPU->PushInstruction(CALL_VM_EXTERN, 33, 0);  //fprintf%s
	CPU->PushInstruction(POP_VM_REG_I, 2, 0);	 //return value, written elements? to reg 1
	CPU->PushInstruction(PUSH_VM_R0_U, 0, 0);	 //File ptr	
	CPU->PushInstruction(CALL_VM_EXTERN, 202, 0); //fclose
	CPU->PushInstruction(HALT_VM_CPU, 0, 0);
	////////////////////////////////////////
  
  systemVM->cpu->SetPushBase(11000);
  systemVM->cpu->PushAsmToMachineCode(source, strlen(source), 11000);    
  systemVM->cpu->SetPushBase(13000);
  systemVM->cpu->PushAsmToMachineCode(subroutine, strlen(subroutine), 13000);
  systemVM->cpu->PushAsmToMachineCode(proc2, strlen(proc2), 13050);
  systemVM->cpu->PushAsmToMachineCode(proc3, strlen(proc3), 13100);
  systemVM->cpu->PushAsmToMachineCode(proc4, strlen(proc4), 13150);
     
  systemVM->cpu->Cycle(11000);   
  systemVM->nativeCalls->consoleOut->SetHwnd(edTw[1].hw);
  systemVM->nativeCalls->consoleOut->updateOutput();
  logger.push("VM Readi?");
  
  //RunMachine(system, pathToSource, 1, 0, 0);  
 }
 
  //CALL_VM_EXTERN 17 0\n
	//  0 0 \n CALL_VM_EXTERN 20 0\n CALL_VM_EXTERN 17 0\n RET_VM 0 0"; //JE_VM_IMMED 15 0\n  CALL_VM_EXTERN 21 0
  //PUSH_VM_IMMED_I 99999 0\n 
  //LAST proc should pop the items?
  //!!!MOV_VM_REG_IMMED_,  ADD_VM_REG_IMMED_I  ...
  //  /*
  ////systemVM->cpu->SetRAM(systemVM->RAM); //->dwMapAddress
  //edTw[1].SetText(systemVM->nativeCalls->consoleOut->...  
  //MessageBoxA(0, "VM Ready?", 0,0);         
 
```
Mouse events captured and processed by KidVM with a glue code to call other Windows system functions (MessageBox) etc.

![image](https://user-images.githubusercontent.com/23367640/195446687-ea2a78a2-1020-4f03-9919-badfa6b05e68.png)


