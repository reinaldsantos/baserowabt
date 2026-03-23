# Baserow Self-Hosted: Clínica Médica

## 1. Visão Geral

Este documento descreve a implementação de uma base de dados Baserow instalada localmente via Docker, configurada para funcionar em rede local, e utilizada para gerenciar uma clínica médica fictícia. O projeto foi desenvolvido como parte de um desafio técnico que exigia:

- Deploy do Baserow com acesso via IP na rede.
- Modelagem de dados com pelo menos duas tabelas e um relacionamento.
- Inserção de pelo menos 10 registros de teste.
- Exposição de uma API REST com operações CRUD (Create, Read, Update, Delete).
- Documentação completa.

## 2. Ambiente Utilizado

- **Sistema Operacional:** Windows 10 Pro
- **Docker Desktop:** Versão 29.2.1 (engine 4.63.0)
- **Baserow:** Imagem `baserow/baserow:1.34.5`
- **IP da Máquina:** `192.168.0.109` (IP atual na rede local – pode variar)
- **Porta exposta:** `8080`

## 3. Passo a Passo da Implantação

### 3.1 Instalação do Docker e Ativação da Virtualização

- Instale o Docker Desktop (baixando do site oficial e seguindo o instalador).
- Ative a virtualização na BIOS/UEFI:
  - **Virtualization Technology (VTx)**
  - **Virtualization Technology for Directed I/O (VTd)**

Após ativar, o Docker Engine passou a funcionar corretamente.

### 3.2 Execução do Container Baserow

O container foi executado com o comando:

```powershell
docker run -d --name baserow-desafio `
  -e BASEROW_PUBLIC_URL=http://192.168.0.109:8080 `
  -v baserow_data:/baserow/data `
  -p 8080:80 -p 8443:443 `
  --restart unless-stopped `
  baserow/baserow:1.34.5
```

- `-v baserow_data:/baserow/data` cria um volume persistente para manter os dados.
- `-e BASEROW_PUBLIC_URL=...` define a URL base usada pelo frontend e pela API.

### 3.3 Configuração de Firewall e Acesso na Rede

Para permitir que outros computadores na mesma rede acessem o Baserow, foi adicionada uma regra no Firewall do Windows:

```powershell
netsh advfirewall firewall add rule name="Baserow 8080" dir=in action=allow protocol=TCP localport=8080
```

## 4. Modelagem do Banco de Dados – Clínica Médica

Foram criadas três tabelas dentro da aplicação `Clinica_Medica`:

### 4.1 Tabela Pacientes (ID: 711)

| Campo | Tipo | Descrição |
|---|---|---|
| Name | Texto | Nome do paciente |
| Date | Data | Data de nascimento |
| Number | Texto | Telefone |
| Completed | Booleano | Ativo / Inativo |
| Email | Texto | E-mail |

### 4.2 Tabela Médicos (ID: 713)

| Campo | Tipo | Descrição |
|---|---|---|
| Nome | Texto | Nome completo do médico |
| CRM | Texto | Registro profissional |
| Especialidade | Texto | Especialidade |

### 4.3 Tabela Consultas (ID: 712)

| Campo | Tipo | Descrição |
|---|---|---|
| DataConsulta | Data | Data da consulta |
| HoraConsulta | Texto | Horário (HH:MM) |
| Status | Single Select | Agendada / Realizada / Cancelada |
| Notes | Long Text | Observações |
| Paciente | Link to Table | Referência para a tabela Pacientes |
| Medico | Link to Table | Referência para a tabela Médicos |

### 4.4 Relacionamentos

- `Consultas.Paciente` → `Pacientes.ID` (muitos-para-um)
- `Consultas.Medico` → `Médicos.ID` (muitos-para-um)

## 5. Registros de Teste

Foram inseridos 11 registros, superando o mínimo de 10:

- **Pacientes:** 5 registros (IDs 6 a 10)
- **Médicos:** 3 registros (IDs 3, 4, 5)
- **Consultas:** 3 registros (IDs 4, 5, 6)

Os dados podem ser visualizados via API ou diretamente na interface do Baserow.

## 6. Autenticação e Tokens

### 6.1 Token de Database (para CRUD)

> **Atenção:** não inclua tokens reais em repositórios públicos. Use um valor de exemplo/placeholder.

- `DATABASE_TOKEN`: `<<SEU_DATABASE_TOKEN_AQUI>>`

Esse token deve ser enviado no header `Authorization`:

```text
Authorization: Token <<SEU_DATABASE_TOKEN_AQUI>>
Content-Type: application/json
```

### 6.2 JWT Token (para operações administrativas)

Obtido via autenticação com e-mail e senha.

Para gerar, use:

```powershell
curl.exe -X POST -H "Content-Type: application/json" `
  -d "{\"email\":\"<<SEU_EMAIL>>\",\"password\":\"<<SUA_SENHA>>\"}" `
  http://192.168.0.109:8080/api/user/token-auth/
```

## 7. Endpoints da API

Base: `http://192.168.0.109:8080/api`

| Operação | Método | Endpoint |
|---|---|---|
| Listar pacientes | GET | `/database/rows/table/711/?user_field_names=true` |
| Criar paciente | POST | `/database/rows/table/711/?user_field_names=true` |
| Listar médicos | GET | `/database/rows/table/713/?user_field_names=true` |
| Criar médico | POST | `/database/rows/table/713/?user_field_names=true` |
| Listar consultas | GET | `/database/rows/table/712/?user_field_names=true` |
| Criar consulta | POST | `/database/rows/table/712/?user_field_names=true` |
| Atualizar registro | PATCH | `/database/rows/table/{table_id}/{row_id}/?user_field_names=true` |
| Deletar registro | DELETE | `/database/rows/table/{table_id}/{row_id}/?user_field_names=true` |

Todas as requisições devem incluir:

```text
Authorization: Token <<SEU_DATABASE_TOKEN_AQUI>>
Content-Type: application/json
```

## 8. Exemplos de Requisições

### 8.1 Listar todos os médicos

```powershell
curl.exe -X GET -H "Authorization: Token <<SEU_DATABASE_TOKEN_AQUI>>" `
  "http://192.168.0.109:8080/api/database/rows/table/713/?user_field_names=true"
```

### 8.2 Criar uma nova consulta

```powershell
curl.exe -X POST `
  -H "Authorization: Token <<SEU_DATABASE_TOKEN_AQUI>>" `
  -H "Content-Type: application/json" `
  -d "{\"DataConsulta\":\"2024-03-25\",\"HoraConsulta\":\"14:30\",\"Status\":\"Agendada\",\"Notes\":\"Consulta de rotina\",\"Paciente\":[6],\"Medico\":[3]}" `
  "http://192.168.0.109:8080/api/database/rows/table/712/?user_field_names=true"
```

### 8.3 Atualizar o telefone de um paciente (campo `Number`)

```powershell
curl.exe -X PATCH `
  -H "Authorization: Token <<SEU_DATABASE_TOKEN_AQUI>>" `
  -H "Content-Type: application/json" `
  -d "{\"Number\":\"99999\"}" `
  "http://192.168.0.109:8080/api/database/rows/table/711/6/?user_field_names=true"
```

### 8.4 Deletar um médico

```powershell
curl.exe -X DELETE `
  -H "Authorization: Token <<SEU_DATABASE_TOKEN_AQUI>>" `
  "http://192.168.0.109:8080/api/database/rows/table/713/6/?user_field_names=true"
```

## 9. Acessando a Interface Web

- Acesso local: `http://localhost:8080` ou `http://192.168.0.109:8080`
- Acesso de outros computadores na mesma rede: `http://192.168.0.109:8080`

Após acessar, faça login com as credenciais cadastradas (e-mail e senha).

## 10. Solução de Problemas Comuns

| Problema | Provável Causa e Solução |
|---|---|
| `ERR_CONNECTION_REFUSED` | Docker não iniciado ou IP mudou. Verifique `docker ps` e `ipconfig`. |
| Site not found após login | `BASEROW_PUBLIC_URL` incorreta. Recrie o container com o IP atual. |
| Colegas não conseguem acessar | Firewall bloqueando. Libere a porta `8080` com `netsh advfirewall ...`. |
| Dados sumiram | O volume `baserow_data` foi removido. Se não removeu, os dados estão lá. |
| Erro `401 Unauthorized` na API | Token inválido ou ausente. Use o token correto e o header `Authorization`. |

## 11. Comandos Úteis para Gerenciamento do Container

| Ação | Comando |
|---|---|
| Iniciar container | `docker start baserow-desafio` |
| Parar container | `docker stop baserow-desafio` |
| Ver status | `docker ps` |
| Ver logs em tempo real | `docker logs -f baserow-desafio` |
| Ver últimas linhas dos logs | `docker logs --tail 50 baserow-desafio` |
| Recriar com novo IP | `docker rm baserow-desafio` e rode novamente `docker run ... -e BASEROW_PUBLIC_URL=http://novo_ip:8080 ...` |

## 12. Estado Atual (23/03/2026)

- Container: rodando, saudável.
- IP atual: `192.168.0.109`
- URL pública: `http://192.168.0.109:8080`
- Acesso local: `http://localhost:8080`
- Dados: intactos (todas as tabelas e registros).
- API: funcional, com token de autenticação ativo.
- Firewall: porta `8080` liberada para entrada.

## 13. Considerações Finais

O desafio foi integralmente cumprido, atendendo a todos os critérios de avaliação:

- Deploy funcional na rede local.
- Modelagem correta da base de dados (três tabelas, relacionamentos, tipos adequados).
- Inserção de mais de 10 registros de teste.
- Operações CRUD realizadas e documentadas via API.
- Documentação clara e completa (este documento).

Qualquer dúvida ou necessidade de manutenção futura, basta seguir os procedimentos descritos acima.

**Autor:** Gabriel Santos  
**Data:** 23/03/2026

#   b a s e r o w a b t  
 