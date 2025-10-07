---
# 📘 README_FULL.md - ElatoAI Voice Client - OrangePi (Custom)

[![Python Version](https://img.shields.io/badge/python-3.10+-blue)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-Custom-orange)](LICENSE)
[![GitHub Repo](https://img.shields.io/badge/github-marcelo/elato-client--python-lightgrey)](https://github.com/marcelo/elato-client-python)

# 🤖 ElatoAI Voice Client - OrangePi

Cliente Python que substitui o ESP32-C3 no sistema ElatoAI, mantendo **100% de compatibilidade** com o backend Deno/Supabase existente.

## ✨ Características

- ✅ **Conversação contínua full-duplex** (fala e ouve simultaneamente)
- ✅ **Codec Opus** para áudio de alta qualidade e baixa latência
- ✅ **LED RGB** com feedback visual de estados
- ✅ **Controle por botão** (long press para iniciar, double click para interromper)
- ✅ **Reconexão automática** em caso de perda de conexão
- ✅ **Compatível com Gemini Live, OpenAI Realtime e ElevenLabs**
- ✅ **Serviço systemd** para execução automática no boot

## 📋 Requisitos

### Hardware
- **SBC:** Orange Pi (Zero 2W, 3 LTS, 5) ou Raspberry Pi 3/4/5
- **Áudio:** Microfone e alto-falante USB (ou placa de som USB)
- **LED RGB:** Ânodo comum de 3 cores (ou 3 LEDs individuais)
- **Botão:** Push button com pull-up interno
- **Cartão SD:** 8GB mínimo (recomendado 16GB+)

### Software
- **SO:** Armbian, Ubuntu 22.04+, Debian 11+ ou Raspberry Pi OS
- **Python:** 3.10+
- **Internet:** Conexão estável (Wi-Fi ou Ethernet)

### Servidor Backend
- Servidor Deno com backend ElatoAI rodando
- Banco Supabase configurado com dispositivo registrado

## 🚀 Instalação Rápida

### Passo 1: Clone o Repositório

```bash
git clone https://github.com/marcelo/elato-client-python.git
cd elato-client-python
```

### Passo 2: Execute o Instalador Automatizado

```bash
sudo chmod +x install.sh
sudo ./install.sh
```

O script irá:
- Atualizar o sistema
- Instalar todas as dependências
- Criar usuário `elato`
- Configurar ambiente virtual Python
- Instalar o serviço systemd

### Passo 3: Configure o Dispositivo

Edite o arquivo de configuração:

```bash
sudo nano /opt/elato/elato-client/device_config.json
```

**Configuração mínima necessária:**

```json
{
  "device": {
    "mac_address": "auto-detectado"
  },
  "server": {
    "ws_url": "ws://192.168.1.100:8080",
    "registration_endpoint": "http://192.168.1.100:8080/api/generate_auth_token"
  }
}
```

> **Importante:** Substitua `192.168.1.100` pelo IP do seu servidor Deno.

### Passo 4: Registre o Dispositivo no Supabase

1. Anote o **MAC address** detectado (mostrado no primeiro boot)
2. Acesse o painel admin do ElatoAI
3. Registre um novo dispositivo com este MAC
4. Associe o dispositivo a um usuário

### Passo 5: Conecte o Hardware

**Conexões GPIO (Orange Pi):**

```
┌─────────────────────────────┐
│ Pino 12 (GPIO1) → LED Vermelho │
│ Pino 16 (GPIO4) → LED Verde   │
│ Pino 18 (GPIO5) → LED Azul    │
│ Pino 22 (GPIO6) → Botão       │
│ GND → GND comum               │
└─────────────────────────────┘
```

**Esquema do LED RGB (Ânodo Comum):**

```
    +3.3V
      │
      ├── LED Vermelho → Resistor 220Ω → Pino 12
      ├── LED Verde    → Resistor 220Ω → Pino 16
      └── LED Azul     → Resistor 220Ω → Pino 18
```

**Esquema do Botão:**

```
Pino 22 ──┬── Botão ── GND
          │
      (Pull-up interno ativado)
```

---

## 🧪 Teste Antes de Ativar o Serviço

### 1. Teste o Áudio

```bash
arecord -d 5 -f cd /tmp/test.wav
aplay /tmp/test.wav
arecord -l
aplay -l
```

Se microfone/alto-falante não for padrão, configure em `device_config.json`.

### 2. Teste o Cliente Manualmente

```bash
cd /opt/elato/elato-client
source venv/bin/activate
python elato_client.py
```

Você deverá ver logs de inicialização com MAC detectado, registro e conexão ao servidor.

### 3. Teste a Interação

- **Long Press (500ms)** → LED amarelo, inicia conversação
- **Fale com a IA** → LED alterna amarelo/azul
- **Double Click** → Interrompe sessão, LED volta verde

---

## ⚙️ Ativar Serviço Automático

```bash
sudo systemctl enable elato-client
sudo systemctl start elato-client
sudo systemctl status elato-client
```

### Comandos Úteis do Serviço

```bash
sudo systemctl stop elato-client
sudo systemctl restart elato-client
sudo systemctl disable elato-client
sudo journalctl -u elato-client -f
```

---

## 🎨 Estados do LED

| Cor | Estado | Descrição |
|-----|--------|-----------|
| 🟢 Verde | IDLE | Pronto para uso |
| 🟡 Amarelo | LISTENING | Sessão ativa, ouvindo |
| 🔵 Azul | SPEAKING | IA falando |
| 🔴 Vermelho | PROCESSING | Processando |
| 🔴 Piscante | ERROR | Erro, reconectando |
| 🔵 Ciano | CONNECTING | Conectando ao servidor |

---

## 🎮 Controles do Botão

- **Long Press (500ms):** inicia sessão, LED amarelo
- **Double Click (<500ms):** interrompe sessão, LED verde

---

## 🔧 Configuração Avançada

Edite `device_config.json` para: pinos GPIO, dispositivos de áudio, comportamento do botão, logs detalhados.

---

## 🐛 Troubleshooting

- MAC não detectado → use `ip link show` e ajuste no `device_config.json`
- Dispositivo não registrado → registre no Supabase
- Áudio não funciona → verifique ALSA e índices de dispositivos
- GPIO não funciona → verifique permissões e teste manual
- Conexão falha → verifique IP, porta e servidor Deno
- `opuslib` não instala → instale `libopus-dev` e reinstale o pacote Python

---

## 📊 Monitoramento e Logs

```bash
sudo journalctl -u elato-client -f
sudo journalctl -u elato-client -n 100
sudo journalctl -u elato-client -p err
```

---

## 🔄 Atualizações

```bash
cd elato-client-python
git pull
sudo systemctl restart elato-client
```

Reinstalação do zero:
```bash
sudo systemctl stop elato-client
sudo systemctl disable elato-client
sudo rm -rf /opt/elato/elato-client
sudo rm /etc/systemd/system/elato-client.service
sudo ./install.sh
```

---

## 🏗️ Arquitetura

(Esquema do cliente e backend como apresentado no material original, incluindo GPIO, áudio, WebSocket e backend Deno/Supabase)

---

## 📝 Compatibilidade

- OS: Armbian 23.11+, Ubuntu 22.04 LTS, Debian 11, Raspberry Pi OS (64-bit)
- Hardware: Orange Pi Zero 2W/3 LTS/5, Raspberry Pi 4/5
- AI Providers: Gemini Live, OpenAI Realtime API, ElevenLabs

---

## 🤝 Suporte

1. Consulte Troubleshooting
2. Verifique logs `sudo journalctl -u elato-client -n 100`
3. Abra uma issue com logs, modelo SBC, SO e `device_config.json` (remova tokens)

---

## 📜 Licença

Custom (base Apache 2.0)

---

## 🎉 Pronto!

Agora você tem um assistente de voz completo rodando no Orange Pi! 🚀
