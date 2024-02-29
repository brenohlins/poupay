const express = require('express');
const bodyParser = require('body-parser');
const sqlite3 = require('sqlite3').verbose();
const jwt = require('jsonwebtoken');

const app = express();
const port = process.env.PORT || 3000;

const db = new sqlite3.Database('sistema_registro.db');

app.use(bodyParser.json());

// Consulta para criar a tabela de usuários
const createUserTableQuery = `
    CREATE TABLE IF NOT EXISTS usuarios (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT UNIQUE NOT NULL,
        password TEXT NOT NULL
    )
`;

// Consulta para criar a tabela de registros
const createRecordsTableQuery = `
    CREATE TABLE IF NOT EXISTS registros (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT NOT NULL,
        value INTEGER NOT NULL,
        timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
    )
`;

// Executa as consultas de criação de tabelas
db.run(createUserTableQuery);
db.run(createRecordsTableQuery);

// Endpoint para registro de usuários
app.post('/register-user', async (req, res) => {
    try {
        const { username, password } = req.body;

        const checkUserQuery = 'SELECT * FROM usuarios WHERE username = ?';
        const existingUser = await executeQuery(checkUserQuery, [username]);

        if (existingUser.length > 0) {
            return res.status(400).send('Nome de usuário já existe');
        }

        const insertUserQuery = 'INSERT INTO usuarios (username, password) VALUES (?, ?)';
        await executeQuery(insertUserQuery, [username, password]);

        res.status(201).send('Usuário registrado com sucesso');
    } catch (error) {
        console.error(error);
        res.status(500).send('Erro interno do servidor');
    }
});

// Endpoint para registro de valores na página do dashboard
app.post('/register-value', async (req, res) => {
    try {
        const { token, value } = req.body;

        jwt.verify(token, process.env.CHAVE_SECRETA_JWT, async (err, decoded) => {
            if (err) {
                return res.status(401).send('Token inválido');
            }

            const username = decoded.username;

            const insertValueQuery = 'INSERT INTO registros (username, value, timestamp) VALUES (?, ?, datetime("now"))';
            await executeQuery(insertValueQuery, [username, value]);

            console.log(`Valor registrado por ${username} com valor ${value}`);

            res.status(200).send('Valor registrado com sucesso');
        });
    } catch (error) {
        console.error(error);
        res.status(500).send('Erro interno do servidor');
    }
});

// Função para executar consultas SQL
async function executeQuery(query, values) {
    return new Promise((resolve, reject) => {
        db.all(query, values, (err, rows) => {
            if (err) {
                reject(err);
            } else {
                resolve(rows);
            }
        });
    });
}

app.listen(port, () => {
    console.log(`Servidor está rodando na porta ${port}`);
});
