# PB-Compass-Docker
Exercícios de Docker realizados no programa de bolsas de DevSecOps da Compass Uol. 

# 1. Rodando um container básico
Execute um container usando a imagem do Nginx e acesse a página padrão no navegador.
Exemplo de aplicação: Use a landing page do TailwindCSS como site estático dentro do container.

```bash
docker pull nginx
```

```bash
docker run -d -p 8080:80 nginx
```

Acessar a página padrão do Nginx via http://localhost:8080 no navegador.

```bash
mkdir tailwind
cd tailwind
curl -o index.html https://tailwindtoolbox.github.io/Landing-Page/
```

```bash
nano Dockerfile

FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

```bash
docker build -t tailwind .
```

```bash
docker run -d -p 8081:80 tailwind
```

Acessar a landing page do Tailwind CSS via http://localhost:8081.

---

# 2. Criando e rodando um container interativo
Inicie um container Ubuntu e interaja com o terminal dele.
Exemplo de aplicação: Teste um script Bash que imprime logs do sistema ou instala pacotes de forma interativa.

```bash
docker pull ubuntu
```

```bash
docker run -it ubuntu
```

```bash
apt update
```

```bash
apt install curl
```

---

# 3. Listando e removendo containers
Liste todos os containers em execução e parados, pare um container em execução e remova um container específico.
Exemplo de aplicação: Gerenciar containers de testes criados para verificar configurações ou dependências.

```bash
docker ps
docker ps -a
```

```bash
docker stop <nome ou ID do container>
```

```bash
docker rm <nome ou ID do container>
```

---

# 4. Criando um Dockerfile para uma aplicação simples em Python
Crie um Dockerfile para uma aplicação Flask que retorna uma mensagem ao acessar um endpoint.
Exemplo de aplicação: Use a API de exemplo Flask Restful API Starter para criar um endpoint de teste.

```bash
mkdir flask-docker
cd flask-docker/
```

```bash
nano app.py

from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello_world():
    return 'Olá, Felipe! Sua aplicação Flask funcionou!'
```

```bash
sudo apt install python3-flask
flask --app app run
```

Acessar a página via http://localhost:5000.

```bash
nano Dockerfile

FROM python:3.9-slim
WORKDIR /app
COPY . /app
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 5000
CMD ["flask", "run", "--host=0.0.0.0"]
```

```bash
nano requirements.txt

Flask==2.2.2
Werkzeug==2.2.2
```

```bash
docker build -t flask-docker .
docker run -p 5000:5000 flask-docker
```

Acessar novamente a página via http://localhost:5000.

---

# 5. Criando e utilizando volumes para persistência de dados
Execute um container MySQL e configure um volume para armazenar os dados do banco de forma persistente.
Exemplo de aplicação: Use o sistema de login e cadastro do Laravel Breeze, que usa MySQL.

```bash
sudo apt install composer
```

```bash
composer create-project --prefer-dist laravel/laravel laravel-mysql
cd laravel-mysql
```

```bash
composer require laravel/breeze --dev
php artisan breeze:install # Selecione Blade with Alpine -> No -> Pest
```

```bash 
nano .env

APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=felipe
DB_PASSWORD=123
```

```bash
nano Dockerfile

FROM php:8.2-apache
RUN docker-php-ext-install pdo pdo_mysql
WORKDIR /var/www/html
COPY . .
RUN curl -sS https://getcomposer.org/installer | php
RUN php composer.phar install
EXPOSE 80
CMD ["apache2-foreground"]
```

```bash
nano docker-compose.yml

version: '3.8'
services:
  app:
    image: laravel
    container_name: laravel_app
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    volumes:
      - .:/var/www/html
    depends_on:
      - mysql
    networks:
      - laravel_net

  mysql:
    image: mysql:8.0
    container_name: mysql_db
    environment:
      MYSQL_ROOT_PASSWORD: 123
      MYSQL_DATABASE: laravel
      MYSQL_USER: felipe
      MYSQL_PASSWORD: 123
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - laravel_net
    ports:
      - "3306:3306"

volumes:
  mysql_data:
    driver: local

networks:
  laravel_net:
    driver: bridge
```

```bash 
docker-compose up -d
``` 

```bash
docker-compose ps # Apenas para verificar se os containers estão funcionando
```

```bash
docker exec -it laravel_app bash
php artisan migrate
exit
```

```bash
docker exec -it mysql_db bash
mysql -u felipe -p
USE laravel;
SHOW TABLES;
exit
exit
```

```bash
docker-compose down
docker-compose up -d
```

```bash
docker exec -it mysql_db bash
mysql -u felipe -p
USE laravel;
SHOW TABLES;
# Verificar que os dados não foram perdidos
```

---

# 6. Criando e rodando um container multi-stage
Utilize um multi-stage build para otimizar uma aplicação Go, reduzindo o tamanho da imagem final.
Exemplo de aplicação: Compile e rode a API do Go Fiber Example dentro do container.

```bash
sudo apt install golang-go
```

```bash
mkdir go-fiber
cd go-fiber
```

```bash
go mod init go-fiber
go get github.com/gofiber/fiber/v2
```

```bash
nano main.go

package main
import "github.com/gofiber/fiber/v2"
func main() {
    app := fiber.New()

    app.Get("/", func(c *fiber.Ctx) error {
        return c.SendString("Olá, Felipe! Sua aplicação Go funcionou!")
    })

    app.Listen(":3000")
}
```

```bash
nano Dockerfile

FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod tidy
COPY . .
RUN go build -o app .
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/app .
EXPOSE 3000
CMD ["./app"]
```

```bash
docker build -t go-fiber .
docker run -p 3000:3000 go-fiber
```

Acessar página via http://localhost:3000/.

```bash
docker images # Verificar se o tamanho da imagem está menor
```

---

# 7. Construindo uma rede Docker para comunicação entre containers
Crie uma rede Docker personalizada e faça dois containers, um Node.js e um MongoDB, se comunicarem.
Exemplo de aplicação: Utilize o projeto MEAN Todos para criar um app de tarefas usando Node.js + MongoDB.
 
```bash
mkdir mean-todos
cd mean-todos
```

```bash
npm init -y
npm install express mongoose body-parser cors
```

```bash
nano server.js

const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');
const cors = require('cors');

const app = express();
const port = 3000;

app.use(cors());
app.use(bodyParser.json());

mongoose.connect('mongodb://mongo:27017/todos', {
  useNewUrlParser: true,
  useUnifiedTopology: true
}).then(() => {
  console.log('Conectado ao MongoDB');
}).catch((err) => {
  console.log('Erro ao conectar ao MongoDB:', err);
});

const Todo = mongoose.model('Todo', {
  task: String  
});

app.get('/todos', async (req, res) => {
  try {
    const todos = await Todo.find();
    res.json(todos);
  } catch (err) {
    res.status(500).json({ error: 'Erro ao buscar tarefas' });
  }
});

app.post('/todos', async (req, res) => {
  try {
    const newTodo = new Todo({
      task: req.body.tarefa    
    });
    await newTodo.save();
    res.status(201).json(newTodo);
  } catch (err) {
    res.status(500).json({ error: 'Erro ao criar tarefa' });
  }
});

app.delete('/todos/:id', async (req, res) => {
  try {
    const todoId = req.params.id;
    const deletedTodo = await Todo.findByIdAndDelete(todoId);
    if (deletedTodo) {
      res.json({ message: 'Tarefa removida com sucesso' });
    } else {
      res.status(404).json({ message: 'Tarefa não encontrada' });
    }
  } catch (err) {
    res.status(500).json({ error: 'Erro ao excluir tarefa' });
  }
});

app.listen(port, () => {
  console.log(`Boa, Felipe! Servidor rodando na porta ${port}`);
});
```

```bash
nano Dockerfile

FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

```bash
nano docker-compose.yml

version: '3.8'

services:  
  nodejs:
    build:
      context: .
    container_name: nodejs-container
    ports:
      - "3000:3000"
    networks:
      - mean-todos-rede
    depends_on:
      - mongo

  mongo:
    image: mongo:latest
    container_name: mongo-container
    volumes:
      - mongodb_data:/data/db
    networks:
      - mean-todos-rede

networks:
  mean-todos-rede:
    driver: bridge

volumes:
  mongodb_data:
    driver: local
```

```bash
docker-compose build
docker-compose up
```

Acessar via navegador http://localhost:3000/todos. Retornará [], indicando que o banco de dados está vazio.

Acessar o Thunder Client (ou outra ferramenta).

Fazer um POST usando o endpoint http://localhost:3000/todos:

{
 
    "tarefa": "Terminar exercício"

}

Acessar via navegador http://localhost:3000/todos. Retornará a tarefa criada.

Faça um GET e um DELETE usando o endpoint http://localhost:3000/todos/:id. 

---

# 8. Criando um compose file para rodar uma aplicação com banco de dados
Utilize Docker Compose para configurar uma aplicação Django com um banco de dados PostgreSQL.
Exemplo de aplicação: Use o projeto Django Polls App para criar uma pesquisa de opinião integrada ao banco.

```bash
sudo apt install python3.12-venv
```

```bash
python3 -m venv venv
source venv/bin/activate
```

```bash
pip install django
```

```bash
django-admin startproject meuprojeto
cd meuprojeto
```

```bash
python manage.py startapp polls
```

```bash
nano settings.py

INSTALLED_APPS = [
    ...
    'polls',
]
```

```bash
cd ..
pip install psycopg2-binary
```

```bash
nano settings.py

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
        'USER': 'felipe',
        'PASSWORD': '123',
        'HOST': 'db',
        'PORT': '5432',
    }
}
```
```bash
cd meuprojeto
```

```bash
nano docker-compose.yml

version: '3.8'

services:
  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: felipe
      POSTGRES_PASSWORD: 123
    networks:
      - mynetwork

  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - db
    networks:
      - mynetwork

volumes:
  postgres_data:

networks:
  mynetwork:
```

```bash
nano Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
```

```bash
nano requirements.txt
Django==3.2
psycopg2-binary==2.9.3
```
```bash
docker-compose up --build
```

Abra um novo terminal e navegue até o diretório **meuprojeto**.

```bash
docker-compose exec web python manage.py migrate
```

Acessar a página do Django via navegador http://localhost:8000.

```bash
docker-compose exec db psql -U felipe -d mydb
```

---

# 9. Criando uma imagem personalizada com um servidor web e arquivos estáticos
Construa uma imagem baseada no Nginx ou Apache, adicionando um site HTML/CSS estático.
Exemplo de aplicação: Utilize a landing page do Creative Tim para criar uma página moderna hospedada no container.

```bash
mkdir site-nginx
cd site-nginx
```

```bash
nano Dockerfile 

FROM nginx:alpine
RUN apk update && apk add git
RUN rm -rf /usr/share/nginx/html/*
RUN git clone --single-branch --depth 1 https://github.com/creativetimofficial/material-kit.git /usr/share/nginx/html
EXPOSE 80
```

```bash
docker build -t nginx-material-kit .
docker run -d -p 8080:80 nginx-material-kit
```

Acessar a landing page do Creative Tim via navegador http://localhost:8080.