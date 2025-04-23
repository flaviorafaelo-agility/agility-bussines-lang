# Bussines Lang

A **Bussines Lang** é uma linguagem declarativa orientada a negócios, criada para acelerar o desenvolvimento de sistemas empresariais com foco em:
- Baixa verbosidade
- Agilidade na modelagem
- Performance na execução (com suporte a VM e Bytecode)
- Integração nativa com IA
- Arquitetura modular

A linguagem foi projetada pela **Agility Soluções** para suportar conceitos como: tipos reutilizáveis, conceitos de domínio (concepts), serviços, views, aplicações e configuração centralizada.

---

## 📁 Estrutura do projeto

```bash
specification/
├── bussines.bnf                  # Definição formal da gramática da linguagem
├── start.bus                     # Arquivo de bootstrap: define quais aplicações subir e em quais portas
├── erp.application.bus           # Declaração da aplicação ERP e seus módulos
├── erp.application.settings      # Configuração da aplicação ERP (ex: banco de dados)
├── vendas.domain.bus             # Declaração do módulo de Vendas e configurações
├── common.types.bus              # Tipos reutilizáveis (ex: cpf, email, endereco)
├── cliente.concept.bus           # Declaração do concept Cliente
├── cliente.ativarConta.service.bus # Serviço para ativar conta do cliente
└── clientes.ui.bus               # Estrutura visual (view) do concept Cliente
```

---

## 🔤 Arquivos e formatos

### `bussines.bnf`

Contém a gramática da linguagem no estilo BNF. Define a estrutura de todos os blocos como: start, application, concept, type, service, implementation, etc.

---

### `start.bus`

Declara de forma simples as aplicações que a VM deve subir:

```txt
applications
  app1
    path sistema.erp.application.bus
    port 8080
    https false

  app2
    path sistema.nfe.application.bus
    port 8443
    https true
    certPath /certs/nfe.crt
    keyPath /certs/nfe.key
```

---

### `erp.application.bus`

Aponta os módulos que fazem parte da aplicação ERP:

```txt
modules
  vendas
  cadastro
```

---

### `erp.application.settings`

Define as configurações da aplicação como banco de dados e opções de autenticação.

```txt
database sqlserver://localhost/erp
settings
  authMode integrated
  maxConnections 100
```

---

### `vendas.domain.bus`

Define o banco e configurações específicas do módulo:

```txt
database sqlserver://localhost/vendas
settings
  version 1.0
```

---

### `common.types.bus`

Define tipos reutilizáveis:

```txt
cpf string 11 \d{11} 999.999.999-99 "Cadastro de Pessoa Física"
email string 50 \S+@\S+\.\S+ "E-mail válido"
endereco
  rua string
  numero int
  cidade string
```

---

### `cliente.concept.bus`

Define a estrutura de dados do cliente, incluindo campos nativos e tipos customizados:

```txt
nome string 100 "Nome do cliente"
email? email
cpf cpf
dataNascimento? date
ativo? bool =true "Está ativo?"
```

---

### `clientes.ui.bus`

Define a estrutura visual do concept `cliente`, como ordem dos campos e agrupamento. (Exemplo simplificado.)

```txt
form
  section "Dados Pessoais"
    nome
    email
    cpf
    dataNascimento
```

---

### `cliente.ativarConta.service.bus`

Declara o serviço `ativarConta`, com entrada, saída e implementação:

```txt
input
  clienteId string

output
  status string
  clienteEmail string

implementation {
  var cliente = get cliente where id = clienteId
  if cliente.ativo == false {
    cliente.ativo = true
    set cliente
  }
  status = "ok"
  clienteEmail = cliente.email
}
```

---

## ✅ Benefícios

- A linguagem elimina repetições.
- Facilita a leitura e manutenção.
- Permite expansão gradual (ex: views, módulos externos, suporte à IA).
- APIs são geradas automaticamente a partir dos `concepts` e `services`.

---