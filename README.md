**Configurar Moodle no Ubuntu**

**INSTALAR OS PACOTES**

Suporte error: https://www.digitalocean.com/community/questions/upgrade-to-php8-1

Para instalar Apache, PHP 8, e MySQL no Ubuntu, siga os passos abaixo:

1. **Atualize os pacotes do sistema**:
    
    ```
    sudo apt update
    sudo apt upgrade  
    ```
    
2. **Instale o Apache**:
    
    ```
    sudo apt install apache2 
    ```
    
3. **Instale o PHP 8 e módulos necessários**:
    
    ```
    sudo apt install php8.1 libapache2-mod-php8.1 php8.1-mysql
    ```
    
4. **Instale o MySQL**:
    
    ```
    sudo apt install mysql-server
    ```
    
5. **Verifique se os serviços estão em execução**:
    
    ```
    sudo systemctl status apache2
    sudo systemctl status mysql
    
    ```
    
6. **Ajuste configurações de firewall (se necessário)**:
    
    ```
    sudo ufw allow in "Apache Full"
    ```
    
7. **Reinicie o Apache para carregar o PHP**:
    
    ```
    sudo systemctl restart apache2
    ```
    

Depois vamos testes php. 

Para testar um arquivo `index.php` no seu servidor Apache, siga estes passos:

1. **Crie o arquivo `index.php`**: 
    
    ```
    sudo nano /var/www/html/index.php
    ```
    
2. **Adicione o código PHP ao arquivo**:
    
    ```php
    <?php
    phpinfo();
    ?>
    ```
    
3. **Salve e feche o arquivo** (Ctrl + O, Enter, Ctrl + X).
4. **Verifique permissões** (opcional, caso necessário ajustar):
    
    ```
    sudo chown www-data:www-data /var/www/html/index.php
    sudo chmod 644 /var/www/html/index.php
    ```
    
5. **Abra o navegador e acesse**:
    
    ```arduino
    http://localhost/index.php
    
    ```
    

Fazer Download do moodle: 

**MySQL**

**Vamos acessar o mysql**

```python
sudo mysql
```

- **Defina a senha para o usuário root**:
    
    ```sql
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Le123123@';
    FLUSH PRIVILEGES;
    ```
    
- **Teste a nova senha**:
    
    ```
    mysql -u root -p
    ```
    

**BANCO DE DADOS**

1 - Criar o banco de dados. Onde está moodle é nome do seu database pode ser qualquer um.

```sql
CREATE DATABASE moodle CHARACTER SET = utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Para visualizar `show databases;`

2 - Vamos criar uma senha para esse banco de dados que acabamos de criar. Onde está *myuser* criar um usuario e *mypassword (com aspas simples)* colocar uma senha **poderosa**. (se for base de teste pode usar 123123 rsrs). 

Antes de criar o usuário pode ser os que já tem.

```sql
use mysql;
select user,host from user;
```

Para criar usuário.

```sql
CREATE USER 'moodleadmin'@'localhost' IDENTIFIED WITH mysql_native_password BY 'moodle123123@';
```

3 - Nessa etapa vamos aplicar as permissões de acesso para esse usuario que criamos. O que ele pode fazer no banco de dados. Nesse caso eu coloquei todos os direitos. 

```sql
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,CREATE TEMPORARY TABLES,DROP,INDEX,ALTER ON moodle.* TO 'moodleadmin'@'localhost';
ou
GRANT ALL PRIVILEGES ON moodle.* TO 'jedi'@'localhost' WITH GRANT OPTION;
```

Uma interface melhor para visualizar as tabelas e criar query.

**MySQL Workbench**

**phpMyAdmin**

**Adminer**

**MOODLE**

Depois de feito o download vamos extrair os arquivos e copiar a pasta para:  /var/www/html/

```python
sudo cp -r moodle leticia@172.28.52.44:/var/www/html/
```

Se precisar aplicar permissões

Suporte: https://docs.moodle.org/404/en/Installing_Moodle

```python
sudo chown -R www-data:www-data /var/www/html/moodle
sudo chmod -R 755 /var/www/html/moodle

sudo chmod 775 /var/www/html/
sudo chmod 777 /var/www/html/
```

Feito isso vamos acessar [localhost/moodle](http://localhost/moodle) e siga os passos abaixo.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a063a051-4fb5-4b47-ad10-54cee14f4f39/1e11c4d3-3dad-4732-9a0b-946c2b49cd76/Untitled.png)

- **Instalar as extensões**:
    
    ```bash
    sudo apt-get update
    sudo apt-get install php8.1-curl php8.1-zip
    
    ls /etc/php/8.1/mods-available/
    ```
    
- **Habilitar as extensões** (caso não sejam habilitadas automaticamente):
    
    ```bash
    sudo phpenmod curl
    sudo phpenmod zip
    
    ```