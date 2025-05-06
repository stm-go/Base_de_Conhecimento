# Configuração IPsec [Legacy] no OPNsense para pfSense

Este guia descreve o processo de configuração de um túnel IPsec legado no OPNsense para pfSense, utilizando IKEv2 e autenticação por Pre-Shared Key (PSK). A configuração está dividida em **Fase 1 (negociação inicial)** e **Fase 2 (definição de tráfego permitido)**.

---

## 🔧 1º Passo – Configuração no Ambiente da Stmgo ou Matriz
📍 Acesso Inicial

Navegue no menu:

**VPN > IPsec > Tunnel Settings [Legacy]**

Clique em **“+ Add P1”** para adicionar uma nova Fase 1 (Phase 1).

---

## 🔐 Phase 1 – Parâmetros de Conexão

### 🔸 General Information

| Parâmetro             | Valor / Descrição                                                                 |
|-----------------------|-----------------------------------------------------------------------------------|
| **Key Exchange version** | IKEv2                                                                            |
| **Interface**            | Interface WAN com IP público (ex: `stmgo` ou `matriz`)                          |
| **Remote Gateway**       | IP público da filial ou cliente remoto                                          |
| **Description**          | Nome do cliente/filial em letras maiúsculas (ex: `FILIAL01`)                   |

---

### 🔸 Phase 1 Proposal (Authentication)

| Parâmetro               | Valor / Descrição                                                                 |
|-------------------------|-----------------------------------------------------------------------------------|
| **Authentication method** | Mutual PSK (Pre-Shared Key)                                                      |
| **My identifier**         | IP público local (selecione "IP Address" e insira IP da `stmgo` ou `matriz`)     |
| **Peer identifier**       | IP público remoto (cliente ou filial)                                            |
| **Pre-Shared Key**        | Gere uma senha forte e armazene com segurança. Será usada nos dois lados do túnel |

---

### 🔸 Phase 1 Proposal (Algorithms)

| Parâmetro            | Valor                  |
|----------------------|------------------------|
| **Encryption Algorithm** | AES 256                |
| **Hash Algorithm**       | SHA256                 |
| **DH Key Group**         | 14 (2048 bits)         |
| **Lifetime**             | 28800 segundos (8h)    |

*Os demais campos podem permanecer com os valores padrão.*

---

## 🔐 Phase 2 – Parâmetros de Tráfego

Clique em **“+ Add P2”** dentro do túnel Fase 1 criado.

### 🔸 General Information

| Parâmetro     | Valor / Descrição                                                             |
|---------------|-------------------------------------------------------------------------------|
| **Description** | Nome do cliente/filial em maiúsculo seguido de `-F2` (ex: `ACCION-F2`)        |

---

### 🔸 Local Network

Define o recurso interno que será exposto à outra ponta do túnel.

| Parâmetro | Valor / Exemplo                                                                 |
|-----------|----------------------------------------------------------------------------------|
| **Type**    | Address                                                                         |
| **Address** | IP da máquina a ser liberada (ex: `172.16.9.106`, uma VM proxy para monitoramento) |

---

### 🔸 Remote Network

Define o recurso remoto que será acessado via IPsec.

| Parâmetro | Valor / Exemplo                                                               |
|-----------|--------------------------------------------------------------------------------|
| **Type**    | Address                                                                       |
| **Address** | IP do firewall ou rede específica no cliente (ex: `182.168.1.1`)              |

---

### 🔸 Phase 2 Proposal (SA/Key Exchange)

| Parâmetro              | Valor            |
|------------------------|------------------|
| **Protocol**             | ESP              |
| **Encryption Algorithm** | AES256           |
| **Hash Algorithm**       | SHA256           |
| **PFS Key Group**        | 12 (2048 bits)   |
| **Lifetime**             | 28800 segundos   |

---

## ✅ Observações Finais

- **Regra de firewall:** após configurar o túnel, **crie uma regra na interface IPsec** para permitir o tráfego necessário, como no exemplo abaixo:

  **Exemplo** (para liberação de monitoramento Zabbix):
  - **Interface:** IPsec
  - **Protocolo:** TCP/UDP
  - **Origem:** rede local de monitoramento (ex: `INFRA net`)
  - **Destino:** `172.16.9.106`
  - **Portas de destino:** `161`, `3000`, `10050`, `10051` (alias: `portas_zabbix`)
  - **Descrição:** Liberação portas Zabbix - Monitoramento

---

# Configuração IPsec no PfSense (Cliente)
Este documento descreve como configurar um túnel IPsec no **PfSense do lado do cliente**, utilizando IKEv2 e autenticação com Pre-Shared Key (PSK).



## 🔧 1º Passo – Configuração no Ambiente do Cliente ou Filial

### 🔹 Acesso ao menu

Navegue em:  
**VPN > IPsec**

Clique em **“+ Add P1”** para criar a Fase 1.

---

## 🔐 Phase 1 – Negociação inicial

### 🔸 General Information

| Parâmetro     | Valor / Descrição                              |
|---------------|------------------------------------------------|
| Description   | STMGO                                          |

### 🔸 IKE Endpoint Configuration

| Parâmetro               | Valor / Descrição                             |
|-------------------------|-----------------------------------------------|
| Key Exchange version    | IKEv2                                          |
| Internet Protocol       | IPv4                                           |
| Interface               | Interface WAN (com IP público do cliente)     |
| Remote Gateway          | IP público da STMGO                           |

### 🔸 Phase 1 Proposal (Authentication)

| Parâmetro           | Valor / Descrição                                  |
|---------------------|----------------------------------------------------|
| Authentication Method | Mutual PSK                                       |
| My Identifier         | IP Address (IP público do cliente)               |
| Peer Identifier       | IP Address (IP público da STMGO)                 |
| Pre-Shared Key        | Chave forte (armazenar com segurança)            |

### 🔸 Phase 1 Proposal (Encryption Algorithm)

| Parâmetro             | Valor                  |
|-----------------------|------------------------|
| Encryption Algorithm  | AES 256 bits           |
| Hash Algorithm        | SHA256                 |
| DH Group              | 14 (2048 bits)         |

Os demais campos podem permanecer com os valores padrão.

---

## 🔐 Phase 2 – Definição de tráfego permitido

Clique em **“+ Add P2”** dentro do túnel criado.

### 🔸 General Information

| Parâmetro     | Valor / Descrição |
|---------------|-------------------|
| Description   | STMGO-F2          |

### 🔸 Networks

| Campo              | Valor / Exemplo                                  |
|--------------------|--------------------------------------------------|
| Local Network      | *(rede do cliente)*               |
| NAT/BINAT          | None *(usar NAT se redes forem iguais nas pontas)* |
| Remote Network     | *(IP do proxy na STMGO)*            |

### 🔸 Phase 2 Proposal (SA/Key Exchange)

| Parâmetro              | Valor             |
|------------------------|-------------------|
| Protocol               | ESP               |
| Encryption Algorithm   | AES 256 bits      |
| Hash Algorithm         | SHA256            |
| PFS Key Group          | 14 (2048 bits)    |

### 🔸 Expiration and Replacement

| Parâmetro     | Valor     |
|---------------|-----------|
| Life Time     | 28800     |
| Rekey Time    | 25920     |
| Rand Time     | 2880      |

---

## 📌 Regra de Firewall (Interface IPsec)

Após configurar o túnel IPsec, é necessário criar uma regra na interface **IPsec** para permitir o tráfego entre os lados. Exemplo:

| Campo           | Valor                             |
|-----------------|-----------------------------------|
| Interface       | IPsec                             |
| Address Family  | IPv4                              |
| Protocol        | Any *(ou especifique TCP/UDP)*    |
| Source          | Any *(ou rede STMGO)*             |
| Destination     | Any *(ou rede local)*             |
| Log             | ✅ Ativar registro, se necessário  |
| Description     | Permitir tráfego IPsec da STMGO   |

---

- **Teste de conectividade:** utilize `ping`, `traceroute` ou o próprio sistema de monitoramento (ex: Zabbix) para validar se o túnel está funcionando conforme esperado.

---

## 🧠 Justificativa das Configurações

### 🔹 Key Exchange version: IKEv2
O IKEv2 é mais seguro e eficiente em relação ao IKEv1, além de suportar reconexão automática e NAT traversal de forma mais robusta.

### 🔹 Autenticação via PSK (Mutual PSK)
A autenticação por chave pré-compartilhada (PSK) é simples e eficaz para túneis site-to-site. Utilizamos uma chave forte e compartilhada entre os pares para garantir segurança na troca inicial.

### 🔹 Identificadores por IP
Utilizamos IPs públicos como identificadores porque essa abordagem evita problemas de resolução de nome DNS entre os peers e é compatível com ambientes que não possuem DNS reverso ou nomes resolvíveis externamente.

### 🔹 Algoritmos Criptográficos
Selecionamos **AES-256** e **SHA256** por serem algoritmos robustos, recomendados pelas melhores práticas de segurança e amplamente suportados por firewalls modernos. O grupo DH 14 (2048 bits) oferece bom equilíbrio entre segurança e desempenho.

### 🔹 Lifetime e Rekey
A definição de 28800 segundos (8 horas) é padrão para manter sessões ativas por longos períodos sem renegociação constante. O tempo de rekey (25920s) e de aleatoriedade (2880s) garantem uma troca segura antes do término da sessão, sem interromper o túnel.

### 🔹 Tráfego Específico (Phase 2)
Limitamos o tráfego da Phase 2 a endereços específicos (ex: proxy ou firewall) por questões de segurança e controle. Isso evita abrir toda a rede local desnecessariamente, minimizando a superfície de ataque.

### 🔹 Regras de Firewall na Interface IPsec
A interface IPsec no pfSense/OPNsense não aplica automaticamente regras de permissão. É necessário definir regras para permitir tráfego entre as redes, como SNMP, Zabbix, ou ICMP, garantindo que apenas o necessário esteja liberado.

---

