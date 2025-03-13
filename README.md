# PB-Compass-Docker
Exercícios de Docker realizados no programa de bolsas de DevSecOps da Compass Uol. 

# Rodando um container básico
Execute um container usando a imagem do Nginx e acesse a página padrão no navegador.
Exemplo de aplicação: Use a landing page do TailwindCSS como site estático dentro do container.

1. Puxar a imagem oficial do Nginx
O primeiro passo é baixar a imagem oficial do Nginx a partir do Docker Hub:

```bash
docker pull nginx
```

2. Executar o contêiner do Nginx
Em seguida, você pode iniciar um contêiner Nginx, mapeando a porta 8080 da sua máquina para a porta 80 do contêiner:

```bash
docker run -d -p 8080:80 nginx
```
Isso fará com que você possa acessar a página padrão do Nginx via **http://localhost:8080** no seu navegador.

3. Criar o diretório e obter o template do Tailwind
Agora, vamos criar um diretório para armazenar o nosso arquivo HTML e baixar um template básico do Tailwind CSS.

```bash
mkdir tailwind
cd tailwind
curl -o index.html https://tailwindtoolbox.github.io/Landing-Page/
```

Isso cria o arquivo index.html no diretório tailwind com o conteúdo do template.

4. Criar o Dockerfile
Dentro do diretório tailwind, crie um arquivo chamado Dockerfile com o seguinte conteúdo:

```bash
Dockerfile

FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

Esse Dockerfile define uma imagem baseada no Nginx Alpine e copia o arquivo index.html para dentro do contêiner, substituindo a página padrão do Nginx.

5. Construir a imagem Docker personalizada
Agora, vamos construir a imagem Docker personalizada com o nome tailwind usando o Dockerfile:

```bash
docker build -t tailwind .
```

Esse comando cria a imagem Docker com o template do Tailwind.

6. Executar o contêiner com a imagem personalizada
Com a imagem tailwind construída, podemos iniciar um contêiner a partir dela, mapeando a porta 8081 da sua máquina para a porta 80 do contêiner:

```bash
docker run -d -p 8081:80 tailwind
```

Isso fará com que você possa acessar o seu site personalizado via **http://localhost:8081** no seu navegador.

7. Acessar o site personalizado
Ao acessar **http://localhost:8081**, você verá a landing page criada com o template do Tailwind CSS.


