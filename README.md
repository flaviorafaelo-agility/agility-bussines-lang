# 🧠 Bussines Lang

**Bussines Lang** é uma linguagem de domínio específica (DSL) desenvolvida pela **Agility Soluções**, voltada para sistemas de negócio com foco em **agilidade, clareza e performance**.

A proposta da linguagem é **eliminar a verbosidade** do desenvolvimento tradicional, permitindo que conceitos de negócio, serviços, estrutura de dados e comportamento sejam definidos de forma **objetiva e sem repetição**.

---

## 🚀 Objetivos

- Reduzir o esforço de codificação em sistemas de negócio
- Evitar duplicação entre modelo, API e lógica
- Permitir escrita declarativa com semântica específica de negócio
- Facilitar automações e integração nativa com inteligência artificial
- Executar via **máquina virtual própria com suporte a bytecode**

---

## 📦 Estrutura do Projeto

```
specification/
├── start.bus                   # Arquivo que define as aplicações que devem ser iniciadas
├── erp.application.bus        # Define a aplicação ERP e seus módulos
├── erp.application.settings   # Configurações específicas da aplicação
├── vendas/
│   ├── vendas.domain.bus      # Define o domínio e o banco de dados
│   ├── cliente.concept.bus    # Representa o conceito de Cliente
│   ├── cliente.ativarConta.service.bus  # Serviço: ativar conta do cliente
│   └── clientes.ui.bus        # Interface declarativa para o concept cliente
├── common.types.bus           # Tipos reutilizáveis: cpf, telefone, endereço, etc.
└── bussines.bnf               # Definição da gramática formal da linguagem
```

---

## 📘 Exemplos

### Concept:
```bus
nome string 100 "Nome do cliente"
cpf cpf
ativo? bool =true "Está ativo?"
```

### Service:
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

## 🔮 Futuro

- Suporte completo a compilação estática
- Integração com ferramentas de IA
- Geração automática de Swagger, documentação e testes
- Interface visual autogerada a partir de `.ui.bus`

---

## 🛠 Desenvolvido por

**Agility Soluções**  
📧 contato@agility.com.br  
🌐 [www.agility.com.br](https://www.agility.com.br)
