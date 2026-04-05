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

# 🤖 Guia de Uso - Seu Copiloto de Código

## 📌 Introdução

Este guia documenta como usar seu **copiloto de código personalizado** para aprender e desenvolver durante o **Bootcamp Backend Java Spring da DIO**. 

O copiloto foi calibrado especificamente para:
- Seguir 100% seu padrão de código
- Respeitar sua curva de aprendizado
- Nunca fazer mudanças não solicitadas
- Aprender novos temas junto com você

---

## 🚀 Como Usar Seu Copiloto

### **1️⃣ Começar a Estudar um Novo Tema**

Quando você quiser aprender sobre um novo conceito, framework ou padrão:

**Formato:**

Ei, vou começar a estudar sobre [TEMA]. Posso contar com você para me ajudar?


**Exemplo prático:**

Vou começar a estudar sobre Autenticação e Autorização com Spring Security. Posso contar com você para me ajudar?


**O copiloto vai:**
- ✅ Aprender o novo tema junto com você
- ✅ Criar exemplos usando SEU padrão de código
- ✅ Nunca adicionar features que você não pediu
- ✅ Explicar conceitos e melhores práticas
- ✅ Responder dúvidas conforme você avança

---

### **2️⃣ Pedir Implementação de Código**

Quando você quiser implementar uma feature ou criar novos endpoints:

**Formato:**

Crie um endpoint [DESCRIÇÃO] que [FUNCIONALIDADE]


**Exemplo prático:**

Crie um endpoint POST /users que registra um novo usuário com email validado


**O copiloto vai:**
- ✅ Seguir 100% seu padrão de código
- ✅ Criar Controller + Service + Repository + DTO + Tests
- ✅ Implementar validações e exception handling
- ✅ Usar a mesma estrutura que você usa no SchoolAPI
- ✅ Explicar cada passo implementado

**Exemplo de resposta:**
```java
// Controller
@RestController
@RequestMapping("/users")
public class UserController {
    @PostMapping
    public ResponseEntity<Object> createUser(@RequestBody UserDto dto) {
        // ... segue seu padrão
    }
}

// Service
@Service
public class UserService {
    @Transactional
    public void create(UserDto dto) {
        // ... lógica de negócio
    }
}

// Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}

// DTO (Record)
public record UserDto(Long id, String name, String email) {
    public User toEntity() { /* ... */ }
}

// Tests
@WebMvcTest(UserController.class)
class UserControllerTest {
    // ... testes completos
}

### **3️⃣ Pedir Revisão ou Refatoração**
Quando você quer que seu código seja revisado:

Formato:

Por favor, revise este código: [CÓDIGO]
Ele segue as boas práticas? O que poderia melhorar?

Exemplo prático:

Por favor, revise este código:

@PostMapping
public ResponseEntity<?> create(@RequestBody DepartmentDto dto) {
    try {
        Department dept = dto.toEntity();
        departmentService.create(dto);
        return ResponseEntity.ok("Criado com sucesso");
    } catch (Exception e) {
        return ResponseEntity.badRequest().body("Erro: " + e);
    }
}

Ele segue as boas práticas?

O copiloto vai:

✅ Identificar pontos de melhoria
✅ Sugerir refatorações mantendo seu estilo
✅ Explicar por que mudar
✅ Fornecer código refatorado pronto

## **4️⃣ Questionar sobre Conceitos**
Quando você quer entender melhor um conceito:

Formato:

Explique como funciona [CONCEITO] e como aplicar isso em [SITUAÇÃO]

Exemplo prático:

Explique como funciona a anotação @Transactional e como aplicar isso em um Service que salva múltiplos registros

O copiloto vai:

✅ Explicar o conceito de forma clara
✅ Fornecer exemplos práticos
✅ Mostrar casos de uso
✅ Ensinar boas práticas

📖 Convenções do Copiloto
Nunca será feito sem você pedir:

❌ Mudanças em arquivos do projeto
❌ Adição de novas dependências
❌ Refatorações ou melhorias não solicitadas
❌ Alteração de padrões estabelecidos
❌ Modificações no banco de dados
❌ Adicionar novos testes além do solicitado

Sempre será feito:

✅ Seguir seu padrão de código EXATAMENTE
✅ Respeitar sua curva de aprendizado
✅ Documentar e explicar decisões
✅ Sugerir boas práticas quando apropriado
✅ Criar testes junto com o código
✅ Manter consistência com seu estilo
✅ Avisar sobre possíveis problemas


