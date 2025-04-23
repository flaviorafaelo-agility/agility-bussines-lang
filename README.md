# Bussines Lang

A **Bussines Lang** é uma linguagem de domínio específico (DSL) criada para modelar aplicações de negócio de forma declarativa, simples e extremamente produtiva. Escrita em arquivos `.bus`, ela tem como principais características:

- Estrutura semântica própria para negócios
- Integração nativa com IA e geração automática de CRUD/API
- Execução baseada em VM e bytecode
- Baixa verbosidade, alta clareza

---

## 🌐 Arquivos da Linguagem

### `start.bus`

Define as aplicações que devem ser iniciadas pela VM.

**Exemplo**:
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

**Resumo**: Arquivo declarativo responsável por iniciar aplicações e servidores HTTP.

---

### `erp.application.bus`

Define os módulos pertencentes a uma aplicação.

**Exemplo**:
```bus
modules
  vendas
  cadastro
```

---

### `erp.application.settings`

Contém configurações da aplicação como base de dados, timeout e flags.

**Exemplo**:
```bus
database bussines_erp
settings
  timeout 30
  retry true
```

---

### `vendas.domain.bus`

Configura o domínio (módulo), como tipo de banco e configs locais.

**Exemplo**:
```bus
database postgres
settings
  useSnakeCase true
```

---

### `common.types.bus`

Define tipos reutilizáveis como `cpf`, `email`, `endereco`.

**Exemplo**:
```bus
cpf string 11 \d{11} 999.999.999-99 "Cadastro de Pessoa Física"

endereco
  rua string "Rua"
  numero int "Número"
  cidade string "Cidade"
```

**BNF**: Igual ao `concept` com mesma estrutura de campos e anotações opcionais.

---

### `cliente.concept.bus`

Define a entidade cliente com campos, máscaras, validações e default.

**Exemplo**:
```bus
nome string 100 "Nome completo"
cpf cpf
email? string 50 \S+@\S+\.\S+ "E-mail"
ativo? bool =true "Cliente ativo?"
```

**Campo derivado**:
```bus
status := (ativo ? "Ativo" : "Inativo") "Status do cliente"
```

---

### `clientes.ui.bus`

(Planejado) Define o layout visual de um `concept` para renderização em UI.

**Exemplo**:
```bus
list
  columns nome, cpf, email, ativo
form
  fields nome, cpf, email, ativo
```

---

### `cliente.ativarConta.service.bus`

Serviço que ativa a conta de um cliente. Input/output seguem mesmo padrão do concept.

**Exemplo**:
```bus
input
  clienteId string

output
  status string

implementation {
  cliente := get cliente where id = clienteId
  cliente.ativo = true
  set cliente
  status = "ok"
}
```

---

## 💻 Anotações da VM

A Bussines VM interpreta bytecodes simples com operações como:

```text
LOAD_VAR produtos
BEGIN_FILTER temp1
ITER_BEGIN produtos
  LOAD_FIELD p nome
  PUSH_CONST "Teclado"
  EQ
  JUMP_IF_TRUE SKIP_APPEND
  APPEND temp1 p
SKIP_APPEND:
ITER_END
STORE_VAR produtos temp1
```

**Exemplo Bussines correspondente**:
```bus
produtos -= p => p.nome == "Teclado"
```

A VM pode usar objetos com `.Call("filterOut", lambda)` para permitir regras dinâmicas, com ou sem pré-compilação via `lambdas_gen.go`.

---

## 🧠 Extras

### Operações sobre arrays:
```bus
produtos += { nome = "Novo" preco = 100.0 estoque = 10 }

produtos -= p => p.nome == "Antigo"

produtos ~= p.nome == "Mouse" {
  preco = 150.00
}
```

---

## 📄 BNF Consolidado

Disponível em: [`specification/bussines.bnf`](./bussines.bnf)

---

## 📁 Estrutura do Projeto

```
specification/
├── bussines.bnf
├── start.bus
├── erp.application.bus
├── erp.application.settings
├── vendas/
│   └── vendas.domain.bus
├── cliente/
│   ├── cliente.concept.bus
│   ├── cliente.ativarConta.service.bus
│   └── clientes.ui.bus
├── common.types.bus
```

---

**Bussines Lang** by Agility Soluções 🚀