# Rodar utilizando Docker (moodle-php-apache)

Partir dessa imagem: https://hub.docker.com/r/moodlehq/moodle-php-apache

Repositório: https://github.com/moodlehq/moodle-docker/blob/main/db.mysql.yml

docker-compose.yml

```python
version: "2"
services:
  moodledb:
    image: mysql:8.0
    command: >
                --character-set-server=utf8mb4
                --collation-server=utf8mb4_bin
                --skip-log-bin
    environment:
      MYSQL_ROOT_PASSWORD: "m@0dl3ing"
      MYSQL_USER: moodle
      MYSQL_PASSWORD: "m@0dl3ing"
      MYSQL_DATABASE: moodle
    volumes:
      - ./mysql_data/:/var/lib/mysql

  moodleapp:
    # apache com php e todas as estenções instaladas com sucesso
    image: moodlehq/moodle-php-apache:8.1
    container_name: moodleapp
    depends_on:
      - moodledb
    environment: 
      MOODLE_DOCKER_WEB_PORT: 8080
      MOODLE_DOCKER_DBNAME: moodle
      MOODLE_DOCKER_DBUSER: moodle
      MOODLE_DOCKER_DBPASS: "m@0dl3ing"
      MOODLE_DOCKER_BROWSER: firefox
      # MOODLE_DOCKER_WEB_HOST:  
      MOODLE_DOCKER_DBTYPE: mysqli
      MOODLE_DOCKER_DBCOLLATION: utf8mb4_bin # utf8mb4: É a codificação de caracteres UTF-8 que suporta até 4 bytes por caractere
    ports:
      - "8080:80"
    volumes:
      - ./moodle_code:/var/www/html
      - ./moodle_data:/var/www/moodledata
      - ./assets:/web/apache2_faildumps.conf:/etc/apache2/conf-enabled/apache2_faildumps.conf
    entrypoint: /bin/bash -c "chmod -R 777 /var/www/moodledata && docker-php-entrypoint apache2-foreground"

volumes:
  mysql_data:
  moodle_code:
  moodle_data:

```

vamos passar a pasta moodle atualizado para volume.

https://download.moodle.org/releases/latest/

**depois vamos configurar config.php**  

vou deixar um template config.docker-template.php 

Depois precisamos passar o config.php para pasta do moodle que está dentro do container. 

```python
<?php  // Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'mysqli';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'localhost';
$CFG->dbname    = 'moodle';
$CFG->dbuser    = 'moodle';
$CFG->dbpass    = 'm@0dl3ing';
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array (
  'dbpersist' => 0,
  'dbport' => '',
  'dbsocket' => '',
  'dbcollation' => 'utf8mb4_unicode_ci',
);

$CFG->wwwroot   = 'http://localhost:8080';
$CFG->dataroot  = '/var/www/moodledata';
$CFG->admin     = 'admin';

$CFG->directorypermissions = 0777;

require_once(__DIR__ . '/lib/setup.php');

// There is no php closing tag in this file,
// it is intentional because it prevents trailing whitespace problems!
```

Antes de rodar o docker compose vamos atualizar copiar o arquivo **config.php** dentro da pasta **moode_data**.

Depois vamos rodar **docker-compose.yml** 

Ele vai rodar normal, vai fazer toda configuração.

**.gitignore**

```python
/moodle_data
/moodle_code
/mysql_data
```

**MELHORIAS**

Por exemplo. as credenciais, chamamos em todos os lugares praticamente. E isso ta muito repeditivo nao é legal. 

vamos criar um arquivo .env 

```python
MOODLE_DOCKER_WWWROOT=./moodle_code
MOODLE_DOCKER_MOODLEDATA=./moodle_data
MOODLE_DOCKER_VOLDB=./mysql_data
 
MOODLE_DOCKER_PHP_VERSION=8.1
MOODLE_DOCKER_DB_VERSION=8.0 
 
MOODLE_DOCKER_WEB_PORT=8080
MOODLE_DOCKER_DB=mysql
MOODLE_DOCKER_DBNAME=moodle

ASSETDIR=./assets
```

docker-compose.yml

```python
version: "3"
services:

  moodledb: # Banco de dados
    image: mysql
    # image: "mysql:${MOODLE_DOCKER_DB_VERSION:-8.0}"
    command: >
                --character-set-server=utf8mb4
                --collation-server=utf8mb4_bin
                --skip-log-bin
    container_name: moodledb
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: moodle
      MYSQL_USER: moodleadmin
      MYSQL_PASSWORD: moodle123123@
    volumes:
      - "${MOODLE_DOCKER_VOLDB}:/var/lib/mysql"

  moodleapp: # Aplicação do Moodle
    # apache com php e todas as estenções instaladas com sucesso
    image: "moodlehq/moodle-php-apache:${MOODLE_DOCKER_PHP_VERSION}" 
    container_name: moodleapp
    depends_on:
      - moodledb
    # restart: always
    # env_file: ".env"
    environment:
      MOODLE_DOCKER_WEB_PORT: 8080
      MOODLE_DOCKER_DBNAME: moodle
      MOODLE_DOCKER_DBUSER: moodleadmin
      MOODLE_DOCKER_DBPASS: "moodle123123@"
      MOODLE_DOCKER_BROWSER: firefox
      # MOODLE_DOCKER_WEB_HOST:  
      MOODLE_DOCKER_DBTYPE: mysqli
      MOODLE_DOCKER_DBCOLLATION: utf8mb4_bin # utf8mb4: É a codificação de caracteres UTF-8 que suporta até 4 bytes por caractere
    ports:
      - "${MOODLE_DOCKER_WEB_PORT}:80"
    volumes:
      - "${MOODLE_DOCKER_WWWROOT}:/var/www/html"
      - "${MOODLE_DOCKER_MOODLEDATA}:/var/www/moodledata"
      - "${ASSETDIR}:/web/apache2_faildumps.conf:/etc/apache2/conf-enabled/apache2_faildumps.conf" 
    entrypoint: /bin/bash -c "chmod -R 777 /var/www/moodledata && docker-php-entrypoint apache2-foreground"
 
```

config.php

```python
<?php  // Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = getenv('MOODLE_DOCKER_DBTYPE');
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'moodledb';
$CFG->dbname    = getenv('MOODLE_DOCKER_DBNAME');
$CFG->dbuser    = getenv('MOODLE_DOCKER_DBUSER');
$CFG->dbpass    = getenv('MOODLE_DOCKER_DBPASS');
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array (
    'dbpersist' => 0,
    'dbport' => '',
    'dbsocket' => '',
    'dbcollation' => 'utf8mb4_unicode_ci',
);

$host = 'localhost';
if (!empty(getenv('MOODLE_DOCKER_WEB_HOST'))) {
    $host = getenv('MOODLE_DOCKER_WEB_HOST');
}
$CFG->wwwroot   = "http://{$host}";
$port = getenv('MOODLE_DOCKER_WEB_PORT');
if (!empty($port)) {
    // Extract port in case the format is bind_ip:port.
    $parts = explode(':', $port);
    $port = end($parts);
    if ((string)(int)$port === (string)$port) { // Only if it's int value.
        $CFG->wwwroot .= ":{$port}";
    }
}
$CFG->dataroot  = '/var/www/moodledata';
$CFG->admin     = 'admin';

$CFG->directorypermissions = 0777;

require_once(__DIR__ . '/lib/setup.php');

// There is no php closing tag in this file,
// it is intentional because it prevents trailing whitespace problems!
```