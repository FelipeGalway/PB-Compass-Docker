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
