# Documentação de Uso para AI — Template

Todo componente DEVE ter UM ÚNICO arquivo `.md`. Este documento define o formato
padrão e as regras para geração dessa documentação.

---

## Regras Fundamentais

### Nomenclatura do Arquivo

O nome do arquivo é a conversão do nome da classe JS de PascalCase para kebab-case.
Cada letra maiúscula (exceto a primeira) é precedida por hífen, tudo minúsculo.

```
PasswordField   → password-field.md
Accordion       → accordion.md
DaterangePicker → daterange-picker.md
SelectAsync     → select-async.md
UploadMultipleFiles → upload-multiple-files.md
```

### Propósito

Instruções para agentes AI (Claude Code) usarem o componente.
**Não é documentação para desenvolvedores** — é referência compacta, sem didática,
otimizada para economia de tokens.

### Um Arquivo, Uma Documentação

- NUNCA gerar documentos separados para componentes auxiliares.
- NUNCA criar seções independentes que tratem o componente auxiliar como autônomo.
- A documentação é sobre o **componente principal**. Componentes auxiliares são documentados
  DENTRO do contexto do componente principal, como extensão.

---

## Template Padrão

Copie e preencha o esqueleto abaixo ao gerar a documentação de um componente.

````markdown
# {Nome Separado} Component

> {Descrição em uma linha do que o componente faz.}

## Uso Básico

```blade
<x-{nome-kebab}
    {prop-obrigatória}="{valor}"
/>
```

## Props

| Prop | Tipo | Default | Descrição |
|---|---|---|---|
| `{prop}` | `{tipo}` | `{default}` | {descrição curta} |

## Slots

| Slot | Descrição |
|---|---|
| `{slot}` | {descrição curta} |

> Omitir seção se não houver slots.

## Data Attributes (JS)

| Atributo | Elemento | Função |
|---|---|---|
| `data-{nome-arquivo}-root` | Root | Inicialização do componente |
| `data-{nome-arquivo}-{atributo}` | {elemento} | {função} |

## Variações

### {Nome da Variação}

```blade
<x-{nome-kebab}
    {props-da-variação}
/>
```

> Omitir seção se não houver variações relevantes.

## Componentes Auxiliares

> Documentar aqui SOMENTE se o componente principal usa sub-componentes.
> Cada auxiliar é uma subseção, nunca um documento separado.

### x-{auxiliar}

```blade
<x-{componente-principal}>
    <x-{auxiliar} {props} />
</x-{componente-principal}>
```

| Prop | Tipo | Default | Descrição |
|---|---|---|---|
| `{prop}` | `{tipo}` | `{default}` | {descrição curta} |

## Exemplos Compostos

### {Cenário}

```blade
{exemplo completo com múltiplas props e/ou slots}
```

## Notas

- {Restrição, dependência ou comportamento não óbvio.}
````

---

## Exemplo Preenchido — Password Field

````markdown
# Password Field Component

> Campo de senha com geração aleatória, cópia, toggle de visibilidade e medidor de força.

## Uso Básico

```blade
<x-password-field name="password" />
```

## Props

| Prop | Tipo | Default | Descrição |
|---|---|---|---|
| `label` | `?string` | `null` | Label do campo. Sufixo `*` ativa indicador de obrigatório. |
| `icon` | `string` | `'key'` | Ícone à esquerda do campo. |
| `size` | `?string` | `null` | `'sm'` para versão compacta. |
| `generate` | `bool` | `false` | Habilita botão de geração de senha. |
| `copy` | `bool` | `false` | Habilita botão de cópia. |
| `eye` | `bool` | `false` | Habilita toggle de visibilidade. |
| `meter` | `bool` | `false` | Habilita medidor de força. |
| `maxlength` | `int` | `100` | Limite de caracteres. |
| `autocomplete` | `string` | `'off'` | Valor do atributo autocomplete. |
| `disabled` | `bool` | `false` | Desabilita o campo. |
| `col` | `?int` | `null` | Span de coluna no grid (`md:col-span-{n}`). |

## Data Attributes (JS)

| Atributo | Elemento | Função |
|---|---|---|
| `data-password-field-root` | Root `<div>` | Inicialização do componente |
| `data-password-field-input` | `<input>` | Campo de senha |
| `data-password-field-box` | `<div>` | Container do campo + botões |
| `data-password-field-generate` | `<button>` | Dispara geração de senha |
| `data-password-field-copy` | `<button>` | Dispara cópia |
| `data-password-field-eye` | `<div>` | Ícone olho aberto |
| `data-password-field-eye-slash` | `<div>` | Ícone olho fechado |
| `data-password-field-meter` | `<div>` | Container do medidor |
| `data-password-field-meter-box` | `<div>` | Grid das barras do medidor |

## Variações

### Compacto

```blade
<x-password-field name="pin" size="sm" :maxlength="6" />
```

### Completo

```blade
<x-password-field
    name="password"
    label="Senha *"
    :generate="true"
    :copy="true"
    :eye="true"
    :meter="true"
/>
```

## Notas

- Geração usa `crypto.getRandomValues` (não `Math.random`).
- Composição fixa: 3 maiúsculas, 3 minúsculas, 4 números, 2 especiais.
- Medidor aparece ao digitar e oculta ao clicar fora do campo.
````
