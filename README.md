# 🌱 Spring Catarina Learn - DDD e SOLID

## 🎯 Objetivo de Aprendizado
Projeto desenvolvido para estudar **Domain Driven Design (DDD)** e **princípios SOLID** em aplicações Spring Boot. Implementa API REST aplicando **Clean Architecture**, **design patterns** e **ferramentas de qualidade de código** para construção de software robusto e maintível.

## 🛠️ Tecnologias Utilizadas
- **Framework:** Spring Boot, Spring Data JPA
- **Arquitetura:** DDD, Clean Architecture
- **Princípios:** SOLID, Design Patterns
- **Banco de dados:** PostgreSQL
- **Qualidade:** SonarQube, SpotBugs
- **Testes:** JUnit, Mockito, TestContainers
- **Documentação:** Swagger/OpenAPI

## 🚀 Demonstração
```java
// Domain Entity seguindo DDD
@Entity
@Table(name = "produtos")
public class Produto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String nome;
    
    @Embedded
    private Preco preco;
    
    // Domain logic
    public void aplicarDesconto(BigDecimal percentual) {
        if (percentual.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Percentual deve ser positivo");
        }
        this.preco = this.preco.aplicarDesconto(percentual);
    }
}
```

## 📁 Estrutura DDD
```
spring-catarina-learn/
├── src/main/java/
│   ├── domain/                    # Camada de domínio
│   │   ├── entities/             # Entidades de negócio
│   │   ├── valueobjects/         # Value Objects
│   │   ├── repositories/         # Interfaces de repositório
│   │   └── services/             # Serviços de domínio
│   ├── application/              # Camada de aplicação
│   │   ├── usecases/            # Casos de uso
│   │   └── dto/                 # Data Transfer Objects
│   ├── infrastructure/           # Camada de infraestrutura
│   │   ├── persistence/         # Implementação JPA
│   │   ├── web/                 # Controllers REST
│   │   └── config/              # Configurações
│   └── shared/                   # Código compartilhado
└── src/test/                     # Testes automatizados
```

## 💡 Principais Aprendizados

### 🏗️ Domain Driven Design
- **Ubiquitous Language:** Linguagem comum entre negócio e desenvolvimento
- **Bounded Contexts:** Delimitação de contextos de domínio
- **Entities vs Value Objects:** Diferenciação de conceitos de domínio
- **Domain Services:** Lógica de negócio complexa
- **Repository Pattern:** Abstração de persistência

### 🔧 Princípios SOLID
- **Single Responsibility:** Uma responsabilidade por classe
- **Open/Closed:** Aberto para extensão, fechado para modificação
- **Liskov Substitution:** Substituição de tipos base
- **Interface Segregation:** Interfaces específicas e coesas
- **Dependency Inversion:** Dependência de abstrações

### 🛠️ Ferramentas de Qualidade
- **SonarQube:** Análise estática de código
- **SpotBugs:** Detecção de bugs potenciais
- **Checkstyle:** Padronização de código
- **JaCoCo:** Cobertura de testes
- **ArchUnit:** Testes de arquitetura

## 🧠 Conceitos Técnicos Estudados

### 1. **Value Objects Implementation**
```java
@Embeddable
public class Preco {
    @Column(name = "valor", precision = 10, scale = 2)
    private BigDecimal valor;
    
    @Column(name = "moeda", length = 3)
    private String moeda;
    
    protected Preco() {} // JPA
    
    public Preco(BigDecimal valor, String moeda) {
        if (valor.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Preço não pode ser negativo");
        }
        this.valor = valor;
        this.moeda = moeda;
    }
    
    public Preco aplicarDesconto(BigDecimal percentual) {
        BigDecimal desconto = valor.multiply(percentual).divide(BigDecimal.valueOf(100));
        return new Preco(valor.subtract(desconto), moeda);
    }
    
    // Immutable - sem setters
    public BigDecimal getValor() { return valor; }
    public String getMoeda() { return moeda; }
}
```

### 2. **Repository Pattern**
```java
// Domain interface
public interface ProdutoRepository {
    Optional<Produto> findById(Long id);
    List<Produto> findByCategoria(String categoria);
    Produto save(Produto produto);
    void delete(Long id);
}

// Infrastructure implementation
@Repository
public class JpaProdutoRepository implements ProdutoRepository {
    
    @Autowired
    private ProdutoJpaRepository jpaRepository;
    
    @Override
    public Optional<Produto> findById(Long id) {
        return jpaRepository.findById(id);
    }
    
    @Override
    public List<Produto> findByCategoria(String categoria) {
        return jpaRepository.findByCategoriaIgnoreCase(categoria);
    }
}
```

### 3. **Use Case Implementation**
```java
@Service
@Transactional
public class CriarProdutoUseCase {
    
    private final ProdutoRepository produtoRepository;
    private final ValidadorProduto validador;
    
    public CriarProdutoUseCase(ProdutoRepository produtoRepository, 
                              ValidadorProduto validador) {
        this.produtoRepository = produtoRepository;
        this.validador = validador;
    }
    
    public ProdutoDto executar(CriarProdutoCommand command) {
        // Validação de negócio
        validador.validar(command);
        
        // Criação da entidade
        Produto produto = new Produto(
            command.getNome(),
            new Preco(command.getValor(), command.getMoeda()),
            command.getCategoria()
        );
        
        // Persistência
        Produto produtoSalvo = produtoRepository.save(produto);
        
        // Conversão para DTO
        return ProdutoDto.from(produtoSalvo);
    }
}
```

## 🚧 Desafios Enfrentados
1. **Complexity management:** Balanceamento entre DDD e simplicidade
2. **Performance:** Otimização com múltiplas camadas
3. **Testing strategy:** Testes de unidade vs integração
4. **Team alignment:** Alinhamento com Ubiquitous Language
5. **Legacy integration:** Integração com sistemas existentes

## 📚 Recursos Utilizados
- [Domain-Driven Design - Eric Evans](https://www.domainlanguage.com/ddd/)
- [Clean Architecture - Robert Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Spring Boot Reference](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
- [SOLID Principles](https://www.baeldung.com/solid-principles)

## 📈 Próximos Passos
- [ ] Implementar Event Sourcing
- [ ] Adicionar CQRS pattern
- [ ] Criar Aggregate Roots complexos
- [ ] Implementar Domain Events
- [ ] Adicionar Hexagonal Architecture
- [ ] Criar testes de arquitetura com ArchUnit

## 🔗 Projetos Relacionados
- [Spring Blog Platform](../spring-blog-platform/) - Aplicação Spring completa
- [Spring Bookstore Management](../spring-bookstore-management/) - Gestão com JPA
- [Spring E-commerce TT](../spring-ecommerce-tt/) - E-commerce avançado

---

**Desenvolvido por:** Felipe Macedo  
**Contato:** contato.dev.macedo@gmail.com  
**GitHub:** [FelipeMacedo](https://github.com/felipemacedo1)  
**LinkedIn:** [felipemacedo1](https://linkedin.com/in/felipemacedo1)

> 💡 **Reflexão:** Este projeto foi fundamental para compreender a aplicação prática de DDD e SOLID em projetos reais. A experiência com ferramentas de qualidade demonstrou como a arquitetura bem estruturada impacta a manutenibilidade do código.