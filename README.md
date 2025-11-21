# Aula 9 – Persistência de Dados NoSQL

## CRUD com MongoDB em Microserviços Java

---

## 🎯 Objetivo da Aula

Nesta aula, você aprenderá a implementar **operações CRUD** utilizando **MongoDB** em um microserviço Java com **Spring Boot 3**. A aula se baseia nos conceitos já aplicados em:

* **Aula 1 – CRUD com Spring Boot**
* **Aula 3 – Documentação de Microserviços (Swagger/OpenAPI)**

O objetivo é compreender como trabalhar com bancos **NoSQL**, modelagem de documentos e acesso a dados usando o **Spring Data MongoDB**.

Para isso utilizaremos o serviço de banco de dados em nuvem **MongoDB Atlas** e o estudo de caso da aula consiste em criar um projeto Spring Boot com MongoDB para um CRUD da tabela de produtos.<br>
Como mencionado na Aula 1, CRUD é um acrônimo que significa **Create, Read, Update e Delete**, que são as quatro operações básicas de manipulação de dados em um banco de dados.

---

## 🧩 O que veremos nesta aula

* Conceitos fundamentais do MongoDB
* Modelagem orientada a documentos
* Configuração do Spring Data MongoDB
* Implementação do CRUD completo
* Criação de DTOs (Record) e Mappers
* Documentação via Swagger/OpenAPI
* Boas práticas e estrutura de projeto

---
<br>

# Introdução ao MongoDB

## 📌 MongoDB é um banco NoSQL orientado a documentos

* Armazena dados em **documentos JSON** (BSON internamente)
* Estrutura flexível (schema-less)
* Ideal para aplicações modernas e escaláveis
* Coleções substituem tabelas e documentos substituem linhas

## 🔍 Vantagens

* Modelagem flexível
* Alta escalabilidade horizontal
* Performance em leituras e escritas
* Facilita evolução da estrutura dos dados

---
<br>

# 1️⃣ Configurando o Projeto


1. **Acesse o Spring Initializr**:

    - [Spring Initializr](https://start.spring.io)
    
        ```bash
            https://start.spring.io
        ```

2. **Configure o projeto**:

   * **Project**: Maven
   * **Language**: Java
   * **Spring Boot**: 3.5.8 (ou a versão mais recente)

3. **Metadados**:

   * **Group**: `br.com.casasbahia`
   * **Artifact**: `crud-mongodb`
   * **Name**: `crud-mongodb`
   * **Package name**: `br.com.casasbahia.crud_mongodb`

4. **Packaging e Java**:

   * **Packaging**: Jar
   * **Configuration File Format**: Properties
   * **Java**: 21

5. **Dependências**:

   * Spring Web
   * Spring Data MongoDB
   * Lombok

6. **Link do Projeto**

    Segue o link para acessar o Spring Initializr com as configurações utilizadas nesse projeto já preenchidas:

    * **Spring Initializr com as configurações já preenchidas**: [Spring Initializr](https://start.spring.io/#!type=maven-project&language=java&platformVersion=3.5.8&packaging=jar&configurationFileFormat=properties&jvmVersion=21&groupId=br.com.casasbahia&artifactId=crud-mongodb&name=crud-mongodb&description=Demo%20project%20for%20Spring%20Boot&packageName=br.com.casasbahia.crud-mongodb&dependencies=web,data-mongodb,lombok)


7. **Gerar o projeto**:

   * Clique em **Generate** e baixe o ZIP.

        <img src="images/spring-initializr.png" alt="Gerar Projeto">


8. **Importe no IntelliJ IDEA**:

   * **File → Open...** → selecione a pasta descompactada.

   * Aguarde o download de dependências.






## 📦 Dependência no `pom.xml`

* Adicionar no `pom.xml` a dependência para o Swagger/OpenAPI:

    ```xml
            <dependency>
                <groupId>org.springdoc</groupId>
                <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
                <version>2.5.0</version>
            </dependency>



    ```

    <img src="images/dependencia-swagger.png" alt="Dependência Swagger">


---
<br>

## 2️⃣ Configurações do MongoDB Atlas

Existem diversas formas de instalar e configurar o MongoDB, mas para facilitar o processo nesta aula utilizaremos o serviço de banco de dados em nuvem **MongoDB Atlas**.

1. **Acesse MongoDB Atlas Free Tier**:

    * [MongoDB Atlas](https://www.mongodb.com/products/platform/atlas-database)  

        <img src="images/mongodb-atlas1.png" alt="MongoDB Atlas">

    * Clique em **Get Started**.

2. Crie uma **conta** e um **cluster** gratuito.

    <img src="images/mongodb-atlas2.png" alt="MongoDB Atlas">

    * Nesse momento pode ser selecionado o Cloud Provider (AWS, GCP, Azure) e a região de preferência.
    * O plano Free Tier é suficiente para essa aula.


3. Configurando o **Banco de Dados e Usuário**:

   * Crie um banco de dados chamado `produtosdb`.
   * Crie uma coleção chamada `produtos`.

        <img src="images/mongodb-atlas3.png" alt="MongoDB Atlas">


4. Obtenha a string de conexão na opção **Get connection string**:

    <img src="images/mongodb-atlas4.png" alt="MongoDB Atlas">

   * Uma prática amplamente adotada é utilizar a string de conexão no formato:

    ```
        mongodb+srv://alonsodecarli_db_user:3YVmKaj8OzuEFJZb@cluster0.0o5yypd.mongodb.net/produtosdb?retryWrites=true&w=majority
    ```

**Observações**: 
* Guarde as credenciais fornecidas, você irá utilizá-las na configuração do Spring Boot.
* As informações disponibilizadas nessa sessão são apenas para fins didáticos para esta aula e serão diferentes para cada usuário, conforme retornos obtidos na criação e configuração do MongoDB Atlas.

---

## ⚙️ Configurações no `application.properties`

1. Usando a string de conexão obtida no MongoDB Atlas, configure o arquivo `src/main/resources/application.properties` da seguinte forma:

```properties
spring.application.name=crud-mongodb
server.port=8080

spring.data.mongodb.uri=mongodb+srv://alonsodecarli_db_user:3YVmKaj8OzuEFJZb@cluster0.0o5yypd.mongodb.net/produtosdb?retryWrites=true&w=majority

logging.level.org.springframework.data.mongodb.core.MongoTemplate=DEBUG
```

## 📚 Fortalecendo o Conhecimento
Vamos entender melhor cada propriedade adicionada no `application.properties`:
* `spring.application.name=crud-mongodb`: Define o nome da aplicação Spring Boot.
* `server.port=8080`: Configura a porta em que o servidor irá rodar.
* `spring.data.mongodb.uri`: Define a URI de conexão com o banco MongoDB Atlas.
  - `mongodb+srv://`: Indica o protocolo de conexão com um cluster MongoDB Atlas.
  - `alonsodecarli_db_user:3YVmKaj8OzuEFJZb@`: Especifica o usuário e a senha para autenticação no banco de dados.
  - `cluster0.0o5yypd.mongodb.net/`: Indica o endereço do cluster MongoDB Atlas.
  - `produtosdb`: Nome do banco de dados a ser utilizado.   
  - `?retryWrites=true&w=majority`: Parâmetros adicionais para a conexão, onde `retryWrites=true` habilita a reescrita automática em caso de falhas e `w=majority` garante que as escritas sejam confirmadas pela maioria dos nós do cluster.
* `logging.level.org.springframework.data.mongodb.core.MongoTemplate=DEBUG`: Habilita logs detalhados para operações do MongoTemplate, útil para depuração.


---
<br>

# 3️⃣ Modelagem da Entidade (Document)

* Criando a entidade `Produto` na pasta `model`:

    ```java
    package com.example.aula9.model;

    import org.springframework.data.annotation.Id;
    import org.springframework.data.mongodb.core.mapping.Document;
    import java.math.BigDecimal;

    @Document(collection = "produtos")
    public record Produto(
            @Id String id,
            String nome,
            String ncm,
            String descricaoNcm,
            BigDecimal preco,
            Integer quantidade
    ) {}
    ```

    <img src="images/crud-mongodb-model.png" alt="CRUD MongoDB Model">




## 📚 Fortalecendo o Conhecimento
    * `@Document(collection = "produtos")`: Indica que essa classe representa um documento na coleção `produtos` do MongoDB.
    * `@Id`: Marca o campo `id` como o identificador único do documento no MongoDB.



---
<br>

# 4️⃣ DTOs (Record) e Mapper

## DTO de entrada

* Criando o DTO `ProdutoRequest` na pasta `dto`:

    ```java
    public record ProdutoRequest(
            String nome,
            String ncm,
            String descricaoNcm,
            BigDecimal preco,
            Integer quantidade
    ) {}
    ```

    <img src="images/crud-mongodb-dtorequest.png" alt="CRUD MongoDB DTO Request">


## DTO de saída

* Criando o DTO `ProdutoResponse` na pasta `dto`:

    ```java
    public record ProdutoResponse(
            String id,
            String nome,
            String ncm,
            String descricaoNcm,
            BigDecimal preco,
            Integer quantidade
    ) {}
    ```

    <img src="images/crud-mongodb-dtoresponse.png" alt="CRUD MongoDB DTO Response">


## Mapper

* Criando a classe `ProdutoMapper` na pasta `mapper`:
    ```java
    @Component
    public class ProdutoMapper {

        public Produto toEntity(ProdutoRequest dto) {
            return new Produto(null,
                    dto.nome(),
                    dto.ncm(),
                    dto.descricaoNcm(),
                    dto.preco(),
                    dto.quantidade());
        }

        public ProdutoResponse toResponse(Produto entity) {
            return new ProdutoResponse(
                    entity.id(),
                    entity.nome(),
                    entity.ncm(),
                    entity.descricaoNcm(),
                    entity.preco(),
                    entity.quantidade());
        }
    }
    ```

    <img src="images/crud-mongodb-mapper.png" alt="CRUD MongoDB Mapper">


## 📚 Fortalecendo o Conhecimento
* **DTOs (Data Transfer Objects)**: São utilizados para transferir dados entre diferentes camadas da aplicação, como entre o controlador e o serviço. Eles ajudam a evitar expor diretamente as entidades do banco de dados.
* **Mapper**: Classe responsável por converter entre entidades e DTOs. Isso facilita a manutenção do código e a separação de responsabilidades.



---
<br>

# 5️⃣ Repositório com Spring Data

* Criando o repositório `ProdutoRepository` na pasta `repository`:

    ```java
    @Repository
    public interface ProdutoRepository extends MongoRepository<Produto, String> {
    }
    ```

    <img src="images/crud-mongodb-repository.png" alt="CRUD MongoDB Repository">


## 📚 Fortalecendo o Conhecimento
* **MongoRepository**: Interface do Spring Data MongoDB que fornece métodos CRUD prontos para uso, como `save`, `findAll`, `findById`, `deleteById`, entre outros.  



---
<br>

# 6️⃣ Serviço de Negócio

* Criando a classe `ProdutoService` na pasta `service`:

    ```java
    @Service
    @RequiredArgsConstructor
    public class ProdutoService {

        private final ProdutoRepository repository;

        public Produto criar(Produto produto) {
            return repository.save(produto);
        }

        public List<Produto> listar() {
            return repository.findAll();
        }

        public Produto buscarPorId(String id) {
            return repository.findById(id)
                    .orElseThrow(() -> new RuntimeException("Produto não encontrado"));
        }

        public Produto atualizar(String id, Produto produto) {
            buscarPorId(id);
            Produto atualizado = new Produto(
                    id,
                    produto.nome(),
                    produto.ncm(),
                    produto.descricaoNcm(),
                    produto.preco(),
                    produto.quantidade());

            return repository.save(atualizado);
        }

        public void deletar(String id) {
            buscarPorId(id);
            repository.deleteById(id);
        }
    }
    ```

    <img src="images/crud-mongodb-service.png" alt="CRUD MongoDB Service">


## 📚 Fortalecendo o Conhecimento
* **Serviço de Negócio**: Camada responsável por implementar a lógica de negócio da aplicação. Ela interage com o repositório para realizar operações de CRUD e pode incluir validações e regras específicas.



---
<br>

# 7️⃣ Controller REST 

* Criando a classe `ProdutoController` na pasta `controller`:

    ```java
    @RestController
    @RequestMapping("/api/produtos")
    @RequiredArgsConstructor
    public class ProdutoController {

        private final ProdutoService service;
        private final ProdutoMapper mapper;

        @PostMapping
        public ResponseEntity<ProdutoResponse> criar(@RequestBody ProdutoRequest request) {
            Produto criado = service.criar(mapper.toEntity(request));
            return ResponseEntity.status(201).body(mapper.toResponse(criado));
        }

        @GetMapping
        public List<ProdutoResponse> listar() {
            return service.listar().stream().map(mapper::toResponse).toList();
        }

        @GetMapping("/{id}")
        public ProdutoResponse buscar(@PathVariable String id) {
            return mapper.toResponse(service.buscarPorId(id));
        }

        @PutMapping("/{id}")
        public ProdutoResponse atualizar(@PathVariable String id,
                                        @RequestBody ProdutoRequest request) {
            return mapper.toResponse(service.atualizar(id, mapper.toEntity(request)));
        }

        @DeleteMapping("/{id}")
        public ResponseEntity<Void> deletar(@PathVariable String id) {
            service.deletar(id);
            return ResponseEntity.noContent().build();
        }
    }
    ```

    <img src="images/crud-mongodb-controller.png" alt="CRUD MongoDB Controller">

## 📚 Fortalecendo o Conhecimento    
* **Controller REST**: Camada responsável por expor os endpoints da API. Ela recebe as requisições HTTP, chama o serviço de negócio e retorna as respostas apropriadas.


---

<br>

# 8️⃣ Documentação com Swagger/OpenAPI

Como já implementado na **Aula 3**, basta acessar:

```
http://localhost:8080/swagger-ui/index.html
```

* Se desejar personalizar:

    ```java
    @Configuration
    public class SwaggerConfig {
        @Bean
        public OpenAPI customAPI() {
            return new OpenAPI()
                    .info(new Info()
                            .title("API de Produtos - MongoDB")
                            .version("1.0")
                            .description("CRUD usando Spring Boot + MongoDB"));
        }
    }
    ```



    <img src="images/swagger.png" alt="Swagger UI">






## Inserindo Dados de Teste

* Utilize o Swagger UI para inserir alguns dados de teste na coleção `produtos` via endpoint `POST /api/produtos`.


    <img src="images/swagger-post1.png" alt="Swagger UI">

* Segue o corpo JSON de exemplo:

    ```json
        {
        "nome": "Notebook Lenovo",
        "ncm": "12345678",
        "descricaoNcm": "Teste",
        "preco": 4500,
        "quantidade": 12
        }
    ```

* Ao **Executar** a requisição, você verá a resposta com o produto criado (retorno 201), incluindo o `id` gerado pelo MongoDB:

    ```json
        {
        "id": "69202db8443d525fa47c550b",
        "nome": "Notebook Lenovo",
        "ncm": "12345678",
        "descricaoNcm": "Teste",
        "preco": 4500,
        "quantidade": 12
        }

* O id gerado pode ser utilizado para testar os demais endpoints (GET, PUT, DELETE).




## Busca de um Produto
* Utilize o endpoint `GET /api/produtos/{id}` para buscar um produto pelo seu id.

    <img src="images/swagger-get1.png" alt="Swagger UI">


<br>


## Visualizando o Registro no MongoDB Atlas
O **Data Explorer** do MongoDB Atlas permite visualizar e gerenciar os dados armazenados no banco de dados de forma intuitiva.
* Acesse o MongoDB Atlas e visualize o documento inserido na coleção `produtos` do banco `produtosdb`:

    <img src="images/mongodb-atlas5.png" alt="MongoDB Atlas">




## 📚 Fortalecendo o Conhecimento
* O Swagger/OpenAPI facilita a documentação e testes dos endpoints da API.
* A interface interativa permite enviar requisições diretamente do navegador, tornando o processo de desenvolvimento mais ágil e eficiente.
* A documentação gerada automaticamente ajuda na comunicação entre equipes e na integração com outras aplicações.


<br>

---

# 9️⃣ Estrutura Final do Projeto

```
src/main/java
 └── com.example.aula9
      ├── controller
      ├── service
      ├── model
      ├── repository
      ├── dto
      └── mapper
```

---

# 🔟 Boas Práticas

* Utilizar **DTOs** para entrada/saída
* Evitar expor entidades diretamente
* Criar exceptions personalizadas
* Utilizar indexações no MongoDB quando necessário
* Padronizar respostas de erro (Aula 6)

---

# ✅ Conclusão

Nesta aula você aprendeu:

* Como configurar MongoDB em um microserviço Java
* Como criar um CRUD completo usando Spring Data MongoDB
* Como modelar documentos, criar DTOs e mapear entidades
* Como integrar tudo ao Swagger

👉 Agora você está pronto para trabalhar com bancos NoSQL em arquiteturas modernas de microserviços!

---
