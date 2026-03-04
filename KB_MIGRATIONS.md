
# KB_MIGRATIONS.md

Este documento define os padrões estruturais e as convenções de implementação para as migrations do banco de dados, assegurando a uniformidade na criação de tabelas e colunas.

# 1. Estrutura Geral das Migrations

*   **Namespace:** Não é utilizado namespace nos arquivos de migration.
*   **Uso de Imports:** As migrations utilizam obrigatoriamente os seguintes componentes:
    *   `Illuminate\Database\Migrations\Migration`
    *   `Illuminate\Database\Schema\Blueprint`
    *   `Illuminate\Support\Facades\Schema`.
*   **Estrutura da Classe:** Utiliza-se o padrão de **Classes Anônimas** que estendem a classe base `Migration`.
*   **Padrão de Métodos:** Os métodos `up()` e `down()` devem possuir o tipo de retorno `void` declarado explicitamente.

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('table_name', function (Blueprint $table) {
            // definições de colunas
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('table_name');
    }
};
```

# 2. Definições de Tipos e Colunas

O projeto adota tipos de dados específicos para garantir compatibilidade e performance:

*   **ID Primário:** `$table->id()` é utilizado como padrão em todas as tabelas.
*   **Strings:** Declaração explícita de tamanho máximo (ex: `$table->string('field', 100)`).
*   **Inteiros:** Utilizado para chaves estrangeiras e contadores (ex: `$table->integer('field')`).
*   **Enums:** Utilizado para estados e papéis de usuário (ex: `$table->enum('field', ['val1', 'val2'])`).
*   **JSON:** Utilizado para colunas de dados flexíveis (ex: `$table->json('field')`).
*   **UUID:** Utilizado para identificadores únicos universais (ex: `$table->uuid('uuid')->unique()`).
*   **Boolean:** Utilizado para flags (ex: `$table->boolean('field')->default(false)`).

# 3. Padrão de Timestamps e Soft Deletes

As migrations não utilizam o método `$table->timestamps()`. Em vez disso, as colunas de data são declaradas manualmente com o tipo `dateTime` para controle granular:

*   **created_at:** `$table->dateTime('created_at')->useCurrent();`.
*   **updated_at:** `$table->dateTime('updated_at')->useCurrent()->useCurrentOnUpdate();`.
*   **deleted_at:** `$table->dateTime('deleted_at')->nullable()->default(null);` (para suporte a SoftDeletes).

# 4. Chaves Estrangeiras e Índices

*   **Chaves Estrangeiras:** Não são utilizadas restrições de `foreignId`. As relações são estabelecidas por colunas `integer` seguindo o prefixo `id_`.
*   **Índices:** Chaves estrangeiras e campos de busca frequente devem receber o método `->index()`.
*   **Unicidade:** Campos de identificação (e-mails, nomes únicos, hashes, uuids) devem receber o modificador `->unique()`.

# 5. Convenções de Nomenclatura

*   **Tabelas:** Nomes em **plural** (ex: `users`, `companies`, `queues`, `queue_tickets`, `bundles`).
*   **Colunas:** Snake case (ex: `company_name`, `queue_ticket_status`).
*   **Prefixos de Relação:** Campos que referenciam outras tabelas utilizam `id_` + nome da tabela no singular (ex: `id_company`, `id_queue`).
