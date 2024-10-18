
matrix:  .byte 2,7,34,68,56,89,156,122,135,0,33,122,122,66,18,255
matrixDimensions:	.word 4
mainMenu:	.ascii	"\n options:" 
		.ascii	"\n1. Print matrix unsigned"
		.ascii	"\n2. Print matrix sign"
		.ascii	"\n3. Change a number in the matrix"
		.ascii	"\n4. Negate a number in the matrix"
		.ascii	"\n5. Swap numbers in the matrix"
		.ascii	"\n6. Find the max sum of  a row (unsign)"
		.ascii	"\n7. Find  the max sum of a row (sign)"
		.asciiz	"\n8. END"
jumpTable: .word	printMatrix
	   .word	printMatrix
	   .word	changeNumber
	   .word	negate
	   .word	swap
	   .word	findMax
	   .word	findMax
	   .word	exit
	   
numStr:	   .asciiz	"    "		

changeStr:	.asciiz "\n*** this is change number in matrix procedure ***"
negStr:		.asciiz "\n*** this is negate value in matrix procedure ***"
swapStr:	.asciiz "\n*** this is swap procedure. Enter two positions ***"
maxRowStr:	.asciiz "\n*** this is sum row procedure for "
signedStr:	.asciiz	"Signed numbers ***"
unsignedStr:	.asciiz "Unsigned numbers ***"
enterRowStr:	.asciiz "\nPlease enter legal row number: "
enterColStr:	.asciiz "\nPlease enter legal column number: "
IllegalColRow: .asciiz "\nIllegal Row/Col value. Has to be between 1 and "
enterNumber:	.asciiz "\nPlease enter number value. between -128 to 127, or 0 to 255"
illegalNumber: .asciiz "\nThe number you have entered is out of range. Please Reenter"


.text
.globl main
main:

#	printString(mainMenu)
	la $a0, mainMenu
	li $v0, 4
	syscall

	li $v0, 5
	syscall
	
	bgt	$v0, 8, main	
	beqz	$v0, main
	
	la	$a0, matrix			# $a0 - is the matrix address
	lw	$a1, matrixDimensions		# $a1 - is the matrix size
	add	$a2, $zero, $v0		# $a2 - is the user selection
		
	addi	$v0, $v0, -1
	sll	$v0, $v0, 2			# calculate index into jump table
	lw	$v0, jumpTable($v0)		
	jalr	$ra, $v0			# call appropriate PROC based on user selection 


	li $a0, 0x0a
	li $v0, 11
	syscall
	j main
exit:
li $v0, 10	# Exit program
syscall

printMatrix:

	addiu $sp, $sp, -4
	sw $ra,0($sp)	

	addiu $sp, $sp, -4
	sw $s0,0($sp)	

	addiu $sp, $sp, -4
	sw $s1,0($sp)	

	addiu $sp, $sp, -4
	sw $s2,0($sp)

	addiu $sp, $sp, -4
	sw $s3,0($sp)
	
	move	$s0, $a0	# $s0 is the matrix address
	move	$s1, $a1	# $s1 is the matrix dementsions for use as loop control
	move 	$s2, $a2	# $s2 is the print type

	add	$s3, $zero, $zero	# $s3 is loop counter for ROWs	
printRowLoop:

		li $a0, 0x0a
		li $v0, 11
		syscall
		mulu	$a0, $s3, $s1	# calculate index for row beginning
		add	$a0, $a0, $s0	# calculate address for current row
		add	$a1, $zero, $s1	# row size
		add	$a2, $zero, $s2 # print type
		jal	printRow 
		addi	$s3, $s3, 1	# increment ROW number
	blt	$s3, $s1, printRowLoop	# continue loop if not finish
	

	lw $s3, 0($sp)
	addiu $sp, $sp, 4

	lw $s2, 0($sp)
	addiu $sp, $sp, 4

	lw $s1, 0($sp)
	addiu $sp, $sp, 4

	lw $s0, 0($sp)
	addiu $sp, $sp, 4

	lw $ra, 0($sp)
	addiu $sp, $sp, 4
	jr $ra

printRow:

	addiu $sp, $sp, -4
	sw $ra,0($sp)	

	addiu $sp, $sp, -4
	sw $s0,0($sp)	

	addiu $sp, $sp, -4
	sw $s1,0($sp)	

	addiu $sp, $sp, -4
	sw $s2,0($sp)

	addiu $sp, $sp, -4
	sw $s3,0($sp)
	
	add	$s0, $zero, $a0	
	add	$s1, $zero, $a1 
	add	$s2, $zero, $a2	
	
	add	$s3, $zero, $zero	# $s3 is loop counter for COLUMNs
printColumnLoop:
		add	$t0, $s0, $s3	# calculate index of current value
		add	$a1, $zero, $s2	# prepare print type
		beq	$a1, 1, printGetNoSigned
printGetSigned:
			lb	$a0, ($t0)	# load current matrix value
			j printCont
printGetNoSigned:
			lbu	$a0, ($t0)
printCont:
		jal	printNum

		li $a0,' '
		li $v0, 11
		syscall
		addi	$s3, $s3, 1	
		blt	$s3, $s1, printColumnLoop	# jump if not finish columns

#	pop	$s3
	lw $s3, 0($sp)
	addiu $sp, $sp, 4
#	pop	$s2
	lw $s2, 0($sp)
	addiu $sp, $sp, 4
#	pop	$s1
	lw $s1, 0($sp)
	addiu $sp, $sp, 4
#	pop	$s0	  
	lw $s0, 0($sp)
	addiu $sp, $sp, 4
#	pop	$ra
	lw $ra, 0($sp)
	addiu $sp, $sp, 4
	jr $ra
		
printNum:

# 	$a0 - the value
#	$a1 - 1 for unsigned, 2 for signed


	beq	$a1, 1, printNumUnsigned
printNumSigned:
		blt	$a0, $zero, printNumMinus
printNumPlus:
		li $t0, '+'	# positive signed number
		j printNumCont
printNumMinus:
		li $t0, '-'	# negative signed number
		neg $a0, $a0	# negate the number for digits saperation
		j printNumCont
printNumUnsigned:
		li $t0, ' '	# for unsigned number: not sign is required
printNumCont:
	sb	$t0, numStr	# prepare the sign for print
	li	$t1, 10
	li	$t2, 3		# digits loop counter
printNumDigitLoop:
		divu	$a0, $t1
		mfhi	$t0		# get the reminder
		addi	$t0, $t0, 0x30	# convert digit to ASCII
		sb	$t0, numStr($t2)
		mflo	$a0		# get the quotient for next digit calculation
		addi	$t2, $t2, -1	# decrement loop counter
	bne	$t2, $zero, printNumDigitLoop
#	printString numStr
	la $a0, numStr
	li $v0, 4
	syscall
	jr $ra
	
getPosition:


#	$a0 - is maximum value for Row/Column
#
#	$v0 - return value: index into matrix

	add	$t0, $zero, $a0		# keep input parameter

#	printString enterRowStr
	la $a0, enterRowStr
	li $v0, 4
	syscall
#	getInteger
	li $v0, 5
	syscall
	bgt	$v0, $t0, getPositionIllegal
	blt	$v0, 1, getPositionIllegal
	add	$t1, $zero, $v0
#	printString enterColStr
	la $a0, enterColStr
	li $v0, 4
	syscall
#	getInteger
	li $v0, 5
	syscall
	bgt	$v0, $t0, getPositionIllegal
	blt	$v0, 1, getPositionIllegal
getPositionLegal:
	addi	$t1, $t1, -1	# decrement by one to start from zero
	mulu	$t1, $t1, $t0	# multiply row by row size
	addi	$v0, $v0, -1	# decrment column by one to start from zero
	add	$v0, $v0, $t1
	j getPositionFinish
	
getPositionIllegal:
#	printString IllegalColRow
	la $a0, IllegalColRow
	li $v0, 4
	syscall
#	printInteger $t0
	add	$a0, $zero, $t0
	li	$v0, 1
	syscall
	j getPosition
	
getPositionFinish:
	jr $ra
	
changeNumber:

#	$a0 - is MATRIX address
#	$a1 - is MATRIX DIMENSIONS
#	$a2 - is 3 (user request code)

#	push $ra
	addiu $sp, $sp, -4
	sw $ra,0($sp)	
#	push $s0
	addiu $sp, $sp, -4
	sw $s0,0($sp)	
#	push $s1
	addiu $sp, $sp, -4
	sw $s1,0($sp)	
	
	add	$s0, $zero, $a0
	add	$s1, $zero, $a1
	
#	printString changeStr
	la $a0, changeStr
	li $v0, 4
	syscall
		
	add	$a0, $a1, $zero	# prepare matrix dimension as parameter for get position
	jal 	getPosition
	add 	$s0, $s0, $v0	# add to matrix address the current position
changeNumberGetNum:	
#	printString enterNumber
	la $a0, enterNumber
	li $v0, 4
	syscall
#	getInteger
	li $v0, 5
	syscall
	sra	$t0, $v0, 8	# arithmetic clear of first 8 bits (first byte)
	beqz	$t0, changeNumberLegal	# if zero, it is positive number up to 255
	sra	$t0, $v0, 7	# arithmetic clear of first 7 bits (for begative values)
	bne 	$t0, -1 ,changeNumberIllegal # if negative it up to -128, it is legal negative. If not, error
changeNumberLegal:
	sb	$v0, 0($s0)
	j 	changeNumberFinish
	
changeNumberIllegal:
#	printString illegalNumber
	la $a0, illegalNumber
	li $v0, 4
	syscall
	j changeNumberGetNum
	
changeNumberFinish:

#	pop	$s1
	lw $s1, 0($sp)
	addiu $sp, $sp, 4
#	pop	$s0	  
	lw $s0, 0($sp)
	addiu $sp, $sp, 4
#	pop	$ra
	lw $ra, 0($sp)
	addiu $sp, $sp, 4
	jr $ra
	
negate:
# This procedure negate a value at requested postion
#	$a0 - is MATRIX address
#	$a1 - is MATRIX DIMENSIONS
#	$a2 - is 4 (user request code)

#	push $ra
	addiu $sp, $sp, -4
	sw $ra,0($sp)	
#	push $s0
	addiu $sp, $sp, -4
	sw $s0,0($sp)	

	add	$s0, $zero, $a0	# save matrix address for later use
	
#	printString negStr
	la $a0, negStr
	li $v0, 4
	syscall
	
	add	$a0, $zero, $a1	# prepare matrix size for get position
	jal	getPosition
	add	$s0, $s0, $v0	# prepare address within matrix
	lb	$t0, 0($s0)	# get current value
	neg	$t0, $t0
	sb	$t0, 0($s0)	
	
#	pop	$s0	  
	lw $s0, 0($sp)
	addiu $sp, $sp, 4
#	pop	$ra
	lw $ra, 0($sp)
	addiu $sp, $sp, 4
	jr $ra
	
swap:

#	push $ra
	addiu $sp, $sp, -4
	sw $ra,0($sp)	
#	push $s0
	addiu $sp, $sp, -4
	sw $s0,0($sp)	
#	push $s1
	addiu $sp, $sp, -4
	sw $s1,0($sp)	
#	push $s2
	addiu $sp, $sp, -4
	sw $s2,0($sp)
	
	add	$s0, $zero, $a0		# save matrix address
	add	$s1, $zero, $a1		# save matrix size
	
#	printString swapStr
	la $a0, swapStr
	li $v0, 4
	syscall
# get first position
	add	$a0, $zero, $s1	# parameter - matrix size
	jal	getPosition
	add	$s2, $s0, $v0	# save first position address
# get second position
	add	$a0, $zero, $s1	# parameter - matrix size
	jal	getPosition
	add	$s0, $s0, $v0	# save second position address
# swap between positions
	lb	$t0, 0($s0)
	lb	$t1, 0($s2)
	sb	$t0, 0($s2)
	sb	$t1, 0($s0)
	
#	pop	$s2
	lw $s2, 0($sp)
	addiu $sp, $sp, 4
#	pop	$s1
	lw $s1, 0($sp)
	addiu $sp, $sp, 4
#	pop	$s0	  
	lw $s0, 0($sp)
	addiu $sp, $sp, 4
#	pop	$ra
	lw $ra, 0($sp)
	addiu $sp, $sp, 4
	jr $ra
	
findMax:

	addiu $sp, $sp, -4
	sw $ra,0($sp)	
#	push $s0
	addiu $sp, $sp, -4
	sw $s0,0($sp)	
#	push $s1
	addiu $sp, $sp, -4
	sw $s1,0($sp)	
#	push $s2
	addiu $sp, $sp, -4
	sw $s2,0($sp)
#	push $s3
	addiu $sp, $sp, -4
	sw $s3,0($sp)
#	push $s4
	addiu $sp, $sp, -4
	sw $s4,0($sp)
#	push $s5
	addiu $sp, $sp, -4
	sw $s5,0($sp)
	
	add	$s0, $zero, $a0	# save matrix address
	add	$s1, $zero, $a1 # save row size
	add	$s2, $a2, -5	# convert to 1 as UNSIGNED, 2 as SIGNED
	
#	printString maxRowStr
	la $a0, maxRowStr
	li $v0, 4
	syscall
	beq $s2, 1, findMaxUnsigned
findMaxSigned:
#		printString signedStr
		la $a0, signedStr
		li $v0, 4
		syscall
		j findMaxCont1
findMaxUnsigned:
#		printString unsignedStr
		la $a0, unsignedStr
		li $v0, 4
		syscall
findMaxCont1:
#	printChar 0x0a	# new line before the row with maximal sum value
	li $a0, 0x0a
	li $v0, 11
	syscall
	
	add	$s3, $zero, $zero	# $s3 is loop counter for ROWs	
	li	$s4, 0	# initialize first row as with highest sum
	beq	$s2, 1, findMaxInitUnsigned
findMaxInitSigned:
		mul	$s5, $s1, -128
		add	$s5, $s5, -1	# set to 1 less the minimal signed sum
	j findMaxSumRowLoop
findMaxInitUnsigned:
		add	$s5, $zero, $zero # set to zero for unsigned sum
		
findMaxSumRowLoop:
		mulu	$a0, $s3, $s1	# calculate index for row beginning
		add	$a0, $a0, $s0	# calculate address for current row
		add	$a1, $zero, $s1	# row size
		add	$a2, $zero, $s2 # sum type
		jal	sumRow 
		beq	$a2, 1, findMaxCheckUnsigned
findMaxCheckSigned:
			bgt	$v0, $s5, findMaxReplaceRow
			j findMaxCont2
findMaxCheckUnsigned:
			bleu	$v0, $s5, findMaxCont2
findMaxReplaceRow:
		add	$s4, $zero, $s3	# save current row as maximal row
		add	$s5, $zero, $v0	# save current value as maximal sum
findMaxCont2:
		addi	$s3, $s3, 1	# increment ROW number
	blt	$s3, $s1, findMaxSumRowLoop	# continue loop if not finish
	
	mulu	$a0, $s4, $s1	# calculate index of maximal sum row
	add	$a0, $a0, $s0	# calculate address of maximal sum row
	add	$a1, $zero, $s1	# prepare row size as parameter
	add	$a2, $zero, $s2 # prepare print type
	jal	printRow
	
#	pop	$s5
	lw $s5, 0($sp)
	addiu $sp, $sp, 4
#	pop	$s4
	lw $s4, 0($sp)
	addiu $sp, $sp, 4
#	pop	$s3
	lw $s3, 0($sp)
	addiu $sp, $sp, 4
#	pop	$s2
	lw $s2, 0($sp)
	addiu $sp, $sp, 4
#	pop	$s1
	lw $s1, 0($sp)
	addiu $sp, $sp, 4
#	pop	$s0	  
	lw $s0, 0($sp)
	addiu $sp, $sp, 4
#	pop	$ra
	lw $ra, 0($sp)
	addiu $sp, $sp, 4
	jr $ra
	
sumRow:

	li	$t0, 0	# initialize column loop
	li	$v0, 0	# initialize column sum
sumRowLoop:
		beq	$a2, 1, sumRowUnsigned
sumRowSigned:
			lb	$t1, 0($a0)	# get current signed value
			add	$v0, $v0, $t1
			j sumRowLoopCont
sumRowUnsigned:
			lbu	$t1, 0(	$a0)
			addu	$v0, $v0, $t1
sumRowLoopCont:	
		addi	$a0, $a0, 1	# increment for next column address
		addi	$t0, $t0, 1	# increment loop counter
	blt	$t0, $a1, sumRowLoop
sumRowFinish:			
	jr $ra


