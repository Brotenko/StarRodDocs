# Writing Code

## Assembly

The N64 uses the NEC VR4300 CPU, closely related to the MIPS R4300i. As such, the compiled machine code in Paper Mario can be disassembled into MIPS assembly language. Star Rod uses a custom MIPS assembler/disassembler to convert between machine code on the ROM and a (more) user-friendly assembly language.

As functions are dumped from the ROM, branches and jump tables are recognized and appropriate labels are created for their destinations. You can use labels as branch targets in your own code. You can also reference pointers (e.g. <code>$Script_Example</code>), script variables (e.g. <code>*StoryProgress</code>), and special functions (e.g. <code>{Func:GetVariable}</code>).

**_Note:_** Register 30 may be referenced as either S8 or FP. Both are accepted.

## Pseudo-Instructions

During the dump, certain sequences of instructions are replaced by pseudo-instructions (PIs) to help find pointers and data structures that would otherwise be 'hidden' within functions. In this way, relocated data structures can have their pointers properly updated. You are encouraged to use these pseudo-instructions in your own code to improve readability and reduce errors.

**_Note:_** Don't worry about the distinction between LIA and LIO. For reversibility only.

### Convenience

| Command           | Description                                     |
| ----------------- | ----------------------------------------------- |
| CLEAR X           | Set X = 0. Equivalent to DADDU X, R0, R0        |
| COPY X, Y         | Set X = Y. Equivalent to DADDU X, Y, R0         |
| SUBI X, Y, 1234   | Subtracts an immediate value.                   |
| SUBIU X, Y, 1234  | Subtracts an immediate value (unsigned).        |
| LIA   X, 12345678 | Load a constant word to register (ADD variant). |
| LIO   X, 12345678 | Load a constant word to register (OR variant).  |
| LIF   FX, 3.25    | Load a constant float to COP1 register.         |

<br>

### Load/Store Address

| Command         | Description                                       |
| --------------- | ------------------------------------------------- |
| LAB    X, ADDR  | Load a byte from ADDR to register.                |
| LABU   X, ADDR  | Load an unsigned byte from ADDR to register.      |
| SAB    X, ADDR  | Store a byte from register to ADDR.               |
| LAH    X, ADDR  | Load a half-word from ADDR to register.           |
| LAHU   X, ADDR  | Load an unsigned half-word from ADDR to register. |
| SAH    X, ADDR  | Store a half-word from register to ADDR.          |
| LAW    X, ADDR  | Load a word from ADDR to register.                |
| SAW    X, ADDR  | Store a word from register to ADDR.               |
| LAF    FX, ADDR | Load a float from ADDR to COP1 register.          |
| SAF    FX, ADDR | Store a float from COP1 register to ADDR.         |
| LAD    FX, ADDR | Load a double float from ADDR to COP1 register.   |

<br>

### Load/Store Table

| Command             | Description                                       |
| ------------------- | ------------------------------------------------- |
| LTB    X, Y (ADDR)  | Load the Yth byte from ADDR to X.                 |
| LTBU   X, Y (ADDR)  | Load the Yth unsigned byte from ADDR to X.        |
| LTH    X, Y (ADDR)  | Load the Yth half-word from ADDR to X.            |
| LTHU   X, Y (ADDR)  | Load the Yth unsigned half-word from ADDR to X.   |
| LTW    X, Y (ADDR)  | Load the Yth word from ADDR to X.                 |
| LTF    FX, Y (ADDR) | Load the Yth float from ADDR to COP1 register FX. |
| STB    X, Y (ADDR)  | Store X at the Yth byte from ADDR.                |
| STH    X, Y (ADDR)  | Store X at the Yth half-word from ADDR.           |
| STW    X, Y (ADDR)  | Store X at the Yth word from ADDR.                |
| STF    FX, Y (ADDR) | Store X at the Yth float from ADDR.               |

<br>

### Stack Operations

Standard MIPS calling conventions require functions to preserve the values of certain registers when they are called. These values are saved to the stack at the beginning of the function and restored at the end. To minimize the opportunity for errors, Star Rod introduces several simple pseudo-instructions for these stack operations: PUSH, POP, and JPOP (identical to POP + JR RA + NOP). You can use these with multiple registers, but be sure to list the same registers in the same order for both.

| Command         | Description                                                                                                                               |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| PUSH  X, Y, ... | Push a list of registers onto the stack, starting at SP[10]. Updates the stack pointer accordingly, padded to 8-byte alignment as needed. |
| POP   X, Y, ... | Restores registers from the stack. Order is important! Use the same list as the corresponding push. Updates the stack pointer.            |
| JPOP  X, Y, ... | Identical to POP followed by JR RA. Stack addition is done in the delay slot of the JR, so this does not need to be followed by a NOP.    |

<br>

### Branch Conditions

| Command             | Description               |
| ------------------- | ------------------------- |
| BLT   X, Y          | Branch if X < Y.          |
| BGT   X, Y          | Branch if X > Y.          |
| BLE   X, Y          | Branch if X <= Y.         |
| BGE   X, Y          | Branch if X >= Y.         |
| BLTL/BGTL/BLEL/BGEL | Branch “likely” variants. |

<br>

### Special

| Command  | Description                                                                                                                    |
| -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| RESERVED | Put this in a delay slot following a PI and a jump to force the final instruction of the compiled PI to occupy the delay slot. |

<br>

## Loops

Loops can be created using the <code>LOOP</code> and <code>ENDLOOP</code> pseudo-instruction. Instructions between them become the body of the loop. Labels and branching logic are created automatically and only a single register is required for the loop index value. You may also exit the loop at any time from within its body with the <code>BREAKLOOP</code> pseudo-instruction.

### For-Loops

```text
LOOP     index = start,step,end
...
ENDLOOP
```

This loop will initialize the value of a register **(index)** to **start** and increment it by **step** at the end of each iteration until it equals or exceeds **end**. The condition is also checked before the first iteration. index can be any register, the other three can be any register or any 16-bit immediate integer value. Setting **step** to a negative value creates a decrementing loop. Omitting **step** defaults to a value of +1. If a register is used for **step** or **end**, you should avoid modifying their values in the loop body.

For loop examples:

```text
LOOP        A0 = 0,5
Executes body with A0 = 0,1,2,3,4
```
```text
LOOP        T1 = 1,2,10
Executes body with T1 = 1,3,5,7,9
```
```text
LOOP        SP = SP,4,T4
Executes body, incrementing SP by 4 until it equals or exceeds T4.
```

### While-Loops

```text
LOOP  index < value
...
ENDLOOP
```

This loop will execute the body so long as the condition is true. The condition is checked prior to each iteration. **index** may be any register and **value** may be a register or a 16-bit immediate value. There are six valid options for the comparison test: >, >=, <, <=, ==, !=

While loop examples:

```text
LOOP        A0 < 10`
LOOP        T0 != T1
LOOP        S4 > 0
```

## Register Naming

Registers can be assigned names with <code>#DEF</code> and reverted to their default names with <code>#UNDEF</code>. For example:

```text
#DEF   A0, *Counter
CLEAR  *Counter
ADDIU  *Counter, *Counter, 1
ADDIU  *Counter, *Counter, 1
#UNDEF A0
COPY   A1, A0
```

In this example, <code>A0</code> may only be referred to as <code>*Counter</code> until the name is cleared by <code>#UNDEF</code>.

## Examples

### Check if a badge is equipped

```text
% in:  A0 = badge ID to check for
% out: V0 = 1 if badge equipped, 0 if not
#DEF    A0, *BadgeID
PUSH    S0, S1             % Push saved registers to stack
LIO     S0, 8010F498       % Initialize loop start
ADDIU   S1, S0, 80         % 0x40 badge slots
LOOP    S0 = S0:2:S1
    LHU     V1, 0 (S0)
    BEQL    V1, *BadgeID, .Done
    ADDIU   V0, R0, 1      % Return TRUE
ENDLOOP
CLEAR   V0                 % Return FALSE
.Done
JPOP    S0, S1             % Pop saved registers from stack
```

## Writing a Script-Callable Function

Functions intended to be called by scripts using the <code>Call</code> command must follow certain rules to operate correctly. Your function must return a value from the <code>eCommandResult</code> enum (detailed below). Most functions will return <code>eScriptContinue</code>, indicating the script should resume execution after the function is complete. Return 0 instead if you’d like to block the script and have it call your function every frame until some task is finished or condition is met.

You may use the int[4] array at <code>(script_context + 0x70)</code> to store temporary values between repeated calls if you don’t want to modify the script variables.

**Valid return values:** <br>
**FF**    eScriptYield    Script will stop executing (until next frame) <br>
**<0**    ???            script returns 1 <br>
**0 **   eScriptBlock    Script will not proceed any further this frame <br>
**1**    ???            script returns 0 <br>
**2 **   eScriptContinue     <br>
**3**    eScriptRepeat     <br>

**Arguments:** <br>
**A0**     pointer to the script_context data structure of the caller <br>
**A1**    isInitialCall boolean, true on the first call, false on subsequent ones <br>
**A2**    address of the callee function <br>
**S0**    pointer to the function argument list in the script <br>
**S1 **   duplicate of A0 <br>
