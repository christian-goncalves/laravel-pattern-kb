# KB_HELPERS_BLADE.md

Este documento define os padrões estruturais para o desenvolvimento de funções auxiliares (Helpers) e a organização de camadas de visualização (Blade), garantindo que a lógica de apresentação e formatação seja reutilizável e consistente.

# 1. Estrutura Geral de Helpers

*   **Localização:** As funções são definidas globalmente e protegidas para evitar colisões de nomes.
*   **Padrão de Nomes:** Utiliza-se **camelCase** (ex: `showValidationError`, `getFormattedTicketNumber`).
*   **Assinatura Típica:** Recebem tipos primitivos ou instâncias de modelos/coleções e retornam strings (frequentemente contendo HTML) ou valores formatados.
*   **Organização Interna:**
    *   **Wrapper de Segurança:** Toda função é envolvida por `if(!function_exists('nome_da_funcao'))`.
    *   **Mapeamento (Lookup Tables):** Uso de arrays internos para traduzir estados ou slugs em labels/ícones.
    *   **Early Return:** Retorno imediato quando condições de erro ou nulas são encontradas.
*   **Dependências Comuns:** Uso das Facades `Crypt`, `Auth`, `Str` e funções globais do Laravel como `asset()`, `public_path()` e `route()`.

### Funções "Core" Identificadas
*   **`showValidationError($fieldName, $validationErrors)`:** Retorna o HTML de erro para um campo específico se ele existir no objeto de erros.
*   **`getFormattedTicketNumber($number, $prefix, $digits)`:** Centraliza a lógica de formatação de identificadores numéricos com preenchimento à esquerda (`str_pad`).
*   **`getCompanyLogoImage($logo, $size)`:** Verifica a existência física do arquivo e retorna a tag `<img>` formatada.

# 2. Estrutura Geral de Blades

*   **Padrão de Layout:** As views são organizadas para estender layouts base, embora o conteúdo fornecido foque na estrutura interna das páginas de conteúdo.
*   **Organização do HTML:** Uso de comentários separadores para delimitar blocos lógicos da interface.
*   **Exibição de Erros e Mensagens:**
    *   **Validação:** Inserção de helpers via `{!! !!}` para renderizar HTML de erro logo abaixo dos inputs.
    *   **Sessão:** Verificação de mensagens globais via `@if(session()->has('message'))`.
    *   **Erros de Servidor:** Uso do helper `showServerError()` centralizado acima ou abaixo de campos críticos.
*   **Padrão de Formulários:**
    *   Obrigatório o uso de `@csrf` logo após a abertura da tag `<form>`.
    *   Atributos `name` dos inputs coincidem com as chaves de validação.
    *   URLs de destino geradas via `route()`.

# 3. Convenções de Dados vindos do Controller

*   **Variável `$subtitle`:** Esperada pela maioria das views para definir o título da página ou seção, geralmente extraída do array `$data['subtitle']` do controlador.
*   **Coleções Genéricas:** Dados de tabelas são passados como coleções. A view verifica a contagem antes de renderizar loops: `@if($collection->count() === 0)`.
*   **Segurança de IDs:** IDs passados em links de ação (Delete, Edit, Details) são obrigatoriamente criptografados na view: `Crypt::encrypt($item->id)`.

# 4. Golden Examples

### Exemplo: Helper Utilitário com Mapeamento
```php
if(!function_exists('getTicketStateText')) {
    function getTicketStateText($state) {
        $rules = [
            'waiting' => 'Aguardando',
            'called' => 'Atendido',
            'dismissed' => 'Dispensado'
        ];
        return $rules[$state] ?? 'Desconhecido';
    }
}
```

### Exemplo: Helper de Validação (Retorno HTML)
```php
if(!function_exists('showValidationError')){
    function showValidationError($fieldName, $validationErrors) {
        if($validationErrors->has($fieldName)){
            return '<p class="text-danger">' . $validationErrors->first($fieldName) . '</p>';
        }
        return '';
    }
}
```

### Exemplo: Formulário com Validação e Erros
```html
<form action="{{ route('auth.login.submit') }}" method="post">
    @csrf
    <label>Usuário</label>
    <input type="text" name="username">
    {!! showValidationError('username', $errors) !!}
    
    {!! showServerError() !!}
    
    <button type="submit">Entrar</button>
</form>
```

### Exemplo: Estrutura de Listagem com Condicional e Criptografia
```html
@if($items->count() === 0)
    <p>Não existem registros.</p>
@else
    @foreach($items as $item)
        <span>{{ $item->name }}</span>
        <a href="{{ route('item.edit', ['id' => Crypt::encrypt($item->id)]) }}">Editar</a>
    @endforeach
@endif
```
