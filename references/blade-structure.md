# Blade — Estrutura do Template (Laravel 13.x)

Referência completa para a refatoração de templates Blade.
Carregar na Fase 3 do workflow.

**Versão alvo:** Laravel 13.x (PHP 8.3+, preferencialmente PHP 8.4).

---

## Bloco @php Centralizado

Toda lógica fica em um único `@php` no topo, após `@props`:

```blade
@props([
    'label' => null,
    'size' => null,
    'disabled' => false,
    'col' => null,
])

@php
    // 1. Booleanos de conveniência
    $isSmall = $size === 'sm';

    // 2. Classes Tailwind pré-calculadas
    $iconSize = $isSmall ? 'size-[16px]' : 'size-[19px]';
    $inputHeight = $isSmall ? 'h-8 text-[14px]' : 'h-10 text-[15px]';

    // 3. Valores derivados
    $colSpan = $col ? 'md:col-span-' . $col : '';

    // 4. Merge de atributos
    $inputAttr = $attributes->merge([...])->except('class');
@endphp
```

Nunca declare blocos `@php` adicionais espalhados pelo template. Criar variáveis booleanas
de conveniência (como `$isSmall`) evita ternários repetidos no HTML e torna o template
mais legível.

---

## Diretiva @class vs Interpolação

Use `@class` para classes condicionais:

```blade
<button @class([
    'form-input-icon-start',
    $iconBoxSize,
    'cursor-pointer' => $generate,
])>
```

Use interpolação `{{ }}` quando houver valores arbitrários com aspas duplas
(ex: `after:content-["*"]`) que quebrariam o `@class`.

---

## Boas Práticas no Template

- Comentários `{{-- --}}` para marcar seções visuais
- Elementos repetitivos substituídos por `@for`
- Prop `col` com concatenação `'md:col-span-' . $col` para grid layout
- Data attributes condicionais com `@if` inline: `@if ($generate) data-password-field-generate @endif`

---

## Atenção com x-icon e Data Attributes

Componentes Blade como `<x-icon>` podem não repassar data attributes arbitrários para o
HTML renderizado. Se o JS precisa encontrar um elemento via `querySelector('[data-password-field-check]')`,
envolva o componente em um `<span>` com o data attribute:

```blade
{{-- ✅ Correto — data attribute no span, querySelector encontra --}}
<span data-password-field-check @class(['hidden' => !$isOptionSelected])>
    <x-icon name="check" class="size-4 fill-blue-600 dark:fill-blue-400" />
</span>

{{-- ❌ Errado — data attribute no x-icon pode não chegar ao HTML --}}
<x-icon name="check" data-password-field-check @class([
    'size-4 fill-blue-600 dark:fill-blue-400',
    'hidden' => !$isOptionSelected,
]) />
```

Para que `<x-icon>` repasse atributos `data-*` ao HTML, o template do componente deve
incluir `{{ $attributes->except('class') }}` no elemento wrapper:

```blade
{{-- Componente x-icon --}}
@props(['name', 'directory' => 'app'])

@php
    $svgIcon = appUtility()->getSvgIcon($name, $directory, $attributes->get('class'));
@endphp

<span {{ $attributes->except('class') }}>{!! $svgIcon !!}</span>
```

---

## Nota: PHP Attributes no Laravel 13.x

O Laravel 13 introduziu PHP Attributes nativos (`#[Fillable]`, `#[Hidden]`, `#[Table]`, etc.)
como alternativa a propriedades de classe em Models, Jobs, Commands, etc. Isso **não afeta**
componentes Blade anônimos (que usam `@props`), mas é relevante se o componente tiver uma
classe PHP associada (`App\View\Components\*`).

Para componentes Blade com classe, o Laravel 13.x mantém o mesmo padrão:

```php
// Componente com classe — Laravel 13.x
class Alert extends Component
{
    public function __construct(
        public string $type,
        public string $message,
    ) {}

    public function render(): View
    {
        return view('components.alert');
    }
}
```

Os PHP Attributes do Laravel 13 são **opcionais** e focados em Models/Jobs/Commands.
Para componentes Blade, a recomendação do projeto permanece: usar **componentes anônimos**
com `@props` sempre que possível, reservando classes PHP apenas quando lógica complexa for
necessária.
