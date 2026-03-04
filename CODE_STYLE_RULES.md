# CODE_STYLE_RULES.md

Este documento estabelece as diretrizes de estilo e padrões estruturais de código para o projeto, visando garantir a consistência e a manutenção da arquitetura.

# 1. Regras Globais (Obrigatórias)

1.  **Organização de Camadas:** O projeto segue o padrão MVC do Laravel, com lógica de persistência no **Model**, processamento e fluxo no **Controller**, e apresentação no **Blade**.
2.  **Convenção de Nomenclatura de Métodos:** Métodos de **Controller** que processam dados de formulários (POST) **DEVEM** obrigatoriamente terminar com o sufixo `Submit`.
3.  **Padrão de Comunicação Controller-View:** O envio de dados para as views **DEVE** ser feito exclusivamente através de um array associativo único chamado `$data`.
4.  **Uso de Subtitles:** O array `$data` **DEVE** conter obrigatoriamente a chave `subtitle` para padronização de títulos de página.
5.  **Política de Validação:** A validação de dados **DEVE** ser feita de forma *inline* no controlador, utilizando `$request->validate()` com arrays de regras e de mensagens customizadas em português.
6.  **Política de IDs:** IDs recebidos via rota ou formulário **DEVEM** ser descriptografados usando `Crypt::decrypt()`. Em caso de falha, o sistema **DEVE** redirecionar para uma rota segura ou interromper a execução via `abort()`.

# 2. Regras por Camada

### Controllers
1.  **DEVE** utilizar injeção de dependência de `Request` nos métodos de submissão.
2.  **DEVE** organizar consultas complexas ou lógicas de agregação em métodos `private` para manter as *actions* limpas.

### Models
1.  **DEVE** utilizar a trait `SoftDeletes` em todos os modelos.
2.  **NÃO DEVE** utilizar propriedades como `$fillable` ou `$guarded` nos modelos, conforme padrão observado.

### Migrations
1.  **DEVE** utilizar classes anônimas que estendem `Migration`.
2.  **DEVE** definir o tipo de retorno `void` nos métodos `up()` e `down()`.

### Routes (web.php)
1.  **DEVE** agrupar rotas por middleware (`auth`, `guest`, `can:role`).
2.  **DEVE** nomear rotas usando notação de pontos (ex: `admin.company.create`).
3.  **DEVE** nomear rotas POST com o sufixo `.submit`.

### Seeders
1.  **DEVE** utilizar a Facade `DB` para inserções, evitando o uso de Models.
2.  **DEVE** realizar `truncate()` em tabelas de massa de dados antes da inserção.

### Helpers / Blade
1.  **DEVE** envolver funções globais em `if(!function_exists('...'))`.
2.  **DEVE** utilizar `{!! showValidationError(...) !!}` para exibir erros abaixo dos inputs.

# 3. Regras de Persistência

1.  **Timestamps Manuais:** As migrations **NÃO DEVEM** usar `$table->timestamps()`. Devem usar declarações manuais de `dateTime`.
    ```php
    $table->dateTime('created_at')->useCurrent();
    $table->dateTime('updated_at')->useCurrent()->useCurrentOnUpdate();
    ```
2.  **Soft Delete:** Todas as tabelas **DEVEM** conter a coluna `deleted_at` do tipo `dateTime` anulável.
3.  **Chaves Estrangeiras:** **DEVEM** seguir o padrão `id_entidade` (singular), ser do tipo `integer` e possuir obrigatoriamente um índice (`index`).
4.  **Relacionamentos:** No Model, relacionamentos **DEVEM** declarar a chave estrangeira explicitamente.
    ```php
    return $this->belongsTo(Company::class, 'id_company');
    ```

# 4. Regras de Resposta

1.  **Redirects:** Falhas de validação ou lógica **DEVEM** usar `redirect()->back()->withInput()`.
2.  **Mensagens de Erro:** Todas as mensagens de erro de validação e de sistema **DEVEM** ser escritas em Português (PT-BR).
3.  **Sessão (Flash Data):**
    *   Erros críticos de negócio ou servidor **DEVEM** ser passados na chave `server_error`.
    *   Mensagens de sucesso **DEVEM** ser passadas na chave `message`.
4.  **Views de Sucesso:** Ações críticas de criação **DEVEM** retornar uma view específica de sucesso em vez de apenas um redirect.
