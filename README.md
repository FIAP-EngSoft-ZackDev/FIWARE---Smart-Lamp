# CP4 - Smart Lamp | FIWARE + ESP32 + AWS

Projeto desenvolvido para o Check Point 4 da disciplina de Edge Computing (FIAP), com o objetivo de conectar um ESP32 (equipado com sensor LDR) ao FIWARE, demonstrando o fluxo bidirecional de dados entre a borda (edge) e a nuvem (cloud).

**Autor:** Isac Oliveira
**Turma:** 1ESPG
**Professor:** Dr. Fábio H. Cabrini

## Sobre o projeto

A Smart Lamp é uma PoC (Proof of Concept) que utiliza o FIWARE como back-end, seguindo o padrão NGSI (Next Generation Service Interfaces), para representar uma lâmpada conectada capaz de:

- Enviar, em tempo real, a leitura de luminosidade do ambiente (sensor LDR) para a plataforma FIWARE.
- Receber comandos remotos (ligar/desligar) enviados via Postman, acionando o LED onboard do ESP32.

## Arquitetura

```
ESP32 (LDR + LED)
      │  MQTT (publish luminosidade / status)
      ▼
Mosquitto (broker MQTT, rodando na AWS EC2)
      │
      ▼
IoT Agent MQTT (traduz MQTT ↔ NGSI)
      │
      ▼
Orion Context Broker (entidade urn:ngsi-ld:Lamp:200)
      ▲
      │  HTTP / NGSI-v2 (GET consulta estado, PATCH envia comando)
      │
   Postman
```

O stack de back-end (Orion, IoT Agent MQTT, STH-Comet e Mosquitto) está hospedado em uma instância AWS EC2, provisionada a partir do repositório [FIWARE Descomplicado](https://github.com/fabiocabrini/fiware), do Prof. Fábio Cabrini.

## Identificação da entidade

| Item | Valor |
|---|---|
| Device ID | `lamp200` |
| Entity Name | `urn:ngsi-ld:Lamp:200` |
| Entity Type | `Lamp` |
| FIWARE Service | `smart` |
| FIWARE Service Path | `/` |
| API Key | `TEF` |
| Protocolo | PDI-IoTA-UltraLight (MQTT) |

### Tópicos MQTT

| Tópico | Direção | Conteúdo |
|---|---|---|
| `/TEF/lamp200/attrs` | ESP32 → broker | Status do LED (`s\|on` / `s\|off`) |
| `/TEF/lamp200/attrs/l` | ESP32 → broker | Valor de luminosidade (`l`) |
| `/TEF/lamp200/cmd` | broker → ESP32 | Comandos recebidos (`lamp200@on\|` / `lamp200@off\|`) |

## Código do ESP32

Arquivo: [`fiware_ngsi_mqtt_esp32.ino`](./fiware_ngsi_mqtt_esp32.ino)

Baseado no template oficial do Prof. Fábio Cabrini, adaptado com:
- IP do broker MQTT apontando para a instância AWS EC2 do projeto.
- Identificação da entidade (`lamp200` / `fiware_200`) correspondente a este projeto.

## Simulação no Wokwi

🔗 [Acessar simulação no Wokwi](https://wokwi.com/projects/475103131124330497)

Para reproduzir o teste completo:

1. Abra o link do Wokwi acima e inicie a simulação.
2. Aguarde a conexão Wi-Fi (`Wokwi-GUEST`) e MQTT no Serial Monitor.
3. Consulte a entidade no Orion via Postman para ver a luminosidade sendo atualizada.
4. Envie um comando `on`/`off` via Postman e observe o LED onboard mudar de estado no simulador.

## Testando via Postman

Collection de referência: [FIWARE Descomplicado.postman_collection.json](https://github.com/fabiocabrini/fiware/blob/main/FIWARE%20Descomplicado.postman_collection.json)

Headers obrigatórios nas requisições ao Orion e ao IoT Agent:

```
fiware-service: smart
fiware-servicepath: /
```

**Consultar estado da entidade:**
```
GET http://<IP-da-EC2>:1026/v2/entities/urn:ngsi-ld:Lamp:200
```

**Enviar comando (ligar o LED):**
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

## Vídeo de demonstração

🔗 [Link do vídeo (time-lapse, até 3 min)](#) <!-- substituir pelo link do vídeo -->

## Referências

- [FIWARE Descomplicado](https://github.com/fabiocabrini/fiware) — Prof. Fábio Henrique Cabrini
- [Documentação NGSI-v2](https://fiware-tutorials.readthedocs.io/en/stable/getting-started/index.html)
- [Datasheet ESP32](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf)
