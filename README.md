# Trabalho de banco de dados 1 - Sistema de Gerenciamento de Consultas Médicas
## Relatório -> [Relatório Sistema de gerenciamento ](https://github.com/CassioLimaP/BD1-Sistema_De_Agendamentos_Consulta_Medica/blob/main/RELATORIO%20SISTEMA%20DE%20GERENCIAMENTO.pdf)
## Código SQL usado para gerar as tabelas, views e triggers: 

```sql
PRAGMA foreign_keys = ON;

CREATE TABLE Endereco (
    cep    VARCHAR(9)  NOT NULL,
    rua    CHAR(20)    NOT NULL,
    bairro CHAR(20)    NOT NULL,
    cidade CHAR(20)    NOT NULL,
    estado CHAR(2)     NOT NULL,
    PRIMARY KEY (cep)
);


CREATE TABLE Usuario (
    id_usuario      INTEGER     NOT NULL PRIMARY KEY AUTOINCREMENT,
    nome_usuario    CHAR(100)   NOT NULL,
    cpf             VARCHAR(14) NOT NULL UNIQUE,
    data_nascimento DATE        NOT NULL,
    senha           VARCHAR(20) NOT NULL,
    cep             VARCHAR(9)  NOT NULL,
    numero          INTEGER     NOT NULL,
    FOREIGN KEY (cep) REFERENCES Endereco(cep)
);

CREATE TABLE Telefone_Usuario (
    id_usuario      INTEGER NOT NULL,
    ddd             INTEGER NOT NULL,
    numero_telefone INTEGER NOT NULL,
    PRIMARY KEY (id_usuario, ddd, numero_telefone),
    FOREIGN KEY (id_usuario) REFERENCES Usuario(id_usuario)
);

CREATE TABLE Cliente (
    id_usuario   INTEGER  NOT NULL,
    tipo_cliente CHAR(20) NOT NULL,
    PRIMARY KEY (id_usuario),
    FOREIGN KEY (id_usuario) REFERENCES Usuario(id_usuario)
);

CREATE TABLE Cliente_PJ (
    id_usuario   INTEGER     NOT NULL,
    cnpj         VARCHAR(18) NOT NULL UNIQUE,
    razao_social CHAR(100)   NOT NULL,
    PRIMARY KEY (id_usuario),
    FOREIGN KEY (id_usuario) REFERENCES Cliente(id_usuario)
);


CREATE TABLE Paciente (
    id_usuario         INTEGER     NOT NULL,
    status_cadastro    CHAR(15)    NOT NULL,
    contato_emergencia VARCHAR(15) NULL,
    PRIMARY KEY (id_usuario),
    FOREIGN KEY (id_usuario) REFERENCES Usuario(id_usuario)
);

CREATE TABLE Dependente (
    id_usuario      INTEGER   NOT NULL,
    id_dependente   INTEGER   NOT NULL,
    nome_dependente CHAR(100) NOT NULL,
    id_paciente     INTEGER   NULL,
    PRIMARY KEY (id_usuario, id_dependente),
    FOREIGN KEY (id_usuario)  REFERENCES Cliente(id_usuario),
    FOREIGN KEY (id_paciente) REFERENCES Paciente(id_usuario)
);

CREATE TABLE Funcionario (
    matricula        INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
    data_contratacao DATE    NOT NULL,
    salario          REAL    NOT NULL,
    id_usuario       INTEGER NOT NULL,
    FOREIGN KEY (id_usuario) REFERENCES Usuario(id_usuario)
);

CREATE TABLE Atendente (
    matricula  INTEGER NOT NULL,
    ramal_mesa INTEGER NOT NULL,
    PRIMARY KEY (matricula),
    FOREIGN KEY (matricula) REFERENCES Funcionario(matricula)
);


CREATE TABLE Medico (
    matricula       INTEGER          NOT NULL,
    crm             VARCHAR(13)     NOT NULL,
    matricula_chefe INTEGER     NULL,
    PRIMARY KEY (matricula),
    FOREIGN KEY (matricula)       REFERENCES Funcionario(matricula),
    FOREIGN KEY (matricula_chefe) REFERENCES Medico(matricula)
);

CREATE TABLE Especialidade (
    id_especialidade   INTEGER  NOT NULL PRIMARY KEY AUTOINCREMENT,
    nome_especialidade CHAR(30) NOT NULL
);

CREATE TABLE Medico_Especialidade (
    matricula_medico INTEGER NOT NULL,
    id_especialidade INTEGER NOT NULL,
    PRIMARY KEY (matricula_medico, id_especialidade),
    FOREIGN KEY (matricula_medico) REFERENCES Medico(matricula),
    FOREIGN KEY (id_especialidade) REFERENCES Especialidade(id_especialidade)
);

CREATE TABLE Plano_de_Saude (
    id_plano       INTEGER  NOT NULL PRIMARY KEY AUTOINCREMENT,
    nome_operadora CHAR(30) NOT NULL
);

CREATE TABLE Cobertura (
    id_paciente     INTEGER NOT NULL,
    id_plano        INTEGER NOT NULL,
    num_carteirinha INTEGER NOT NULL,
    validade        DATE    NOT NULL,
    PRIMARY KEY (id_paciente, id_plano),
    FOREIGN KEY (id_paciente) REFERENCES Paciente(id_usuario),
    FOREIGN KEY (id_plano)    REFERENCES Plano_de_Saude(id_plano)
);


CREATE TABLE Data_disponivel (
    id_data INTEGER    NOT NULL PRIMARY KEY AUTOINCREMENT,
    hora    TIME       NOT NULL,
    dia_mes VARCHAR(5) NOT NULL
);

CREATE TABLE Agenda_data_disponivel (
    id_usuario INTEGER NOT NULL,
    id_data    INTEGER NOT NULL,
    PRIMARY KEY (id_usuario, id_data),
    FOREIGN KEY (id_usuario) REFERENCES Usuario(id_usuario),
    FOREIGN KEY (id_data)    REFERENCES Data_disponivel(id_data)
);

CREATE TABLE Consulta (
    id_consulta         INTEGER     NOT NULL PRIMARY KEY AUTOINCREMENT,
    data_consulta       DATE        NOT NULL,
    horario_consulta    TIME        NOT NULL,
    estado                   CHAR(15)    NOT NULL,
    valor_consulta      REAL        NOT NULL,
    tipo_consulta       CHAR(10)    NOT NULL,
    id_paciente         INTEGER     NOT NULL,
    matricula_medico    INTEGER     NOT NULL,
    matricula_atendente INTEGER     NOT NULL,
    id_plano            INTEGER     NULL,
    FOREIGN KEY (id_paciente)         REFERENCES Paciente(id_usuario),
    FOREIGN KEY (matricula_medico)    REFERENCES Medico(matricula),
    FOREIGN KEY (matricula_atendente) REFERENCES Atendente(matricula),
    FOREIGN KEY (id_plano)            REFERENCES Plano_de_Saude(id_plano)
);

CREATE TABLE Prescricao (
    id_consulta     INTEGER NOT NULL,
    data_prescricao DATE    NOT NULL,
    PRIMARY KEY (id_consulta),
    FOREIGN KEY (id_consulta) REFERENCES Consulta(id_consulta)
);

CREATE TABLE Remedio (
    id_remedio        INTEGER   NOT NULL PRIMARY KEY AUTOINCREMENT,
    descricao_remedio CHAR(200) NOT NULL
);

CREATE TABLE Prescricao_Remedio (
    id_consulta INTEGER NOT NULL,
    id_remedio  INTEGER  NOT NULL,
    dosagem     VARCHAR(30) NOT NULL,
    frequencia   VARCHAR(30) NOT NULL,
    PRIMARY KEY (id_consulta, id_remedio),
    FOREIGN KEY (id_consulta) REFERENCES Prescricao(id_consulta),
    FOREIGN KEY (id_remedio)  REFERENCES Remedio(id_remedio)
);

CREATE TABLE Atestado (
    id_atestado         INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
    id_consulta         INTEGER NOT NULL,
    periodo_afastamento VARCHAR(30)  NOT NULL,
    FOREIGN KEY (id_consulta) REFERENCES Consulta(id_consulta)
);

CREATE UNIQUE INDEX uq_atestado_consulta ON Atestado(id_consulta);

CREATE TABLE Prontuario (
    id_prontuario      INTEGER   NOT NULL PRIMARY KEY AUTOINCREMENT,
    data_abertura      DATE      NOT NULL,
    tipo_sanguineo     CHAR(3)   NOT NULL,
    observacoes_gerais CHAR(200) NULL,
    id_paciente        INTEGER   NOT NULL,
    FOREIGN KEY (id_paciente) REFERENCES Paciente(id_usuario)
);

CREATE TABLE Alergia (
    id_alergia        INTEGER   NOT NULL PRIMARY KEY AUTOINCREMENT,
    nome_alergia      CHAR(30)  NOT NULL,
    descricao_alergia CHAR(100) NULL
);

CREATE TABLE Prontuario_Alergia (
    id_prontuario INTEGER NOT NULL,
    id_alergia    INTEGER NOT NULL,
    PRIMARY KEY (id_prontuario, id_alergia),
    FOREIGN KEY (id_prontuario) REFERENCES Prontuario(id_prontuario),
    FOREIGN KEY (id_alergia)    REFERENCES Alergia(id_alergia)
);

CREATE TABLE Prontuario_Remedio (
    id_prontuario INTEGER NOT NULL,
    id_remedio    INTEGER NOT NULL,
    dosagem      VARCHAR(30) NOT NULL,
    frequencia    VARCHAR(30)  NOT NULL,
    PRIMARY KEY (id_prontuario, id_remedio),
    FOREIGN KEY (id_prontuario) REFERENCES Prontuario(id_prontuario),
    FOREIGN KEY (id_remedio)    REFERENCES Remedio(id_remedio)
);

---- TRIGGERS

CREATE TRIGGER preenche_endereco
BEFORE INSERT ON Usuario
FOR EACH ROW
BEGIN
    INSERT OR IGNORE INTO Endereco (cep, rua, bairro, cidade, estado)
    VALUES (NEW.cep, 'A preencher', 'A preencher', 'A preencher', 'A preencher');
END;

CREATE TRIGGER cria_prontuario
AFTER INSERT ON Paciente
FOR EACH ROW
BEGIN
    INSERT INTO Prontuario (data_abertura, tipo_sanguineo, observacoes_gerais, id_paciente)
    VALUES (DATE('now'), 'A preencher', NULL, NEW.id_usuario);
END;


---- VIEWS

CREATE VIEW view_medicos_especialidade AS
SELECT
    u.nome_usuario  AS nome_medico,
    m.crm,
    e.nome_especialidade
FROM Medico m
JOIN Funcionario f        ON m.matricula        = f.matricula
JOIN Usuario u            ON f.id_usuario       = u.id_usuario
JOIN Medico_Especialidade me ON m.matricula     = me.matricula_medico
JOIN Especialidade e      ON me.id_especialidade = e.id_especialidade
ORDER BY e.nome_especialidade, u.nome_usuario;

CREATE VIEW view_todos_usuarios AS
SELECT id_usuario, nome_usuario
FROM Usuario;
```
