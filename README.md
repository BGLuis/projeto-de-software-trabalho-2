# 1. Introdução

## Visão Geral do Sistema

**Gatuno** é uma plataforma web moderna para leitura, gerenciamento e catalogação de mangás e livros digitais. O sistema oferece uma experiência completa desde a descoberta de conteúdo até a leitura personalizada, integrando funcionalidades de scraping automatizado, organização em coleções e controle detalhado de conteúdo sensível.

## Objetivos do Projeto

O desenvolvimento do sistema Gatuno visa atender aos seguintes objetivos:

1. **Centralização de Conteúdo**: Agregar mangás de múltiplas fontes web em um único catálogo unificado
2. **Experiência de Leitura Aprimorada**: Prover leitor responsivo com navegação intuitiva e histórico de leitura
3. **Automação de Atualização**: Implementar sistema de scraping para manter conteúdo atualizado automaticamente
4. **Personalização**: Permitir criação de coleções personalizadas e configurações individuais de sensibilidade de conteúdo
5. **Gerenciamento Eficiente**: Oferecer ferramentas administrativas robustas para curadoria de metadados e manutenção do catálogo
6. **Escalabilidade**: Arquitetura preparada para crescimento de usuários e volume de dados

## Escopo do Sistema

### Funcionalidades Implementadas

**Para Usuários Anônimos:**

-   Navegação e busca no catálogo público de livros
-   Visualização de detalhes de livros (sinopse, autor, tags, capítulos)
-   Leitura de capítulos disponíveis publicamente
-   Criação de nova conta no sistema

**Para Usuários Autenticados:**

-   Todas as funcionalidades de usuários anônimos
-   Criação e gerenciamento de coleções personalizadas
-   Marcação de capítulos como lidos (histórico de leitura)
-   Atualização de perfil pessoal
-   Configuração de sensibilidade de conteúdo (filtro de tags sensíveis)
-   Gerenciamento de sessões e autenticação JWT

**Para Administradores:**

-   Todas as funcionalidades de usuários autenticados
-   CRUD completo de livros, capítulos e páginas
-   Upload de capas e páginas (imagens)
-   Configuração de websites para scraping automatizado
-   Execução de jobs de scraping de capítulos
-   Gerenciamento de metadados (tags, autores, categorias de conteúdo sensível)
-   Operações de mescla (merge) de metadados duplicados
-   Monitoramento de status de processamento
-   Dashboards administrativos e ferramentas de manutenção

### Limitações e Exclusões

-   Não implementa sistema de comentários ou reviews
-   Não possui funcionalidades sociais (seguir usuários, compartilhamento)
-   Scraping requer configuração manual de websites (não autodescoberta)
-   Sistema de notificações não implementado na versão atual
-   Sem suporte para formatos de arquivo (somente imagens de páginas)

## Tecnologias Utilizadas

### Backend

-   **Framework**: NestJS (Node.js)
-   **Linguagem**: TypeScript
-   **ORM**: TypeORM
-   **Autenticação**: JWT (JSON Web Tokens)
-   **Validação**: class-validator, class-transformer

### Frontend

-   **Framework**: Angular 18+
-   **Linguagem**: TypeScript
-   **State Management**: RxJS
-   **Renderização**: SSR (Server-Side Rendering)

### Banco de Dados

-   **SGBD**: MySQL 8.0
-   **Arquitetura**: Master-Slave Replication (1 master + 2 slaves)
-   **Cache**: Redis 8.0

### Infraestrutura

-   **Containerização**: Docker, Docker Compose
-   **Reverse Proxy**: Traefik (produção)
-   **Scraping**: Selenium Grid (Hub + Chrome Nodes)
-   **Monitoramento**: Prometheus, Grafana, AlertManager

### Padrões Arquiteturais

-   Repository Pattern
-   Service Layer Pattern
-   Dependency Injection
-   Cache-Aside Pattern
-   Master-Slave Replication Pattern
-   Soft Delete Pattern

## Estrutura do Documento

Este documento está organizado em 4 seções principais:

-   **Seção 1 - Introdução**: Visão geral, objetivos, escopo e tecnologias
-   **Seção 2 - Modelos de Usuário e Requisitos**: Atores, casos de uso, diagramas de sequência do sistema e contratos de operação
-   **Seção 3 - Modelos de Projeto**: Arquitetura, diagramas de componentes, classes, sequência, comunicação e estados
-   **Seção 4 - Modelos de Dados**: Esquemas de banco de dados e estratégias de mapeamento objeto-relacional

# 2. Modelos de Usuário e Requisitos

Esta seção apresenta os modelos relacionados aos usuários do sistema e aos requisitos funcionais, incluindo a descrição de atores, casos de uso e seus respectivos diagramas comportamentais.

## 2.1 Descrição de Atores

O sistema Gatuno possui três tipos principais de atores, cada um com níveis crescentes de privilégios e capacidades:

### Usuário Anônimo

Visitante não autenticado que acessa a plataforma. Este ator tem acesso limitado apenas às funcionalidades públicas de consulta e visualização de conteúdo.

**Responsabilidades:**

-   Navegar pelo catálogo de livros
-   Visualizar informações detalhadas de livros e capítulos
-   Ler conteúdo disponível publicamente
-   Consultar metadados (tags, autores, categorias de conteúdo sensível)
-   Criar uma nova conta no sistema

### Usuário Autenticado

Usuário registrado que realizou login no sistema. Este ator possui acesso a funcionalidades personalizadas e pode interagir de forma mais profunda com a plataforma.

**Responsabilidades:**

-   Todas as responsabilidades do Usuário Anônimo
-   Gerenciar seu perfil pessoal
-   Criar e organizar coleções personalizadas de livros
-   Marcar capítulos como lidos e manter histórico de leitura
-   Gerenciar sessões de autenticação (refresh token, logout)
-   Visualizar conteúdo de acordo com suas preferências de sensibilidade

### Administrador

Usuário com privilégios administrativos elevados, responsável pela gestão completa do conteúdo da plataforma. Este ator é identificado através de uma role específica (ADMIN).

**Responsabilidades:**

-   Todas as responsabilidades do Usuário Autenticado
-   Gerenciar ciclo de vida completo dos livros (criar, atualizar, excluir)
-   Administrar capítulos, páginas e capas dos livros
-   Fazer upload de conteúdo (imagens de capas e páginas)
-   Curar e mesclar metadados duplicados (tags, autores, conteúdo sensível)
-   Configurar e gerenciar websites para scraping automatizado
-   Monitorar status de processamento de livros
-   Realizar operações de manutenção (verificação, correção, reset de cache)

## 2.2 Modelo de Casos de Uso

O sistema implementa 58 casos de uso organizados em 12 pacotes funcionais:

### Pacotes de Casos de Uso

1. **Autenticação** (UC-01 a UC-04): Registro, login, refresh token, logout
2. **Perfil de Usuário** (UC-05 a UC-06): Visualização e atualização de perfil
3. **Catálogo de Livros** (UC-07 a UC-13): Busca, listagem, visualização de livros
4. **Leitura e Histórico** (UC-14): Marcação de capítulos lidos
5. **Coleções** (UC-15 a UC-21): CRUD de coleções personalizadas
6. **Administração de Livros** (UC-22 a UC-32): Gestão completa de livros
7. **Administração de Capítulos** (UC-33 a UC-40): Gestão de capítulos e páginas
8. **Administração de Capas** (UC-41 a UC-46): Upload e gerenciamento de capas
9. **Scraping** (UC-47 a UC-49): Configuração e execução de scraping automatizado
10. **Administração de Metadados - Tags** (UC-50 a UC-52): CRUD e merge de tags
11. **Administração de Metadados - Autores** (UC-53 a UC-55): CRUD e merge de autores
12. **Administração de Metadados - Conteúdo Sensível** (UC-56 a UC-58): Gestão de categorias de conteúdo sensível

**Diagrama de Casos de Uso:**

![Diagrama de Casos de Uso](./image/use-case/Diagrama%20de%20Casos%20de%20Uso.svg)

## 2.3 Diagrama de Sequência do Sistema

Os diagramas de sequência modelam a interação temporal entre atores e o sistema para os principais casos de uso, demonstrando o fluxo de mensagens e a ordem de execução das operações.

### Diagramas Implementados

O sistema possui 6 diagramas de sequência cobrindo 22 casos de uso distribuídos em áreas funcionais:

1. **Autenticação** (UC-01 a UC-04): Fluxos de registro, login, refresh token e logout
2. **Livros Públicos** (UC-07, UC-09, UC-10, UC-13): Busca, listagem e visualização de livros
3. **Funcionalidades de Usuário** (UC-06, UC-14, UC-15, UC-17, UC-18): Perfil, histórico de leitura e coleções
4. **Administração de Livros** (UC-24, UC-25, UC-30, UC-41): CRUD de livros e upload de capas
5. **Administração de Capítulos** (UC-33, UC-34, UC-35, UC-37, UC-47, UC-56): Gestão de capítulos e scraping
6. **Administração de Metadados** (UC-50, UC-51, UC-52, UC-55, UC-58): CRUD e merge de metadados

**Diagramas:**

![Autenticação](./image/sequence-auth/Sequ%C3%AAncia%20-%20Autentica%C3%A7%C3%A3o.svg)
![Livros Públicos](./image/sequence-books-public/Sequ%C3%AAncia%20-%20Consulta%20P%C3%BAblica%20de%20Livros.svg)
![Funcionalidades de Usuário](./image/sequence-user-features/Sequ%C3%AAncia%20-%20Funcionalidades%20de%20Usu%C3%A1rio%20Autenticado.svg)
![Administração de Capítulos](./image/sequence-admin-chapters/Sequ%C3%AAncia%20-%20Administra%C3%A7%C3%A3o%20de%20Cap%C3%ADtulos%20e%20P%C3%A1ginas.svg)
![Administração de Metadados](./image/sequence-admin-metadata/Sequ%C3%AAncia%20-%20Administra%C3%A7%C3%A3o%20de%20Metadados.svg)

### 2.3.1 Contratos de Operação

Os contratos de operação especificam formalmente as pré-condições e pós-condições para as principais operações do sistema.

| Contrato  | Operação                                                                      | Referências  | Pré-condições                                                     | Pós-condições                                                         |
| --------- | ----------------------------------------------------------------------------- | ------------ | ----------------------------------------------------------------- | --------------------------------------------------------------------- |
| **CO-01** | `register(email, password, username, name)`                                   | UC-01        | Email e username não cadastrados                                  | Instância de User criada, senha criptografada, sessão JWT retornada   |
| **CO-02** | `login(email, password)`                                                      | UC-02        | Usuário existe e não está excluído                                | Par de tokens JWT gerado, refresh token armazenado no Redis           |
| **CO-03** | `createBook(title, synopsis, status, authorIds, tagIds, sensitiveContentIds)` | UC-24        | Usuário com role ADMIN, IDs de autores/tags/conteúdo existem      | Book criado, associações many-to-many estabelecidas, cache invalidado |
| **CO-04** | `createCollection(name, description, userId)`                                 | UC-15        | Usuário autenticado, nome não duplicado                           | CollectionBook criada, relação com User estabelecida                  |
| **CO-05** | `markChapterAsRead(userId, chapterId)`                                        | UC-14        | Usuário autenticado, capítulo existe                              | ChapterRead criado/atualizado com timestamp atual                     |
| **CO-06** | `scrapeChapters(bookId, websiteId, startChapter, endChapter)`                 | UC-33, UC-47 | Admin autenticado, website configurado, Selenium Grid operacional | Chapters e Pages criados, imagens armazenadas, cache invalidado       |

# 3. Modelos de Projeto

Esta seção apresenta os modelos de projeto do sistema Gatuno, incluindo visões arquiteturais, estruturais e comportamentais em diferentes níveis de abstração.

## 3.1 Arquitetura

A arquitetura do sistema é documentada através do **C4 Model**, fornecendo visão hierárquica em 4 níveis de abstração:

### Level 1: Context Diagram

Mostra o sistema Gatuno no contexto externo, seus usuários e sistemas externos.

![C4 Context](./image/c4-context/C4%20Context%20Diagram.svg)

### Level 2: Container Diagram

Detalha a arquitetura de alto nível com aplicações executáveis e armazenamento de dados: Frontend (Angular 18+ SSR), Backend API (NestJS), MySQL 8.0 (Master-Slave), Redis 8.0, Selenium Grid, Prometheus/Grafana/AlertManager.

![C4 Container](./image/c4-container/C4%20Container%20Diagram.svg)

### Level 3: Component Diagram

Detalha os componentes internos da API Backend: Módulos de Domínio (Books, Chapters, Users, Collections), Módulos Transversais (Auth, Cache, Storage, Scraping), Camadas (Controllers, Services, Repositories, DTOs).

![C4 Component](./image/c4-component/C4%20Component%20Diagram.svg)

### Level 4: Deployment Diagram

Descreve a infraestrutura física de implantação com Docker Compose.

![C4 Deployment](./image/c4-deployment/C4%20Deployment%20Diagram.svg)

## 3.2 Diagrama de Componentes e Implantação

Além dos diagramas C4, o sistema possui diagramas UML tradicionais para componentes e implantação:

### Diagrama de Componentes

Representa os componentes do sistema e suas dependências, organizados por camada arquitetural.

![Componentes](./image/component-diagram/Diagrama%20de%20Componentes.svg)

### Diagrama de Implantação - Desenvolvimento

Infraestrutura local com 10 containers Docker: Frontend Angular, Backend NestJS, MySQL Master + 2 Slaves, Redis, Selenium Hub + Chrome Node, Prometheus, Grafana, AlertManager.

![Implantação - Dev](./image/deployment-dev/Diagrama%20de%20Implanta%C3%A7%C3%A3o%20-%20Desenvolvimento.svg)

### Diagrama de Implantação - Produção

Infraestrutura de produção com 12 containers incluindo Traefik como reverse proxy com SSL/TLS.

![Implantação - Prod](./image/deployment-prod/Diagrama%20de%20Implanta%C3%A7%C3%A3o%20-%20Produ%C3%A7%C3%A3o.svg)

## 3.3 Diagrama de Classes

O diagrama de classes apresenta a estrutura estática do sistema, organizado em 4 pacotes principais:

### Pacotes

1. **Entities**: Entidades de domínio mapeadas para tabelas do banco de dados

    - User, Role, Book, Author, Chapter, Page, Cover, Tag, SensitiveContent, ChapterRead, CollectionBook, Website

2. **Controllers**: Camada de apresentação (REST API)

    - 14 controllers implementando endpoints HTTP para todas as funcionalidades

3. **Services**: Camada de lógica de negócio

    - Business logic, orquestração de operações complexas, validações

4. **DTOs**: Data Transfer Objects para validação de entrada/saída
    - CreateBookDto, UpdateBookDto, LoginDto, RegisterDto, etc.

**Características Modeladas:**

-   Atributos e métodos de cada classe
-   Relacionamentos (associações, composições, agregações)
-   Multiplicidades
-   Herança e interfaces
-   Tipos de dados e visibilidade

![Diagrama de Classes](./image/class/Diagrama%20de%20Classes.svg)

## 3.4 Diagramas de Sequência

Diagramas de sequência detalhados mostram a interação entre objetos (Controllers, Services, Repositories, Cache, Database) para implementação dos casos de uso.

### Diagramas por Área Funcional

1. **Autenticação** - UC-01, UC-02, UC-03, UC-04
2. **Livros Públicos** - UC-07, UC-09, UC-10, UC-13
3. **Funcionalidades de Usuário** - UC-06, UC-14, UC-15, UC-17, UC-18
4. **Administração de Livros** - UC-24, UC-25, UC-30, UC-41
5. **Administração de Capítulos** - UC-33, UC-34, UC-35, UC-37, UC-47, UC-56
6. **Administração de Metadados** - UC-50, UC-51, UC-52, UC-55, UC-58

![Sequência - Autenticação](./image/sequence-auth/Sequ%C3%AAncia%20-%20Autentica%C3%A7%C3%A3o.svg)
![Sequência - Livros Públicos](./image/sequence-books-public/Sequ%C3%AAncia%20-%20Consulta%20P%C3%BAblica%20de%20Livros.svg)
![Sequência - Usuário](./image/sequence-user-features/Sequ%C3%AAncia%20-%20Funcionalidades%20de%20Usu%C3%A1rio%20Autenticado.svg)
![Sequência - Admin Capítulos](./image/sequence-admin-chapters/Sequ%C3%AAncia%20-%20Administra%C3%A7%C3%A3o%20de%20Cap%C3%ADtulos%20e%20P%C3%A1ginas.svg)
![Sequência - Admin Metadados](./image/sequence-admin-metadata/Sequ%C3%AAncia%20-%20Administra%C3%A7%C3%A3o%20de%20Metadados.svg)

## 3.5 Diagramas de Comunicação

Diagramas de comunicação (também conhecidos como diagramas de colaboração) enfatizam a estrutura organizacional dos objetos que trocam mensagens, complementando os diagramas de sequência com foco na topologia de comunicação.

### Diagramas por Domínio

1. **Autenticação** - UC-01, UC-02, UC-03, UC-04
2. **Livros** - UC-07, UC-09, UC-13
3. **Coleções** - UC-15, UC-17, UC-18
4. **Administração** - UC-24, UC-30, UC-41
5. **Scraping** - UC-33, UC-47

**Características:**

-   Numeração sequencial de mensagens (1, 1.1, 1.2, 2, etc.)
-   Links de comunicação entre objetos
-   Parâmetros e valores de retorno
-   Decisões e condições

![Comunicação - Autenticação](./image/communication-auth/Diagrama%20de%20Comunica%C3%A7%C3%A3o%20-%20Autentica%C3%A7%C3%A3o.svg)
![Comunicação - Livros](./image/communication-books/Diagrama%20de%20Comunica%C3%A7%C3%A3o%20-%20Consulta%20de%20Livros.svg)
![Comunicação - Coleções](./image/communication-collections/Diagrama%20de%20Comunica%C3%A7%C3%A3o%20-%20Cole%C3%A7%C3%B5es%20de%20Usu%C3%A1rio.svg)
![Comunicação - Admin](./image/communication-admin/Diagrama%20de%20Comunica%C3%A7%C3%A3o%20-%20Administra%C3%A7%C3%A3o%20de%20Livros.svg)
![Comunicação - Scraping](./image/communication-scraping/Diagrama%20de%20Comunica%C3%A7%C3%A3o%20-%20Web%20Scraping.svg)

## 3.6 Diagramas de Estados

Os diagramas de máquina de estados modelam o comportamento dinâmico de objetos e processos do sistema, mostrando transições entre estados ao longo do ciclo de vida.

### Máquinas de Estado Implementadas

1. **User Session** - Ciclo de vida de sessão de usuário (Anônimo → Autenticado → Logout)
2. **Book Lifecycle** - Estados de um livro (Pending → Processing → Completed/Failed)
3. **Chapter Lifecycle** - Estados de capítulo com scraping (Pending → Scraping → Completed/Failed/Retry)
4. **Collection Management** - Operações em coleções (Empty → Has Books → Full)
5. **File Upload** - Transação de upload (Validating → Saving → Completed/Failed)
6. **Cache System** - Operações de cache (Miss → Fetching → Cached → Invalidated)

**Elementos Modelados:**

-   Estados simples e compostos
-   Transições com eventos e guardas
-   Ações de entrada/saída (entry/exit)
-   Estados iniciais e finais
-   Choice points e fork/join

![Estado - Sessão de Usuário](./image/state-user-session/Estado%20-%20Sess%C3%A3o%20de%20Usu%C3%A1rio.svg)
![Estado - Ciclo de Vida do Livro](./image/state-book-lifecycle/Estado%20-%20Ciclo%20de%20Vida%20do%20Livro.svg)
![Estado - Ciclo de Vida do Capítulo](./image/state-chapter-lifecycle/Estado%20-%20Ciclo%20de%20Vida%20do%20Cap%C3%ADtulo.svg)
![Estado - Gerenciamento de Coleção](./image/state-collection/Estado%20-%20Cole%C3%A7%C3%A3o%20de%20Livros.svg)
![Estado - Upload de Arquivo](./image/state-file-upload/Estado%20-%20Upload%20e%20Armazenamento%20de%20Arquivos.svg)
![Estado - Sistema de Cache](./image/state-cache-system/Estado%20-%20Cache%20do%20Sistema.svg)

# 4. Modelos de Dados

Esta seção apresenta os modelos de dados do sistema Gatuno, incluindo o esquema relacional e as estratégias de mapeamento objeto-relacional.

## 4.1 Esquema de Banco de Dados

O sistema utiliza **MySQL 8.0** como SGBD principal com arquitetura de replicação Master-Slave (1 master + 2 slaves) para distribuição de carga de leitura.

### Modelo Entidade-Relacionamento

O modelo de dados é composto por 12 entidades principais e 5 tabelas de junção para relacionamentos many-to-many:

#### Entidades Principais

1. **users**: Usuários do sistema com autenticação
2. **roles**: Perfis de acesso (ex: ADMIN, USER)
3. **books**: Livros/mangás do catálogo
4. **authors**: Autores das obras
5. **chapters**: Capítulos dos livros
6. **pages**: Páginas individuais de cada capítulo
7. **covers**: Capas dos livros
8. **tags**: Tags/categorias para classificação
9. **sensitive_content**: Categorias de conteúdo sensível com peso (0-5)
10. **chapters_read**: Histórico de leitura (quem leu qual capítulo)
11. **collection_book**: Coleções personalizadas de usuários
12. **websites**: Configurações de websites para scraping

#### Tabelas de Junção (Many-to-Many)

1. **users_roles**: Usuários ↔ Roles
2. **books_authors**: Livros ↔ Autores
3. **books_tags**: Livros ↔ Tags
4. **books_sensitive_content**: Livros ↔ Conteúdo Sensível
5. **collection_book_books**: Coleções ↔ Livros

### Relacionamentos Principais

-   **User → Role**: Many-to-Many (tabela users_roles)
-   **Book → Author**: Many-to-Many (tabela books_authors)
-   **Book → Tag**: Many-to-Many (tabela books_tags)
-   **Book → SensitiveContent**: Many-to-Many (tabela books_sensitive_content)
-   **Book → Chapter**: One-to-Many (FK em chapters)
-   **Chapter → Page**: One-to-Many (FK em pages)
-   **Book → Cover**: One-to-Many (FK em covers)
-   **User → ChapterRead**: One-to-Many (FK em chapters_read)
-   **Chapter → ChapterRead**: One-to-Many (FK em chapters_read)
-   **User → CollectionBook**: One-to-Many (FK em collection_book)
-   **CollectionBook ↔ Book**: Many-to-Many (tabela collection_book_books)

### Características do Esquema

-   **Chaves Primárias**: UUID para entidades principais, INT para entidades de alta volumetria (pages, chapters_read)
-   **Soft Delete**: Coluna `deletedAt` em entidades principais (users, books, chapters, tags, authors, covers)
-   **Auditoria**: Colunas `createdAt` e `updatedAt` para rastreamento temporal
-   **Enumerações**: Status de scraping (PENDING, PROCESSING, COMPLETED, FAILED, PAUSED), tipo de livro (MANGA, MANHWA, MANHUA, COMIC, NOVEL)
-   **JSON Fields**: Dados semi-estruturados como alternativeTitle, originalUrl, altNames, ignoreFiles
-   **Constraints**: UNIQUE em email, userName, role.name, tag.name, sensitive_content.name
-   **Índices**: Otimização em colunas de busca (title, email, userName) e filtros (scrapingStatus, deletedAt)

![Diagrama ER](./image/database/Diagrama%20Entidade-Relacionamento.svg)

## 4.2 Estratégias de Mapeamento Objeto-Relacional

O sistema utiliza **TypeORM** como framework ORM para mapear classes TypeScript para o esquema relacional MySQL.

### Estratégias Implementadas

#### 1. Mapeamento de Entidades

-   **Decorators Declarativos**: `@Entity`, `@Column`, `@PrimaryGeneratedColumn`
-   **Tipos de Dados**: Mapeamento explícito (VARCHAR, TEXT, INT, TIMESTAMP, ENUM, JSON)
-   **Constraints**: `unique`, `nullable`, `default` definidos nos decorators
-   **Nomenclatura**: Tabelas em snake_case, classes em PascalCase

#### 2. Relacionamentos

**One-to-Many / Many-to-One:**

-   Decorator `@OneToMany` no lado "um" (ex: Book → Chapter)
-   Decorator `@ManyToOne` no lado "muitos" (ex: Chapter → Book)
-   `@JoinColumn` define nome da coluna FK
-   Propriedade FK exposta para queries otimizadas

**Many-to-Many Simples:**

-   Decorator `@ManyToMany` em ambos os lados
-   `@JoinTable` no lado "dono" da relação
-   Tabela de junção gerenciada automaticamente pelo TypeORM

**Many-to-Many com Atributos:**

-   Entidade intermediária explícita (ex: ChapterRead)
-   Dois relacionamentos `@ManyToOne`
-   Permite adicionar atributos na relação (ex: readAt timestamp)

#### 3. Repository Pattern

-   Abstrair acesso ao banco através de repositories
-   Métodos customizados para queries complexas
-   QueryBuilder para consultas dinâmicas e otimizadas
-   Injeção de dependências via NestJS

#### 4. Transações

-   Garantir atomicidade em operações complexas
-   Uso de `dataSource.transaction()` para operações multi-entidade
-   Rollback automático em caso de erro

#### 5. Soft Delete

-   `@DeleteDateColumn` para soft delete pattern
-   Métodos `softDelete()` e `restore()`
-   Queries padrão ignoram registros deletados
-   Opção `withDeleted: true` para incluir deletados

#### 6. Auditoria

-   `@CreateDateColumn` para timestamp de criação
-   `@UpdateDateColumn` para timestamp de modificação
-   Classe base `AuditableEntity` para reutilização

#### 7. Otimizações

**Lazy vs Eager Loading:**

-   Lazy Loading por padrão
-   Eager Loading seletivo com `eager: true`
-   Controle explícito via parâmetro `relations`

**Índices:**

-   `@Index()` para colunas de busca
-   Índices compostos para queries frequentes
-   Índices únicos para constraints de unicidade

**Caching:**

-   Integração com Redis para cache de queries
-   Invalidação de cache em operações de escrita
-   TTL configurável por tipo de dado

**Paginação:**

-   `take` e `skip` para paginação eficiente
-   `findAndCount()` retorna dados e total simultaneamente

### Exemplo de Mapeamento

```typescript
@Entity('books')
export class Book {
	@PrimaryGeneratedColumn('uuid')
	id: string;

	@Column({ type: 'varchar', length: 500 })
	@Index()
	title: string;

	@Column({
		type: 'enum',
		enum: ScrapingStatus,
		default: ScrapingStatus.PENDING,
	})
	scrapingStatus: ScrapingStatus;

	@ManyToMany(() => Author)
	@JoinTable({ name: 'books_authors' })
	authors: Author[];

	@OneToMany(() => Chapter, (chapter) => chapter.book, { cascade: true })
	chapters: Chapter[];

	@CreateDateColumn()
	createdAt: Date;

	@DeleteDateColumn()
	deletedAt: Date;
}
```
