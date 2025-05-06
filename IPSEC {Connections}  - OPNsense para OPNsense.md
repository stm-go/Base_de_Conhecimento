# Conexão IPsec entre dois OPNsense (Site-to-Site)

## Ambiente

- **Matriz (STMGO):** OPNsense com IP WAN fixo
- **Filial:** OPNsense com IP WAN fixo ou acessível
- **Modo:** Site-to-Site
- **Autenticação:** PSK
- **IKE Version:** IKEv2

---

## 🔧 Etapas - OPNsense (STMGO e FILIAL)

### 1. Ativar IPsec
- Navegue até: `VPN > IPsec > Connections`
- Marque **Enable IPsec** e clique em **Apply**

### 2. Criar Pre-Shared Key
Acesse:
- `VPN > IPsec > Pre-Shared Keys`
- Adicione uma nova entrada:

| Campo             | Valor                  |
|------------------|------------------------|
| Local Identifier | `stmgo`                |
| Remote Identifier| `cliente01`            |
| Pre-Shared Key   | `senha forte`          |
| Type             | `PSK`                  |

Repita na outra ponta invertendo os identificadores:

| Campo             | Valor                  |
|------------------|------------------------|
| Local Identifier | `cliente01`            |
| Remote Identifier| `stmgo`                |
| Pre-Shared Key   | mesma senha            |
| Type             | `PSK`                  |

---

### 3. Configurar Conexão IPsec (Phase 1)

#### Matriz (STMGO)

| Campo              | Valor                              |
|-------------------|-------------------------------------|
| Local Address      | Interface WAN (`186.195.113.196`)  |
| Remote Address     | WAN da filial                      |
| Description        | `VPN-STMGO-cliente01`              |
| Authentication     | PSK                                |
| Local ID           | `stmgo`                            |
| Remote ID          | `cliente01`                        |
| IKE Version        | v2                                 |
| DH Group           | 14 (2048)                          |
| Encryption         | AES-256                            |
| Hash               | SHA256                             |
| Lifetime           | 28800                              |

#### Filial (cliente01)

Mesmos valores, porém invertendo:
- Local Address: WAN da filial
- Remote Address: `WAN stmgo ou Matriz`
- Description: `VPN-cliente01-STMGO`
- Local ID: `cliente01`
- Remote ID: `stmgo`

---

### 4. Configurar Child SA (Phase 2)

#### STMGO
- Local Subnet: `192.168.10.0/24`
- Remote Subnet: `192.168.20.0/24`

#### Filial
- Local Subnet: `192.168.20.0/24`
- Remote Subnet: `192.168.10.0/24`

**Parâmetros recomendados:**
- Protocol: ESP
- Encryption: AES-256
- Hash: SHA256
- PFS: off ou group 14
- Lifetime: 3600

---

### 🔐 Regras de Firewall - Floating Rules

Criar 3 regras em:
> `Firewall > Rules > Floating`

1. **ESP**
   - Interface: WAN
   - Protocol: ESP
   - Destination: This Firewall
   - Log: Ativado
   - Description: `Allow IPsec ESP`

2. **ISAKMP**
   - Interface: WAN
   - Protocol: UDP
   - Port: 500
   - Destination: This Firewall
   - Description: `Allow IPsec ISAKMP`

3. **NAT-T**
   - Interface: WAN
   - Protocol: UDP
   - Port: 4500
   - Destination: This Firewall
   - Description: `Allow IPsec NAT-T`

---

### 🔐 Regras de Firewall - IPsec

Vá em:
> `Firewall > Rules > IPsec`

Adicione regra:

- Action: Pass
- Protocol: Any
- Source: `192.168.20.0/24`
- Destination: `192.168.10.0/24`
- Description: `Permitir cliente acessar STMGO`

E vice-versa na outra ponta.

---

## ✅ Checklist de Verificação

- [ ] WAN das duas pontas está acessível?
- [ ] PSK igual e identificadores corretos?
- [ ] Fase 1 com parâmetros compatíveis?
- [ ] Fase 2 com redes corretas e mesma criptografia?
- [ ] Regras ESP, ISAKMP, NAT-T aplicadas nas duas WANs?
- [ ] Regras IPsec para permitir tráfego?
- [ ] Tunnel aparece como “Established”?
- [ ] Ping e comunicação entre as redes funciona?

---

## 📝 Observações

- Em alguns casos, pode ser necessário marcar “Prefer older IPsec SAs” em `VPN > IPsec > Advanced Settings` se tiver múltiplos túneis.
- Utilize logs em `System > Log Files > IPsec` para depuração de erros.

