# Como Rodar o Premium Gás Localmente (Windows)

Guia passo a passo para executar o sistema **100% local**, sem servidores em nuvem,
em um computador com Windows.

---

## Requisitos

| Componente | O que instalar | Link |
|---|---|---|
| Banco de dados | PostgreSQL 16+ | https://www.enterprisedb.com/downloads/postgresql |
| Java | Temurin 21 LTS (instalador .msi) | https://adoptium.net/ |
| Node.js | Versão LTS (vem com npm) | https://nodejs.org/ |
| API | JAR pronto (`gerenciador-estoque-api-0.0.1-SNAPSHOT.jar`) | repositório `gerenciador_estoque_jar` |
| Site | Pasta `gerenciador_estoque_app` | repositório do app |

---

## 1. Instalar PostgreSQL (banco de dados)

1. Baixe o instalador em https://www.enterprisedb.com/downloads/postgresql (versão 16 ou superior).
2. Instale com as opções padrão.
3. **Na tela de senha do usuário `postgres`, digite `Ra28041996`** (é a senha que a API espera).
4. Abra o **SQL Shell (psql)** ou o **pgAdmin** e crie o banco:

```sql
CREATE DATABASE gerenciador_estoque;
```

> O banco é criado vazio. As tabelas e os **dados iniciais** (Gás P13 e Água Galão 20L)
> são criados automaticamente pela API na primeira subida (Flyway roda as migrations V1-V8).

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

## 3. Instalar Node.js

1. Baixe em https://nodejs.org/ (versão **LTS**, instalador `.msi`) — o npm vem junto.
2. Verifique no **Prompt de Comando**:

```
node -v
npm -v
```

---

## 4. Copiar o JAR da API para o PC

1. Baixe o arquivo `gerenciador-estoque-api-0.0.1-SNAPSHOT.jar` (cerca de 47 MB) do
   repositório **privado** `gerenciador_estoque_jar` (ou copie da pasta `target/` da API
   após um build).
2. Coloque em uma pasta local, por exemplo: `C:\gerenciador-estoque\`.

> **Importante:** não extraia o arquivo. O JAR é executado diretamente
> (ele é um zip internamente — extrair transforma em pastas e quebra a execução).

---

## 5. Rodar a API

No **Prompt de Comando**:

```
cd C:\gerenciador-estoque
java -jar gerenciador-estoque-api-0.0.1-SNAPSHOT.jar
```

- A API sobe na porta **8080**.
- Deixe esta janela aberta enquanto usar o sistema.
- O perfil `local` é o padrão: conecta em `localhost:5432` no banco `gerenciador_estoque`
  com usuário `postgres` / senha `Ra28041996` (já embutida no JAR de apresentação).

---

## 6. Rodar o site (app)

1. Copie a pasta `gerenciador_estoque_app` inteira para o Windows
   (pode excluir `node_modules` e `dist`, se existirem — serão recriados).
2. No **Prompt de Comando**:

```
cd C:\gerenciador-estoque-app
npm install
npm run dev
```

3. Abra o navegador em **http://localhost:5173**

> Em desenvolvimento, o Vite redireciona as chamadas `/api` para
> `http://localhost:8080` (proxy configurado no `vite.config.ts`), então o site
> e a API conversam sem configuração extra. O CORS da API já libera
> `http://localhost:5173` por padrão.

---

## Resumo do fluxo

1. PostgreSQL rodando (porta 5432, banco `gerenciador_estoque` criado)
2. `java -jar gerenciador-estoque-api-0.0.1-SNAPSHOT.jar` (porta 8080)
3. `npm run dev` na pasta do app (porta 5173)
4. Abrir **http://localhost:5173** no navegador

---

## Dica: subir a API com um duplo clique

Crie um arquivo `iniciar-api.bat` na pasta do JAR com o conteúdo:

```bat
@echo off
java -jar gerenciador-estoque-api-0.0.1-SNAPSHOT.jar
pause
```

E um `iniciar-app.bat` na pasta do app:

```bat
@echo off
cd /d %~dp0
npm run dev
pause
```
