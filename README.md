# 📦 Code Review - Desafio Técnico VExpenses

Este repositório contém minha análise técnica do projeto fornecido no desafio para back-end da VExpenses.

O objetivo foi revisar o código existente, identificar possíveis melhorias, pontos de atenção e sugestões para manter a aplicação escalável, segura e alinhada a boas práticas modernas de desenvolvimento com Laravel.

---

## 📂 Estrutura do Projeto

A aplicação segue uma arquitetura baseada em **camadas** (UseCase, Domain, Repository, Integration, etc.), muito próxima dos princípios de **Clean Architecture** e **DDD (Domain-Driven Design)**.

Abaixo apresento minha análise dividida por tópicos relevantes.

---

# ✅ Checklist de Melhoria do Projeto

## Gerais

-   [ ] Organização dos imports por domínio

```php
use App\UseCases\{
    User\Show,
    User\Register,
};

```

-   [ ] **Corrigir nomes de classes com letra maiúscula**, respeitando o padrão PSR-4. Exemplo: `new show()` deve ser `new Show()`.
-   [ ] Evitar criação de variáveis desnecessárias.
-   [ ] Comentários mais descritivos.

## Rotas

-   [ ] Usar versionamento nas rodas `api/v1/endpoint`
-   [ ] Agrupar pelos controller com o methodo `->controoler`
-   [ ] Remover uso de `auth.basic` no login; usar autenticação via token Sanctum, ja implementado no projeto
-   [ ] Refatorar endpoints para aderir melhor ao padrão REST (ex: `PATCH /users/{id}/account` para status, rotas sem uri, rotas post com parametros na uri. ).
-   [ ] Garantir que todas as rotas sensíveis estejam protegidas por middleware adequado.

## Http

### Controllers

#### Base

-   [ ] **Avaliar mover a lógica de `response()` para uma Trait `ApiResponse`**,se outros controllers (como de APIs externas, admin, etc.) forem criados e não estenderem esse base controller diretamente.

#### Users

-   [ ] **Deixar para os controllers somente os verbos RESTful**, como `index`, `store`, `show`, `update`, `destroy`. Métodos como `register` e `login` devem ser movidos para um `AuthController`.

-   [ ] **Garantir que todas as respostas passem por Resources**, evitando retorno de dados crus (modelos, arrays ou coleções diretamente).

-   [ ] **Evitar acesso direto ao Eloquent nos controllers**; usar sempre `UseCase`, `Repository` ou `Service` para separar lógica de negócio da camada HTTP.

-   [ ] **Evitar múltiplas chamadas de `Auth::user()`** em um mesmo método. Armazenar em variável `$user = Auth::user();` para clareza e performance.

-   [ ] **Seguir padrão de nomenclatura das ações RESTful** (`store`, `update`, `index`, etc.). Evitar nomes genéricos como `create()` para não conflitar com métodos automáticos do Laravel (`Route::resource`).

-   [ ] **Evitar lógica condicional ou de decisão dentro do controller**. Delegar esse tipo de regra para a camada de `UseCase` ou `Service`.

-   [ ] **Usar `FormRequest` tanto para validação quanto para autorização** (`authorize()`), evitando checagens manuais no controller.

##### Company

-   [ ] **Evitar múltiplas chamadas de `Auth::user()`** no mesmo método. Salvar em uma variável `$user = Auth::user();` para clareza e performance.

-   [ ] **Evitar acesso direto ao Eloquent dentro do controller**, como o uso de `Company::find(...)`. Prefira retornar o que o UseCase já retornou, ou encapsular essa lógica em um repositório.

-   [ ] **Evitar `.first()` após `find()`**, pois `find()` já retorna um único modelo ou `null`. Esse trecho está redundante:
    ```php
    Company::find(Auth::user()->company_id)->first()
    ```

##### Card

-   [ ] \*\*Evitar múltiplas chamadas a Auth::user()

##### Account

-   [ ] \*\*Sem validação de entrada (em `register`)
-   [ ] Evitar múltiplas chamadas a `Auth::user()`
-   [ ] Refatorar endpoints para aderir melhor ao padrão REST
-   [ ] Isolar Active e Block em controller INVOKABLE

#### Request

-   [ ] \*\*Trait SanitizesInput não esta sendo usado
-   [ ] Passar campos types ou qualquer outro Enum para um class enum (Ex in:USER,VIRTUAL,MANAGER)
-   [ ] passar o `bail`, para retornar os erros em fila
-   [ ] definir limites de tamando
-   [ ] validar campos de email
-   [ ] para o campo document criar um Rules para validar o documento cpf/cnpj
-   [ ] campo de senha adiciona confirmed, e criar um `confirm_password
-   [ ] Caso necessário adicionar messagem personalizadas

#### Resource

-   [ ] \*\*O parâmetros `$order` não está sendo usado, se não for necessário, remova para evitar confusão.
-   [ ] Uso de `$this->resource['campo']` $this->campo

#### DefaultResponse

-   [ ] **Adicionar o campo `message` na resposta padrão**, permitindo mensagens customizadas e facilitando o feedback para o usuário no frontend. Exemplo: `"Usuário criado com sucesso"` ou `"Erro ao processar requisição"`.

-   [ ] **Aceitar o parâmetro `message` no construtor do `DefaultResponse`**, com valor padrão condicional baseado em `success`.

-   [ ] **Adicionar o método `toResponse()`**, que retorna diretamente um `JsonResponse`, permitindo uso direto nos controllers:
        `return (new DefaultResponse(...))->toResponse();`

-   [ ] **Garantir que o campo `errors` sempre exista na estrutura da resposta**, mesmo quando estiver vazio, evitando necessidade de `isset()` no frontend.

-   [ ] **Manter a estrutura da resposta sempre padronizada**, com os campos:

    -   `success`
    -   `message`
    -   `code`
    -   `method`
    -   `request`
    -   `data`
    -   `errors`

-   [ ] **Evitar retornar dados paginados diretamente**, garantindo que estejam encapsulados dentro de uma `ResourceCollection` que utiliza `LengthAwarePaginator`.

-   [ ] **Validar o tipo dos erros recebidos**, lançando exceção se não forem instâncias de `InternalError`, garantindo integridade da resposta.

-   [ ] **Centralizar lógica de paginação no método `formatPaginatedData()`**, respeitando query strings e mantendo compatibilidade com filtros da URL.

-   [ ] **Manter método `__get()` apenas para leitura de atributos da resposta**, não usar como API principal para acesso ao conteúdo.

## Domains

-   [ ] \*\*O domínio não deve cria nem um tipo de Record.
-   [ ] Assinatura do método `handle()`
-   [ ] Validação de tipo hardcoded usar Enum para comportar `['USER', 'VIRTUAL', 'MANAGER']`
-   [ ] Padronização dos erros como no ACCOUNT_NOT_FOUND
-   [ ] Falta `toArray()` para exportar os dados.
-   [ ] Sugestão de estrutura:

```php
📦 Domains
 ┣ 📂User
 ┃ ┣ 📜UserEntity.php     <- estrutura interna do usuário (opcional)
 ┃ ┗ 📜Create.php         <- validação e preparação
📦 UseCases
 ┗ 📂User
   ┗ 📜Register.php       <- orquestra domínio + repositório
📦 Repositories
 ┗ 📂User
   ┣ 📜UserRepository.php <- abstração de persistência
   ┗ 📜CanUseEmail.php    <- regras específicas
```

## UseCases

-   [ ] Use PascalCase para nomes de classes: `Show` em vez de `show`.
-   [ ] Evite nomes genéricos (`$a`, `$b`, `$c`). Use nomes claros e semânticos, como `$userId`, `$companyId`, `$user`.
-   [ ] Separar Login do UseCases, para Auth
-   [ ] Ajuste de Tipagem
-   [ ] Adicionar método comum para tratamento de erros (se ainda não estiver na trait Logger)
-   [ ] Tipagem no método `handle()`
-   [ ] Inicializar propriedades que armazenam dados

## Repositories

-   [ ] Substituir todos os usos de `whereRaw` por métodos seguros do Eloquent (ex: `where('name', 'like', ...)`.
-   [ ] Evitar criar instância do `parent` antes de definir `model`
-   [ ] Usa o método reutilizável `create()` da base, o que reduz duplicação.
-   [ ] Clonar o builder com o filtro, ou aplicar o filtro direto no método `find`
-   [ ] Status HardCode passar para um Enum
-   [ ] Garantir que o filtro por `company_id` é aplicado antes do update
-   [ ] Garantir que a senha já venha hasheada no domínio ou hashear antes de salvar no repositório.

## Integrações

-   [ ] Implementar lógica de retry/circuit breaker para chamadas externas
-   [ ] Melhorar o mapeamento de erros das integrações para exceções de domínio mais específicas
-   [ ] Usar camelCase (`actualPage`) ou snake_case (`actual_page`).
-   [ ] Remover setters para imutabilidade, como essa classe representa uma resposta, geralmente faz sentido ser imutável.
-   [ ] No método `invalidateAuthenticationToken()`, o Laravel usa `Cache::forget('key')` e não `Cache::delete`
-   [ ] Falta de tipos de retorno explícitos em alguns métodos

## 7. Middleware

-   [ ] Adicionar fallback/logging na resolução dinâmica de políticas.
-   [ ] Garantir que todos os métodos de controllers tenham políticas correspondentes.

## 8. Policies

-   [ ] Documentar ou remover métodos vazios nas policies.
-   [ ] Garantir que todas as regras de autorização estejam cobertas.

## 9. Configurações

-   [ ] Garantir que configs sensíveis (ex: credenciais de BaaS) estejam apenas no `.env` e nunca versionadas.
-   [ ] Revisar arquivos de configuração para boas práticas de segurança.

## 10. Testes

-   [ ] Adicionar testes negativos para falhas de integração (ex: BaaS fora do ar).
-   [ ] Cobrir endpoints de empresa e cartão com testes de feature.
-   [ ] Adicionar testes unitários para domínios e repositórios.
-   [ ] Garantir cobertura de casos de borda e falhas de autorização.
