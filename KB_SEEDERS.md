# KB_SEEDERS.md

Este documento define os padrões estruturais e as convenções de implementação para os Seeders do projeto, garantindo uma alimentação de banco de dados consistente para ambientes de desenvolvimento e teste.

# 1. Estrutura Geral dos Seeders

*   **Namespace:** `Database\Seeders`.
*   **Imports Utilizados:**
    *   `Illuminate\Database\Seeder` (Classe base).
    *   `Illuminate\Support\Facades\DB` (Query Builder para inserção).
    *   `Illuminate\Support\Str` (Geração de strings e UUIDs).
*   **Estrutura da Classe:** As classes estendem `Seeder` e implementam o método obrigatório `run()`.
*   **Método `run()`:** Declarado com o tipo de retorno `void`.

# 2. Padrão de Inserção de Dados

O projeto utiliza predominantemente o **Query Builder** através da Facade `DB`, em vez de modelos Eloquent.

*   **Comando:** `DB::table('nome_tabela')->insert($data);`.
*   **Inserção em Lote:** O padrão é acumular registros em um array e realizar uma única operação de `insert` ao final do loop ou método.
*   **Limpeza de Dados:** Em seeders de massa de dados, utiliza-se `DB::table('tabela')->truncate();` no início do método `run()` para garantir um estado limpo.

# 3. Estrutura dos Arrays de Dados

*   **Construção Dinâmica (Loops):** Para gerar múltiplos registros, utiliza-se um loop `for` que popula um array temporário.
*   **Definição Estática:** Para usuários ou registros de configuração fixa, utiliza-se um array multidimensional direto.
*   **Organização de Campos:** Chaves em *snake_case* correspondentes às colunas da migration.
*   **Feedback no Console:** Uso de comandos `echo` para informar a quantidade de registros criados durante a execução.

# 4. Uso de Helpers e Funções

*   **Strings e Identificadores:**
    *   `Str::uuid()` para campos de identificação única.
    *   `Str::random(64)` para geração de hashes e tokens.
*   **Segurança:** Uso da função global `bcrypt()` para encriptação de senhas.
*   **Dados Complexos:** Uso de `json_encode()` para armazenar arrays ou objetos em colunas do tipo JSON.
*   **Timestamps:** Atribuição manual de `now()` para as colunas `created_at` e `updated_at`, e `null` para `deleted_at`.

# 5. Convenções de Execução

Embora o arquivo `DatabaseSeeder.php` não esteja detalhado, os seeders individuais demonstram dois padrões de organização:
*   **Seeders Atômicos:** Focados em uma única tabela (ex: `UserSeeder`, `CompanySeeder`).
*   **Seeders de Relacionamento:** Métodos privados dentro da classe (ex: `generateQueues()`, `generateBundles()`) para organizar a criação de dados que dependem de chaves estrangeiras entre múltiplas tabelas no mesmo fluxo.

# 6. Golden Examples

### Exemplo 1: Inserção em Lote com Loop (`CompanySeeder`)
Demonstra a criação dinâmica de registros com UUID e timestamps manuais.
```php
public function run(): void
{
    $companies = [];
    for($index = 1; $index <= 3; $index++) {
        $companies[] = [
            'company_name' => 'Empresa ' . $index,
            'uuid' => Str::uuid(),
            'created_at' => now(),
            'updated_at' => now(),
            'deleted_at' => null,
        ];
    }
    DB::table('companies')->insert($companies);
    echo count($companies) . " empresas criadas.\n";
}
```

### Exemplo 2: Dados Estáticos e Segurança (`UserSeeder`)
Demonstra a definição manual de registros e uso de `bcrypt`.
```php
public function run(): void
{
    $users = [
        [
            'email' => 'admin@localhost.com',
            'password' => bcrypt('password123'),
            'role' => 'sys-admin',
            'active' => true,
        ],
    ];
    DB::table('users')->insert($users);
}
```

### Exemplo 3: Limpeza e Organização Interna (`QueueAndBundleSeeder`)
Demonstra o uso de `truncate` e a divisão do seeder em métodos privados para clareza estrutural.
```php
public function run(): void
{
    DB::table('table_name')->truncate();
    $data = $this->generateData();
}

private function generateData()
{
    // Lógica de preparação do array
    return $array;
}
```
