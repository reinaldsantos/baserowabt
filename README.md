# 🏥 Baserow Self-Hosted – Clínica Médica

![Docker](https://img.shields.io/badge/Docker-29.2.1-blue)
![Baserow](https://img.shields.io/badge/Baserow-1.34.5-purple)
![Status](https://img.shields.io/badge/status-running-success)
![License](https://img.shields.io/badge/license-academic-lightgrey)

---

## 📑 Índice

- [1. Visão Geral](#1-visão-geral)
- [2. Ambiente Utilizado](#2-ambiente-utilizado)
- [3. Implantação](#3-implantação)
  - [3.1 Docker](#31-docker)
  - [3.2 Firewall](#32-firewall)
- [4. Modelagem do Banco](#4-modelagem-do-banco)
- [5. Registros de Teste](#5-registros-de-teste)
- [6. Autenticação](#6-autenticação)
- [7. API](#7-api)
- [8. Exemplos de Requisições](#8-exemplos-de-requisições)
- [9. Acesso à Interface Web](#9-acesso-à-interface-web)
- [10. Troubleshooting](#10-troubleshooting)
- [11. Comandos Docker Úteis](#11-comandos-docker-úteis)
- [12. Estado Atual](#12-estado-atual)
- [13. Considerações Finais](#13-considerações-finais)

---

## 1. Visão Geral

Este projeto implementa uma base de dados **Baserow self-hosted** utilizando Docker, acessível em rede local, simulando o gerenciamento de uma clínica médica.  
O desafio técnico consistia em:

- Deploy via IP na rede local
- Modelagem com relacionamentos
- Inserção de dados de teste (mínimo 10 registros)
- Exposição de API REST com operações CRUD
- Documentação completa

---

## 2. Ambiente Utilizado

| Item                  | Valor                         |
|-----------------------|-------------------------------|
| Sistema Operacional   | Windows 10 Pro                |
| Docker Desktop        | 29.2.1 (engine 4.63.0)        |
| Baserow               | `baserow/baserow:1.34.5`      |
| IP da máquina         | `192.168.0.109`               |
| Porta                 | `8080`                        |

---

## 3. Implantação

### 3.1 🐳 Docker

Execute o seguinte comando no **PowerShell ou CMD** para criar e iniciar o container:

```bash
docker run -d `
  --name baserow-desafio `
  -e BASEROW_PUBLIC_URL=http://192.168.0.109:8080 `
  -v baserow_data:/baserow/data `
  -p 8080:80 `
  -p 8443:443 `
  --restart unless-stopped `
  baserow/baserow:1.34.5
Explicação dos parâmetros:

-d → executa em segundo plano (detached)

--name → define um nome para o container

-e → define a variável de ambiente BASEROW_PUBLIC_URL (necessária para o frontend funcionar)

-v → cria um volume persistente (baserow_data) para manter os dados

-p → mapeia a porta 8080 do host para a porta 80 do container (acesso web) e 8443 para 443 (HTTPS opcional)

--restart unless-stopped → reinicia automaticamente se o container parar acidentalmente

3.2 🔥 Firewall (Windows)
Para permitir que outros computadores na mesma rede acessem o Baserow, libere a porta 8080 no Firewall do Windows:

bash
netsh advfirewall firewall add rule name="Baserow 8080" dir=in action=allow protocol=TCP localport=8080
Verifique se a regra foi criada:
netsh advfirewall firewall show rule name="Baserow 8080"

4. Modelagem do Banco
🧑‍⚕️ Pacientes (ID: 711)
Campo	Tipo	Descrição
Name	Texto	Nome do paciente
Date	Data	Data de nascimento
Number	Texto	Telefone
Completed	Booleano	Ativo / Inativo
Email	Texto	E-mail
👨‍⚕️ Médicos (ID: 713)
Campo	Tipo	Descrição
Nome	Texto	Nome completo
CRM	Texto	Registro profissional
Especialidade	Texto	Especialidade
📅 Consultas (ID: 712)
Campo	Tipo	Descrição
DataConsulta	Data	Data da consulta
HoraConsulta	Texto	Horário (HH:MM)
Status	Single Select	Agendada / Realizada / Cancelada
Notes	Long Text	Observações
Paciente	Link	Referência à tabela Pacientes
Medico	Link	Referência à tabela Médicos
🔗 Relacionamentos
Consultas.Paciente → Pacientes.ID (muitos‑para‑um)

Consultas.Medico → Médicos.ID (muitos‑para‑um)

5. Registros de Teste
Total: 11 registros (mínimo exigido: 10)

Tabela	Quantidade
Pacientes	5
Médicos	3
Consultas	3
6. Autenticação
🔑 Database Token (para CRUD)
text
ZflqGRqnGhdjsYEZ4gzZVP0wH9e1J2KN
Todas as requisições devem incluir o cabeçalho:

http
Authorization: Token ZflqGRqnGhdjsYEZ4gzZVP0wH9e1J2KN
Content-Type: application/json
🔐 JWT Token (para operações administrativas)
Para gerar um JWT token, utilize o comando abaixo (substitua e‑mail/senha se necessário):

bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"email":"reinaldasntos312@gmail.com","password":"12345678"}' \
  http://192.168.0.109:8080/api/user/token-auth/
7. API
Base URL: http://192.168.0.109:8080/api

Operação	Método	Endpoint
Listar pacientes	GET	/database/rows/table/711/?user_field_names=true
Criar paciente	POST	/database/rows/table/711/?user_field_names=true
Listar médicos	GET	/database/rows/table/713/?user_field_names=true
Criar médico	POST	/database/rows/table/713/?user_field_names=true
Listar consultas	GET	/database/rows/table/712/?user_field_names=true
Criar consulta	POST	/database/rows/table/712/?user_field_names=true
Atualizar registro	PATCH	/database/rows/table/{table_id}/{row_id}/?user_field_names=true
Deletar registro	DELETE	/database/rows/table/{table_id}/{row_id}/?user_field_names=true
8. Exemplos de Requisições
📥 Listar todos os médicos
bash
curl -X GET \
  -H "Authorization: Token ZflqGRqnGhdjsYEZ4gzZVP0wH9e1J2KN" \
  "http://192.168.0.109:8080/api/database/rows/table/713/?user_field_names=true"
📤 Criar uma consulta
bash
curl -X POST \
  -H "Authorization: Token ZflqGRqnGhdjsYEZ4gzZVP0wH9e1J2KN" \
  -H "Content-Type: application/json" \
  -d '{
    "DataConsulta": "2024-03-25",
    "HoraConsulta": "14:30",
    "Status": "Agendada",
    "Notes": "Consulta de rotina",
    "Paciente": [6],
    "Medico": [3]
  }' \
  "http://192.168.0.109:8080/api/database/rows/table/712/?user_field_names=true"
✏️ Atualizar o telefone de um paciente (campo Number)
bash
curl -X PATCH \
  -H "Authorization: Token ZflqGRqnGhdjsYEZ4gzZVP0wH9e1J2KN" \
  -H "Content-Type: application/json" \
  -d '{"Number": "99999"}' \
  "http://192.168.0.109:8080/api/database/rows/table/711/6/?user_field_names=true"
❌ Deletar um médico
bash
curl -X DELETE \
  -H "Authorization: Token ZflqGRqnGhdjsYEZ4gzZVP0wH9e1J2KN" \
  "http://192.168.0.109:8080/api/database/rows/table/713/6/?user_field_names=true"
9. Acesso à Interface Web
Local: http://localhost:8080

Rede local: http://192.168.0.109:8080

Credenciais de login
E‑mail: reinaldasntos312@gmail.com

Senha: 12345678

10. Troubleshooting
Problema	Provável causa e solução
ERR_CONNECTION_REFUSED	Docker não iniciado ou IP mudou. Verifique docker ps e ipconfig.
Site not found	BASEROW_PUBLIC_URL incorreta. Recrie o container com o IP atual.
Colegas não acessam	Firewall bloqueando. Libere a porta 8080 com o comando netsh advfirewall.
Dados sumiram	Volume baserow_data foi removido. Se não removeu, os dados ainda existem.
401 Unauthorized	Token inválido ou ausente. Envie o cabeçalho Authorization: Token <token>.
11. Comandos Docker Úteis
Ação	Comando
Iniciar container	docker start baserow-desafio
Parar container	docker stop baserow-desafio
Ver status	docker ps
Ver logs em tempo real	docker logs -f baserow-desafio
Ver últimas 50 linhas	docker logs --tail 50 baserow-desafio
Recriar com novo IP	docker rm baserow-desafio
docker run ... (com novo IP)
12. Estado Atual
✅ Container rodando e saudável

🌐 IP atual: 192.168.0.109

🔗 API funcional e acessível

💾 Dados intactos

🔥 Porta 8080 liberada no firewall

13. Considerações Finais
Todos os critérios do desafio foram cumpridos:

✔️ Deploy funcional na rede local

✔️ Modelagem correta (tabelas + relacionamentos)

✔️ Dados de teste inseridos (>10 registros)

✔️ CRUD funcional via API

✔️ Documentação completa

👨‍💻 Autor: Gabriel Santos
📅 Data: 23/03/2026