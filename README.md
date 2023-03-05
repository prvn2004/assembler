1. Write a two pass assembler for the assembly language. (75%)
The assembler must,
o Read assembly language from a text file, assigning label values and instruction
opcodes. The format of the assembly language is described.
o Diagnose common assembly errors such as unknown instruction, no such label,
duplicate label.
o Produce an object file of the produced machine code. This file should be a binary
file. Code starts at address zero.
o Produce a listing file. There is a choice of the format of the listing file. It can
either be a simple memory dump, or show the bytes produced for each instruction,
and that instruction's mnemonic. The formats are shown. (Extra marks are
available for the latter type of listing file)
o You must write the assembler in ISO C89. (`gcc -std=c89 -pedantic -W -Wall' is a
good way to check things.) or others are also fine.
2. Test your assembler with the sample programs listed.
3. Test your assembler with additional programs and submit evidence of this.
4. Write a bubble sort program in SIMPLE Assembler. The start of this file is provided, you
have to fill in the blanks. 

Example Programs
This is a valid, but nonsense assembly file. Your assembler should not issue any errors (it could
issue warning though).
; test1.asm
label: ; an unused label
 ldc 0
 ldc -5
 ldc +5
loop: br loop ; an infinite loop
br next ;offset should be zero
next:
 ldc loop ; load code address
 ldc var1 ; forward ref
var1: data 0 ; a variable
It is shown as text below.
This example contains many errors. Your assembler should spot them all (it need not copy the
error message exactly, but should issue something appropriate).
; test2.asm
; Test error handling
label:
label: ; duplicate label definition
br nonesuch ; no such label
ldc 08ge ; not a number
ldc ; missing operand
add 5 ; unexpected operand
ldc 5, 6; extra on end of line
0def: ; bogus label name
fibble; bogus mnemonic
0def ; bogus mnemonic 
If you implement the SET pseudo instruction, this program should assemble
; test3.asm
; Test SET
val: SET 75
ldc val
adc val2
val2: SET 66
Here is an implementation of memcpy which assembles to  object file  and listing
Here's a real file, (the one we will be testing with) you should be able to assemble and then
emulate it. See if you can figure out what it's doing. .
 ldc 0x1000
 a2sp
 adj -1
 ldc result
 stl 0
 ldc count
 ldnl 0
 call main
 adj 1
 HALT
;
main: adj -3
 stl 1
 stl 2
 ldc 0 ; zero accumulator
 stl 0
loop: adj -1
 ldl 3
 stl 0
 ldl 1
 call triangle
 adj 1
 ldl 3
 stnl 0
 ldl 3
 adc 1
 stl 3
 ldl 0
 adc 1
 stl 0
 ldl 0 ; reload it
 ldl 2
 sub
 brlz loop
 ldl 1 ; get return address
 adj 3
 return
; 
triangle:adj -3
 stl 1
 stl 2
 ldc 1
 shl
 ldl 3
 sub
 brlz skip
 ldl 3
 ldl 2
 sub
 stl 2
skip: ldl 2
 brz one
 ldl 3
 adc -1
 stl 0
 adj -1
 ldl 1
 stl 0
 ldl 3
 adc -1
 call triangle
 ldl 1
 stl 0
 stl 1
 ldl 3
 call triangle
 adj 1
 ldl 0
 add
 ldl 1
 adj 3
 return
one: ldc 1
 ldl 1
 adj 3
 return
;
count: data 10
result: data 0 
