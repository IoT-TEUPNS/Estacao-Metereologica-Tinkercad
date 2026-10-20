# Estação Meteorológica no Tinkercad (Arduino Uno)
 
Simulação de uma estação meteorológica no **Tinkercad Circuits**, baseada no projeto "Estação Meteorológica IoT" do *Kit Iniciante IoT v2.3 – Casa da Robótica*. O circuito mede temperatura, umidade, pressão atmosférica, concentração de gás/fumaça e presença de chuva, e exibe os dados no Monitor Serial.
 
> **Simulação no Tinkercad:** https://www.tinkercad.com/things/3EiBCuBfALG-estacao-metereologica
 
---
 
## Sobre o projeto
 
O projeto original do kit usa uma **NodeMCU ESP8266** com sensores DHT11, BMP180, MQ-135 e sensor de chuva, enviando os dados pela internet para as plataformas **ThingSpeak** e **Blynk**.
 
O Tinkercad Circuits só simula placas Arduino e não oferece Wi-Fi, a NodeMCU ESP8266 nem alguns dos sensores originais (como o BMP180 e o sensor de chuva). Por isso, este repositório apresenta uma **versão adaptada**, que mantém a lógica da estação (leitura de cinco grandezas e exibição dos dados), mas usa componentes disponíveis no simulador.
 
## Componentes
 
| Grandeza medida | Componente original (kit) | Componente usado no Tinkercad |
|---|---|---|
| Temperatura | DHT11 | Sensor de temperatura **TMP36** |
| Umidade | DHT11 | **Potenciômetro** (simula 0 a 100 %) |
| Pressão atmosférica | BMP180 | **Potenciômetro** (simula 950 a 1050 hPa) |
| Qualidade do ar | MQ-135 | **Sensor de gás** do Tinkercad |
| Chuva | Sensor de chuva | **Botão** (pressionado = chovendo) |
| Indicador de chuva | – | **LED** vermelho + resistor de 220 Ω |
| Microcontrolador | NodeMCU ESP8266 | **Arduino Uno** |
 
Também são usados uma protoboard e um resistor de **10 kΩ** para o sensor de gás.
 
## Ligações
 
| Componente | Ligação no Arduino |
|---|---|
| TMP36 | 5V, **A0** (sinal) e GND |
| Potenciômetro (umidade) | Extremos em 5V e GND, pino central em **A1** |
| Potenciômetro (pressão) | Extremos em 5V e GND, pino central em **A2** |
| Sensor de gás | H1 e A1/A2 em 5V, H2 em GND, saída B1/B2 em **A3**, com resistor de 10 kΩ de B1/B2 para o GND |
| Botão | Um lado em **D2**, o outro em GND (usa `INPUT_PULLUP`) |
| LED | **D3** → resistor de 220 Ω → ânodo (perna longa); cátodo (perna curta) em GND |
 
## Código
 
```cpp
const int pinTemp = A0;   
const int pinUmid = A1;   
const int pinPres = A2;   
const int pinAr   = A3;   
const int pinChuva = 2;   
const int ledChuva = 3;   
 
void setup() {
  Serial.begin(9600);
  pinMode(pinChuva, INPUT_PULLUP);
  pinMode(ledChuva, OUTPUT);
}
 
void loop() {
  float tensao = analogRead(pinTemp) * 5.0 / 1024.0;
  float temperatura = (tensao - 0.5) * 100.0;
 
  float umidade = map(analogRead(pinUmid), 0, 1023, 0, 100);
 
  float pressao = map(analogRead(pinPres), 0, 1023, 950, 1050);
 
  int ar = analogRead(pinAr);
 
  String situacaoAr;
  if (ar < 300) {
    situacaoAr = "Boa";
  } else if (ar < 600) {
    situacaoAr = "Moderada";
  } else {
    situacaoAr = "Ruim";
  }
 
  // Chuva: botão pressionado = LOW = chovendo
  bool chovendo = (digitalRead(pinChuva) == LOW);
  digitalWrite(ledChuva, chovendo);
 
  Serial.println("===== Estacao Meteorologica =====");
  Serial.print("Temperatura: "); Serial.print(temperatura); Serial.println(" C");
  Serial.print("Umidade: "); Serial.print(umidade); Serial.println(" %");
  Serial.print("Pressao: "); Serial.print(pressao); Serial.println(" hPa");
  Serial.print("Concentracao de gas (bruto): "); Serial.println(ar);
  Serial.print("Qualidade do ar: "); Serial.println(situacaoAr);
  Serial.print("Chuva: ");
  Serial.println(chovendo ? "Esta chovendo" : "Nao esta chovendo");
  Serial.println();
 
  delay(1000);
}
```
 
## Como executar
 
1. Abra o circuito no Tinkercad (link acima) ou monte-o seguindo a tabela de ligações.
2. Clique em **Iniciar simulação** e abra o **Monitor Serial**.
3. Interaja com os componentes durante a simulação:
   - clique no **TMP36** e arraste o controle deslizante para alterar a temperatura;
   - gire os **potenciômetros** para variar umidade e pressão;
   - clique no **sensor de gás** e altere a concentração de fumaça;
   - **segure o botão** para simular chuva (o LED acende).
## Resultados
 
O Monitor Serial exibe um bloco de leituras a cada 1 segundo:
 
```
===== Estacao Meteorologica =====
Temperatura: 45.70 C
Umidade: 47.00 %
Pressao: 1003.00 hPa
Concentracao de gas (bruto): ...
Qualidade do ar: ...
Chuva: Nao esta chovendo
```
 
Nos testes realizados:
 
- **Temperatura:** variou conforme o controle do TMP36 (de cerca de 24,7 °C, valor padrão, até 45,7 °C).
- **Umidade:** acompanhou o giro do potenciômetro (valores observados de 17 % a 56 %).
- **Pressão:** variou na faixa simulada (valores observados de 976 a 1003 hPa).
- **Chuva:** a mensagem alternou entre "Esta chovendo" e "Nao esta chovendo" ao pressionar e soltar o botão, com o LED acompanhando o estado.
- **Gás:** a leitura aumenta com a aproximação da fumaça, e o programa classifica a qualidade do ar em *Boa*, *Moderada* ou *Ruim*.
## Principais dificuldades
 
1. **Limitações do simulador.** O Tinkercad não possui NodeMCU ESP8266, Wi-Fi, BMP180 nem sensor de chuva, o que impede o envio de dados ao ThingSpeak/Blynk. A solução foi adaptar o projeto para Arduino Uno, substituindo sensores por componentes equivalentes (potenciômetros e botão).
2. **Pinagem do sensor de gás.** O componente do Tinkercad não tem pinos como VCC, GND e saída, mas terminais identificados por siglas (A1, A2, B1, B2, H1 e H2). Foi preciso entender que H1/H2 são o aquecedor e que A/B são os terminais do elemento sensor.
3. **Leitura travada em 1023.** Inicialmente o sensor de gás sempre indicava o valor máximo. A causa era a falta de um **resistor de 10 kΩ** entre a saída do sensor e o GND, que forma o divisor de tensão necessário para a leitura variar.
4. **Interpretação do valor do gás.** A leitura parecia "invertida", porque a "qualidade do ar" aumentava com a fumaça. Na verdade o sensor mede a **concentração de gás**, e valores maiores indicam ar pior. O código foi corrigido para exibir a concentração e uma classificação (Boa/Moderada/Ruim).
5. **Terminais do botão.** O botão tem quatro terminais (1a, 1b, 2a, 2b), ligados em pares por dentro. Foi necessário entender que basta usar um terminal de cada lado.
6. **Interação durante a simulação.** Alguns sensores (como o TMP36 e o sensor de gás) só mudam de valor ao clicar neles durante a simulação, o que inicialmente parecia um erro no circuito.
## Limitações
 
- Umidade, pressão e chuva são **simuladas** por potenciômetros e botão, não por sensores reais.
- Os limites de classificação do ar (300 e 600) foram escolhidos para a simulação e não correspondem a uma calibração em ppm como a do MQ-135 real.
- Não há envio de dados pela internet.

## Referência
 
*Kit Iniciante IoT v2.3 – Casa da Robótica*, projeto "NodeMCU ESP8266 e IoT: Desenvolvendo uma Estação Meteorológica".
