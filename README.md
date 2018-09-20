# Assembly x8086 Example
## Excercise
Make a program that allows the entry of a text of up to 250 characters ended by the sign $ and ask for the entry of another text of up to 8 characters ended with the sign $ and look for this last text in the first.
You must print the location where the 2nd text is in the 1st in binary 8bits.

Ex:
 ```asm
1st text: How are you$
2nd text: are$
Output: 00000100 (4 position)
 ```

 ```asm
 .8086
.MODEL SMALL
.STACK 100H
.DATA
	;Declaring variables
	STRING1 DB 250 DUP ('$')
	SUBSTRING1 DB 8 DUP ('$')
	TEXT1 DB "Enter String: $"
	TEXT2 DB 0DH, 0AH, "Enter Substring: $"
	TEXT3 DB 0DH, 0AH, "Substring found $"
	TEXT4 DB 0DH, 0AH, "Substring not found $"
	TEXT5 DB 0DH, 0AH, "Position in binary: $"

.CODE
MAIN PROC
	MOV AX, @DATA
	MOV DS, AX

	;CX = 254 (loop)
	MOV CX, 254
	;i = 0
	MOV BX, 0
	
	MOV AH, 9
	MOV DX, OFFSET TEXT1
	INT 21H
STR1:
	;Enter String
	MOV AH, 1
	INT 21H

	;Place letter entered in STRING1[i]
	MOV STRING1[BX], AL
	CMP AL, '$'
	JE OTHER

	;i++
	INC BX
	LOOP STR1
OTHER:
	;I = 0
	MOV BX, 0
	;CX = 14 (loop)
	MOV CX, 14

	MOV AH, 9
	MOV DX, OFFSET TEXT2
	INT 21H
SUBSTR1:
	;Enter Substring
	MOV AH, 1
	INT 21H

	;Place letter entered in SUBSTRING1[i]
	MOV SUBSTRING1[BX], AL
	CMP AL, '$'
	JE KEEPGOING

	;i++
	INC BX
	LOOP SUBSTR1
KEEPGOING:
	;i = 0
	MOV BX, 0
AGAIN:
	;j = 0
	MOV DI, 0
LOOPSUBSTR:
	;substring[j] -> AL
	MOV AL, SUBSTRING1[DI]
	;si AL = $ ---> FINDEPAL
	CMP AL, '$'
	JE ENDWORD

	;string[i+j] -> AH
	MOV AH, STRING1[BX+DI]
	;si AH = $ ---> FINTEXTO
	CMP AH, '$'
	JE ENDTEXT

	;substring[j] = string[i+j] ?
	CMP AL, AH
	JE OTHERWORD
	INC BX

	;i = 255 ?
	CMP BX, 255
	JE ENDTEXT
	JMP AGAIN

OTHERWORD:
	;j++
	INC DI
	JMP LOOPSUBSTR

ENDTEXT:
	MOV AH, 9
	MOV DX, OFFSET TEXT4
	INT 21H
	JMP FIN
ENDWORD:
	MOV AH, 9
	MOV DX, OFFSET TEXT3
	INT 21H

	;cx = 8 (loop)
	MOV CX, 8

	MOV AH, 9
	MOV DX, OFFSET TEXT5
	INT 21H
LOOPPOSITION:
	;example with substring found in
	;position [5] (00000101)
	;BL = 00000101 -> jc 0
	;BL = 00001010 -> jc 0
	;BL = 00010100 -> jc 0
	;BL = 00101000 -> jc 0
	;BL = 01010000 -> jc 0
	;BL = 10100000 -> jc 0
	;BL = 01000000 -> jump carry 1, then ONE
	SHL BL, 1
	JC ONE
	MOV DL, '0'
	JMP PRINT
ONE:
	MOV DL, '1'
PRINT:
	;MOV AH prints DL
	MOV AH, 6
	INT 21H
	LOOP LOOPPOSITION
FIN:
	MOV AX, 4C00H
	INT 21H
ENDP
END MAIN
 ```