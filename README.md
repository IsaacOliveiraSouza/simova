# Estudo: Fluxo entre Xmova e NFS

Este guia é um resumo de como funcionam algumas partes do **Xmova** (app) e do **NFS** (sistema), com foco na criação de tabelas, funcionamento do app, EntryPoints e processamento de dados.

---

## 🗂 Criação de Tabelas

As tabelas **não são criadas diretamente no banco** com `CREATE TABLE`.  
O processo é feito por registros em tabelas de controle:

- `nfs_ds_tabela` → registra a nova tabela.  
- `nfs_ds_tabela_campo` → registra os campos da tabela.  

👉 **Depois**: no painel de administração do sistema, rode um **check full**.  
Isso fará o sistema criar fisicamente as tabelas no banco.

📌 Extras:
- Ao criar uma tabela nova, o sistema gera automaticamente uma tela de **CRUD**, que aparece na aba **"Outros"**.  
- Essa tela precisa ser vinculada a um grupo (na tabela `ds_grupo_permissão`) e configurada no **SIUDT** para dar as permissões.

---

## 📱 Estrutura do Xmova

O Xmova usa sua própria linguagem.  
📄 Documentação: [Xmova Android Docs](https://xmova-server-h1.simova.ws/doc/app/xmovaandroid/doc.html)

Ele trabalha com **3 arquivos principais**:

1. **APP** → configurações específicas do aplicativo.  
2. **lang** → define textos, labels e nomes de campos.  
3. **model** → onde acontece a "mágica":
   - Define telas do app.  
   - Instancia **entidades** e seus atributos.  
   - Cada atributo gera automaticamente um **input** no app.  
   - Entidades podem ser instanciadas dentro de outras → isso abre uma lista de objetos para escolher.  

### 🔄 Sincronização das Entidades
- **Sync In** → recebe dados *antes da sincronização*, normalmente puxando registros do banco.  
- **Sync Out** → envia dados do app para o NFS, usando um **EntryPoint** (referenciado pelo *server name*).

---

## 🛠 EntryPoints

Os **EntryPoints** são como "funções" do sistema.  
Eles ficam registrados na tabela `nfs_entryPoint`.

- São executados quando chamados pelo app.  
- Podem ser escritos em **PHP** ou **SQL**.  
- Servem para:
  - Buscar dados.  
  - Processar dados (apesar de existir outra forma de processar também).  

---

## ⚙️ Processamento de Dados

Quando o Xmova Mobile envia dados, existe uma forma **padrão** de processar sem escrever código.

Isso é feito na tabela `par_parametros_mobile`, onde se define **quais campos do mobile vão para quais campos do banco**.

- O campo **LIST_TYPE** = *server name*.  
- Em cada registro você diz:  
  - App (installcode X)  
  - Campo X do mobile → Campo X da tabela do sistema  

---

## 🎯 Resumindo

1. Criar tabela → registrar em `nfs_ds_tabela` e `nfs_ds_tabela_campo` → rodar **check full** → CRUD gerado automaticamente.  
2. Xmova → usa **APP, lang e model** para configurar telas e entidades.  
3. Entidades → podem ser **sync in** (puxa dados) ou **sync out** (manda dados).  
4. EntryPoints → blocos de código (PHP/SQL) registrados em `nfs_entryPoint`.  
5. Processamento → feito via `par_parametros_mobile`, mapeando campos do app ↔ banco.  

---
