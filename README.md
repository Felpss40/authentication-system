# 🛡️ AdvancedAuth API - Arquitetura de Licenciamento e Autenticação

Este projeto consiste na arquitetura de back-end de um **Sistema de Licenciamento de Software**. Ele foi projetado para gerenciar a distribuição, autenticação e proteção de softwares (Desktop/Web), garantindo que apenas usuários com licenças válidas tenham acesso ao sistema e aos seus módulos internos.

O grande foco desta aplicação é a **segurança de rede e a prevenção contra crackers e engenharia reversa**, utilizando técnicas avançadas de criptografia e validação contínua de estado (Heartbeat).

> **⚠️ Aviso de Escopo e Segurança**
> 
> Este repositório atua como um portfólio de engenharia de software e contém uma **versão sanitizada e parcial** da aplicação. Como este sistema opera ativamente em ambiente de produção, artefatos críticos (chaves criptográficas RSA, bancos de dados reais, `.env`, binários e painéis administrativos) foram estritamente removidos para garantir a segurança da infraestrutura.

---

## 💡 A Ideia do Projeto

A motivação principal foi construir uma infraestrutura de controle de acesso que não dependesse apenas de validações simples (como checar uma string no banco de dados). O sistema precisava ser resiliente contra interceptações de rede (*Packet Sniffing*) e manipulação de respostas (*Man-in-the-Middle*).

Para isso, a API atua como um "cofre" inteligente: ela não apenas valida a licença do usuário, mas estabelece um túnel seguro de comunicação com o software cliente, monitora o uso em tempo real e entrega os arquivos sensíveis da aplicação apenas em memória, após a autenticação ser confirmada.

---

## 🚀 Destaques Arquiteturais e de Segurança

Abaixo estão os principais desafios técnicos resolvidos por este back-end:

### 1. Criptografia Híbrida de Ponta a Ponta (RSA + AES)
Nenhuma informação sensível viaja em texto puro. O sistema utiliza criptografia assimétrica (RSA-2048) para realizar o *handshake* inicial e trocar de forma segura as chaves simétricas (AES-256). A partir daí, todo o *payload* das requisições e respostas é encriptado, invalidando tentativas de interceptação via ferramentas como Fiddler ou Wireshark.

### 2. Gestão de Sessões por *Heartbeat* e Bloqueio de HWID
Para evitar o compartilhamento ilícito de licenças, o sistema atrela a key do usuário ao seu **Hardware ID (HWID)** no primeiro login. Além disso, implementa um sistema de *Heartbeat* (pulso): o cliente deve enviar sinais periódicos para a API. Se o sinal falhar ou se uma segunda máquina tentar conectar com a mesma key, a sessão anterior é imediatamente revogada.

### 3. Entrega Dinâmica de Módulos
Em vez de o software cliente conter todo o código crítico, partes essenciais (módulos, DLLs ou scripts) ficam armazenadas no servidor. A API possui rotas seguras para entregar esses módulos dinamicamente apenas para sessões validadas e criptografadas, dificultando o *dumping* dos arquivos.

### 4. API Pública para Revendedores (Reseller System)
O sistema conta com rotas específicas para criação de integrações B2B. Revendedores autorizados recebem tokens JWT com escopos limitados para gerar, renovar e gerenciar chaves de acesso de seus próprios clientes, tudo de forma automatizada e registrada.

### 5. Defesa Ativa (Anti-Abuse)
* **Rate Limiting:** Regras estritas de limite de requisições por IP para mitigar ataques de força bruta (Brute-Force) ou DDoS em endpoints de autenticação.
* **Blacklist Dinâmica:** Sistema integrado que permite banir permanentemente acessos vindos de determinados IPs ou HWIDs suspeitos.

---

## 🛠️ Stack Tecnológico

A escolha das tecnologias priorizou a performance I/O, a segurança criptográfica nativa e a escalabilidade:

* **Ecossistema:** Node.js, Express.js
* **Armazenamento:** SQLite (configurado para alta concorrência)
* **Segurança Base:** Módulo nativo `crypto` (Node.js), JSON Web Tokens (JWT) para autorização de rotas administrativas.
* **Design Pattern:** MVC (Model-View-Controller) para uma separação clara entre as regras de negócio de *Auth*, *Logs*, *Sessions* e *Keys*.

## 🔌 Experiência do Desenvolvedor (DX)

Para demonstrar a facilidade de consumo desta API por diferentes plataformas, a pasta `examples/` deste repositório contém implementações de clientes robustos. Eles demonstram todo o fluxo de *handshake* RSA e encriptação AES de forma nativa nas seguintes linguagens:
- **C#** - **Python**
- **Node.js**
