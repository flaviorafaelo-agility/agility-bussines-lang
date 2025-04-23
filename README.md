

# bussines.bnf

Start              ::= 'applications' Newline AppList
AppList            ::= App | App Newline AppList
App                ::= Identifier Newline Indent 'path' Whitespace Path
                       Newline Indent 'port' Whitespace Number
                       AppExtras

AppExtras          ::= ε
                     | Newline Indent 'https' Whitespace Bool AppExtras
                     | Newline Indent 'certPath' Whitespace Path AppExtras
                     | Newline Indent 'keyPath' Whitespace Path AppExtras

Concept            ::= FieldList
FieldList          ::= Field | Field Newline FieldList

Field              ::= Identifier Whitespace TypeName FieldOptions
                    | Identifier ':=' '(' Expression ')' CaptionOpt
                    | Identifier ':=' FunctionCall CaptionOpt

FieldOptions       ::= LengthOpt PatternOpt MaskOpt DefaultOpt CaptionOpt

LengthOpt          ::= ε | Whitespace Length
PatternOpt         ::= ε | Whitespace Pattern
MaskOpt            ::= ε | Whitespace MaskOrFormat
DefaultOpt         ::= ε | Whitespace Default
CaptionOpt         ::= ε | Whitespace Caption

TypeName           ::= BaseType ArrayOpt
ArrayOpt           ::= ε | '[]'

BaseType           ::= 'string' | 'int' | 'float' | 'bool' | 'date' | 'datetime' | CustomType
CustomType         ::= Identifier

Identifier         ::= Name NullableOpt
NullableOpt        ::= ε | '?'

Name               ::= Letter NameTail
NameTail           ::= ε | NameTailChar NameTail
NameTailChar       ::= Letter | Digit | '_'

Length             ::= Digit DigitTail
DigitTail          ::= ε | Digit DigitTail

FunctionCall       ::= FunctionName '(' Expression ')'
FunctionName       ::= 'sum' | 'avg' | 'count' | 'min' | 'max'

Caption            ::= '"' Text '"'
Expression         ::= Text
Default            ::= '=' Value

Whitespace         ::= ' ' | '\t'
Newline            ::= '\n' | '\r\n'
Indent             ::= Whitespace WhitespaceOpt
WhitespaceOpt      ::= ε | WhitespaceOpt Whitespace

Letter             ::= 'a' | ... | 'z' | 'A' | ... | 'Z'
Digit              ::= '0' | ... | '9'

Text               ::= '"' TextBody '"'
TextBody           ::= ε | TextChar TextBody
TextChar           ::= AnyCharacterExceptQuote

Bool               ::= 'true' | 'false'
Value              ::= Text | Number | Bool
Path               ::= Text


# cliente.ativarConta.service.bus


Summary:
O bloco service define uma operação de negócio exposta ou reutilizável pela aplicação.
Cada serviço possui dados de entrada (input) e saída (output), e uma lógica de execução declarada no bloco implementation.

- A entrada (input) e saída (output) seguem a mesma semântica dos campos em concept.
- Pode-se usar input[] ou output[] para indicar listas de objetos.
- A lógica é implementada diretamente em implementation, com variáveis, if, for, get, set, query, entre outros.

BNF:
<service> ::= <input-block> <output-block> <implementation-block>

<input-block> ::= "input" <newline> <field-list>
                | "input[]" <newline> <field-list>

<output-block> ::= "output" <newline> <field-list>
                 | "output[]" <newline> <field-list>

<field-list> ::= <field> | <field> <newline> <field-list>

<field> ::= <identifier> <whitespace> <type-name>
            [<whitespace> <length>] 
            [<whitespace> <pattern>] 
            [<whitespace> <mask-or-format>] 
            [<whitespace> <default>] 
            [<whitespace> <caption>]

<identifier> ::= <name> ["?"]
<name> ::= <letter> { <letter> | <digit> | "_" }
<type-name> ::= "string" | "int" | "float" | "bool" | "date" | "datetime" | <custom-type>

<implementation-block> ::= "implementation" <whitespace>? "{" <newline> <instruction-list> "}"

<instruction-list> ::= { <instruction> <newline> }

<instruction> ::= <var-decl> | <assignment> | <if-block> | <for-block> | <get> | <set> | <del> | <query> | <return>

<var-decl> ::= "var" <identifier> [": " <type-name>] ["=" <expression>]
<assignment> ::= <identifier> "=" <expression>
<if-block> ::= "if" "(" <expression> ")" "{" <instruction-list> "}" ["else" "{" <instruction-list> "}"]
<for-block> ::= "for" <identifier> "in" <expression> "{" <instruction-list> "}"]
<get> ::= "get" <concept> "where" <expression>
<set> ::= "set" <concept> <assignment-list> "where" <expression>
<del> ::= "del" <concept> "where" <expression>
<query> ::= "query" <concept> <field-list-inline>? "where" <expression>
<return> ::= "return" <expression>?

<assignment-list> ::= <assignment> { "," <assignment> }
<expression> ::= expressão matemática, lógica, concatenação, função etc.

<newline> ::= "\n" | "\r\n"
<whitespace> ::= " " | "\t"
<letter> ::= "a" ... "z" | "A" ... "Z"
<digit> ::= "0" ... "9"

Explicações:
- input / output podem ser campos simples ou uma estrutura (com múltiplos campos).
- input[] / output[] indicam listas de entrada ou saída.
- implementation define o comportamento com instruções semelhantes a linguagens tradicionais.
- Suporte nativo a if, for, get, set, del, query, return, etc.
- Campos seguem mesma regra de concept: tipos, máscara, nullable, etc.

Exemplo 1 - Lista simples:

input
  uf string

output[]
  id string
  nome string
  email string

implementation {
  var resultado = query cliente id, nome, email where estado = uf
  return resultado
}

Exemplo 2 - Entrada como array + lógica:

input[]
  produtoId string
  quantidade int

output[]
  produtoId string
  disponivel bool
  mensagem string

implementation {
  var resposta = []

  for item in input {
    var estoque = get produto where id = item.produtoId
    var ok = estoque.disponivel >= item.quantidade

    resposta += {
      produtoId: item.produtoId,
      disponivel: ok,
      mensagem: ok ? "Disponível" : "Estoque insuficiente"
    }
  }

  return resposta
}

Exemplo 3 - Entrada como array + lógica:

input
  clienteId string

output
  status string
  mensagem string

implementation {
  var cliente = get cliente where id = clienteId

  if (cliente.ativo == false) {
    cliente.ativo = true
    set cliente ativo = true where id = clienteId

    status = "ok"
    mensagem = "Conta ativada com sucesso"
  } else {
    status = "ignorado"
    mensagem = "Cliente já estava ativo"
  }
}

Exemplo 4

output
  status string
  clienteId string

implementation {
  var cliente: cliente = {
    nome = "Flávio"
    email = "flavio@exemplo.com"
    cpf = "12345678900"
    ativo = true
  }

  set cliente

  clienteId = cliente.id
  status = "ok"
}

Exemplo 5

output
  status string
  total int

implementation {
  produtos := {
    nome string
    preco float
    estoque int
  }[] = [
    {
      nome = "Notebook"
      preco = 4200.00
      estoque = 10
    },
    {
      nome = "Mouse Gamer"
      preco = 150.50
      estoque = 30
    }
  ]

  // Adiciona um novo produto
  produtos += {
    nome = "Teclado Mecânico"
    preco = 320.00
    estoque = 25
  }

  // Atualiza produto pelo nome
  produtos ~= p.nome == "Notebook" {
    preco = 3500
    estoque = 15
  }

  // Remove item da lista
  produtos -= p => p.nome == "Teclado Mecânico"

  // Conta os itens restantes
  total := 0
  for p in produtos {
    total += 1
  }

  status = "ok"
}



# cliente.concept.bus

Summary:
O concept define a estrutura e o comportamento implícito de uma entidade de negócio, sem precisar declarar CRUD, validações ou operações manualmente.

Ao declarar um concept, a linguagem infere automaticamente:
- persistência,
- endpoints REST (ou outros protocolos),
- estrutura de dados,
- integração com tipos personalizados (definidos em *.types.bus),
- e suporte nativo à IA.

Campos definidos em concepts podem usar a mesma semântica de um type, mesmo que não estejam formalmente declarados em arquivos .types.bus.

Campos podem ter:
- tipos primitivos ou personalizados
- tamanho fixo
- regex de validação
- máscara de entrada
- valor padrão
- descrição
- ser opcionais com `?`
- ser arrays com `[]`
- ser campos derivados (calculados) com `:=`
- suportar projeções com funções como `sum(...)`, `count(...)` etc.

Tipos primitivos:

string  
Representa texto simples. Letras, números, símbolos.  
Ex: Nome string

int  
Número inteiro.  
Ex: Idade int

float  
Número com casas decimais.  
Ex: Preco float

bool  
Valor verdadeiro ou falso.  
Ex: Ativo bool

date  
Data no formato YYYY-MM-DD.  
Ex: DataNascimento date

datetime  
Data e hora no formato YYYY-MM-DD HH:mm:ss.  
Ex: CriadoEm datetime

BNF:

<concept> ::= <field-list>

<field-list> ::= <field> | <field> <newline> <field-list>

<field> ::= <identifier> <whitespace> <type-name> 
            [<whitespace> <length>] 
            [<whitespace> <pattern>] 
            [<whitespace> <mask-or-format>] 
            [<whitespace> <default>] 
            [<whitespace> <caption>]
          | <identifier> ":=" "(" <expression> ")" [<whitespace> <caption>]
          | <identifier> ":=" <projection> [<whitespace> <caption>]

<identifier> ::= <name> ["?"]

<name> ::= <letter> { <letter> | <digit> | "_" }

<type-name> ::= <base-type> [ "[]" ]

<base-type> ::= "string" | "int" | "float" | "bool" | "date" | "datetime" | <custom-type>

<custom-type> ::= <identifier>  ; Referência a um tipo definido em *.types.bus

<length> ::= <digit>+

<pattern> ::= <regex>           ; Ex: \d{11}, ^[A-Z]+$

<mask-or-format> ::= <text>     ; Ex: 999.999.999-99, (99) 99999-9999

<default> ::= "=" <value>

<caption> ::= "\"" <text> "\""

<projection> ::= <projection-name> "(" <expression> ")"

<projection-name> ::= "sum" | "avg" | "count" | "min" | "max"

<expression> ::= <expr-text>    ; expressão matemática usando campos conhecidos

<whitespace> ::= " " | "\t"

<newline> ::= "\n" | "\r\n"

<letter> ::= "a" | ... | "z" | "A" | ... | "Z"

<digit> ::= "0" | ... | "9"

<value> ::= <text> ou <number> ou "true" ou "false" ou string entre aspas

Explicações:

<concept>: conteúdo completo do arquivo .concept.bus

<field-list>: lista de campos definidos (um por linha)

<field>: campo com nome, tipo e atributos opcionais ou campo calculado/derivado

<identifier>: nome do campo, podendo terminar com `?` para nullable

<type-name>: tipo do campo, podendo ser array (ex: string[], int[], endereco[])

<custom-type>: tipo reutilizável definido em *.types.bus

<length>: tamanho fixo de string

<pattern>: expressão regular para validação

<mask-or-format>: máscara visual (para entrada e exibição)

<default>: valor padrão atribuído quando omitido

<caption>: descrição amigável para label, tooltip, docs, IA

<expression>: expressão aritmética entre campos (campo derivado)

<projection>: cálculo aplicado sobre campo(s) de array (ex: sum(itens.total))

Exemplo de uso:

Titulo string 100 "Título do conteúdo"
CPF cpf
Email? string 50 \S+@\S+\.\S+ "E-mail do cliente"
Telefones? string[] 14 \(\d{2}\)\d{5}-\d{4} "Lista de telefones"
Endereco endereco
DataCadastro datetime
Ativo? bool =true "Está ativo?"

// Campo calculado (linha)
Total := (Preco * Quantidade - Desconto) "Total do item"

// Campo com projeção (agregação)
Itens ItemPedido[] "Itens do pedido"
TotalGeral := sum(Itens.Total) "Total geral do pedido"
QtdItens := count(Itens) "Quantidade de linhas"

Regras para o compilador/VM:

- Ao processar um arquivo .concept.bus, a engine:
  - Sobe na árvore de diretórios até encontrar o *.domain.bus.
  - Carrega as configurações globais do módulo.
  - Aplica as regras do módulo a todos os concepts daquele diretório (e subdiretórios, se configurado).

- Erro se não encontrar um módulo:
  Ex: Módulo não definido para concept "cliente.concept.bus". Adicione um arquivo .domain.bus no diretório.

- Validação cruzada:
  O database definido no módulo pode restringir certas operações suportadas pelos concepts.

Benefícios:
- Zero repetição — o concept não precisa saber do module.
- Organização natural por pasta.
- Escalável: cada time pode trabalhar em seu módulo isoladamente.
- Facilita geração automática de pacotes/API por domínio.

Possibilidades futuras:
- Suporte a múltiplos módulos com configuração central (ex: monorepo).
- Importação de tipos entre módulos.


# clientes.ui.bus

ainda pensar nesse tema

# common.types.bus

Summary:
O type define tipos reutilizáveis com semântica própria, podendo ser simples (como cpf string) ou complexos (como endereco com campos aninhados). 
Ele permite encapsular regras de negócio e estrutura, garantindo reuso e manutenção centralizada. Tipos definidos aqui podem ser utilizados em concepts.

Tipos primitivos:

Os tipos primitivos são os blocos fundamentais da linguagem. Eles representam dados básicos e são amplamente usados em concepts e types.

string
Representa um texto simples. Pode conter letras, números, símbolos e espaços.
Ex: Nome string

int
Representa um número inteiro, positivo ou negativo.
Ex: Idade int

float
Representa um número com casas decimais (ponto flutuante).
Ex: Preco float

bool
Representa um valor booleano: true ou false.
Ex: Ativo bool

date
Representa uma data no formato YYYY-MM-DD.
Ex: DataNascimento date

datetime
Representa uma data com hora no formato YYYY-MM-DD HH:mm:ss.
Ex: CriadoEm datetime

Convenções importantes:
- Tipos sempre em minúsculo (string, int, date).
- Tipos definidos em *.types.bus podem ser referenciados livremente nos concepts.
- A ideia é estender o domínio de forma semântica, ex: cpf não é só uma string — tem validação e formato.
- Tipos podem ser definidos como array com `[]`, ex: string[], int[], endereco[]
- Campos opcionais (nullable) devem terminar com `?`, ex: email?, ativo?, enderecos?

Tipos futuros:

Estes tipos ainda não são obrigatórios na primeira versão da linguagem, mas estão planejados para ampliar o poder expressivo e semântico da Bussines Lang.

enum
Representa uma lista fixa de valores possíveis.
Ideal para status, categorias, níveis, etc.
Ex: Status enum { Ativo, Inativo, Pendente }

list<T>
Representa uma lista de elementos do tipo T.
Usado para coleções como itens, telefones, endereços.
Ex: Telefones list<telefone>

map<K, V>
Representa uma estrutura de chave-valor.
Útil para armazenar dados flexíveis ou pares dinâmicos.
Ex: CamposPersonalizados map<string, string>

file
Representa um arquivo carregado ou armazenado.
Pode incluir imagem, PDF, XML, etc.
Ex: Contrato file

image
Representa uma imagem, com possível preview ou manipulação.
Ex: FotoPerfil image

money
Representa um valor monetário com precisão adequada para cálculos financeiros.
Ex: TotalPedido money

ref<T>
Representa uma referência a outro concept, criando relacionamento entre entidades.
Ex: Cliente ref<cliente>

BNF:

<type> ::= <field-list>

<field-list> ::= <field> | <field> <newline> <field-list>

<field> ::= <identifier> <whitespace> <type-name> 
            [<whitespace> <length>] 
            [<whitespace> <pattern>] 
            [<whitespace> <mask-or-format>] 
            [<whitespace> <default>] 
            [<whitespace> <caption>]

<complex-type> ::= <identifier> <newline> <indent> <field-list>

<identifier> ::= <name> ["?"]

<name> ::= <letter> { <letter> | <digit> | "_" }

<type-name> ::= <base-type> [ "[]" ]

<base-type> ::= "string" | "int" | "float" | "bool" | "date" | "datetime" | <custom-type>

<custom-type> ::= <identifier>

<mask-or-format> ::= <text>    ; Detectado por estrutura visual (formato ou máscara)

<default> ::= "=" <value>      ; Valor padrão (string, número, boolean, data)

<caption> ::= "\"" <text> "\""

<whitespace> ::= " " | "\t"

<newline> ::= "\n" | "\r\n"

<indent> ::= <whitespace>+

<letter> ::= "a" | ... | "z" | "A" | ... | "Z"

<digit> ::= "0" | ... | "9"

<value> ::= <text> ou <number> ou "true" ou "false" ou string entre aspas

Explicações:

<type>: conteúdo completo do arquivo .types.bus

<field-list>: lista de tipos definidos (um por linha ou bloco)

<field>: define um tipo simples (cpf string) ou inicia um tipo complexo (endereco)

<complex-type>: tipo que possui subcampos (indentados)

<identifier>: nome do campo, podendo terminar com `?` para indicar que é nullable

<type-name>: tipo base, podendo ser array (ex: string[], int[], endereco[])

<custom-type>: referência a outro tipo definido no mesmo ou outro arquivo

<length>: usado apenas com string, indica o tamanho fixo

<pattern>: regex de validação (ex: \d{11})

<mask-or-format>: formato visual (ex: 999.999.999-99, dd/MM/yyyy)

<default>: valor atribuído quando o campo for omitido

<caption>: descrição amigável exibida como tooltip, label ou ajuda

Exemplo:

cpf string 11 \d{11} 999.999.999-99 "Cadastro de Pessoa Física"
email? string 50 \S+@\S+\.\S+ "E-mail do cliente"
ativo? bool "Está ativo?" =true
dataNascimento? date "Nascimento"
emails? string[] 50 \S+@\S+\.\S+ "Lista de e-mails"

endereco
  rua string "Nome da rua"
  numero int "Número da casa"
  cep string 8 \d{8} "CEP"
  cidade string "Cidade"

enderecos? endereco[] "Lista de endereços"


# erp.application.bus

Summary:
A application representa o sistema completo, agrupando múltiplos módulos (*.domain.bus). Define configurações globais como nome da aplicação, versão, autor, e até regras globais de compilação ou geração de artefatos.

application (ERP)
src/
├── erp.application.bus
├── vendas/
│   ├── vendas.domain.bus
│   ├── pedido.concept.bus
│   └── cliente.concept.bus
└── cadastro/
    ├── cadastro.domain.bus
    ├── usuario.concept.bus
    └── perfil.concept.bus



 BNF:
 <application-def> ::= "application" <whitespace> <identifier> <newline> <application-body>

<application-body> ::= <application-line> | <application-line> <newline> <application-body>

<application-line> ::= "version" <whitespace> <text>
                     | "description" <whitespace> <text>
                     | "author" <whitespace> <text>
                     | "default-database" <whitespace> <db-type>

<db-type> ::= "sqlite" | "postgresql" | "sqlserver" | "mysql"

<identifier> ::= <letter> { <letter> | <digit> | "_" }

<text> ::= { qualquer caractere visível }

<whitespace> ::= " " | "\t"

<newline> ::= "\n" | "\r\n"
   

Exemplo de conteúdo:
Campo | Obrigatório | Descrição
application | ✅ | Nome da aplicação
version | opcional | Versão atual
description | opcional | Descrição da aplicação
author | opcional | Nome do criador/desenvolvedor
default-database | opcional | Define o banco padrão, caso não seja definido em cada módulo

Exemplo de conteúdo:
application ERP
version 1.0.0
description "Sistema integrado de gestão empresarial"
author "Agility Soluções"
default-database postgresql
date-format dd/MM/yyyy
currency-symbol R$

 Regras de associação
O compilador começa pelo arquivo .application.bus e varre recursivamente os diretórios.
Todos os arquivos .domain.bus localizados abaixo dessa pasta pertencem à aplicação.
Cada .domain.bus continua agrupando seus próprios concepts, e herda configurações globais se não definidas localmente.

# erp.application.settings

{
  "name": "ERP Comercial",
  "database": "/data/erp.sqlite",
  "user": "admin",
  "password": "1234",
  "logLevel": "info",
  "enableSwagger": true
}


# start.bus

Summary:
O arquivo `start.bus` define o ponto de entrada da VM da Bussines Lang, listando as aplicações que devem ser iniciadas, as portas a serem utilizadas e configurações opcionais como HTTPS e certificados. Ele é centralizado e fica fora da estrutura dos módulos.

Esse arquivo permite executar múltiplas aplicações simultaneamente e configurar cada uma com diferentes portas, caminhos e opções de segurança.

Estrutura geral:
- Cada aplicação é identificada por um nome (ex: app1, app2)
- O conteúdo de cada bloco especifica:
  - o caminho completo para o arquivo `*.application.bus`
  - a porta a ser usada para expor a aplicação
  - se HTTPS deve ser ativado
  - caminhos para o certificado e a chave, se necessário

BNF:

<start> ::= "applications" <newline> { <app-block> }

<app-block> ::= <identifier> <newline> <indent> <app-config>

<app-config> ::= 
    "path" <whitespace> <path>
  | "port" <whitespace> <port-number>
  | "https" <whitespace> ("true" | "false")
  | "certPath" <whitespace> <file-path>
  | "keyPath" <whitespace> <file-path>
  | <newline> <indent> <app-config>

<path> ::= <identifier> "." { <identifier> "." } <identifier>

<port-number> ::= <digit>+

<file-path> ::= <text>

<identifier> ::= <letter> { <letter> | <digit> | "_" }

<whitespace> ::= " " | "\t"
<indent> ::= <whitespace>+
<newline> ::= "\n" | "\r\n"
<letter> ::= "a" | ... | "z" | "A" | ... | "Z"
<digit> ::= "0" | ... | "9"
<text> ::= qualquer sequência de caracteres (sem aspas)

Explicações:

<start>: conteúdo completo do arquivo `start.bus`

<app-block>: define uma aplicação que será iniciada pela VM

<path>: caminho completo para o arquivo `*.application.bus`

<port>: número da porta TCP usada pela aplicação

<https>: ativa HTTPS (se `true`, `certPath` e `keyPath` devem ser definidos)

<certPath>, <keyPath>: caminhos absolutos ou relativos para os arquivos do certificado SSL

Exemplo de uso:

applications
  erp
    path sistema.erp.application.bus
    port 8080
    https false

  fiscal
    path sistema.fiscal.application.bus
    port 8443
    https true
    certPath /certs/fiscal.crt
    keyPath /certs/fiscal.key

Comportamento da VM:

- Para cada item em `applications`, a VM:
  - carrega a aplicação do caminho indicado
  - inicia um servidor HTTP(S) na porta informada
  - registra as APIs baseadas nos `concepts` encontrados
  - aplica as configurações indicadas no settings da aplicação (ex: database)

Benefícios:
- Início centralizado de múltiplas aplicações
- Configuração declarativa por arquivo
- Suporte nativo a HTTPS e certificados
- Organização limpa e fora dos módulos da aplicação
- Pronto para ambientes multiapp e balanceamento

Possibilidades futuras:
- Configuração de domínio externo (hostnames)
- Estratégias de autenticação globais
- Registro em serviço de discovery


application (ERP)

start.bus                        ← Arquivo de inicialização global da VM

sistema/                         ← Raiz das aplicações e módulos
├── erp/
│   ├── erp.application.bus                ← Define a aplicação ERP
│   ├── erp.application.settings           ← Configurações da aplicação ERP (JSON)
│   ├── types/
│   │   └── common.types.bus               ← Tipos reutilizáveis
│   ├── vendas/
│   │   ├── vendas.domain.bus              ← Define o módulo Vendas
│   │   ├── vendas.domain.settings         ← Configurações do módulo Vendas
│   │   ├── pedidos/
│   │   │   └── pedido.concept.bus
│   │   └── clientes/
│   │       └── cliente.concept.bus
│   ├── financeiro/
│   │   ├── financeiro.domain.bus          ← Define o módulo Financeiro
│   │   ├── financeiro.domain.settings     ← Configurações do módulo Financeiro
│   │   ├── contas-pagar/
│   │   │   └── conta.concept.bus
│   │   └── pagamentos/
│   │       └── pagamento.concept.bus
│   └── ecommerce/
│       ├── ecommerce.domain.bus           ← Define o módulo Ecommerce
│       ├── ecommerce.domain.settings      ← Configurações do módulo Ecommerce
│       ├── produtos/
│       │   └── produto.concept.bus
│       └── carrinhos/
│           └── carrinho.concept.bus


# vendas.domain.bus

Summary:
O module representa o agrupador lógico de vários concepts, e define metadados comuns a eles, como o tipo de banco de dados, descrição do domínio, e outras configurações globais.

Cada módulo corresponde a um domínio funcional ou técnico do sistema (ex: vendas, cadastro, estoque).

Campos suportados:
Campo | Obrigatório | Descrição
module | ✅ | Nome do módulo
description | opcional | Descrição curta sobre o objetivo do módulo
database | opcional | Tipo de banco suportado pelos concepts vinculados (ex: sqlite, postgresql, sqlserver, mysql)
date-format dd/MM/yyyy
currency-symbol R$

bnf:
<module-def> ::= "module" <whitespace> <identifier> <newline> <module-body>

<module-body> ::= <module-line> | <module-line> <newline> <module-body>

<module-line> ::= "description" <whitespace> <text>
                | "database" <whitespace> <db-type>

<db-type> ::= "sqlite" | "postgresql" | "sqlserver" | "mysql"

<identifier> ::= <letter> { <letter> | <digit> | "_" }

<text> ::= { qualquer caractere visível }

<whitespace> ::= " " | "\t"

<newline> ::= "\n" | "\r\n"

<letter> ::= "a" | ... | "z" | "A" | ... | "Z"

<digit> ::= "0" | ... | "9"

Regras de uso:
O nome do módulo é usado como namespace para os concepts.
O database define a base para gerar código SQL ou ORMs.
Pode haver validações para garantir que todos os concepts do módulo estejam compatíveis com o banco especificado.