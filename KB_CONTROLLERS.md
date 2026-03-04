# KB_CONTROLLERS.md

Este documento descreve os padrões estruturais e convenções arquiteturais aplicados nos controladores da aplicação, garantindo consistência no fluxo de dados e segurança na manipulação de recursos.

## 1. Padrão de Comunicação com Views (`$data`)
Os controladores não passam variáveis soltas para as views. Utiliza-se um **array associativo padronizado** chamado `$data`, que centraliza todas as informações necessárias para a renderização, incluindo metadados como o `subtitle`.

```php
public function index()
{
    $data = [
        'subtitle' => 'Título da Página',
        'collection' => $this->getInternalData()
    ];
    return view('folder.view_name', $data);
}
```

## 2. Convenções de Métodos de Submissão
Métodos que processam requisições de alteração de estado (POST, PUT, DELETE) seguem a nomenclatura **`actionNameSubmit`**. Essa convenção diferencia claramente o carregamento do formulário (GET) do processamento dos dados.

## 3. Padrão de Validação
A validação é executada diretamente no controlador via `$request->validate`, separando as regras de negócio das mensagens de erro customizadas para garantir uma interface amigável.

*   **Estrutura:** Um array para regras e um segundo array para mensagens.
*   **Tratamento de Erros:** Em caso de falha, o fluxo é interrompido automaticamente pelo Laravel, retornando os erros para a sessão.

## 4. Tratamento de IDs e Segurança
IDs expostos em URLs ou formulários são obrigatoriamente protegidos com **`Crypt::decrypt`**. O tratamento de erro na descriptografia varia conforme o contexto:

*   **Redirect:** Utilizado em áreas administrativas para retornar à listagem em caso de ID inválido.
*   **Abort:** Utilizado quando o recurso é essencial e a falha indica manipulação maliciosa (ex: `abort(403)` ou `abort(404)`).
*   **Null:** Utilizado em métodos auxiliares privados para validação interna.

## 5. Padrões de Queries (Eloquent)
A persistência e recuperação de dados seguem padrões específicos para garantir integridade:

*   **`find($id)` ou `first()`:** Consultas simples onde a existência é validada manualmente por condicionais.
*   **`firstOrFail()`:** Utilizado para garantir que o fluxo seja interrompido caso o recurso não exista ou o usuário não tenha acesso.
*   **Soft Deletes:** Uso de `withTrashed()` para recuperar registros excluídos logicamente e `restore()` para reativação.
*   **Agregações Dinâmicas:** Uso de `withCount` com subqueries para calcular totais de registros relacionados sem carregar as coleções completas na memória.

## 6. Padrão de Upload de Arquivos
O processo de upload segue um fluxo rígido de segurança e organização:
1.  Verificação da existência do arquivo (`hasFile`).
2.  Geração de nome único via **`Str::uuid()`** para evitar colisões.
3.  Armazenamento físico usando `move(public_path(...))` para garantir acesso público controlado.

## 7. Golden Examples

### Exemplo: `createCompanySubmit` (Fluxo de Criação e Upload)
```php
public function createCompanySubmit(Request $request)
{
    $request->validate([/* regras */], [/* mensagens */]); //

    if($request->hasFile('company_logo')) {
        $fileName = Str::uuid() . '.' . $request->company_logo->extension();
        $request->company_logo->move(public_path('assets/images/path'), $fileName);
        $logo = $fileName;
    } //

    $model = new Company();
    $model->field = $request->field;
    $model->save(); //

    return view('admin.success', $data); //
}
```

### Exemplo: `createQueueSubmit` (Validação Complexa e Unicidade)
```php
public function createQueueSubmit(Request $request)
{
    $request->validate([
        'name' => 'required|min:5|max:100',
        'status' => 'required|in:active,inactive',
    ], [/* mensagens */]); //

    $exists = Model::where('field', $request->field)->exists();
    if($exists){
        return redirect()->back()->withInput()->with(['server_error' => 'Mensagem']);
    } //

    $new = new Model();
    $new->fill($request->all());
    $new->save(); //
}
```

### Exemplo: Helpers Privados (`ClientAdminController`)
Padrão de encapsulamento para validação de escopo e descriptografia.
```php
private function decryptUserId($id)
{
    try {
        return Crypt::decrypt($id);
    } catch (\Exception $e) {
        return null;
    }
}

private function checkUserIsValid($id)
{
    return User::withTrashed()
        ->where('id', $id)
        ->where('context_id', Auth()->user()->context_id)
        ->first();
}
```

---
> **Nota de Referência:** O `AuthController` deve ser utilizado como exemplo de **lógica funcional** (autenticação, gestão de sessão e hashes), porém sua formatação pode divergir levemente dos padrões visuais de `$data` definidos acima.
