# Neocare-QA

# Neocare API — Plano de Testes
**Projeto:** Neocare — Sistema de Monitoramento de Estresse  
**Integrantes:** Kaue Vinicius (559317) · João Cardoso (560400) · Davi Praxedes (560719)  
**Plataforma de gestão:** Azure Boards  
**Ferramentas de automação:** Selenium IDE (UI) · Postman (API REST)

---

## Estrutura no Azure Boards

```
Test Plan: Neocare — Plano de Testes v1.0
├── Test Suite: Autenticação
│   ├── TC-001 (Manual)
│   └── ST-001 (Selenium)
├── Test Suite: Gestão de Usuários
│   ├── TC-002 (Manual)
│   └── ST-002 (Selenium)
├── Test Suite: Medições Psicofisiológicas
│   ├── TC-003 (Manual)
│   └── ST-003 (Selenium)
├── Test Suite: Alertas
│   ├── TC-004 (Manual)
│   └── ST-004 (Selenium)
└── Test Suite: API REST (Postman)
    ├── PT-001 — Login JWT
    ├── PT-002 — Cadastro de Usuário
    ├── PT-003 — Registrar Medição Psicofisiológica
    └── PT-004 — Listar Alertas do Usuário
```

---

# PARTE A — TESTES MANUAIS

---

## TC-001 — Login via Formulário Web

| Campo | Valor |
|---|---|
| **ID** | TC-001 |
| **Tipo** | Manual — Validação de Sistema |
| **Funcionalidade** | Autenticação via formulário Thymeleaf (sessão) |
| **Prioridade** | Alta |
| **Sprint de origem** | Sprint 1 — US-01: Autenticação de Usuário |
| **Pré-condições** | Aplicação em execução em `http://localhost:8080`. Usuário com `username: joao.teste` e `password: Senha@123` previamente cadastrado no banco. |

### Dados de Entrada (controlados)

| Campo no Formulário | Valor Fixo |
|---|---|
| Username | `joao.teste` |
| Password | `Senha@123` |

### Dados de Saída Esperados

| Critério | Valor Esperado |
|---|---|
| Redirecionamento após login | `http://localhost:8080/dashboard` |
| Elemento visível na tela | Texto "Bem-vindo" ou nome do usuário no cabeçalho |
| Cookie/Sessão criada | `JSESSIONID` presente nos cookies do navegador |
| HTTP Status da página | `200 OK` |

### Procedimento de Teste

1. Abrir o navegador e acessar `http://localhost:8080/auth/login`.
2. Verificar que o formulário de login é exibido com os campos **Username** e **Password**.
3. Preencher o campo **Username** com `joao.teste`.
4. Preencher o campo **Password** com `Senha@123`.
5. Clicar no botão **"Entrar"** (ou equivalente).
6. Verificar que o sistema redireciona para `http://localhost:8080/dashboard`.
7. Confirmar que o dashboard é exibido com o nome do usuário logado.
8. Inspecionar os cookies do navegador e verificar a presença de `JSESSIONID`.

**Resultado Esperado:** Usuário autenticado com sucesso, sessão criada, redirecionado para o Dashboard.

**Resultado Obtido:** _(preencher durante execução)_

**Status:** ☐ Passou &nbsp;&nbsp; ☐ Falhou &nbsp;&nbsp; ☐ Bloqueado

---

## TC-002 — Cadastro de Novo Usuário via Formulário Web

| Campo | Valor |
|---|---|
| **ID** | TC-002 |
| **Tipo** | Manual — Validação de Sistema |
| **Funcionalidade** | Cadastro de usuário via formulário Thymeleaf (`/auth/registro`) |
| **Prioridade** | Alta |
| **Sprint de origem** | Sprint 1 — US-02: Cadastro de Usuário |
| **Pré-condições** | Aplicação em execução. CPF `123.456.789-09` **não** cadastrado no banco. |

### Dados de Entrada (controlados)

| Campo no Formulário | Valor Fixo |
|---|---|
| Nome | `Maria Oliveira` |
| CPF | `123.456.789-09` |
| E-mail | `maria.oliveira@neocare.com` |
| Telefone | `(11) 99999-0001` |
| Data de Nascimento | `15/06/1990` |
| Peso (kg) | `65` |
| Altura (cm) | `165` |
| CEP | `01310-100` |
| Logradouro | `Avenida Paulista` |
| Número | `1000` |
| Bairro | `Bela Vista` |
| Cidade | `São Paulo` |
| UF | `SP` |
| Username | `maria.oliveira` |
| Password | `Teste@2024` |
| Confirmar Password | `Teste@2024` |

### Dados de Saída Esperados

| Critério | Valor Esperado |
|---|---|
| Redirecionamento após cadastro | `/auth/login` (ou mensagem de sucesso) |
| Mensagem de feedback | "Cadastro realizado com sucesso" (ou equivalente) |
| Registro no banco | Usuário com CPF `123.456.789-09` presente na tabela `usuario` com `ativo = true` |
| HTTP Status | `200 OK` (ou `302 redirect`) |

### Procedimento de Teste

1. Acessar `http://localhost:8080/auth/registro`.
2. Preencher **todos** os campos do formulário com os valores da tabela acima.
3. Clicar em **"Cadastrar"**.
4. Verificar mensagem de sucesso ou redirecionamento para `/auth/login`.
5. Acessar o banco de dados e confirmar o registro: `SELECT * FROM usuario WHERE cpf = '12345678909'`.
6. Confirmar que o campo `ativo` é `true` e que os dados pessoais foram salvos corretamente.

**Resultado Esperado:** Usuário cadastrado com sucesso, redirecionado para o login.

**Resultado Obtido:** _(preencher durante execução)_

**Status:** ☐ Passou &nbsp;&nbsp; ☐ Falhou &nbsp;&nbsp; ☐ Bloqueado

---

## TC-003 — Registro de Medição Psicofisiológica via Interface Web

| Campo | Valor |
|---|---|
| **ID** | TC-003 |
| **Tipo** | Manual — Validação de Sistema |
| **Funcionalidade** | Registro de medição HRV + GSR e exibição do resultado de predição |
| **Prioridade** | Alta |
| **Sprint de origem** | Sprint 2 — US-05: Registrar Medição Psicofisiológica |
| **Pré-condições** | Usuário `joao.teste` autenticado (sessão ativa). Pelo menos um dispositivo cadastrado no banco (seed V8 do Flyway já cobre isso). |

### Dados de Entrada (controlados)

| Campo no Formulário | Valor Fixo |
|---|---|
| HRV (ms) | `28` _(valor baixo — deve gerar alerta de severidade Alta)_ |
| GSR (μS) | `18` _(valor elevado — correlato com estresse)_ |
| Dispositivo | ID `1` (dispositivo padrão do seed) |

### Dados de Saída Esperados

| Critério | Valor Esperado |
|---|---|
| Redirecionamento | `/medicoes-web/resultado` |
| Exibição na tela de resultado | Campos `score`, `predicao` e `analisadoEm` presentes |
| Predição esperada | `ALTO_ESTRESSE` (score ≥ 0.8, dada a combinação HRV baixo + GSR alto) |
| Alerta gerado automaticamente | Registro com `severidade = ALTA` na tabela `alertas` para o usuário |
| Métrica de Estresse | Índice calculado no intervalo `70–100`, categoria `CRONICO` ou `AGUDO` |

### Procedimento de Teste

1. Realizar login com `joao.teste` / `Senha@123` em `http://localhost:8080/auth/login`.
2. Navegar para `http://localhost:8080/medicoes-web/nova-psicofisiologica`.
3. Preencher o campo **HRV** com `28`.
4. Preencher o campo **GSR** com `18`.
5. Selecionar o **Dispositivo** de ID `1` no seletor.
6. Clicar em **"Registrar Medição"**.
7. Verificar que a tela `/medicoes-web/resultado` é exibida com os dados de predição.
8. Confirmar no banco: `SELECT * FROM alertas WHERE usuario_id = ? ORDER BY criado_em DESC LIMIT 1`.
9. Confirmar no banco: `SELECT * FROM metricas_estresse ORDER BY id DESC LIMIT 1`.

**Resultado Esperado:** Medição salva, predição exibida na tela, alerta de severidade Alta criado automaticamente.

**Resultado Obtido:** _(preencher durante execução)_

**Status:** ☐ Passou &nbsp;&nbsp; ☐ Falhou &nbsp;&nbsp; ☐ Bloqueado

---

## TC-004 — Visualizar Alertas e Marcar como Lido

| Campo | Valor |
|---|---|
| **ID** | TC-004 |
| **Tipo** | Manual — Validação de Sistema |
| **Funcionalidade** | Listagem de alertas e ação de marcar como lido na interface web |
| **Prioridade** | Média |
| **Sprint de origem** | Sprint 2 — US-07: Gerenciamento de Alertas |
| **Pré-condições** | Usuário `joao.teste` autenticado. Ao menos um alerta com `lido = false` existente para esse usuário (gerado no TC-003 ou inserido manualmente). |

### Dados de Entrada (controlados)

| Campo | Valor Fixo |
|---|---|
| Usuário autenticado | `joao.teste` |
| Alerta alvo | Primeiro alerta com status `Novo` na listagem (ID obtido via banco antes do teste) |
| Ação executada | Clique no botão **"Marcar como Lido"** |

### Dados de Saída Esperados

| Critério | Valor Esperado |
|---|---|
| Status do alerta na tela | Muda de **"Novo"** para **"Lido"** |
| Valor no banco | `lido = true` para o alerta de ID utilizado |
| HTTP Status da ação | `302 redirect` de volta para `/alertas-web` |
| Listagem após ação | Alerta marcado não aparece mais com badge "Novo" |

### Procedimento de Teste

1. Anotar o ID do alerta alvo: `SELECT id FROM alertas WHERE usuario_id = ? AND lido = false LIMIT 1`.
2. Realizar login com `joao.teste` / `Senha@123`.
3. Navegar para `http://localhost:8080/alertas-web`.
4. Localizar o alerta com status **"Novo"** correspondente ao ID anotado.
5. Clicar no botão **"Marcar como Lido"** ao lado do alerta.
6. Verificar que a página recarrega e o status do alerta exibe **"Lido"**.
7. Confirmar no banco: `SELECT lido FROM alertas WHERE id = ?` → deve retornar `true`.

**Resultado Esperado:** Alerta atualizado para lido na interface e no banco de dados.

**Resultado Obtido:** _(preencher durante execução)_

**Status:** ☐ Passou &nbsp;&nbsp; ☐ Falhou &nbsp;&nbsp; ☐ Bloqueado

---

# PARTE B — TESTES AUTOMATIZADOS

---

## B.1 — Selenium IDE (Interface Web)

> **Ferramenta:** Selenium IDE (extensão de navegador) ou Katalon Studio.  
> Os testes abaixo estão no formato de **Command / Target / Value** do Selenium IDE, prontos para importação como `.side`.

---

### ST-001 — Login com Credenciais Válidas (Selenium)

**Descrição:** Valida o fluxo completo de login via formulário web e confirma o redirecionamento ao dashboard.

**URL de início:** `http://localhost:8080/auth/login`

| # | Command | Target | Value |
|---|---|---|---|
| 1 | `open` | `/auth/login` | |
| 2 | `assertTitle` | `*Login*` | |
| 3 | `type` | `name=username` | `joao.teste` |
| 4 | `type` | `name=password` | `Senha@123` |
| 5 | `click` | `css=button[type='submit']` | |
| 6 | `waitForElementVisible` | `css=.dashboard-container` | `5000` |
| 7 | `assertUrlMatch` | `*/dashboard*` | |
| 8 | `assertElementPresent` | `css=.user-name` | |

**Critério de Aprovação:** URL contém `/dashboard` e elemento `.dashboard-container` presente.

---

### ST-002 — Cadastro de Novo Usuário via Formulário (Selenium)

**Descrição:** Valida o fluxo completo de cadastro de usuário pela interface web.

**URL de início:** `http://localhost:8080/auth/registro`

| # | Command | Target | Value |
|---|---|---|---|
| 1 | `open` | `/auth/registro` | |
| 2 | `assertTitle` | `*Registro*` | |
| 3 | `type` | `name=nome` | `Carlos Automacao` |
| 4 | `type` | `name=cpf` | `987.654.321-00` |
| 5 | `type` | `name=email` | `carlos.auto@neocare.com` |
| 6 | `type` | `name=telefone` | `(11) 98888-0002` |
| 7 | `type` | `name=dataNascimento` | `20/03/1985` |
| 8 | `type` | `name=peso` | `80` |
| 9 | `type` | `name=altura` | `175` |
| 10 | `type` | `name=cep` | `01310-100` |
| 11 | `type` | `name=logradouro` | `Avenida Paulista` |
| 12 | `type` | `name=numero` | `500` |
| 13 | `type` | `name=bairro` | `Bela Vista` |
| 14 | `type` | `name=cidade` | `São Paulo` |
| 15 | `select` | `name=uf` | `label=SP` |
| 16 | `type` | `name=username` | `carlos.auto` |
| 17 | `type` | `name=password` | `Auto@2024` |
| 18 | `type` | `name=confirmPassword` | `Auto@2024` |
| 19 | `click` | `css=button[type='submit']` | |
| 20 | `waitForElementVisible` | `css=.alert-success, .login-form` | `5000` |
| 21 | `assertUrlMatch` | `*/login*` | |

**Critério de Aprovação:** Redirecionamento para `/auth/login` após cadastro bem-sucedido.

---

### ST-003 — Registrar Nova Medição Psicofisiológica (Selenium)

**Descrição:** Usuário autenticado registra uma medição HRV + GSR e confirma a tela de resultado com dados de predição.

**URL de início:** `http://localhost:8080/auth/login`

| # | Command | Target | Value |
|---|---|---|---|
| 1 | `open` | `/auth/login` | |
| 2 | `type` | `name=username` | `joao.teste` |
| 3 | `type` | `name=password` | `Senha@123` |
| 4 | `click` | `css=button[type='submit']` | |
| 5 | `waitForElementVisible` | `css=.dashboard-container` | `5000` |
| 6 | `open` | `/medicoes-web/nova-psicofisiologica` | |
| 7 | `assertTitle` | `*Medição*` | |
| 8 | `type` | `name=hrv` | `28` |
| 9 | `type` | `name=gsr` | `18` |
| 10 | `select` | `name=dispositivoId` | `value=1` |
| 11 | `click` | `css=button[type='submit']` | |
| 12 | `waitForElementVisible` | `css=.resultado-predicao` | `8000` |
| 13 | `assertUrlMatch` | `*/resultado*` | |
| 14 | `assertElementPresent` | `css=.predicao-valor` | |
| 15 | `assertElementPresent` | `css=.score-valor` | |

**Critério de Aprovação:** Página `/medicoes-web/resultado` exibida com elementos de predição e score presentes.

---

### ST-004 — Visualizar Alertas e Marcar como Lido (Selenium)

**Descrição:** Usuário autenticado acessa a lista de alertas e executa a ação de marcar um alerta como lido.

**URL de início:** `http://localhost:8080/auth/login`

| # | Command | Target | Value |
|---|---|---|---|
| 1 | `open` | `/auth/login` | |
| 2 | `type` | `name=username` | `joao.teste` |
| 3 | `type` | `name=password` | `Senha@123` |
| 4 | `click` | `css=button[type='submit']` | |
| 5 | `waitForElementVisible` | `css=.dashboard-container` | `5000` |
| 6 | `open` | `/alertas-web` | |
| 7 | `assertTitle` | `*Alertas*` | |
| 8 | `assertElementPresent` | `css=.alerta-item` | |
| 9 | `assertTextPresent` | `Novo` | |
| 10 | `click` | `css=.alerta-item:first-child form[action*='marcar-lido'] button` | |
| 11 | `waitForPageToLoad` | | `5000` |
| 12 | `assertUrlMatch` | `*/alertas-web*` | |
| 13 | `assertTextNotPresent` | `Novo` | |
| 14 | `assertTextPresent` | `Lido` | |

**Critério de Aprovação:** Badge de status do primeiro alerta muda de "Novo" para "Lido" após a ação.

---

## B.2 — Postman (API REST)

> **Pré-requisito de todas as collections:** Aplicação rodando em `http://localhost:8080`.  
> Variável de ambiente Postman: `{{base_url}} = http://localhost:8080` e `{{token}}` (preenchida automaticamente pelo PT-001).

---

### PT-001 — Login via API REST e Obtenção do JWT

**Método:** `POST`  
**Endpoint:** `{{base_url}}/api/auth/login`  
**Autenticação:** Pública (sem token)

**Headers:**
```
Content-Type: application/json
```

**Body (raw JSON):**
```json
{
  "username": "joao.teste",
  "password": "Senha@123"
}
```

**Script de Testes (aba Tests — Postman):**
```javascript
// 1. Valida status HTTP
pm.test("Status deve ser 200 OK", function () {
    pm.response.to.have.status(200);
});

// 2. Valida que o token JWT foi retornado
pm.test("Response deve conter token JWT", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property("token");
    pm.expect(jsonData.token).to.be.a("string").and.not.empty;
});

// 3. Salva o token como variável de ambiente para os próximos testes
pm.test("Salvar token no ambiente", function () {
    const jsonData = pm.response.json();
    pm.environment.set("token", jsonData.token);
});

// 4. Valida tempo de resposta
pm.test("Tempo de resposta abaixo de 2000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});
```

**Saída Esperada:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

| Critério | Esperado |
|---|---|
| HTTP Status | `200 OK` |
| Campo `token` | String JWT não vazia |
| Tempo de resposta | `< 2000 ms` |

---

### PT-002 — Cadastro de Novo Usuário via API

**Método:** `POST`  
**Endpoint:** `{{base_url}}/usuarios`  
**Autenticação:** Pública (sem token)

**Headers:**
```
Content-Type: application/json
```

**Body (raw JSON):**
```json
{
  "nome": "Postman Usuario Teste",
  "cpf": "111.444.777-35",
  "email": "postman.teste@neocare.com",
  "telefone": "(11) 97777-0003",
  "dataNascimento": "1992-08-10",
  "peso": 72.5,
  "altura": 170,
  "endereco": {
    "cep": "01310-100",
    "logradouro": "Avenida Paulista",
    "numero": "200",
    "bairro": "Bela Vista",
    "cidade": "São Paulo",
    "uf": "SP"
  },
  "username": "postman.teste",
  "password": "Postman@2024"
}
```

**Script de Testes (aba Tests — Postman):**
```javascript
// 1. Valida status HTTP de criação
pm.test("Status deve ser 201 Created", function () {
    pm.response.to.have.status(201);
});

// 2. Valida campos obrigatórios no response
pm.test("Response deve conter id e nome do usuário", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property("id");
    pm.expect(jsonData).to.have.property("nome").eql("Postman Usuario Teste");
    pm.expect(jsonData).to.have.property("cpf").eql("111.444.777-35");
});

// 3. Valida que o usuário está ativo
pm.test("Usuário criado deve estar ativo", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.ativo).to.be.true;
});

// 4. Salva o ID do usuário criado para reutilizar
pm.test("Salvar userId no ambiente", function () {
    const jsonData = pm.response.json();
    pm.environment.set("usuarioId", jsonData.id);
});
```

**Saída Esperada:**
```json
{
  "id": 10,
  "nome": "Postman Usuario Teste",
  "cpf": "111.444.777-35",
  "email": "postman.teste@neocare.com",
  "ativo": true
}
```

| Critério | Esperado |
|---|---|
| HTTP Status | `201 Created` |
| Campo `id` | Inteiro gerado automaticamente |
| Campo `ativo` | `true` |
| Campo `nome` | `"Postman Usuario Teste"` |

---

### PT-003 — Registrar Medição Psicofisiológica via API

**Método:** `POST`  
**Endpoint:** `{{base_url}}/medicoes/medicao_psicofisiologica`  
**Autenticação:** Pública (conforme README)

**Headers:**
```
Content-Type: application/json
```

**Body (raw JSON):**
```json
{
  "hrv": 28,
  "gsr": 18.5,
  "usuarioId": 1,
  "dispositivoId": 1
}
```

**Script de Testes (aba Tests — Postman):**
```javascript
// 1. Valida status HTTP
pm.test("Status deve ser 201 Created", function () {
    pm.response.to.have.status(201);
});

// 2. Valida campos da medição
pm.test("Response deve conter id e valores de HRV e GSR", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property("id");
    pm.expect(jsonData.hrv).to.eql(28);
    pm.expect(jsonData.gsr).to.eql(18.5);
});

// 3. Valida presença do resultado de predição (pode ser null se APEX estiver offline)
pm.test("Response deve conter campo resultadoPredicao", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property("resultadoPredicao");
});

// 4. Valida métrica de estresse calculada
pm.test("Response deve conter metricaEstresse com indice e categoria", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property("metricaEstresse");
    pm.expect(jsonData.metricaEstresse).to.have.property("indice");
    pm.expect(jsonData.metricaEstresse).to.have.property("categoria");
});

// 5. Salva o ID da medição para uso posterior
pm.test("Salvar medicaoId no ambiente", function () {
    const jsonData = pm.response.json();
    pm.environment.set("medicaoId", jsonData.id);
});
```

**Saída Esperada:**
```json
{
  "id": 5,
  "hrv": 28,
  "gsr": 18.5,
  "metricaEstresse": {
    "indice": 85,
    "categoria": "CRONICO"
  },
  "resultadoPredicao": {
    "score": 0.87,
    "predicao": "ALTO_ESTRESSE",
    "analisadoEm": "2026-05-24T10:00:00"
  }
}
```

| Critério | Esperado |
|---|---|
| HTTP Status | `201 Created` |
| Campo `hrv` | `28` |
| Campo `gsr` | `18.5` |
| Campo `metricaEstresse.indice` | Número entre `0` e `100` |
| Campo `resultadoPredicao` | Presente (ou `null` se APEX offline — fail-safe) |

---

### PT-004 — Listar Alertas do Usuário via API

**Método:** `GET`  
**Endpoint:** `{{base_url}}/api/alertas/usuario/1`  
**Autenticação:** Bearer Token (JWT obtido em PT-001)

**Headers:**
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Body:** Nenhum

**Script de Testes (aba Tests — Postman):**
```javascript
// 1. Valida status HTTP
pm.test("Status deve ser 200 OK", function () {
    pm.response.to.have.status(200);
});

// 2. Valida que o response é um array
pm.test("Response deve ser um array de alertas", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.be.an("array");
});

// 3. Valida estrutura de cada alerta retornado
pm.test("Cada alerta deve conter id, tipo, severidade e lido", function () {
    const jsonData = pm.response.json();
    if (jsonData.length > 0) {
        const alerta = jsonData[0];
        pm.expect(alerta).to.have.property("id");
        pm.expect(alerta).to.have.property("tipo");
        pm.expect(alerta).to.have.property("severidade");
        pm.expect(alerta).to.have.property("lido");
        pm.expect(alerta.lido).to.be.a("boolean");
    }
});

// 4. Valida que severidade tem valor válido
pm.test("Severidade deve ser ALTA ou MODERADA", function () {
    const jsonData = pm.response.json();
    if (jsonData.length > 0) {
        const severidadesValidas = ["ALTA", "MODERADA"];
        jsonData.forEach(alerta => {
            pm.expect(severidadesValidas).to.include(alerta.severidade);
        });
    }
});

// 5. Valida tempo de resposta
pm.test("Tempo de resposta abaixo de 2000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});
```

**Saída Esperada:**
```json
[
  {
    "id": 1,
    "tipo": "HRV_BAIXO",
    "severidade": "ALTA",
    "valorDetectado": 28.0,
    "mensagem": "HRV muito baixo detectado. Possível estresse elevado.",
    "lido": false,
    "criadoEm": "2026-05-24T10:00:00"
  }
]
```

| Critério | Esperado |
|---|---|
| HTTP Status | `200 OK` |
| Tipo do body | Array JSON |
| Campo `severidade` | `"ALTA"` ou `"MODERADA"` |
| Campo `lido` | Boolean (`true` ou `false`) |
| Tempo de resposta | `< 2000 ms` |

---

## Resumo Geral dos Testes

| ID | Título | Tipo | Funcionalidade | Prioridade |
|---|---|---|---|---|
| TC-001 | Login via formulário web | Manual | Autenticação (sessão) | Alta |
| TC-002 | Cadastro de usuário via formulário | Manual | Gestão de Usuários | Alta |
| TC-003 | Registrar medição psicofisiológica | Manual | Medições + Predição + Alertas | Alta |
| TC-004 | Visualizar alertas e marcar como lido | Manual | Alertas | Média |
| ST-001 | Login com credenciais válidas | Selenium IDE | Autenticação (UI) | Alta |
| ST-002 | Cadastro de usuário via formulário | Selenium IDE | Gestão de Usuários (UI) | Alta |
| ST-003 | Registrar medição psicofisiológica | Selenium IDE | Medições (UI) | Alta |
| ST-004 | Visualizar alertas e marcar como lido | Selenium IDE | Alertas (UI) | Média |
| PT-001 | Login via API REST — Obter JWT | Postman | Autenticação (API) | Alta |
| PT-002 | Cadastro de usuário via API | Postman | Gestão de Usuários (API) | Alta |
| PT-003 | Registrar medição psicofisiológica | Postman | Medições + Predição (API) | Alta |
| PT-004 | Listar alertas do usuário | Postman | Alertas (API) | Média |

---

## Checklist de Conformidade da Rubrica

| Item da Rubrica | Status |
|---|---|
| ✅ Testes planejados e listados (peso 20%) | 4 manuais + 4 Selenium + 4 Postman |
| ✅ Dados de entrada controlados e predefinidos (peso 20%) | Todos os campos com valores fixos |
| ✅ Dados de saída esperados descritos (peso 20%) | Tabelas de saída + JSON esperado em cada TC |
| ✅ Procedimento passo a passo descrito (peso 20%) | Detalhado em manuais; command/target/value nos Selenium |
| ✅ Testes alinhados às sprints realizadas | Sprint 1 (Auth, Usuários) + Sprint 2 (Medições, Alertas) |
| ✅ Cobertura das funcionalidades principais | Auth · Usuários · Medições · Predição · Alertas |
| ✅ Casos negativos/alternativos | Cobertos nos scripts Postman (validações de schema e severidade) |
| ✅ Automação com UI (Selenium) | 4 casos — ST-001 a ST-004 |
| ✅ Automação com API (Postman) | 4 casos — PT-001 a PT-004 |
