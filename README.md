<div align="center">
  <img src="https://readme-typing-svg.demolab.com?font=Inter&weight=800&size=30&pause=1000&color=D32F2F&center=true&vCenter=true&width=700&lines=CP4+-+Smart+Lamp;FIWARE+%2B+ESP32+%2B+AWS;Equipe+Smart+Solutions" alt="Animacao Titulo" />
</div>

<p align="center">
  <img src="https://img.shields.io/badge/ESP32-000000?style=for-the-badge&logo=espressif&logoColor=white" alt="ESP32" />
  <img src="https://img.shields.io/badge/C%2B%2B-00599C?style=for-the-badge&logo=cplusplus&logoColor=white" alt="C++" />
  <img src="https://img.shields.io/badge/AWS%20EC2-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white" alt="AWS EC2" />
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker" />
  <img src="https://img.shields.io/badge/MQTT-660066?style=for-the-badge&logo=mqtt&logoColor=white" alt="MQTT" />
  <img src="https://img.shields.io/badge/Postman-FF6C37?style=for-the-badge&logo=postman&logoColor=white" alt="Postman" />
  <img src="https://img.shields.io/badge/FIWARE-1F8ACB?style=for-the-badge&logo=data%3Aimage%2Fpng%3Bbase64%2C&logoColor=white" alt="FIWARE" />
</p>

Projeto desenvolvido para o **Check Point 4** da disciplina de Edge Computing (FIAP), com o objetivo de conectar um ESP32 (equipado com sensor LDR) ao FIWARE, demonstrando o fluxo bidirecional de dados entre a borda (**edge**) e a nuvem (**cloud**).

---

## 👥 Equipe — Smart Solutions

| Integrante | RM |
|---|---|
| Eduarda Soares Moraes | RM569369 |
| Isac Nilton Fernandes de Oliveira | RM573282 |
| João Benedito de Oliveira Simplício | RM570206 |
| Julia Souza Matarazzo | RM571340 |
| Mariana Malagutti Gomes Peixoto | RM570290 |

**Turma:** 1ESPG &nbsp;|&nbsp; **Professor:** Fábio H. Cabrini

---

## 🚀 Sobre o projeto

A **Smart Lamp** é uma PoC (*Proof of Concept*) que utiliza o FIWARE como back-end, seguindo o padrão **NGSI** (*Next Generation Service Interfaces*), para representar uma lâmpada conectada capaz de:

- 📡 Enviar, em tempo real, a leitura de luminosidade do ambiente (sensor LDR) para a plataforma FIWARE.
- 🕹️ Receber comandos remotos (ligar/desligar) enviados via Postman, acionando o LED onboard do ESP32.

## 🏗️ Arquitetura

```
📟 ESP32 (LDR + LED)
      │  MQTT (publish luminosidade / status)
      ▼
🦟 Mosquitto (broker MQTT, rodando na AWS EC2)
      │
      ▼
🔀 IoT Agent MQTT (traduz MQTT ↔ NGSI)
      │
      ▼
🧠 Orion Context Broker (entidade urn:ngsi-ld:Lamp:200)
      ▲
      │  HTTP / NGSI-v2 (GET consulta estado, PATCH envia comando)
      │
📬 Postman
```

O stack de back-end (Orion, IoT Agent MQTT, STH-Comet e Mosquitto) está hospedado em uma instância **AWS EC2**, provisionada a partir do repositório [FIWARE Descomplicado](https://github.com/fabiocabrini/fiware), do Prof. Fábio Cabrini.

## 🏷️ Identificação da entidade

| Item | Valor |
|---|---|
| 🔑 Device ID | `lamp200` |
| 🧩 Entity Name | `urn:ngsi-ld:Lamp:200` |
| 💡 Entity Type | `Lamp` |
| 🗂️ FIWARE Service | `smart` |
| 🗂️ FIWARE Service Path | `/` |
| 🔐 API Key | `TEF` |
| 📶 Protocolo | PDI-IoTA-UltraLight (MQTT) |

### 📡 Tópicos MQTT

| Tópico | Direção | Conteúdo |
|---|---|---|
| `/TEF/lamp200/attrs` | ESP32 ➡️ broker | Status do LED (`s\|on` / `s\|off`) |
| `/TEF/lamp200/attrs/l` | ESP32 ➡️ broker | Valor de luminosidade (`l`) |
| `/TEF/lamp200/cmd` | broker ➡️ ESP32 | Comandos recebidos (`lamp200@on\|` / `lamp200@off\|`) |

## 💻 Código do ESP32

Arquivo: [`fiware_ngsi_mqtt_esp32.ino`](./fiware_ngsi_mqtt_esp32.ino)

Baseado no template oficial do Prof. Fábio Cabrini, adaptado com:
- ⚙️ IP do broker MQTT apontando para a instância AWS EC2 do projeto.
- 🏷️ Identificação da entidade (`lamp200` / `fiware_200`) correspondente a este projeto.

## 🧪 Simulação no Wokwi

<p align="center">
  <a href="https://wokwi.com/projects/475103131124330497" target="_blank">
    <img src="https://img.shields.io/badge/Abrir_Simulação-Wokwi-1A73E8?style=for-the-badge&logo=arduino&logoColor=white" alt="Botão Wokwi" />
  </a>
</p>

Para reproduzir o teste completo:

1. ▶️ Abra o link do Wokwi acima e inicie a simulação.
2. 📶 Aguarde a conexão Wi-Fi (`Wokwi-GUEST`) e MQTT no Serial Monitor.
3. 🔍 Consulte a entidade no Orion via Postman para ver a luminosidade sendo atualizada.
4. 💡 Envie um comando `on`/`off` via Postman e observe o LED onboard mudar de estado no simulador.

## 📬 Testando via Postman

<p align="center">
  <a href="https://github.com/fabiocabrini/fiware/blob/main/FIWARE%20Descomplicado.postman_collection.json" target="_blank">
    <img src="https://img.shields.io/badge/Collection_de_Referência-Postman-FF6C37?style=for-the-badge&logo=postman&logoColor=white" alt="Botão Postman" />
  </a>
</p>

Headers obrigatórios nas requisições ao Orion e ao IoT Agent:

```
fiware-service: smart
fiware-servicepath: /
```

**🔍 Consultar estado da entidade:**
```
GET http://<IP-da-EC2>:1026/v2/entities/urn:ngsi-ld:Lamp:200
```

**🕹️ Enviar comando (ligar o LED):**
```
PATCH http://<IP-da-EC2>:1026/v2/entities/urn:ngsi-ld:Lamp:200/attrs
Content-Type: application/json

{
  "on": {
    "type": "command",
    "value": ""
  }
}
```

## 🎥 Vídeo de demonstração

<p align="center">
  <a href="https://youtu.be/IbLhBxr1tMY?feature=shared" target="_blank">
    <img src="https://img.shields.io/badge/Assistir_no-YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white" alt="Botão YouTube" />
  </a>
</p>

## 📚 Referências

<p align="center">
  <a href="https://github.com/fabiocabrini/fiware" target="_blank">
    <img src="https://img.shields.io/badge/FIWARE_Descomplicado-Prof._Fábio_Cabrini-181717?style=for-the-badge&logo=github&logoColor=white" alt="Botão FIWARE Descomplicado" />
  </a>
  <a href="https://fiware-tutorials.readthedocs.io/en/stable/getting-started/index.html" target="_blank">
    <img src="https://img.shields.io/badge/Documentação-NGSI--v2-3ECF8E?style=for-the-badge&logo=readthedocs&logoColor=white" alt="Botão NGSI-v2" />
  </a>
  <a href="https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf" target="_blank">
    <img src="https://img.shields.io/badge/Datasheet-ESP32-E7352C?style=for-the-badge&logo=espressif&logoColor=white" alt="Botão Datasheet ESP32" />
  </a>
</p>
