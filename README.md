.MODEL SMALL
.STACK 100H

.DATA
    mensaje1 DB 13,10,'CALCULADORA HEXADECIMAL',13,10,'$'
    mensaje2 DB 13,10,'Ingrese el primer numero (00-FF): $'
    mensaje3 DB 13,10,'Ingrese el operador (+ o -): $'
    mensaje4 DB 13,10,'Ingrese el segundo numero (00-FF): $'
    mensaje5 DB 13,10,'Resultado hexadecimal: $'
    mensaje6 DB 13,10,'Indicador (acarreo/prestamo): $'
    mensaje7 DB 13,10,'Operador no valido.$'
    salto DB 13,10,'$'

    numero1 DB ?
    numero2 DB ?
    resultado DB ?
    indicador DB ?
    temporal DB ?

.CODE

MAIN PROC
    MOV AX, @DATA
    MOV DS, AX

    ; Mostrar titulo
    MOV AH, 09H
    LEA DX, mensaje1
    INT 21H

    ; Leer primer numero hexadecimal
    MOV AH, 09H
    LEA DX, mensaje2
    INT 21H

    CALL LEER_HEX
    MOV numero1, AL

    ; Leer operador
    MOV AH, 09H
    LEA DX, mensaje3
    INT 21H

    MOV AH, 01H
    INT 21H
    MOV BH, AL

    ; Leer segundo numero hexadecimal
    MOV AH, 09H
    LEA DX, mensaje4
    INT 21H

    CALL LEER_HEX
    MOV numero2, AL

    ; Comprobar el operador
    CMP BH, '+'
    JE SUMAR

    CMP BH, '-'
    JE RESTAR

    JMP OPERADOR_INVALIDO

SUMAR:
    MOV AL, numero1
    ADD AL, numero2

    MOV resultado, AL
    MOV BL, 0
    ADC BL, 0
    MOV indicador, BL

    JMP MOSTRAR_RESULTADO

RESTAR:
    MOV AL, numero1
    SUB AL, numero2

    MOV resultado, AL
    MOV BL, 0
    ADC BL, 0
    MOV indicador, BL

    JMP MOSTRAR_RESULTADO

MOSTRAR_RESULTADO:
    MOV AH, 09H
    LEA DX, mensaje5
    INT 21H

    MOV AL, resultado
    CALL IMPRIMIR_HEX

    MOV AH, 09H
    LEA DX, mensaje6
    INT 21H

    MOV AL, indicador
    CALL IMPRIMIR_DIGITO

    MOV AH, 09H
    LEA DX, salto
    INT 21H

    JMP FINALIZAR

OPERADOR_INVALIDO:
    MOV AH, 09H
    LEA DX, mensaje7
    INT 21H

FINALIZAR:
    MOV AH, 4CH
    INT 21H

MAIN ENDP

; PROCEDIMIENTO PARA LEER UN BYTE HEXADECIMAL
LEER_HEX PROC
    CALL LEER_DIGITO
    SHL AL, 4
    MOV temporal, AL

    CALL LEER_DIGITO
    OR AL, temporal

    RET
LEER_HEX ENDP

; PROCEDIMIENTO PARA LEER UN DIGITO HEXADECIMAL
LEER_DIGITO PROC
    MOV AH, 01H
    INT 21H

    CMP AL, '9'
    JBE DIGITO_NUMERICO

    CMP AL, 'F'
    JBE DIGITO_MAYUSCULA

    SUB AL, 20H

DIGITO_MAYUSCULA:
    SUB AL, 'A'
    ADD AL, 10
    RET

DIGITO_NUMERICO:
    SUB AL, '0'
    RET
LEER_DIGITO ENDP

; PROCEDIMIENTO PARA IMPRIMIR UN BYTE HEXADECIMAL
IMPRIMIR_HEX PROC
    PUSH AX

    MOV AH, AL
    SHR AL, 4
    CALL IMPRIMIR_DIGITO

    POP AX
    AND AL, 0FH
    CALL IMPRIMIR_DIGITO

    RET
IMPRIMIR_HEX ENDP

; PROCEDIMIENTO PARA IMPRIMIR UN DIGITO HEXADECIMAL
IMPRIMIR_DIGITO PROC
    CMP AL, 9
    JBE NUMERO

    ADD AL, 7

NUMERO:
    ADD AL, '0'
    MOV DL, AL
    MOV AH, 02H
    INT 21H

    RET
IMPRIMIR_DIGITO ENDP

END MAIN
