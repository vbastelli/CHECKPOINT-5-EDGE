# Vinheria Agnello IoT Monitoring System

## Descrição

Este projeto foi desenvolvido para a Vinheria Agnello com o objetivo de monitorar e controlar as condições de armazenamento de vinhos em sua adega. Utilizamos sensores para medir luminosidade, temperatura e umidade, e os dados são enviados para uma plataforma FIWARE via protocolo MQTT para armazenamento e análise.

## Objetivos

1. Monitorar luminosidade, temperatura e umidade do ambiente de armazenamento do vinho.
2. Armazenar os dados históricos coletados dos sensores.
3. Exportar os dados para geração de gráficos e relatórios.
4. Integrar a solução ao ecossistema de negócios da União Europeia utilizando FIWARE e NGSIv2.

## Funcionalidades

- Leitura de dados de luminosidade (LDR) e de temperatura/umidade (DHT22) usando ESP32.
- Envio de dados coletados para a plataforma FIWARE via protocolo MQTT.
- Interface de dados em conformidade com o padrão NGSI.
- Armazenamento e exportação dos dados para análises históricas.

## Requisitos

### Hardware

- **ESP32**: Microcontrolador utilizado para capturar e enviar os dados via Wi-Fi.
- **Sensor DHT22**: Sensor de temperatura e umidade.
- **Sensor LDR**: Sensor de luminosidade.
- **Fonte de Alimentação**: 5V para ESP32.
- **Cabo USB**: Para programar o ESP32 e alimentá-lo.

### Software

- **Arduino IDE**: Para programar o ESP32.
- **Bibliotecas do Arduino**: `WiFi.h`, `PubSubClient.h`, `DHT.h`.
- **Plataforma FIWARE**: Utilizada para o back-end.
- **Aplicativo MyMQTT**: Para visualização e manipulação dos dados IoT.
- **Docker e FIWARE Descomplicado**: Para rodar o backend FIWARE (Orion, IoT Agent, etc.).

## Instalação e Configuração

### 1. Configuração do ESP32

#### Passo 1: Instalar o Arduino IDE
Faça o download e instale o Arduino IDE a partir do site oficial: [Arduino IDE](https://www.arduino.cc/en/software).

#### Passo 2: Configurar o ESP32 no Arduino IDE
Adicione a URL para gerenciar placas ESP32 em *Preferências* no Arduino IDE:

```
https://dl.espressif.com/dl/package_esp32_index.json
```

Em seguida, vá para **Ferramentas > Placas > Gerenciador de Placas**, procure por "ESP32" e instale.

#### Passo 3: Instalar Bibliotecas
No Arduino IDE, vá para **Sketch > Incluir Biblioteca > Gerenciar Bibliotecas** e instale as seguintes:

- **WiFi**: Para conectividade Wi-Fi.
- **PubSubClient**: Para comunicação MQTT.
- **DHT**: Para leitura do sensor de temperatura e umidade.

#### Passo 4: Carregar o Código

Copie o código abaixo e carregue no ESP32. Esse código lê os dados dos sensores e envia via MQTT para o FIWARE.

```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>

// Definição dos pinos dos sensores
#define DHTPIN 4
#define DHTTYPE DHT22
#define LDRPIN 34

// Definição das credenciais de Wi-Fi
const char* ssid = "SEU_SSID";
const char* password = "SUA_SENHA";

// Definição do broker MQTT
const char* mqtt_server = "BROKER_IP";
WiFiClient espClient;
PubSubClient client(espClient);
DHT dht(DHTPIN, DHTTYPE);

// Função para conexão com Wi-Fi
void setup_wifi() {
  delay(10);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }
}

// Função de callback MQTT
void callback(char* topic, byte* message, unsigned int length) {
}

// Função de reconexão ao broker MQTT
void reconnect() {
  while (!client.connected()) {
    if (client.connect("ESP32Client")) {
      client.subscribe("vinheria/sensores");
    } else {
      delay(5000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dht.begin();
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Leitura dos sensores
  float h = dht.readHumidity();
  float t = dht.readTemperature();
  int ldrValue = analogRead(LDRPIN);

  // Publicação dos dados no tópico MQTT
  String payload = "Temperatura: " + String(t) + "C, Umidade: " + String(h) + "%, Luminosidade: " + String(ldrValue);
  client.publish("vinheria/sensores", payload.c_str());

  delay(2000); // Delay de 2 segundos entre leituras
}
```

### 2. Configuração do FIWARE

#### Passo 1: Instalar Docker e clonar o repositório FIWARE Descomplicado

1. No terminal do Ubuntu, execute:
   ```bash
   sudo apt update
   sudo apt install docker.io
   sudo docker run hello-world
   ```

2. Clone o repositório FIWARE Descomplicado:
   ```bash
   git clone https://github.com/FiwareDescomplicado/fiware.git
   ```

#### Passo 2: Subir os contêineres FIWARE

Dentro do diretório clonado, suba os contêineres:
```bash
docker-compose up -d
```

### 3. Teste e Visualização dos Dados no MyMQTT

1. Baixe o aplicativo **MyMQTT** na Play Store.
2. Configure a conexão com o broker MQTT usando o endereço IP do servidor.
3. Assine o tópico `vinheria/sensores` para visualizar os dados enviados pelo ESP32.

## Estrutura do Projeto

```
vinheria-agnello-iot/
│
├── src/
│   ├── main.cpp           # Código fonte do ESP32
│   ├── lib/
│   └── includes/
│
├── docker-compose.yml      # Arquivo de configuração do FIWARE
└── README.md               # Instruções detalhadas
```

## Conclusão

Este projeto fornece uma solução completa de IoT para monitoramento das condições de armazenamento de vinhos. Com sensores DHT22 e LDR, os dados são coletados e enviados via MQTT para o back-end FIWARE. As leituras históricas podem ser exportadas para análises avançadas, ajudando a Vinheria Agnello a melhorar o controle de qualidade dos vinhos.

