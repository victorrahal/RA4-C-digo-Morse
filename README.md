# Compilador da Linguagem RPN — Nome em Código Morse

**Instituição:** Pontifícia Universidade Católica do Paraná — Campus Curitiba
**Ano/Semestre:** 2026/1
**Disciplina:** Compiladores
**Trabalho:** Individual

---

## Autor

| Nome | GitHub |
|------|--------|
| Victor Rahal Basseto | [@victorrahal](https://github.com/victorrahal) |

**Linguagem de implementação:** Python 3

---

## Sobre o trabalho

Este projeto individual reaproveita a linguagem de programação baseada em
**notação polonesa reversa (RPN)** desenvolvida ao longo do semestre e a
**estende com o comando `MORSE`**, capaz de transmitir um texto em código Morse
piscando um LED. O objetivo é gerar o nome do autor — `1 VICTOR RAHAL BASSETO`
 — em código Morse, compilando para Assembly ARMv7 executável no simulador
**Cpulator (placa DE1-SoC)**.

O pipeline completo do compilador é preservado: análise léxica, análise
sintática LL(1), construção da tabela de símbolos, verificação de tipos,
geração da árvore atribuída e geração de código Assembly.

A única saída em disco é o arquivo `saida/Assembly.s`.

---

## Estrutura do projeto

```
morse_compiler/
├── analisadorSemantico.py      # ponto de entrada do compilador
├── README.md
├── requirements.txt            # sem dependencias externas (apenas a stdlib)
├── .gitignore
├── docs/                       # toda a documentacao do projeto
│   ├── cod_morse.md
│   ├── gram_ebnf.md
│   ├── regras_de_tipos.md
│   ├── constTabelaSimbolos.md
│   └── prepararEntradaSemantica.md
├── src/                        # codigo-fonte do compilador
│   ├── tokensConfig.py
│   ├── estadosLexicos.py
│   ├── lerTokens.py
│   ├── construirGramatica.py
│   ├── parsear.py
│   ├── gerarArvore.py
│   ├── construirTabelaSimbolos.py
│   ├── verificarTipos.py
│   ├── gerarArvoreAtribuida.py
│   ├── gerarAssembly.py
│   └── main.py
├── testes/                     # codigo de teste (unittest)
│   ├── testeLexico.py
│   ├── testeSintatico.py
│   ├── testePrepararEntradaSemantica.py
│   ├── testeTabelaSimbolos.py
│   ├── testeSemantico.py
│   ├── testeAssembly.py
│   ├── testeIntegracao.py
│   └── programas/              # programas .txt usados como entrada dos testes
│       ├── nome.txt
│       ├── teste1.txt / teste2.txt / teste3.txt
│       └── teste_comentario_*.txt
└── saida/                      # Assembly.s gerado (unico artefato de saida)
```

Os programas de entrada (`.txt`) ficam em `testes/programas/`. O compilador
aceita tanto um **nome simples** (resolvido nessa pasta) quanto um **caminho
completo**, como um compilador de linha de comando.

---

## Como executar

### Pré-requisito

Python 3.10 ou superior. Sem dependências externas.

### Executar o compilador

Na raiz do projeto:

```bash
python analisadorSemantico.py <arquivo_de_teste>
```

Gerar o nome em código Morse:

```bash
python analisadorSemantico.py nome.txt
```

Outros exemplos:

```bash
python analisadorSemantico.py teste1.txt   # programa aritmético válido
python analisadorSemantico.py teste2.txt   # programa com erros semânticos
```

O programa exibe no terminal o andamento de cada fase (léxico, sintático,
semântico) e, **se não houver erros semânticos**, grava `saida/Assembly.s`.
O código de saída é `0` quando não há erros e `1` quando há.

### Executar os testes

A suíte usa apenas a biblioteca padrão (`unittest`), sem dependências externas:

```bash
# Todos os testes de uma vez
python -m unittest discover -s testes -p "teste*.py"

# Por módulo
python testes/testeLexico.py                  # análise léxica (+ KW_MORSE)
python testes/testeSintatico.py               # análise sintática LL(1) (+ MORSE)
python testes/testePrepararEntradaSemantica.py # preparação da entrada
python testes/testeTabelaSimbolos.py          # tabela de símbolos (+ MORSE)
python testes/testeSemantico.py               # verificação de tipos (+ MORSE)
python testes/testeAssembly.py                # geração de Assembly (+ Morse)
python testes/testeIntegracao.py              # pipeline completo ponta-a-ponta
```

---

## Descrição da Linguagem

A linguagem é baseada em **notação polonesa reversa (RPN)**. Cada instrução é
delimitada por parênteses. Todo programa deve começar com `(START)` e terminar
com `(END)`.

### Estrutura geral

```
(START)
(instrução 1)
(instrução 2)
...
(END)
```

### Comentários

Comentários são delimitados por `*{` e `}*` e podem aparecer em qualquer
posição: linha inteira, final de linha ou entre tokens de uma expressão.

```
*{ comentário em linha inteira }*
(5 3 +) *{ comentário no fim da linha }*
(5 *{ entre operandos }* 3 +)
```

### Expressões aritméticas

Operações seguem a notação `(operando operando operador)`:

```
(5 3 +)           -> soma: 5 + 3
(10 2 -)          -> subtração: 10 - 2
(4 3 *)           -> multiplicação: 4 * 3
(9.0 3.0 |)       -> divisão real: 9.0 / 3.0
(9 4 /)           -> divisão inteira: 9 // 4
(9 4 %)           -> resto: 9 % 4
(2 3 ^)           -> potenciação: 2 ** 3
```

Expressões podem ser aninhadas sem limite:

```
((A B *) (C D +) -)    -> (A*B) - (C+D)
```

### Comandos especiais

| Sintaxe | Descrição |
|---------|-----------|
| `(V MEM X)` | Armazena o valor `V` na variável `X` |
| `(X)` | Retorna o valor armazenado em `X` (0 se não inicializada) |
| `(N RES)` | Retorna o resultado da linha N posições atrás (N ≥ 0) |
| `(P MORSE)` | Transmite o payload `P` em código Morse piscando o LED |

### O comando `MORSE`

O comando `(P MORSE)` transmite o payload `P` em código Morse. O payload pode
ser:

- um **literal inteiro** — ex.: `(1 MORSE)` transmite o dígito `1` (`·----`);
- uma **sequência de letras maiúsculas** — ex.: `(VICTOR MORSE)` soletra
  `V-I-C-T-O-R`, letra por letra.

Ponto importante: o payload é **texto literal** — os dígitos a transmitir ou as
letras a soletrar. Ele **não é uma leitura de variável**. Por isso `(VICTOR
MORSE)` não exige que `VICTOR` tenha sido declarada e não gera erro de variável
não declarada.

As durações seguem o enunciado:

| Elemento | Duração | Estado do LED |
|----------|---------|---------------|
| Ponto (`·`) | 300 ms | ligado |
| Traço (`–`) | 600 ms | ligado |
| Espaço dentro da letra | 450 ms | desligado |
| Espaço entre letras | 900 ms | desligado |
| Espaço entre palavras | 2000 ms | desligado |

Detalhes da geração de código, do modelo de execução no Cpulator e do ajuste de
velocidade estão em [`docs/codigo_morse.md`](docs/codigo_morse.md).

### Estruturas de controle

```
(IF (condicao) (ramo_verdadeiro) (ramo_falso))
(WHILE (condicao) (corpo))
(FOR inicio fim VARIAVEL (corpo))
```

As condições usam operadores relacionais `<` e `>`:

```
(IF (TOTAL 10 <) (3 3 +) (2 2 -))
(WHILE (CONT 5 <) ((CONT 1 +) MEM CONT))
(FOR 1 5 I (I 2 *))
```

---

## Tipos suportados

| Tipo | Descrição | Exemplos |
|------|-----------|---------|
| `INT` | Número inteiro | `5`, `42`, `0` |
| `REAL` | Número real de ponto flutuante | `3.14`, `9.0`, `2.5` |
| `BOOL` | Valor lógico — resultado de operações relacionais | resultado de `<` ou `>` |

### Regras de promoção de tipo

Nas operações `+`, `-`, `*`, `^` e `|`:

| Operando 1 | Operando 2 | Resultado |
|-----------|-----------|-----------|
| `INT` | `INT` | `INT` |
| `INT` | `REAL` | `REAL` |
| `REAL` | `INT` | `REAL` |
| `REAL` | `REAL` | `REAL` |

As operações `/` e `%` **exigem ambos os operandos INT** e retornam `INT`.
A operação `|` sempre retorna `REAL`.
As condições `<` e `>` aceitam `INT` ou `REAL` e retornam `BOOL`.
O comando `MORSE` **não produz valor** (é um comando de saída).

---

## Regras de definição e uso de variáveis

- Identificadores são sequências de **letras latinas maiúsculas** (ex: `X`, `TOTAL`, `CONTADOR`).
- Uma variável é **definida** no momento de sua primeira atribuição via `MEM`.
- O tipo da variável é inferido na definição e **não pode ser alterado**.
- Usar uma variável antes de defini-la é **erro semântico** (`VARIAVEL_NAO_DECLARADA`).
- Reatribuir uma variável com tipo diferente do original é **erro semântico** (`REDEFINICAO_INCOMPATIVEL`).
- A variável de controle do `FOR` é automaticamente do tipo `INT`.
- O escopo de todas as variáveis é **global** por arquivo.
- O payload de `MORSE` **não** é uma variável: não é declarado, lido nem verificado como tal.

---

## Exemplo principal — `nome.txt`

```
(START)
*{ Nome em codigo Morse - Victor Rahal Basseto }*
(1 MORSE)
(VICTOR MORSE)
(RAHAL MORSE)
(BASSETO MORSE)
(END)
```

Compilando com `python analisadorSemantico.py nome.txt`, o `saida/Assembly.s`
resultante pisca o LED transmitindo `1 VICTOR RAHAL BASSETO` em código Morse.

---

## Exemplos de programas semanticamente válidos

### teste1.txt — programa completo com todos os recursos aritméticos

```
(START)
(5 3 +)
(10 2 -)
(4 3 *)
(9.0 3.0 |)
(9 4 /)
(9 4 %)
(2 3 ^)
(15 MEM TOTAL)
(2.5 MEM PESO)
(TOTAL)
(1 RES)
((5 3 +) 2 *)
(IF (TOTAL 10 <) (3 3 +) (2 2 -))
(0 MEM CONT)
(WHILE (CONT 5 <) ((CONT 1 +) MEM CONT))
(FOR 1 5 I (I 2 *))
(END)
```

### teste3.txt — expressões aninhadas e comentários em várias posições

```
(START)
(2 3 +) *{ comentario no fim da linha }*
*{ comentario em linha inteira }*
((3 2 +) (5 1 -) *)
(((2 3 +) 4 *) 2 /)
(9 4 %)
(8.0 *{ comentario entre tokens }* 2.0 |)
(2 3 ^)
(0 MEM BASE)
(1 MEM FATOR)
(5 MEM LIMITE)
(BASE)
(1 RES)
((BASE FATOR +) MEM BASE)
(IF (BASE LIMITE <) (BASE 2 *) (BASE FATOR -))
(0 MEM N)
(WHILE (N 4 <) ((N 1 +) MEM N))
(FOR 1 5 K (K K *))
(END)
```

---

## Exemplo de programa semanticamente inválido

### teste2.txt — múltiplos erros semânticos

```
(START)
(10 MEM INTVAR)
(3.5 MEM REALVAR)
(3.5 2 /)           *{ ERRO: divisao inteira com REAL }*
(5 2.5 %)           *{ ERRO: modulo com REAL }*
(XSEMDEF 5 +)       *{ ERRO: variavel nao declarada }*
(100 RES)           *{ ERRO: RES referencia linha inexistente }*
(END)
```

Erros detectados:
- `OPERADOR_EXIGE_INT` — `/` e `%` usados com REAL;
- `VARIAVEL_NAO_DECLARADA` — `XSEMDEF` nunca definida (reportado **uma única vez**);
- `RES_INDICE_INVALIDO` — índice 100 maior que o histórico disponível.

---

## Saída (`saida/`)

A cada execução bem-sucedida é gerado **um único artefato**:

| Arquivo | Formato | Descrição |
|---------|---------|-----------|
| `Assembly.s` | Assembly ARMv7 | Código gerado — **apenas se não houver erros semânticos** |

Diferente de versões anteriores do projeto, **nenhum outro artefato é gravado**
(sem JSON, Markdown, gráficos ou manifesto): apenas o Assembly final.

---

## Documentação adicional

- [`docs/codigo_morse.md`](docs/codigo_morse.md) — o comando `MORSE`, durações, modelo no Cpulator e geração de código
- [`docs/gramatica_ebnf.md`](docs/gramatica_ebnf.md) — gramática em formato EBNF (com o comando `MORSE`)
- [`docs/regras_de_tipos.md`](docs/regras_de_tipos.md) — regras de validação de tipos em cálculo de sequentes
- [`docs/construirTabelaSimbolos.md`](docs/construirTabelaSimbolos.md) — tabela de símbolos e detecção de erros de declaração
- [`docs/prepararEntradaSemantica.md`](docs/prepararEntradaSemantica.md) — integração léxico + sintático e preparação da entrada semântica