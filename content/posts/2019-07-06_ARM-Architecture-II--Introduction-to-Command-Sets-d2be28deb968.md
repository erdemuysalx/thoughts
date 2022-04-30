---
title: 'ARM Architecture II: Introduction to Command Sets'
description: >-
  In this article, finally, we will start to take a glance at ARM commands. But
  before that, you should check your microprocessor’s model…
date: '2019-07-06T18:35:36.612Z'
categories: []
keywords: []
slug: /@erdemuysalx/arm-architecture-ii-introduction-to-command-sets-d2be28deb968
---

In this article, finally, we will start to take a glance at ARM’s arithmetical, logical, data transfer, and branching instructions. But before that, you should check your microprocessor’s model and serial number. Commands might differ according to these two numbers. The reason for this is that ARM has updated its designs due to technological developments.

The arm commands process data in the register. Therefore, the data must be transferred to the registers before arithmetic, logic, or other types of processing. This information is very important to us. The way to do this is to load data into registers by using load/store commands. ARM commands generally take three or two operands. To explain this with an example let’s check the syntax of Instructions:

INS Operan1, Operand2, Operand3

where:

*   INS: Name of the instruction
*   Operand1: Destination register (Operand getting result)
*   Operand2: First register for operation (Operand getting 1st source)
*   Operand3: Second register for operation (Operand getting 2nd source)

Quick example;

_ADD(INS) r0(Operand1), r1(Operand2), r2(Operand3)  
// This_ instructions _set is basically adds r1 and r2 registers and writes the results into the r0 register.  
//!--> r0 = r1+r2;  (in C)_

Some ARM instructions may also take some prefixes and suffixes.

### Types of ARM Commands

#### Data Transfer and Process Instructions

MOV(Move) and MVN(Move Not) instructions: These instructions are almost available in every microprocessor as well as in ARM microprocessors. These are very simple to use. Basically, provide to load or store the desired initial values to the register before processing. The instruction duty and usage structure are as follows.

**MOV:** The `MOV` instruction copies the value of `_Operand2_` into `_Rd_`.

**MVN:** The `MVN` instruction takes the value of `_Operand2_`, performs a bitwise logical NOT operation on the value and places the result into `_Rd_`.

**Syntax:**

```
MOV{S}
```

**Examples:**

_// We can assigned a register value as well as we can assign a constant number.  
MOV r0, r1    
MOV r0, 5  
MVN r0, r1  //_!--> _r0 =_ NOT(r1)

where:

*   S: is an optional suffix. If S is specified, the condition code flags are updated on the result of the operation
*   cond: is an optional condition code.
*   Rd: is the destination register.
*   Operand2: is a flexible second operand.
*   imm 16: is any value in the range 0–65535. Immediate numbers are numerical constants.

#### Arithmetic and Logical Computation Instructions

**Arithmetic instructions:** These instructions are fundamental arithmetic operations for all kinds of processors. They compute the sum (or difference), multiplication (or division)of two registers, and store the result in a register. Multiplication and division operations can also be performed via barrel shift.

**ADD:** The `ADD` instruction adds the value of `_Register1_` to`_Register2_`.

**SUB:** The `SUB` instruction subtracts the value of `_Register2_` from `_Register1_`.

**MUL:** The `MUL` instruction multiplies the value of `_Register1_` by`_Register2_`.

**SDIV:** The `SDIV` instruction divides the value of `_Register1_` by`_Register2_`.

_Note: Division with a constant number might not be supported by all ARM processors._

**Syntax:**

```
ADD{S}
```

**Examples:**

Let’s write assembly instructions that perform the following operations.

> result = (a + b) - (c + d);

> a = b \* b;

> c = d / e;

//!--> result = (a + b) - (c + d);  
ADD   r0, r2, r3  //!--> result = a + b;  (in C)  
ADD   r1, r4, r5  //!--> temp= c + d;  (in C)  
SUB   r0, r0, r1  //!--> result = result - temp;  (in C)

MUL r0, r2, r3    //!--> a = b \* c; (only 32 bits stored)  (in C)  
SDIV  r0, r2, r4  //!--> c = d / e; (signed divide)  (in C)

**Logic (Bitwise) instructions:** The ARM instruction set provides instructions such as AND, OR, XOR, and BIC, which sets, and clears the bits according to the need of the program. Usually, you find these as part of if-else, while statements in high-level languages.

**AND:** The `AND` instruction adds the value of `_Register1_` to`_Register2_`.

**OR:** The `OR` instruction subtracts the value of `_Register2_` from `_Register1_`.

**XOR:** The `XOR` instruction multiplies the value of `_Register1_` by`_Register2_`.

**BIC:** The `BIC` instruction divides the value of `_Register1_` by`_Register2_`.

**Syntax:**

AND `_Register0_`, `_Register1_`, `_Register2_`  //!--> r0 = r1 & r2;  (in C)   
ORR `_Register3_`, `_Register4_`, `_Register5_`  //!--> r3 = r4 | r5;  (in C)   
EOR `_Register0_`, `_Register1_`, `_Register2_`  //!--> r0 = r1 ^ r2;  (in C)  
BIC `_Register3_`, `_Register4_`, `_Register5_`  //!--> r3 = r4 & (!r5);  (in C)

**Examples:**

//!--> r0 = 01101001    
//!--> r1 = 11000111  
AND r3, r0, r1; r3  //!--> 01000001  
ORR r3, r0, r1; r3  //!--> 11101111   
EOR r3, r0, r1; r3  //!--> 10101110   
BIC r3, r0, r1; r3  //!--> 00101000

#### Branch Control Instructions

These instructions change the flow of execution via jumping to another instruction or subroutine (conditional jump (e.g., branch if register == 0)).

**Barrel Shifter:** As we mentioned in our first articles, the ARM’s arithmetic logic unit has a 32-bit barrel shifter that is capable of shifting and rotating operations. In order to be able to do this, the value must be in the register Rm. Briefly, the results are pre-processed by the barrel shifter before being processed in ALU.

MOV r0, r0, LSL #1  // Multiply r0 by two  
MOV r1, r1, LSR #2  // Divide r1 by four  
MOV r2, r2, ASR #2  // Divide r2 by four(signed).

If you remember from digital electronic circuits, shifting a number 1 step left is equal to multiplying the number by 2. Shifting the number to 2 steps right means dividing the number by 4.

The last column specifies how many cycle times each command takes. Although the basic logic of all barrel shifter commands is the same, the only direction of shifting may vary. There are two types of usage of barrel shifters. These uses may be shifting by the value of any register or shifting by a specified fixed number.

In the next article, I am planning to focus on the registers, CPU, and memory structure. Until then, I wish you no blue screen of death.

Continue reading this series of articles;

*   [ARM Architecture I](https://erdemuysalx.medium.com/arm-architecture-174ec191d939)
*   [Installing IDE for Embedded Developing; ARM Keil & Code Composer Studio](https://erdemuysalx.medium.com/installing-ides-for-developing-arm-keil-code-composer-studio-d0dcb069ff74)
*   [ARM Architecture II: Introduction to Command Sets](https://erdemuysalx.medium.com/arm-architecture-ii-introduction-to-command-sets-d2be28deb968)

#### References

*   [https://cseweb.ucsd.edu/classes/fa15/cse30/lectures/lec7\_detailed.pdf](https://cseweb.ucsd.edu/classes/fa15/cse30/lectures/lec7_detailed.pdf)
*   [http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.kui0100a/armasm\_cihcjfjg.htm](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.kui0100a/armasm_cihcjfjg.htm)
*   [http://www.davespace.co.uk/arm/introduction-to-arm/barrel-shifter.html](http://www.davespace.co.uk/arm/introduction-to-arm/barrel-shifter.html)