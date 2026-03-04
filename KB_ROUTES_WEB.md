
# KB_ROUTES_WEB.md

Este documento define os padrões estruturais e as convenções de organização das rotas da aplicação, garantindo uma arquitetura de acesso consistente e segura.

# 1. Estrutura Geral do web.php

A organização do arquivo de rotas segue uma ordem lógica de importação e separação por blocos de acesso:

*   **Imports:** O arquivo inicia com a importação da Facade `Route`, seguida pelos controladores em ordem alfabética ou agrupamento lógico e, por fim, os Middlewares customizados.
*   **Organização por Comentários:** Blocos de acesso são visualmente separados por linhas de comentário (`// -------------------`) e títulos explicativos.
*   **Ordem dos Blocos:**
    1.  Rotas públicas/convidados (`guest`).
    2.  Rotas autenticadas agrupadas por permissão/perfil (`can:role`).
    3.  Rotas autenticadas gerais (ex: logout, alteração de senha).
    4.  Rotas de interfaces específicas com Middlewares próprios.

# 2. Agrupamento por Middleware

O projeto utiliza extensivamente o agrupamento de rotas para aplicar políticas de segurança e contexto:

*   **Sintaxe:** `Route::middleware(['middleware_name'])->group(function() { ... });`.
*   **Middlewares observados:**
    *   `guest`: Acesso exclusivo para usuários não autenticados.
    *   `auth`: Restringe o acesso apenas a usuários logados.
    *   `can:role`: Implementa autorização baseada em papéis (ex: `can:client`, `can:sys-admin`, `can:client-admin`).
    *   **Classes de Middleware:** Uso de classes para sessões de contextos específicos (ex: `TicketDispenserSession::class`).

# 3. Convenção de URLs

*   **Padrão de Paths:** URLs utilizam **kebab-case** (letras minúsculas separadas por hífen). Exemplo: `/conclude-registration`, `/define-password-success`.
*   **Parâmetros de Rota:** Definidos entre chaves. Parâmetros de ID são genéricos (`{id}`) ou específicos quando há múltiplos no path. Exemplo: `{code}`, `{id}`, `{queue_id}/{ticket_id}/{status}`.

# 4. Convenção de Nomes de Rotas

As rotas são obrigatoriamente nomeadas seguindo o padrão de **notação por pontos**:

*   **Hierarquia:** Reflete o nível de acesso e a entidade (ex: `admin.company.create`).
*   **Sufixo de Submissão:** Rotas de processamento (POST) obrigatoriamente terminam com `.submit`. Exemplo: `login.submit`, `queue.create.submit`, `admin.company.control.access.submit`.

# 5. Mapeamento Controller → Método

*   **Métodos GET:** Mapeados para nomes que descrevem a visualização ou ação (ex: `index`, `createQueue`, `editQueue`, `details`).
*   **Métodos POST:** Sempre mapeados para métodos que possuem o sufixo **`Submit`**. Exemplo: `loginSubmit`, `createQueueSubmit`, `editSubmit`.

# 6. Uso de HTTP Verbs

O projeto utiliza uma estrutura simplificada de verbos:

*   **Route::get:** Utilizado para recuperação de páginas, formulários, ações de exclusão direta e restauração de registros.
*   **Route::post:** Utilizado exclusivamente para submissão de dados e processamento de formulários.
*   *Nota:* Não foi observado o uso de PUT, PATCH ou DELETE nos fontes fornecidos.

# 7. Golden Examples

### Exemplo 1: Fluxo de Autenticação (Guest)
```php
Route::middleware(['guest'])->group(function (){
    Route::get('/login', [AuthController::class, 'login'])->name('login');
    Route::post('/login', [AuthController::class, 'loginSubmit'])->name('login.submit');
});
```

### Exemplo 2: CRUD com Parâmetros e Submissão (Auth + Can)
```php
Route::middleware(['auth', 'can:sys-admin'])->group(function(){
    Route::get('/admin/company/create', [AdminController::class, 'createCompany'])->name('admin.company.create');
    Route::post('/admin/company/create', [AdminController::class, 'createCompanySubmit'])->name('admin.company.create.submit');
    Route::get('/admin/company/details/{id}', [AdminController::class, 'companyDetails'])->name('admin.company.details');
});
```

### Exemplo 3: Rotas de Controle e Gestão (Multi-parâmetros)
```php
Route::middleware(['auth', 'can:client'])->group(function(){
    Route::get('/queue/edit/{id}', [MainController::class, 'editQueue'])->name('queue.edit');
    Route::post('/queue/edit', [MainController::class, 'editQueueSubmit'])->name('queue.edit.submit');
    Route::get('/caller/queue-caller/{queue_id}/{ticket_id}/{status}', [TicketCallerController::class, 'queueCaller'])->name('caller.queue.caller');
});
```

### Exemplo 4: Middleware de Classe Customizada
```php
Route::middleware([TicketDispenserSession::class])->group(function (){
    Route::get('/dispenser', [TicketDispenserController::class, 'index'])->name('dispenser');
    Route::post('/dispenser/get-ticket', [TicketDispenserController::class, 'getTicket'])->name('dispenser.get.ticket');
});
```
