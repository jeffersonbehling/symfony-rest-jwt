# symfony-rest-jwt
Projeto criado para implementar Json Web Token para autenticar em uma API através de Token.

### Ambiente

- Ubuntu 16.04
- PHP 7.2
- MySQL 5.7
- Symfony 4.2.4
- Composer 1.8

### Dependências utilizadas

`composer require lexik/jwt-authentication-bundle`

`composer require maker`

`composer require api`

`composer require migrations`

### Servidor
Foi utilizado o servidor imbutido do PHP. Para isso, foi executado o comando abaixo.

`php -S 127.0.0.1:8000 -t public`

Se estiver tudo certo, você já poderá acessar a página de boas vindas do Symfony. Para isso abra o navegador e acesse a URL http://127.0.0.1:8000.

Para testar sua api, acesse a URL http://127.0.0.1:8000/api. Deverá aparecer uma tela da API PLATFORM.

### Criando as Entidades
Neste exemplo, será criado 3 entidades, sendo elas:

- Users
- Categories
- Authors
- Movies

Onde a tabela **Movies** terá relação com a tabela **Categories** e com a tabela **Authors**.

Utilize o comando `bin/console make:entity` para gerar as tabelas acima. O código das entidades estão logo abaixo.


```php
<?php
// TABLE USERS

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Table(name="users")
 * @ORM\Entity(repositoryClass="App\Repository\UserRepository")
 */
class User
{
    /**
     * @ORM\Id()
     * @ORM\GeneratedValue()
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(name="name", type="string", length=255)
     */
    private $name;

    /**
     * @ORM\Column(name="email", type="string", length=255)
     */
    private $email;

    /**
     * @ORM\Column(name="password", type="string", length=50)
     */
    private $password;

    /**
     * @ORM\Column(name="is_active", type="boolean", nullable=true, options={"default": true})
     */
    private $isActive;
    
    // Getters and Setters
```

```php
<?php
// TABLE CATEGORIES

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ApiResource()
 * @ORM\Table(name="categories")
 * @ORM\Entity(repositoryClass="App\Repository\CategoryRepository")
 */
class Category
{
    /**
     * @ORM\Id()
     * @ORM\GeneratedValue()
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(name="name", type="string", length=100)
     */
    private $name;

    /**
     * @ORM\Column(name="description", type="text", nullable=true)
     */
    private $description;
    
    // Getters and Setters
```

```php
<?php
// TABLE AUTHORS

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ApiResource()
 * @ORM\Table(name="authors")
 * @ORM\Entity(repositoryClass="App\Repository\AuthorsRepository")
 */
class Author
{
    /**
     * @ORM\Id()
     * @ORM\GeneratedValue()
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(name="name", type="string", length=255)
     */
    private $name;

    /**
     * @ORM\Column(name="birthdate", type="date")
     */
    private $birthdate;

    /**
     * @ORM\Column(name="email", type="string", length=255, unique=true)
     */
    private $email;
    
    // Getters and Setters
```

```php
<?php
// TABLE MOVIES

namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ApiResource()
 * @ORM\Table(name="movies")
 * @ORM\Entity(repositoryClass="App\Repository\MovieRepository")
 */
class Movie
{
    /**
     * @ORM\Id()
     * @ORM\GeneratedValue()
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(name="title", type="string", length=255)
     */
    private $title;

    /**
     * @ORM\Column(name="synopsis", type="text")
     */
    private $synopsis;

    /**
     * @ORM\Column(name="release_date", type="date")
     */
    private $releaseDate;

    /**
     * @ORM\ManyToOne(targetEntity="App\Entity\Category")
     * @ORM\JoinColumn(name="fk_category_id", nullable=false)
     */
    private $fkCategoryId;

    /**
     * @ORM\ManyToOne(targetEntity="App\Entity\Author")
     * @ORM\JoinColumn(name="fk_author_id", nullable=false)
     */
    private $fkAuthorId;
    
    // Getters and Setters
```

Após ter gerado as entidades, configure a conexão do banco de dados no arquivo `.env`.

```dotenv
# file .env
...
DATABASE_URL=mysql://db_user:db_password@127.0.0.1:3306/db_name
...

# UPDATE fields db_user, db_password and db_name
```

Após isso, execute o comando `bin/console doctrine:database:create` para criar sua base de dados. Feito isso vamos criar uma migrations para podermos "versionar" nossa base de dados de acordo com as Entidades que acabamos de criar.

Execute o comando `bin/console doctrine:migrations:diff` e depois o comando `bin/console doctrine:migrations:migrate` para criar as tabelas na base de dados.


### Json Web Token

Primeiramente, precisamos criar a chave privada e a chave pública para que o JWT consiga gerar e validar o token. A chave private é utilizada para CRIAR o token, enquanto a chave pública é utilizada para VALIDAR o token. Neste exemplo as chaves são criadas dentro do diretório `config/jwt`. No momento que for gerar as chaves, será solicitado uma **palavra-chave** que será utilizada para "encriptar" o token (não se esqueça da **palavra-chave**). Para gerar as chaves, execute os comandos abaixo.

```
$ mkdir -p config/jwt
$ openssl genrsa -out config/jwt/private.pem -aes256 4096
$ openssl rsa -pubout -in config/jwt/private.pem -out config/jwt/public.pem
``` 

Se você olhar no diretório `config/jwt`, irá perceber que tem dois arquivos lá, `public.pem` e `private.pem`. Agora precisamos informar ao Bundle LexikJWTAuthentication onde os arquivos se encontram no nosso projeto.
Atualize o arquivo `config/packages/lexik_jwt_authentication.yaml` para algo semelhante ao código abaixo.

```yaml
lexik_jwt_authentication:
    secret_key: '%kernel.project_dir%/config/jwt/private.pem'
    public_key: '%kernel.project_dir%/config/jwt/public.pem'
    pass_phrase: 'YOUR_PASSPHRASE' # palavra-chave utilizada na geração das chaves 
    token_ttl:  3600 # tempo de validade do token em segundos
```
