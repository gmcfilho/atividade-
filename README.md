# Tabelas Identificadas e Regras de Negócio — Cenários 1, 2 e 3

Este documento consolida, para cada cenário, as tabelas identificadas a partir do modelo (entidades/relacionamentos do código) e as regras de negócio implementadas.

---

## Cenário 1 – Sistema de Clínica Veterinária

### Tabelas Identificadas

#### tutor
```sql
CREATE TABLE tutor (
    id       SERIAL PRIMARY KEY,
    nome     VARCHAR(100) NOT NULL,
    endereco VARCHAR(200) NOT NULL,
    telefone VARCHAR(20)  NOT NULL
);
```

#### animal
```sql
CREATE TABLE animal (
    id       SERIAL PRIMARY KEY,
    nome     VARCHAR(100) NOT NULL,
    especie  VARCHAR(50)  NOT NULL,
    raca     VARCHAR(50)  NOT NULL,
    id_tutor INTEGER NOT NULL REFERENCES tutor(id)
);
```

#### consulta
```sql
CREATE TABLE consulta (
    id        SERIAL PRIMARY KEY,
    id_animal INTEGER        NOT NULL REFERENCES animal(id),
    data      DATE           NOT NULL,
    motivo    TEXT           NOT NULL,
    valor     NUMERIC(10, 2) NOT NULL
);
```

### Regras de Negócio

| # | Regra |
|---|-------|
| RN01 | Não é permitido registrar uma consulta para um animal que não esteja cadastrado no sistema. |
| RN02 | O valor da consulta não pode ser negativo. |
| RN03 | Um tutor pode ter mais de um animal cadastrado. |
| RN04 | Um animal pertence a um único tutor (chave estrangeira obrigatória). |
| RN05 | É possível consultar todo o histórico de atendimentos de um animal específico. |
| RN06 | É possível listar todos os animais de um determinado tutor. |

### Fluxo simulado na Main

`Tutor → Animal → Consulta`

1. Cadastra um tutor
2. Cadastra um animal vinculado ao tutor
3. Lista os animais do tutor
4. Registra uma consulta para o animal (movimento)
5. Exibe o histórico de consultas do animal
6. Tenta registrar consulta com valor negativo (demonstra RN02)

---

## Cenário 2 – Sistema de Oficina Mecânica

### Tabelas Identificadas

#### cliente
```sql
CREATE TABLE cliente (
    id       SERIAL PRIMARY KEY,
    nome     VARCHAR(100) NOT NULL,
    telefone VARCHAR(20)  NOT NULL
);
```

#### veiculo
```sql
CREATE TABLE veiculo (
    id         SERIAL PRIMARY KEY,
    placa      VARCHAR(10)  NOT NULL UNIQUE,
    modelo     VARCHAR(100) NOT NULL,
    ano        INTEGER      NOT NULL,
    id_cliente INTEGER      NOT NULL REFERENCES cliente(id)
);
```

#### ordem_servico
```sql
CREATE TABLE ordem_servico (
    id         SERIAL PRIMARY KEY,
    id_veiculo INTEGER        NOT NULL REFERENCES veiculo(id),
    descricao  TEXT           NOT NULL,
    valor      NUMERIC(10, 2) NOT NULL,
    status     VARCHAR(20)    NOT NULL DEFAULT 'ABERTA'
);
```

### Regras de Negócio

| # | Regra |
|---|-------|
| RN01 | Não é permitido abrir uma ordem de serviço para um veículo que não esteja cadastrado. |
| RN02 | O valor do serviço não pode ser negativo. |
| RN03 | Um cliente pode ter mais de um veículo cadastrado. |
| RN04 | Ao abrir uma ordem de serviço, o status inicial é sempre "ABERTA". |
| RN05 | Uma ordem de serviço pode ser marcada como "CONCLUIDA". |
| RN06 | É possível consultar todo o histórico de manutenções de um determinado veículo. |

### Fluxo simulado na Main

`Cliente → Veículo → Ordem de Serviço`

1. Cadastra um cliente
2. Cadastra um veículo vinculado ao cliente
3. Lista os veículos do cliente
4. Abre uma ordem de serviço para o veículo (movimento)
5. Exibe o histórico de manutenções do veículo
6. Tenta abrir ordem com valor negativo (demonstra RN02)

---

## Cenário 3 – Sistema de Escola de Cursos Livres

### Tabelas Identificadas

#### aluno
```sql
CREATE TABLE aluno (
    id       SERIAL PRIMARY KEY,
    nome     VARCHAR(100) NOT NULL,
    email    VARCHAR(150) NOT NULL UNIQUE,
    telefone VARCHAR(20)  NOT NULL
);
```

#### curso
```sql
CREATE TABLE curso (
    id                  SERIAL PRIMARY KEY,
    nome                VARCHAR(100) NOT NULL,
    descricao           TEXT         NOT NULL,
    carga_horaria       INTEGER      NOT NULL,
    vagas_totais        INTEGER      NOT NULL,
    vagas_disponiveis   INTEGER      NOT NULL
);
```

#### matricula
```sql
CREATE TABLE matricula (
    id              SERIAL PRIMARY KEY,
    id_aluno        INTEGER        NOT NULL REFERENCES aluno(id),
    id_curso        INTEGER        NOT NULL REFERENCES curso(id),
    data_matricula  DATE           NOT NULL,
    valor           NUMERIC(10, 2) NOT NULL,
    UNIQUE (id_aluno, id_curso)
);
```

### Regras de Negócio

| # | Regra |
|---|-------|
| RN01 | Não é permitido matricular um aluno que não esteja cadastrado no sistema. |
| RN02 | Não é permitido matricular em um curso que não esteja cadastrado no sistema. |
| RN03 | Não é possível realizar matrícula em curso sem vagas disponíveis. |
| RN04 | O mesmo aluno não pode ser matriculado duas vezes no mesmo curso. |
| RN05 | O valor pago na matrícula não pode ser negativo. |
| RN06 | Ao realizar uma matrícula, o número de vagas disponíveis do curso deve ser decrementado. |
| RN07 | É possível consultar todos os alunos matriculados em um curso. |
| RN08 | É possível consultar todos os cursos em que um aluno está matriculado. |

### Fluxo simulado na Main

`Aluno → Curso → Matrícula`

1. Cadastra dois alunos
2. Cadastra um curso com apenas 1 vaga
3. Matricula o aluno1 com sucesso
4. Lista alunos do curso
5. Tenta matricular aluno1 novamente (demonstra RN04 — matrícula duplicada)
6. Tenta matricular aluno2 (demonstra RN03 — sem vagas)
7. Lista os cursos do aluno1
8. Tenta matrícula com valor negativo (demonstra RN05)

---

## Resumo Comparativo

| Cenário | Entidades | Relacionamento Principal | Regra de Validação de Valor |
|---|---|---|---|
| 1 – Clínica Veterinária | tutor, animal, consulta | tutor → animal → consulta | valor da consulta ≥ 0 |
| 2 – Oficina Mecânica | cliente, veiculo, ordem_servico | cliente → veiculo → ordem_servico | valor do serviço ≥ 0 |
| 3 – Escola de Cursos Livres | aluno, curso, matricula | aluno ↔ curso (via matricula) | valor da matrícula ≥ 0 |
