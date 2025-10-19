# Rodar utilizando Docker (bitnami)

- **Docker e Docker Compose**
    
    **Docker Compose** é uma ferramenta que permite definir e executar aplicativos **Docker multicontêiner em um único arquivo**, facilitando a orquestração de vários contêineres que trabalham em conjunto.
    
    **Exemplo:**
    
    Suponha que seu aplicativo web dependa de um banco de dados e um servidor web. Com o Docker Compose, você pode definir esses serviços em um arquivo chamado **`docker-compose.yml`**.
    
    ```yaml
    # Exemplo de docker-compose.yml
    version: '3.9'
    services:
      web: # Meu app
        image: nginx:latest
        ports:
          - "8080:80"
      database: # Banco de dados
        image: postgres:latest
        environment:
          POSTGRES_PASSWORD: mysecretpassword
    
    ```
    
    Este arquivo descreve dois serviços: um contêiner com o servidor web Nginx e outro com um banco de dados PostgreSQL. O Docker Compose simplifica a execução desses serviços juntos.
    
    ```bash
    # Usando Docker Compose para iniciar os serviços
    docker-compose up
    ```
    
    Isso inicia ambos os contêineres de acordo com a configuração definida no arquivo **`docker-compose.yml`**.
    
    1. **Docker:**
        - O Docker é uma plataforma que permite criar, distribuir e executar aplicativos em contêineres.
        - Contêineres são ambientes leves e isolados que encapsulam um aplicativo e suas dependências, garantindo consistência em diferentes ambientes.
        - Docker pode referir-se tanto à plataforma como um todo quanto ao software que você instala em seu sistema operacional para criar e gerenciar contêineres.
    2. **Docker Compose:**
        - Docker Compose é uma ferramenta que permite definir e gerenciar aplicativos Docker multicontêiner usando um arquivo de configuração YAML.
        - Ele simplifica a orquestração de vários contêineres que precisam trabalhar em conjunto.
        - O arquivo **`docker-compose.yml`** define serviços, redes e volumes, permitindo que você especifique toda a configuração de seu aplicativo em um único arquivo.
    
    Resumindo, o Docker refere-se à plataforma de contêineres e ao software para criar e gerenciar contêineres. O Docker Compose é uma ferramenta para orquestrar vários contêineres, permitindo definir a configuração de um aplicativo multicontêiner em um único arquivo. A imagem Docker, por sua vez, é um artefato que contém um sistema de arquivos com o aplicativo e suas dependências, e pode ser usado para criar contêineres.
    
- **Instalação do Docker e Docker Compose**
    
    Instalação do Docker para windows:
    
    https://docs.docker.com/desktop/install/windows-install/
    
    Acessa esse link e faça o download e instala normalmente. Esse Docker para windows contem todos os serviços do Docker e Docker compose que precisamos. 
    
    No Linux:
    
    https://docs.docker.com/engine/install/ubuntu/
    
    ```python
    sudo apt update
    sudo apt install apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    
    sudo apt update
    sudo apt install docker-ce docker-ce-cli containerd.io
    sudo docker --version
    
    sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    docker-compose --version
    ```
    

Podemos utilizar uma imagem pronta do bitnami

https://hub.docker.com/r/bitnami/moodle

https://hub.docker.com/_/mysql

```python
version: "3"
services:
  moodledb:
    image: mysql
    container_name: moodledb
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: moodle
      MYSQL_USER: moodleadmin
      MYSQL_PASSWORD: moodle123123@
    volumes:
      - ./mysql_data/:/var/lib/mysql
  moodleapp:
    image: bitnami/moodle
    container_name: moodleapp
    depends_on:
      - moodledb
    environment:
      ALLOW_EMPTY_PASSWORD: yes
      MOODLE_DATABASE_TYPE: mysqli
      MOODLE_DATABASE_HOST: moodledb
      MOODLE_DATABASE_NAME: moodle
      MOODLE_DATABASE_USER: moodleadmin
      MOODLE_DATABASE_PASSWORD: moodle123123@
      MOODLE_SITE_NAME: OpenCodigo Academy
      MOODLE_LANG: pt_br
    ports:
      - "8089:8089"
    volumes:
      - ./moodle_data:/bitnami/moodle
      - ./moodledata_data:/bitnami/moodledata 
```

1. **Version**: Especifica a versão do Compose, aqui usamos a versão 3.
2. **Services**: Define os serviços que serão executados.
3. **moodledb (serviço de banco de dados)**:
    - **image**: Usamos a imagem oficial do MySQL.
    - **container_name**: Nomeamos o container como `moodledb`.
    - **environment**: Configura variáveis de ambiente para o MySQL, incluindo senhas e nome do banco de dados.
    - **volumes**: Mapeamos um volume local `./mysql_data/` para o diretório de dados do MySQL no container `/var/lib/mysql`.
4. **moodleapp (serviço Moodle)**:
    - **image**: Usamos a imagem do Bitnami para o Moodle.
    - **container_name**: Nomeamos o container como `moodleapp`.
    - **depends_on**: Especifica que o `moodleapp` depende do `moodledb` estar em execução primeiro.
    - **environment**: Configura variáveis de ambiente para o Moodle, incluindo detalhes do banco de dados e configurações do site.
    - **ports**: Mapeamos a porta `8089` do host para a `8089` do container.
    - **volumes**: Mapeamos diretórios locais para os diretórios do Moodle no container.

### Passos para rodar o arquivo:

1. **Instalar Docker e Docker Compose**: Certifique-se de que Docker e Docker Compose estão instalados na máquina.
2. **Criar o arquivo docker-compose.yml**: Copie e cole a configuração no arquivo `docker-compose.yml`.