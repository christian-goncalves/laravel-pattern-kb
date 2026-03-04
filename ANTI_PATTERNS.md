# ANTI_PATTERNS.md

Este documento detalha práticas e padrões que **NÃO DEVEM** ser adotados neste projeto para evitar o desvio de padrão (*drift*) em relação à arquitetura estabelecida nos fontes.

---

# 1. Proibições Globais

1.  **NÃO FAÇA** uso de **Form Requests** para validação. Toda a validação deve ser feita de forma *inline* nos métodos `Submit` dos controladores utilizando `$request->validate()`.
2.  **NÃO FAÇA** uso do método helper `compact()` para passar dados para as views. Utilize obrigatoriamente um array associativo chamado `$data`.
3.  **NÃO FAÇA** uso do método `$table->timestamps()` nas migrations. As colunas de data devem ser declaradas manualmente como `dateTime` para controle de precisão e compatibilidade.
4.  **NÃO FAÇA** uso de **Resource Controllers**. Cada rota deve ser declarada explicitamente (GET ou POST) com nomes baseados em pontos.
5.  **NÃO FAÇA** uso de **Route Model Binding** implícito. IDs devem ser recebidos como parâmetros simples, descriptografados e validados manualmente dentro do controlador.

---

# 2. Anti-padrões por Camada

### Controllers
*   **NÃO FAÇA** métodos de processamento de formulários que não terminem com o sufixo `Submit`.
    *   *Anti-exemplo:* `public function saveUser(Request $request)`
*   **NÃO FAÇA** a descriptografia de IDs fora de um bloco `try-catch` que trate a exceção e redirecione o usuário.
*   **NÃO FAÇA** retornos de mensagens de erro que não utilizem a chave de sessão `server_error`.

### Models
*   **NÃO FAÇA** a declaração de relacionamentos sem especificar explicitamente a chave estrangeira (FK), mesmo que o Laravel a deduza.
    *   *Anti-exemplo:* `return $this->belongsTo(Company::class);`.
*   **NÃO FAÇA** a criação de novos modelos que não incluam a trait `SoftDeletes`.

### Migrations
*   **NÃO FAÇA** uso de `foreignId()` ou `constrained()`. Utilize `integer()` com o prefixo `id_` e adicione explicitamente o método `index()`.
*   **NÃO FAÇA** tabelas sem a coluna `deleted_at` definida como `dateTime` anulável.

### Routes (web.php)
*   **NÃO FAÇA** nomes de rotas que utilizem hífens ou camelCase para separação. Utilize obrigatoriamente pontos.
    *   *Anti-exemplo:* `->name('admin-create-user')`.
*   **NÃO FAÇA** rotas soltas fora de grupos de middlewares (`auth`, `guest` ou `can`).

### Seeders
*   **NÃO FAÇA** uso de **Factories** ou do método `Model::create()`. Utilize sempre a Facade `DB::table()->insert()`.
*   **NÃO FAÇA** o preenchimento de senhas sem o uso de `bcrypt()` ou `Hash::make()`.

### Blade & Helpers
*   **NÃO FAÇA** o uso de IDs descriptografados em links HTML. IDs em URLs **DEVEM** ser passados através de `Crypt::encrypt()`.
*   **NÃO FAÇA** a exibição de erros de validação manualmente com `@error`. Utilize os helpers padronizados `{!! showValidationError(...) !!}` e `{!! showServerError() !!}`.

---

# 3. Sinais de Drift (Checklist de Revisão)

Ao revisar um código gerado ou alterado, marque como **rejeitado** se encontrar:
*   [ ] Mensagens de erro ou validação em Inglês.
*   [ ] Colunas de data chamadas `created_at` e `updated_at` criadas via `$table->timestamps()`.
*   [ ] Chaves estrangeiras sem o prefixo `id_`.
*   [ ] Relacionamentos no Model sem o segundo parâmetro de chave estrangeira.
*   [ ] Ausência do campo `deleted_at` em novas tabelas ou `SoftDeletes` em novos Models.
*   [ ] Uso de `redirect()->to()` em vez de `redirect()->route()`.
*   [ ] Views que recebem variáveis soltas em vez de um único array `$data`.
*   [ ] Consultas de listagem que não consideram registros deletados (`withTrashed()`) quando o contexto exige.
