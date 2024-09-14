# Exemplos Práticos de Resiliência em Aplicações Node.js
Para demonstração foram utilizadas as Bibliotecas e Frameworks:

- `express`: Framework web para Node.js que facilita a criação de servidores e APIs. Usado para criar o servidor HTTP e rotas.

- `cockatiel`: Biblioteca que implementa padrões de resiliência, como timeout e bulkhead, para chamadas assíncronas.
    - `timeout`: Define um tempo máximo para uma operação, abortando se o tempo exceder o limite.
    - `bulkhead`: Restringe o número de operações simultâneas para prevenir que uma parte do sistema sobrecarregue.
- `express-rate-limit`: Middleware para Express que limita o número de requisições de um IP específico em um determinado período. Usado para implementar rate limiting.

- `opossum`: Biblioteca que implementa o padrão de Circuit Breaker, que ajuda a evitar chamadas a serviços que estão falhando. Permite definir limites de tempo, porcentagens de falhas e intervalos de reset. Link: https://github.com/nodeshift/opossum


## 1. Criar o Projeto Node.js

**1.1 Criar um diretório para o projeto e inicializar um novo projeto Node.js:**

 ```sh
 mkdir node-resilience-example
 cd node-resilience-example
 npm init -y
```
**1.2 Instalar as dependências necessárias:**

```
npm install express cockatiel express-rate-limit opossum
```

## 2. Exemplos de Código

### 2.1 Timeout

Crie um arquivo chamado **`server-timeout.js`**:

```javascript
const express = require('express');

const app = express();
const port = 8080;

// Função para criar uma Promise que simula um timeout
function timeoutPromise(ms, promise) {
    return new Promise((resolve, reject) => {
        const timeout = setTimeout(() => {
            reject(new Error('Tempo limite excedido!'));
        }, ms);

        promise
            .then((result) => {
                clearTimeout(timeout);
                resolve(result);
            })
            .catch((error) => {
                clearTimeout(timeout);
                reject(error);
            });
    });
}

// Função simulando chamada externa
async function externalService() {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve('Resposta da chamada externa');
        }, 5000); 
    });
}

// Rota de health check
app.get('/api/health', (req, res) => {
    res.send('OK');
});

// Rota que faz a chamada simulada com timeout
app.get('/api/timeout', async (req, res) => {
    try {
        const result = await timeoutPromise(3000, externalService());
        res.send(result);
    } catch (error) {
        res.status(500).send(`Erro: ${error.message}`);
    }
});

// Iniciando o servidor
app.listen(port, () => {
    console.log(`Servidor rodando em http://localhost:${port}`);
});


```

Utilize o comando para executar a aplicação
```javascript
node server-timeout.js
```
 
Utilize o comando pra realizar a chamada do endpoint
```javascript
curl localhost:8080/api/timeout
```

#### 2.1.2 Desafio
Ajustar configurações de timeout e corrigir erro de timeout execedido ao invocar o serviço
![Screen Shot 2024-09-13 at 21 42 04](https://github.com/user-attachments/assets/a451d1a1-ef3f-4116-8ab0-246d6548b7a3)

### 2.2 Rate Limit

### 2.3 Bulkhead

### 2.4 Rate Limit

### 2.5 Circuit Breaker
O Circuit Breaker ajuda a proteger a aplicação contra falhas em cascata, evitando chamadas excessivas para serviços que estão falhando.

```javascript
const express = require('express');
const CircuitBreaker = require('opossum');

const app = express();
const port = 8080;

// Função simulando chamada externa com 50% de falhas
async function externalService() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const shouldFail = Math.random() > 0.8;  // Simula o percentual de falhas
            if (shouldFail) {
                reject(new Error('Falha na chamada externa'));
            } else {
                resolve('Resposta da chamada externa');
            }
        }, 2000);  // Simula uma chamada que demora 2 segundos
    });
}

// Configuração do Circuit Breaker
const breaker = new CircuitBreaker(externalService, {
    timeout: 3000,  // Tempo limite de 3 segundos para a chamada
    errorThresholdPercentage: 50,  // Abre o circuito se 50% das requisições falharem
    resetTimeout: 10000  // Tenta fechar o circuito após 10 segundos
});

// Lidando com sucesso e falhas do Circuit Breaker
breaker.fallback(() => 'Resposta do fallback...');
breaker.on('open', () => console.log('Circuito aberto!'));
breaker.on('halfOpen', () => console.log('Circuito meio aberto, testando...'));
breaker.on('close', () => console.log('Circuito fechado novamente'));
breaker.on('reject', () => console.log('Requisição rejeitada pelo Circuit Breaker'));
breaker.on('failure', () => console.log('Falha registrada pelo Circuit Breaker'));
breaker.on('success', () => console.log('Sucesso registrado pelo Circuit Breaker'));

// Rota de health check
app.get('/api/health', (req, res) => {
    res.send('OK');
});

// Rota que faz a chamada simulada com o Circuit Breaker
app.get('/api/circuitbreaker', async (req, res) => {
    try {
        const result = await breaker.fire();
        res.send(result);
    } catch (error) {
        res.status(500).send(`Erro: ${error.message}`);
    }
});

// Iniciando o servidor
app.listen(port, () => {
    console.log(`Servidor rodando em http://localhost:${port}`);
});

```
#### 2.5.1 Desafio
Ajustar o o percentual de falhas para que o circuit breaker obtenha sucesso ao receber as requisições após sua abertura.



