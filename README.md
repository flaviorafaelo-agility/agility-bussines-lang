# Bussines Lang

**Bussines Lang** é uma linguagem de programação declarativa e semântica desenvolvida pela Agility Soluções, projetada para agilidade no desenvolvimento de sistemas empresariais com forte integração nativa a IA, CRUD implícito e performance de execução via máquina virtual.

## 🌟 Objetivos
- Reduzir a verbosidade do código
- Declarar regras e estrutura com semântica empresarial
- Suportar geração de APIs REST, documentação e interfaces automaticamente
- Oferecer integração com VM baseada em bytecode, com execução performática
- Permitir expressividade com recursos como `if`, `for`, `sum`, `get`, `set`, entre outros

---

## 📁 Estrutura de Arquivos e Diretórios

| Arquivo                                 | Descrição |
|----------------------------------------|-----------|
| `start.bus`                             | Define quais aplicações devem ser inicializadas pela VM, suas portas e opções de HTTPS. |
| `erp.application.bus`                  | Define os módulos que pertencem a uma aplicação, como `vendas`, `cadastro`, etc. |
| `erp.application.settings`             | Contém configurações específicas da aplicação, como o caminho da base de dados. |
| `vendas.domain.bus`                    | Declara o banco de dados utilizado pelo módulo `vendas`, bem como suas configurações. |
| `cliente.concept.bus`                  | Define a estrutura da entidade `cliente`, com seus campos, validações, tipos e regras. |
| `common.types.bus`                     | Define tipos reutilizáveis como `cpf`, `endereco`, `telefone` com validações e máscaras. |
| `cliente.ativarConta.service.bus`      | Define o serviço `ativarConta` com entrada, saída e a implementação em bloco de código. |
| `clientes.ui.bus`                      | Representa o layout visual (UI) para o concept `cliente` — a ser interpretado pelo motor de renderização de UI. |
| `bussines.bnf`                         | Arquivo com a gramática completa em formato BNF da linguagem Bussines Lang. |

---

## 🛠️ Componentes principais

- **Concepts:** definem entidades com estrutura e comportamento implícito (CRUD, validação, integração).
- **Types:** definem tipos reutilizáveis com semântica, máscaras, regex, etc.
- **Domains:** agrupam concepts e definem configurações de persistência.
- **Applications:** organizam os módulos de domínio.
- **Services:** encapsulam lógica com input/output e bloco de `implementation {}`.
- **UI:** descreve a visualização declarativa de um concept ou serviço.
- **Start:** ponto inicial da aplicação, define as aplicações a serem iniciadas.

---

## 📦 Exemplo de Inicialização

Arquivo `start.bus`:

```bus
applications
  erp
    path sistema.erp.application.bus
    port 8080
    https false

  nfe
    path sistema.nfe.application.bus
    port 8443
    https true
    certPath /certs/nfe.crt
    keyPath /certs/nfe.key
```

---

## 🧠 Futuro

- Geração de bytecode com serialização de lambdas
- Execução performática via `call`
- Debugger integrado para rastreio de execução
- Integração com IA para geração de serviços e recomendações de campos

---

## 🏢 Desenvolvido por

Agility Soluções — Tecnologia com inteligência de negócio.

