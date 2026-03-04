
# KB_MODELS.md

Este documento define os padrões estruturais e as convenções de implementação para as classes de Model do projeto, servindo como guia para a criação de novos recursos.

# 1. Estrutura Geral de Models
*   **Namespace:** Todos os modelos utilizam o namespace `App\Models`.
*   **Imports típicos:** 
    *   `Illuminate\Database\Eloquent\Model` para modelos padrão.
    *   `Illuminate\Foundation\Auth\User as Authenticable` para modelos de autenticação.
    *   `Illuminate\Database\Eloquent\SoftDeletes` para suporte a exclusão lógica.
*   **Uso de Traits:** A trait `SoftDeletes` é declarada obrigatoriamente dentro do corpo da classe para habilitar o controle de registros deletados.
*   **Herança:** As classes herdam de `Model` ou, no caso do modelo de usuário, de `Authenticable`.
*   **Propriedades de Classe:** O projeto utiliza as convenções padrão do Eloquent para `$table` e `$primaryKey` (implícitos), não havendo declarações explícitas de `$fillable`, `$guarded` ou `$casts` nos fontes fornecidos.

# 2. Convenções de Nome e Chaves
*   **Chaves Estrangeiras:** Seguem o padrão de prefixo `id_` seguido pelo nome da entidade relacionada no singular (ex: `id_company`, `id_queue`).
*   **Declaração de Relacionamentos:** A chave estrangeira é sempre declarada de forma explícita como o segundo argumento nos métodos de relacionamento.
*   **Nomenclatura de Métodos:**
    *   **Singular:** Para relacionamentos `belongsTo` que retornam uma única instância (ex: `company()`, `queue()`).
    *   **Plural:** Para relacionamentos `hasMany` que retornam uma coleção (ex: `users()`, `queues()`, `tickets()`).

# 3. Soft Deletes e Campos de Data
*   **Soft Deletes:** O padrão é adotado em todos os modelos através da trait `SoftDeletes`.
*   **Campo deleted_at:** Os modelos esperam a existência do campo `deleted_at` do tipo `dateTime`, configurado como anulável por padrão.
*   **Campos de Data:** O projeto utiliza `created_at` e `updated_at` gerenciados automaticamente, definidos nas migrations como `dateTime`.

# 4. Relacionamentos
Padrões reais encontrados com declaração explícita de chaves:

*   **belongsTo:**
    ```php
    public function company()
    {
        return $this->belongsTo(Company::class, 'id_company');
    }
    ```

*   **hasMany:**
    ```php
    public function tickets()
    {
        return $this->hasMany(QueueTicket::class, 'id_queue');
    }
    ```

*   **belongsToMany:** Não encontrado no padrão atual das fontes.
*   **whereHas / contagens:** Não encontrado no padrão atual das fontes.

# 5. Scopes e Métodos de Domínio no Model
*   **Scopes:** Não encontrado no padrão atual das fontes.
*   **Métodos Utilitários:** Não encontrado no padrão atual das fontes.

# 6. Política de Acesso e Carregamento
*   **Uso de withTrashed:** Embora a trait seja declarada nos models, o uso de `withTrashed()` ou `onlyTrashed()` para filtragem de relações não aparece definido dentro das estruturas das classes de modelo nas fontes.
*   **Eager Loading ($with):** Não há uso da propriedade `$with` para carregamento automático de relações nas fontes analisadas.

# 7. Golden Examples (Exemplos Canônicos)

### Model: `Queue`
Exemplifica o uso padrão de herança, SoftDeletes e o bicefalo de relacionamentos (pai e filhos) com chaves explícitas.
```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Queue extends Model
{
    use SoftDeletes;

    public function company()
    {
        return $this->belongsTo(Company::class, 'id_company');
    }

    public function tickets()
    {
        return $this->hasMany(QueueTicket::class, 'id_queue');
    }
}
```

### Model: `User`
Exemplifica a herança de `Authenticable` e a integração do padrão de chaves do projeto em modelos de autenticação.
```php
namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticable;
use Illuminate\Database\Eloquent\SoftDeletes;

class User extends Authenticable
{
    use SoftDeletes;

    public function company()
    {
        return $this->belongsTo(Company::class, 'id_company');
    }
}
```

### Model: `Company`
Exemplifica um modelo centralizador com múltiplos relacionamentos de saída (`hasMany`) todos utilizando a convenção de FK explícita.
```php
class Company extends Model
{
    use SoftDeletes;

    public function users()
    {
        return $this->hasMany(User::class, 'id_company');
    }

    public function queues()
    {
        return $this->hasMany(Queue::class, 'id_company');
    }
}
```
