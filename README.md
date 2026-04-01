[Vídeo de demonstração da ALU](https://youtu.be/9Lx4HaIXJZw)

# ALU e CPU de 8 bits

Projeto de uma Unidade Lógica e Aritmética (ALU) e uma CPU simplificada de 8 bits, desenvolvido no simulador [Digital](https://github.com/hneemann/Digital).

---

## Sumário
#  ALU & CPU de 8 bits (Simulador Digital)

Este projeto apresenta uma **Unidade Lógica e Aritmética (ALU)** de 8 bits integrada a uma CPU funcional, construída do zero a partir de portas lógicas fundamentais no simulador **Digital**. O objetivo foi aplicar conceitos de arquitetura de computadores para criar um sistema capaz de processar 7 operações distintas de forma paralela e combinacional.

---

##  Arquitetura do Sistema

A CPU opera em um ciclo clássico de **Busca e Execução**, onde a ALU é o "motor" principal de processamento.

### O Fluxo de Dados
* **Acumulador (AC):** O registrador principal que armazena os resultados. Ele pode ser carregado externamente ou via realimentação, permitindo operações encadeadas (ex: somar um número ao resultado da conta anterior).
* **Paralelismo Real:** Diferente de um software sequencial, aqui todos os módulos (soma, divisão, multiplicador, etc.) calculam o resultado ao mesmo tempo.
* **Seleção por Opcode:** O sinal de controle enviado pela CPU atua como um seletor no MUX principal, decidindo qual das respostas prontas será enviada para os registradores de saída.

---

##  Detalhamento dos Módulos da ALU

### 1. Aritmética: Soma e Subtração
Em vez de criar dois circuitos isolados, eu utilizei a eficiência do Complemento de Dois para re-utilizar hardware.

* **Soma:** Implementada via *Ripple Carry Adder*. O "vai um" (carry) propaga-se bit a bit, da posição menos significativa para a mais significativa.
* **Subtração:** Utiliza o mesmo somador. Invertemos os bits do operando B (NOT) e injetamos um `1` no transporte inicial (Cin). Isso transforma matematicamente a subtração em uma soma, reaproveitando 100% do circuito somador.



### 2. Multiplicação (Long Binary Multiplication)
A multiplicação opera de forma idêntica ao método escolar, mas em base binária:

1.  **Geração de Produtos:** Para cada bit do multiplicador (B), o circuito decide se "copia" o multiplicando (usando portas AND) ou se insere zeros.
2.  **Shift & Add:** Cada linha resultante é deslocada para a esquerda conforme sua posição e somada às demais em uma árvore de somadores.
3.  **Resultado de 16 bits:** Como o produto de dois números de 8 bits pode atingir 16 bits, o resultado é segmentado entre o **AC** (8 bits menos significativos) e o **MQ** (8 bits mais significativos).



### 3. Divisão (Subtração com Restauração)
Este é o módulo mais complexo, funcionando como uma "escada" de 8 blocos decisores combinacionais.

* **A Lógica:** Em cada etapa (bit), o circuito tenta subtrair o divisor do dividendo parcial. 
* **Decisão:** Se o resultado for positivo (coube), o bit do quociente é `1` e o resto segue para o próximo bloco. Se for negativo (não coube), o circuito "restaura" o valor anterior e marca `0` no quociente.
* **Saídas:** Ao final da cascata, o **Resto** é enviado para o AC e o **Quociente** consolidado vai para o registrador MQ.



### 4. Operações Lógicas e Shifts
* **Shifts Lógicos:** Implementados puramente via roteamento (splitters). Deslocar a fiação para a esquerda dobra o valor; para a direita, divide por 2 com truncamento. É a operação mais rápida da ALU por não exigir portas lógicas.
* **NAND & XOR:** Operações bit a bit fundamentais. O NAND atua como porta universal, enquanto o XOR é utilizado para detecção de diferenças entre operandos e funções criptográficas simples.

---

## 📋 Tabela de Instruções (Opcodes)

| Opcode | Operação | Saída AC (Principal) | Saída MQ (Auxiliar) |
| :--- | :--- | :--- | :--- |
| **0 / 1** | Divisão | Resto | Quociente |
| **2 / 3** | Multiplicação | LSB (Baixo) | MSB (Alto) |
| **4** | Soma | Resultado | - |
| **5** | Subtração | Resultado | - |
| **6 / 7** | Shift Esquerda/Direita | Resultado | - |
| **8 / 9** | NAND / XOR | Resultado | - |

---

## 💻 Integração com a CPU
A ALU não trabalha isolada; ela é coordenada por uma estrutura de controle:

* **ROM (Memória de Programa):** Armazena as instruções (opcodes).
* **RAM (Memória de Dados):** Armazena os valores (operandos) que serão operados contra o Acumulador.
* **Program Counter (PC):** Incrementa a cada ciclo de clock, garantindo que o programa execute uma instrução após a outra.

---