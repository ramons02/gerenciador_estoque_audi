# Como Rodar o Premium Gás Localmente (Windows)

Guia passo a passo para executar o sistema **100% local**, sem servidores em nuvem,
em um computador com Windows.

**Simplicidade:** o JAR já traz a **API + o site embutidos** em um único arquivo.
Não precisa de Node.js, npm nem terminal. É só duplo clique.

---

## Requisitos (instalação única)

| Componente | O que instalar | Link |
|---|---|---|
| Banco de dados | PostgreSQL 16+ | https://www.enterprisedb.com/downloads/postgresql |
| Java | Temurin 21 LTS (instalador .msi) | https://adoptium.net/ |

Não é necessário Node.js — o site vem dentro do JAR.

---

## 1. Instalar PostgreSQL (banco de dados)

1. Baixe o instalador em https://www.enterprisedb.com/downloads/postgresql (versão 16 ou superior).
2. Instale com as opções padrão (defina uma **senha para o usuário `postgres`** e guarde-a).
3. Abra o **SQL Shell (psql)** ou o **pgAdmin** e crie o banco e o usuário do sistema:

```sql
CREATE DATABASE gerenciador_estoque;
CREATE ROLE premium_gas LOGIN PASSWORD 'SUA_SENHA_FORTE';
GRANT ALL PRIVILEGES ON DATABASE gerenciador_estoque TO premium_gas;
```

> Troque `SUA_SENHA_FORTE` por uma senha sua (só letras e números, sem acentos).
> Ela será usada no arquivo `premium-gas.env` do passo 3.

> O banco é criado vazio. As tabelas e os **dados iniciais** (Gás P13 e Água Galão 20L)
> são criados automaticamente na primeira execução (Flyway roda as migrations V1-V9).

---

## 2. Instalar Java 21

1. Baixe o instalador em https://adoptium.net/ (Temurin 21 LTS, Windows x64, `.msi`).
2. Instale com as opções padrão.
3. Verifique no **Prompt de Comando**:

```
java -version
```

Deve exibir uma versão `21.x`.

---

## 3. Copiar os arquivos do sistema para o PC

1. Baixe do repositório `gerenciador_estoque_jar` (privado) os **3 arquivos**:
   - `gerenciador-estoque-api-0.0.1-SNAPSHOT.jar` (cerca de 47 MB)
   - `iniciar-premium-gas.bat`
   - `premium-gas.env.exemplo`
2. Coloque **os três na mesma pasta**, por exemplo: `C:\PremiumGas\`.
3. Renomeie `premium-gas.env.exemplo` para `premium-gas.env` e **edite** (Bloco de
   Notas) colocando a senha do passo 1:

```
SPRING_DATASOURCE_USERNAME=premium_gas
SPRING_DATASOURCE_PASSWORD=SUA_SENHA_FORTE
```

> **Importante:** não extraia o JAR. Ele é executado diretamente
> (internamente é um zip — extrair transforma em pastas e quebra a execução).
> E nunca envie o `premium-gas.env` para ninguém.

---

## 4. Iniciar (duplo clique)

1. Dê **duplo clique no `iniciar-premium-gas.bat`**.
2. A janela do servidor abre **minimizada**.
3. Quando tudo estiver pronto, o **navegador abre sozinho** no sistema
   (`http://localhost:8080`).

Para fechar: feche a janela do servidor (ou pressione Ctrl+C nela).

> Alternativa manual: `java -jar gerenciador-estoque-api-0.0.1-SNAPSHOT.jar`
> e abrir `http://localhost:8080` no navegador.

---

## Resumo do fluxo

1. PostgreSQL rodando (porta 5432, banco `gerenciador_estoque` e usuário `premium_gas` criados)
2. `premium-gas.env` preenchido na mesma pasta do JAR
3. Duplo clique no `iniciar-premium-gas.bat` (API + site na porta 8080)
4. Navegador abre sozinho em `http://localhost:8080`

---

## Atualização do sistema

Quando sair versão nova do JAR: baixe o JAR novo do repositório e **substitua o arquivo**
na pasta `C:\PremiumGas\` (o `.bat` e o `premium-gas.env` não mudam). Próximo duplo
clique já usa a versão nova.

## Solução de problemas

| Problema | Causa provável | Solução |
|---|---|---|
| Janela fecha na hora | Java não instalado | Verifique `java -version` (passo 2) |
| Janela mostra "Configuracao pendente" | `premium-gas.env` não existe | Renomeie o `.exemplo` e preencha a senha (passo 3) |
| Navegador abre página de erro | Banco não conecta | Confira se o PostgreSQL está rodando, o banco `gerenciador_estoque` existe e as credenciais do `premium-gas.env` estão certas (passo 1) |
| Porta 8080 já em uso | Outro programa usando a porta | Feche o outro programa ou reinicie o PC |