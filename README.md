# Motor-Passo-CC-X-NUCLEO-IHM04A1
Acionamento de motor de passo utilizando a topologia de Ponte H de um hardware de potência dedicado utilizando três sinais de controle, sendo eles: Step, fornecidos por um gerador de sinais,
Direção (DIR) e Enable (EN) por botões. Para isso, será utilizado um hardware de 
potência dedicado (ST X-NUCLEO IHM04A1) que energizará as bobinas do atuador 
conforme a lógica contida no microcontrolador e escolhida pelo usuário de acordo com 
o pressionamento do botão de Direção. A opção por desenergizar as bobinas fica 
disponível ao se pressionar o botão Enable.

## Diagrama de blocos
A seguir, diagrama de blocos do projeto.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/27f57278-fa02-4b42-aef3-03dde7f4c021" width="500px"/>
</div>


## Esquemático e diagrama de ligações
A seguir, podemos visualizar o driver usado no projeto, bem como a descrição 
de seus respectivos pinos. Foram adicionados os botões para Enable e Direção, e foi 
indicado o sinal de Step. No desenvolvimento do circuito eletrônico, foi usado um 
módulo de potência ST X-NUCLEO-IHM04A1 como ponte H.
Os pinos IN1A e IN2A foram usados para entrada dos sinais de controle no 
Shield.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/cae7976a-aad0-4c19-8905-1dfb61960a45" width="500px"/>
</div>

## Fluxograma
A etapa anterior a elaboração do código de acionamento do motor consiste na 
realização de um fluxograma que represente e facilite a organização da estratégia de 
programação.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/2084528d-35d2-4ba5-b7d5-e7b7bce1f231" width="500px"/>
</div>

## Software da aplicação

```C++
// Inclusão bibliotecas
#include "mbed.h"
// definição I/O
DigitalOut EN_A(D2);
DigitalOut IN1_A(D5);
DigitalOut IN2_A(D4);
DigitalOut EN_B(A4);
DigitalOut IN1_B(A0);
DigitalOut IN2_B(A1);

InterruptIn e(D6);
InterruptIn d(PA_9);
InterruptIn s(PB_8);
bool dir = 0;
bool enable = 0;
int passo = 0;

void liga(void) { enable = !enable; }
void inverte(void) { dir = !dir; }

void step(void) {
  if (dir == 0) {
    passo++;
  }
  if (dir == 1) {
    passo--;
  }
  if (passo == 4) {
    passo = 0;
  }
  if (passo == -1) {
    passo = 3;
  }
}

int main() {
  // configurações iniciais
  EN_A = 0; // Desliga Ponte-H (A)
  EN_B = 0; // Desliga Ponte-H (B)
  IN1_A = 0;
  IN2_A = 0; // Ponte-H (A) Motor 'travado'
  IN1_A = 0;
  IN1_B = 0; // Ponte-H (B) Motor 'travado’
  /* da T.V. do L6206:
  EN = 1; IN1 = 1; IN2 = 0 --> Vs - GND
  EN = 1; IN1 = 0; IN2 = 1 --> GND - Vs */
  e.fall(&liga);
  d.rise(&inverte);
  s.fall(&step);
  while (1) {
    if (enable == 1) {
      EN_A = 0; // Desliga Ponte-H (A)
      EN_B = 0; // Desliga Ponte-H (B)
      IN1_A = 0;
      IN2_A = 0; // Ponte-H (A) Motor 'travado'
      IN1_A = 0;
      IN1_B = 0; // Ponte-H (B) Motor 'travado’
    }
    if (enable == 0) {
      if (passo == 0) {
        // passo 0
        EN_A = 1;
        IN1_A = 1;
        IN2_A = 0;
        EN_B = 1;
        IN1_B = 1;
        IN2_B = 0;
      }
      if (passo == 1) {
        // passo 1
        EN_A = 1;
        IN1_A = 0;
        IN2_A = 1;
        EN_B = 1;
        IN1_B = 1;
        IN2_B = 0;
      }
      if (passo == 2) {
        // passo 2
        EN_A = 1;
        IN1_A = 0;
        IN2_A = 1;
        EN_B = 1;
        IN1_B = 0;
        IN2_B = 1;
      }
      if (passo == 3) {
        // passo 3
        EN_A = 1;
        IN1_A = 1;
        IN2_A = 0;
        EN_B = 1;
        IN1_B = 0;
        IN2_B = 1;
      }
    }
  }
}
```

Detalhando melhor o desenvolvimento de cada um dos sinais de controle, observa-se 
que todos foram elaborados a partir do uso de interrupções. Para o sinal de enable (EN), 
a estratégia usada foi inverter o sinal toda vez que o botão for pressionado pelo usuário, 
ligando ou desligando o motor de acordo com o nível lógico estabelecido. Para o sinal 
de direção (DIR) a estratégia adotada foi semelhante, mas além de inverter o sinal por 
meio da função “inverte” foi necessário alterar a ordem de acionamento das fases 
(função “step”) aumentando ou decrementando o valor da variável “passo” que 
coordena a ordem em que cada estado é percorrido ao longo do loop. 
O sinal de STEP, por sua vez, foi definido com base no gerador de sinais como 
previamente citado e é representado por uma onda quadrada capaz de coordenar tanto o 
tempo de permanência em cada estado com o tempo de transição entre eles. O duty 
cycle usado foi de 50%, com low level igual a 0V e high level de 1,65V, gerando 
distância de 3,3V pico-a-pico, compatível com o nível lógico alto da placa núcleo. Além 
disso, outra configuração importante foi a frequência do sinal de 700 Hz, estabelecida 
empiricamente conforme a observação da resposta do giro do motor frente a exposição a 
diferentes frequências.

## Montagem
Após esquemático concluído, a montagem física foi realizada com o auxílio de 
uma protoboard para fixação dos botões de Enable e Direção e dos resistores de pulldown. Para gerar o PWM desejado, a estratégia adotada consistiu em usar o gerador de 
sinais para obtenção de uma onda quadrada. Para a configuração desse componente foi 
considerado um duty cycle de 50% com tensão pico-a-pico de 3,3V, que foi 
escolhido de acordo com a tensão de trabalho da placa núcleo.

<div align="center">
<img src ="<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/9bd5fe39-3cb3-41bc-bcfb-8be71f0a9f62" width="500px"/>
</div>


