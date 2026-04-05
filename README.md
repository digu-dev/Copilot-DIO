# 🤖 Meu Copiloto de Código - Bootcamp DIO

## 📌 O que é este repositório?

Este repositório documenta minha jornada de aprendizado no **Bootcamp Backend Java Spring da DIO**, com o objetivo de consolidar meus conhecimentos e criar um **copiloto de IA personalizado** que me acompanhe durante todo o processo de desenvolvimento.

O copiloto foi calibrado para entender:
- Meu **padrão de código** atual
- Meu **nível de conhecimento** nas tecnologias
- Meu **estilo de desenvolvimento** e boas práticas
- Quando **estudar novos tópicos** junto comigo (sem fazer mudanças não solicitadas)

---

## 🛠️ Tech Stack Atual

### **Linguagens**
- Java (Principal)
- SQL

### **Frameworks & Bibliotecas**
- Spring Boot 3.4.x
- Spring MVC
- Spring Data JPA / JPA Hibernate
- Lombok
- Bean Validation
- SpringDoc OpenAPI (Swagger 3)

### **Banco de Dados**
- PostgreSQL
- MySQL

### **Ferramentas & Plataformas**
- Git & GitHub
- Maven
- Docker (Nível Básico)
- Swagger / OpenAPI 3
- Azure (Az-900 em andamento)

### **Conceitos & Padrões**
- ✅ API REST
- ✅ Programação Orientada a Objetos (POO)
- ✅ Arquitetura MVC
- ✅ SOLID
- ✅ Testes Unitários (JUnit & MockMvc)
- ✅ Tratamento de Exceções Global
- ✅ DTOs para proteção de entidades
- ✅ Queries otimizadas (JOIN FETCH para N+1)

---

## 📚 Meu Padrão de Desenvolvimento

### **Arquitetura em Camadas**

### **Controller → Service → Repository → Database ↓ DTOs**


### **Padrão de Controllers**
- REST endpoints com padrão CRUD (`POST`, `PUT`, `DELETE`, `GET`)
- Tratamento de exceções em cada método
- Retorno de `ResponseEntity` com status HTTP apropriados
- Uso de `Optional` para null-safety

### **Padrão de Services**
- Lógica de negócio isolada
- Uso de `@Transactional` para garantir integridade
- Chamadas ao Repository para persistência

### **Exemplo Real (Meu Código)**

**Controller:**
```java
@RestController
@RequestMapping("/departments")
public class DepartmentController {

    @Autowired
    DepartmentService departmentService;

    @PostMapping
    public ResponseEntity<Object> createDepartment(@RequestBody DepartmentDto dto) {
       try{
        Department createdDepartment = dto.toEntity();
        departmentService.create(dto);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
        .path("/{id}")
        .buildAndExpand(createdDepartment.getId()).toUri();
        return ResponseEntity.created(location).build();
       }catch(IllegalArgumentException e){
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(e.getMessage());
       }catch(ResourceAlreadyExistsException e){
        return ResponseEntity.status(HttpStatus.CONFLICT).body(e.getMessage());
       }
    }

    @PutMapping("/{id}")
    public ResponseEntity<Object> updateDepartment(@PathVariable("id") Long id, @RequestBody DepartmentDto dto) {
        try {
            Optional<Department> existingDepartment = departmentService.findById(id);
            if (existingDepartment.isEmpty()) {
                return ResponseEntity.notFound().build();
            }else{
                Department department = existingDepartment.get();
                department.setName(dto.name());
                departmentService.update(department);
                return ResponseEntity.noContent().build();
                }
                
            }catch (IllegalArgumentException e) {
                return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(e.getMessage());
            }catch (ResourceAlreadyExistsException e) {
                return ResponseEntity.status(HttpStatus.CONFLICT).body(e.getMessage());
         }
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Object> deleteDepartment(@PathVariable("id") Long id) {
        Optional<Department> existingDepartment = departmentService.findById(id);
        if (existingDepartment.isEmpty()) {
            return ResponseEntity.notFound().build();
        }
        departmentService.delete(existingDepartment.get());
        return ResponseEntity.noContent().build();
    }

    @GetMapping("/{id}")
    public ResponseEntity<DepartmentDto> findDepartmentById(@PathVariable("id") Long id) {

        Optional<Department> existingDepartment = departmentService.findById(id);
        if(existingDepartment.isPresent()){
            Department department = existingDepartment.get();
            DepartmentDto dto = new DepartmentDto(department.getId(), department.getName());
            return ResponseEntity.ok(dto);
        }else{
            return ResponseEntity.notFound().build();
        }
    }
    
    @GetMapping
    public ResponseEntity<List<Department>> findAllDepartments() {
        List<Department> departments = departmentService.findAll();
        if (departments.isEmpty()) {
            return ResponseEntity.notFound().build();
        }else{
        return ResponseEntity.ok(departments);
        }
    } 

    @GetMapping("/name")
    public ResponseEntity<DepartmentDto> findByName(@RequestParam("name") String name) {
        Optional<Department> existingDepartment = departmentService.findByName(name);
        if(existingDepartment.isPresent()){
            Department department = existingDepartment.get();
            DepartmentDto dto = new DepartmentDto(department.getId(), department.getName());
            return ResponseEntity.ok(dto);
        }else{
            return ResponseEntity.notFound().build();
        }
    }
}

```
@Service
public class DepartmentService {
    
    @Autowired
    private DepartmentRepository departmentRepository;
    
    @Transactional
    public void create(DepartmentDto dto) {
        // Lógica de negócio e validações
        departmentRepository.save(dto.toEntity());
    }

    @Transactional
    public void update(Department department) {
        departmentRepository.save(department);
    }

    public void delete(Department department) {
        departmentRepository.delete(department);
    }

    public Optional<Department> findById(Long id) {
        return departmentRepository.findById(id);
    }

    public List<Department> findAll() {
        return departmentRepository.findAll();
    }

    public Optional<Department> findByName(String name) {
        return departmentRepository.findByName(name);
    }
}

public record DepartmentDto(
    Long id,
    String name
) {
    public Department toEntity() {
        Department department = new Department();
        department.setId(this.id);
        department.setName(this.name);
        return department;
    }
}

@Repository
public interface DepartmentRepository extends JpaRepository<Department, Long> {
    Optional<Department> findByName(String name);
}

@RestControllerAdvice
public class ResourceExceptionHandler {

    @ExceptionHandler(RuntimeException.class) 
    public ResponseEntity<StandardError> entityNotFound(RuntimeException e, HttpServletRequest request) {
        HttpStatus status = HttpStatus.NOT_FOUND;
        StandardError err = new StandardError(
                Instant.now(),
                status.value(),
                "Resource not found",
                e.getMessage(),
                request.getRequestURI()
        );
        return ResponseEntity.status(status).body(err);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<StandardError> genericError(Exception e, HttpServletRequest request) {
        HttpStatus status = HttpStatus.INTERNAL_SERVER_ERROR;
        StandardError err = new StandardError(
                Instant.now(),
                status.value(),
                "Internal Server Error",
                "An unexpected error occurred, please try again later.",
                request.getRequestURI()
        );
        return ResponseEntity.status(status).body(err);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<StandardError> handleIllegalArgument(IllegalArgumentException e, HttpServletRequest request) {
        HttpStatus status = HttpStatus.BAD_REQUEST;
        StandardError err = new StandardError(
                Instant.now(),
                status.value(),
                "Bad Request",
                e.getMessage(),
                request.getRequestURI()
        );
        return ResponseEntity.status(status).body(err);
    }

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<StandardError> handleValidationException(ValidationException e, HttpServletRequest request) {
        HttpStatus status = HttpStatus.BAD_REQUEST;
        StandardError err = new StandardError(
                Instant.now(),
                status.value(),
                "Validation Error",
                e.getMessage(),
                request.getRequestURI()
        );
        return ResponseEntity.status(status).body(err);
    }

    @ExceptionHandler(ResourceAlreadyExistsException.class)
    public ResponseEntity<StandardError> handleResourceAlreadyExists(ResourceAlreadyExistsException e, HttpServletRequest request) {
        HttpStatus status = HttpStatus.CONFLICT;
        StandardError err = new StandardError(
                Instant.now(),
                status.value(),
                "Conflict",
                e.getMessage(),
                request.getRequestURI()
        );
        return ResponseEntity.status(status).body(err);
    }
}

public class StandardError {
    private Instant timestamp;
    private Integer status;
    private String error;
    private String message;
    private String path;

    public StandardError(Instant timestamp, Integer status, String error, String message, String path) {
        this.timestamp = timestamp;
        this.status = status;
        this.error = error;
        this.message = message;
        this.path = path;
    }

    // Getters e Setters
}

public class ResourceAlreadyExistsException extends RuntimeException {
    public ResourceAlreadyExistsException(String message) {
        super(message);
    }
}

@WebMvcTest(DepartmentController.class)
class DepartmentControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockitoBean
    private DepartmentService departmentService;

    private final ObjectMapper objectMapper = new ObjectMapper();

    @Test
    void createDepartment_shouldReturn201_whenValidDto() throws Exception {
        DepartmentDto dto = new DepartmentDto(null, "Engineering");
        Department saved = new Department();
        saved.setId(1L);
        saved.setName("Engineering");
        
        when(departmentService.create(any(DepartmentDto.class)))
            .thenReturn(saved);

        mockMvc.perform(post("/departments")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(dto)))
                .andExpect(status().isCreated());
    }

    @Test
    void updateDepartment_shouldReturn204_whenFound() throws Exception {
        DepartmentDto dto = new DepartmentDto(null, "Updated Name");
        Department existing = new Department();
        existing.setId(1L);
        existing.setName("Old Name");

        when(departmentService.findById(1L)).thenReturn(Optional.of(existing));
        doNothing().when(departmentService).update(any(Department.class));

        mockMvc.perform(put("/departments/1")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(dto)))
                .andExpect(status().isNoContent());
    }

    @Test
    void updateDepartment_shouldReturn404_whenNotFound() throws Exception {
        DepartmentDto dto = new DepartmentDto(null, "Updated Name");

        when(departmentService.findById(99L)).thenReturn(Optional.empty());

        mockMvc.perform(put("/departments/99")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(dto)))
                .andExpect(status().isNotFound());
    }

    @Test
    void deleteDepartment_shouldReturn204_whenFound() throws Exception {
        Department existing = new Department();
        existing.setId(1L);
        existing.setName("Engineering");

        when(departmentService.findById(1L)).thenReturn(Optional.of(existing));
        doNothing().when(departmentService).delete(any(Department.class));

        mockMvc.perform(delete("/departments/1"))
                .andExpect(status().isNoContent());
    }

    @Test
    void deleteDepartment_shouldReturn404_whenNotFound() throws Exception {
        when(departmentService.findById(99L)).thenReturn(Optional.empty());

        mockMvc.perform(delete("/departments/99"))
                .andExpect(status().isNotFound());
    }

    @Test
    void getDepartmentById_shouldReturn200WithDto_whenFound() throws Exception {
        Department department = new Department();
        department.setId(1L);
        department.setName("Engineering");

        when(departmentService.findById(1L)).thenReturn(Optional.of(department));

        mockMvc.perform(get("/departments/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1))
                .andExpect(jsonPath("$.name").value("Engineering"));
    }

    @Test
    void getDepartmentById_shouldReturn404_whenNotFound() throws Exception {
        when(departmentService.findById(99L)).thenReturn(Optional.empty());

        mockMvc.perform(get("/departments/99"))
                .andExpect(status().isNotFound());
    }

    @Test
    void getAllDepartments_shouldReturn200WithList_whenNotEmpty() throws Exception {
        Department dept1 = new Department();
        dept1.setId(1L);
        dept1.setName("Engineering");

        Department dept2 = new Department();
        dept2.setId(2L);
        dept2.setName("Sciences");

        when(departmentService.findAll()).thenReturn(Arrays.asList(dept1, dept2));

        mockMvc.perform(get("/departments"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$", hasSize(2)))
                .andExpect(jsonPath("$[0].name").value("Engineering"))
                .andExpect(jsonPath("$[1].name").value("Sciences"));
    }

    @Test
    void getAllDepartments_shouldReturn404_whenEmpty() throws Exception {
        when(departmentService.findAll()).thenReturn(Collections.emptyList());

        mockMvc.perform(get("/departments"))
                .andExpect(status().isNotFound());
    }

    @Test
    void getDepartmentByName_shouldReturn200WithDto_whenFound() throws Exception {
        Department department = new Department();
        department.setId(1L);
        department.setName("Engineering");

        when(departmentService.findByName("Engineering")).thenReturn(Optional.of(department));

        mockMvc.perform(get("/departments/name?name=Engineering"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1))
                .andExpect(jsonPath("$.name").value("Engineering"));
    }

    @Test
    void getDepartmentByName_shouldReturn404_whenNotFound() throws Exception {
        when(departmentService.findByName("NonExistent")).thenReturn(Optional.empty());

        mockMvc.perform(get("/departments/name?name=NonExistent"))
                .andExpect(status().isNotFound());
    }
}

🚀 Como Usar Seu Copiloto (Este Repositório)
1. Começar a Estudar um Novo Tema
Quando você quiser aprender sobre um novo conceito, framework ou padrão:

Code
Ei, vou começar a estudar sobre [TEMA].
Posso contar com você para me ajudar?
Exemplo:

Code
Vou começar a estudar sobre Autenticação e Autorização com Spring Security.
Posso contar com você para me ajudar?
O copiloto vai:

✅ Aprender o novo tema junto com você
✅ Criar exemplos usando SEU padrão de código
✅ Nunca adicionar features que você não pediu
✅ Explicar conceitos e melhores práticas
2. Pedir Implementação de Código
Quando você quiser implementar uma feature:

Code
Crie um endpoint [DESCRIÇÃO] que [FUNCIONALIDADE]
Exemplo:

Code
Crie um endpoint POST /users que registra um novo usuário com email validado
O copiloto vai:

✅ Seguir 100% seu padrão de código
✅ Criar Controller + Service + Repository + DTO + Tests
✅ Implementar validações e exception handling
✅ Explicar cada passo
3. Pedir Revisão ou Refatoração
Code
Por favor, revise este código: [CÓDIGO]
Ele segue as boas práticas? O que poderia melhorar?
4. Questionar sobre Conceitos
Code
Explique como funciona [CONCEITO] e como aplicar isso em [SITUAÇÃO]
📖 Convenções do Copiloto
Nunca será feito sem você pedir:
❌ Mudanças em arquivos do projeto
❌ Adição de novas dependências
❌ Refatorações ou melhorias não solicitadas
❌ Alteração de padrões estabelecidos
Sempre será feito:
✅ Seguir seu padrão de código EXATAMENTE
✅ Respeitar sua curva de aprendizado
✅ Documentar e explicar decisões
✅ Sugerir boas práticas quando apropriado
✅ Criar testes junto com o código
✅ Manter consistência com seu estilo
🎯 Referência Rápida: Seu Padrão
Aspecto	Padrão	Exemplo
URL Base	/resource-name	/departments, /teachers
POST	Criar novo	POST /departments → 201 Created
PUT	Atualizar	PUT /departments/{id} → 204 No Content
DELETE	Deletar	DELETE /departments/{id} → 204 No Content
GET by ID	Buscar um	GET /departments/{id} → 200 OK com DTO
GET All	Listar todos	GET /departments → 200 OK com List
GET Query	Busca param	GET /departments/name?name=... → 200 OK
Exception	Centralizado	@RestControllerAdvice com StandardError
DTO	Sempre	Nunca expor entidades diretamente
Service	@Transactional	Lógica isolada, com validações
Tests	MockMvc	@WebMvcTest com Mockito
HTTP Status	Apropriado	201 Created, 204 No Content, 404 Not Found, 409 Conflict
📊 Estrutura do Projeto Típico
Code
src/main/java/
├── controller/
│   ├── DepartmentController.java
│   ├── TeacherController.java
│   ├── StudentController.java
│   └── ...
├── service/
│   ├── DepartmentService.java
│   ├── TeacherService.java
│   ├── StudentService.java
│   └── ...
├── repository/
│   ├── DepartmentRepository.java
│   ├── TeacherRepository.java
│   ├── StudentRepository.java
│   └── ...
├── entity/
│   ├── Department.java
│   ├── Teacher.java
│   ├── Student.java
│   └── ...
├── dto/
│   ├── DepartmentDto.java
│   ├── TeacherDto.java
│   ├── StudentDto.java
│   └── ...
└── exception/
    ├── ResourceExceptionHandler.java
    ├── ResourceAlreadyExistsException.java
    ├── StandardError.java
    └── ...

src/test/java/
└── tests/controller/
    ├── DepartmentControllerTest.java
    ├── TeacherControllerTest.java
    ├── StudentControllerTest.java
    └── ...

resources/
├── application.properties (ou application.yml)
└── templates/ (se necessário)

pom.xml
docker-compose.yml
README.md
.gitignore
📋 Dependências Maven Padrão
XML
<dependencies>
    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webmvc</artifactId>
    </dependency>

    <!-- Spring Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Bean Validation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <!-- PostgreSQL Driver -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <!-- Swagger/OpenAPI -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>3.0.2</version>
    </dependency>

    <!-- Dev Tools -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>

    <!-- Testing -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
🐳 Docker Compose Padrão
YAML
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: postgres_db
    environment:
      POSTGRES_DB: schoolapi_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5431:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app_network

volumes:
  postgres_data:

networks:
  app_network:
    driver: bridge
⚙️ Application Properties Padrão
properties
# Server
server.port=8080
server.servlet.context-path=/api

# Database
spring.datasource.url=jdbc:postgresql://localhost:5431/schoolapi_db
spring.datasource.username=postgres
spring.datasource.password=postgres
spring.datasource.driver-class-name=org.postgresql.Driver

# JPA/Hibernate
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQL10Dialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=true

# Logging
logging.level.root=INFO
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate.SQL=DEBUG

# Swagger
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.api-docs.path=/v3/api-docs
📝 Sobre Você
Rodrigo

🎓 Estudante de Sistemas de Informação (3º Período)
🧬 Graduado em Ciências Biológicas com Pós-graduação em Neurociência
🎯 Foco em Backend (Java/Spring) e DevOps
📜 Certificado Java (Basic) e em preparação para Azure Fundamentals (AZ-900)
🚀 Participando do Bootcamp Backend Java Spring - DIO
🔗 Referências Úteis
Spring Boot Documentation
Spring Data JPA Guide
REST API Best Practices
SOLID Principles
JWT Authentication
Docker for Developers
PostgreSQL Documentation
Maven Guide
JUnit 5 Documentation
Mockito Documentation
📌 Próximas Etapas
✅ Copiloto calibrado e pronto
📚 Começar os módulos do bootcamp
🏗️ Implementar projetos práticos
🧪 Aprender novos padrões e frameworks
🚀 Deployar na nuvem (Azure)
Estou pronto para aprender junto com você! 🚀

Quando quiser estudar um novo tema ou implementar uma feature, é só chamar!

📌 Resumo de Como Usar
Para começar a estudar algo novo:

Code
Vou começar a estudar sobre [TEMA]. Posso contar com você?
Para pedir uma feature:

Code
Crie um endpoint [DESCRIÇÃO] que [FUNCIONALIDADE]
Para questionar conceitos:

Code
Explique como funciona [CONCEITO]
Para revisar código:

Code
Revise este código: [CÓDIGO]. Segue as boas práticas?
Última atualização: 05 de abril de 2026 ✨

Code

Pronto! 📄 Arquivo único `.md` completo com tudo que você precisava. Você pode copiar isso e criar um novo repositório no GitHub para o bootcamp da DIO. 

Quer que eu faça ajustes em algo? 🎯
