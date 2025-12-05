# 📦 Gestão de Estoque Inteligente com Write-back (Power BI + Power Automate)

Este projeto demonstra a implementação de **Write-back** (escrita de dados) diretamente a partir de um dashboard do Power BI. O sistema permite não apenas visualizar o nível de estoque, mas **realizar pedidos de reposição** automaticamente, registrando a solicitação em um banco de dados (Excel Online) e notificando a equipe via **Microsoft Teams** em tempo real.

![Badge Power BI](https://img.shields.io/badge/Power_BI-Desktop-yellow?style=flat&logo=powerbi)
![Badge Power Automate](https://img.shields.io/badge/Power_Automate-Flow-blue?style=flat&logo=powerautomate)
![Badge Excel](https://img.shields.io/badge/Excel-Online-green?style=flat&logo=microsoft-excel)

## 🎯 Objetivo do Projeto

Transformar um dashboard passivo de leitura em uma ferramenta ativa de gestão.
- **Problema:** Analistas identificam falta de estoque no BI, mas precisam abrir outro sistema ou e-mail para pedir reposição.
- **Solução:** Um botão integrado no relatório que envia o pedido instantaneamente com os dados do produto selecionado.

## 🛠️ Arquitetura da Solução

1.  **Banco de Dados:** Excel Online (hospedado no OneDrive/SharePoint).
2.  **Front-end:** Power BI Desktop (Visualização e Gatilho).
3.  **Back-end:** Power Automate (Automação de fluxo na nuvem).
4.  **Notificação:** Microsoft Teams (Canal de Logística).

---

## 🚀 Passo a Passo da Implementação

### 1. Preparação dos Dados (O Truque do Caminho do Excel) 💡
A base de dados consiste em um arquivo Excel com duas tabelas oficiais (`Ctrl + T`): `Tabela_Estoque` e `Tabela_Pedidos`.

**Desafio Técnico:** O Power BI Web Connector não aceita links de compartilhamento padrão do Excel Online (aqueles com `:x:/g/`).
**Solução:**
1.  Abrir o arquivo no **Excel Online**.
2.  Clicar em *Edição* > **Abrir na Área de Trabalho (Open in Desktop app)**.
3.  No Excel do PC, ir em **Arquivo > Informações > Copiar Caminho**.
4.  **Limpeza da URL:** Ao colar no Power BI, é necessário remover o parâmetro `?web=1` do final do link para garantir o acesso direto ao arquivo físico na nuvem.

### 2. Conexão no Power BI
- Utilizado o conector **Web** (não Excel) para garantir atualização automática na nuvem.
- Autenticação via **Conta Organizacional** (OAuth2).
- Transformação de dados no Power Query (tipagem de colunas para Data e Número Inteiro).

### 3. Integração com Power Automate
Foi utilizado o visual nativo **Power Automate for Power BI**.

**Configuração do Gatilho:**
Para que o fluxo funcione, é necessário arrastar os campos de dados (`ID_Produto`, `Nome`, `Estoque_Minimo`, `User Email`) para a área de "Dados" do visual **antes** de criar o fluxo. Isso define a "Entidade" que o Power BI envia para o robô.

### 4. O Fluxo de Automação (Logic App)
O fluxo segue a seguinte lógica:
1.  **Gatilho:** Botão do Power BI clicado.
2.  **Ação (Excel Business):** "Adicionar uma linha em uma tabela".
    - Mapeamento dinâmico: O campo `Produto` no Excel recebe o valor `Power BI Data - Nome`.
    - **ID Único:** Utilizada a expressão `guid()` para gerar um ID de pedido único e evitar conflitos de chave primária.
    - **Timestamp:** Utilizada a expressão `utcNow()` para registrar a data/hora exata do clique.
3.  **Ação (Teams):** "Postar mensagem em um chat ou canal".
    - Envia um cartão com os detalhes do pedido para o Canal da Equipe de Logística.

---

## 📸 Como Utilizar

1.  No dashboard, selecione um produto na tabela de estoque (ex: "Teclado Mecânico").
2.  O botão "Solicitar Compra" será habilitado.
3.  **No Power BI Desktop:** Segure `Ctrl` e clique no botão.
4.  **No Power BI Service (Online):** Apenas clique no botão.
5.  O sistema processará o pedido, adicionará a linha no Excel e enviará o alerta no Teams.

## ⚠️ Aprendizados e Troubleshooting

* **Erro "Entity is missing":** Acontece ao tentar testar o fluxo diretamente pelo editor do Power Automate sem disparar pelo botão do relatório. A correção é sempre testar clicando no botão do dashboard.
* **Ambientes do Power Automate:** O fluxo criado via Power BI muitas vezes é salvo no ambiente "Personal Productivity" ou "Default". É necessário verificar o ambiente correto no portal `make.powerautomate.com` para edições avançadas.

---
