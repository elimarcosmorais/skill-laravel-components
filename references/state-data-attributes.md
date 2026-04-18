# Estado via Data Attributes — Separação Estado vs Apresentação

Carregar quando o componente possuir estados visuais multi-valor (selecionado, highlight, ativo).

---

## O Problema

Quando o JavaScript manipula diretamente dezenas de classes CSS para representar estados
visuais (selecionado, hover, highlight, ativo, desabilitado), cada alteração pode conflitar
com outra. Classes adicionadas por um fluxo não são removidas por outro, classes do Blade
divergem das classes do JS, e o resultado é um ciclo de bugs cascata.

## O Princípio

**JS gerencia estado (data attributes) → CSS gerencia apresentação (Tailwind).**

O JavaScript nunca deve adicionar ou remover classes de cor, fundo ou tipografia para
representar mudanças de estado em elementos que possuem múltiplos estados visuais.

---

## Quando Aplicar

| Cenário                                                             | Técnica                                         |
| ------------------------------------------------------------------- | ----------------------------------------------- |
| Toggle binário simples (mostrar/ocultar)                            | `classList.toggle('hidden', booleano)`          |
| Estado visual com múltiplos estados (selecionado, highlight, ativo) | Data attribute + Tailwind `data-[*]:`           |
| Animações de abertura/fechamento (opacity, scale)                   | `classList.add/remove` de classes de animação   |
| Cores de medidor/progresso com níveis                               | `element.className = template` (reset completo) |

**Regra prática:** Se o JS precisa gerenciar mais de 2 classes CSS que representam o mesmo
tipo de informação visual em elementos que podem estar em estados diferentes — use data attributes.

---

## Convenção de Nomenclatura

Data attributes de estado usam o nome completo do componente seguido de um nome semântico:

```
data-{nome-arquivo}-{estado}

Exemplos:
data-select-styled-active      → item selecionado no SelectStyled
data-select-styled-highlight   → item focado via teclado no SelectStyled
data-accordion-expanded        → seção aberta no Accordion
data-tabs-active               → aba ativa no Tabs
```

---

## Como Funciona — JavaScript

O JS apenas alterna a presença/ausência do data attribute:

```javascript
// ✅ Correto — JS gerencia estado via data attributes
#selectOption(selectedOption) {
    this.#options.forEach((option) => {
        if (option === selectedOption) {
            option.setAttribute('data-select-styled-active', '');
        } else {
            option.removeAttribute('data-select-styled-active');
        }
    });
}

#highlightOption(index, visibleOptions) {
    visibleOptions.forEach((option) =>
        option.removeAttribute('data-select-styled-highlight'),
    );

    if (index >= 0 && index < visibleOptions.length) {
        visibleOptions[index].setAttribute('data-select-styled-highlight', '');
    }
}

#clearSelection() {
    this.#options.forEach((option) =>
        option.removeAttribute('data-select-styled-active'),
    );
}
```

```javascript
// ❌ Errado — JS manipula classes de cor/fundo diretamente
#selectOption(selectedOption) {
    this.#options.forEach((option) => {
        const isSelected = option === selectedOption;
        option.classList.toggle('bg-gray-100', isSelected);
        option.classList.toggle('dark:bg-zinc-875', isSelected);
        option.classList.toggle('text-blue-600', isSelected);
        option.classList.toggle('dark:text-blue-400', isSelected);
    });
}
```

---

## Como Funciona — Blade (Tailwind CSS)

O Blade declara todos os estilos condicionais usando os seletores `data-[*]:` do Tailwind.
O Blade também define o estado inicial via `@if`:

```blade
{{-- ✅ Correto — estado via data attributes, visual via Tailwind --}}
<li data-select-styled-option
    @if ($isOptionSelected) data-select-styled-active @endif
    class="flex items-center px-3.5 py-2.5 cursor-pointer
           hover:bg-gray-100 dark:hover:bg-zinc-875
           data-select-styled-active:bg-gray-100 dark:data-select-styled-active:bg-zinc-875
           data-select-styled-active:text-blue-600 dark:data-select-styled-active:text-blue-400
           data-select-styled-highlight:bg-gray-100 dark:data-select-styled-highlight:bg-zinc-875">
```

```blade
{{-- ❌ Errado — classes condicionais via @class que divergem do JS --}}
<li data-select-styled-option @class([
    'flex items-center px-3.5 py-2.5 cursor-pointer',
    'bg-gray-100 dark:bg-zinc-900 text-blue-600 dark:text-blue-400' => $isOptionSelected,
    'hover:bg-gray-100 dark:hover:bg-zinc-875' => !$isOptionSelected,
])>
```

---

## Tabela de Correspondência

| Ação            | JS faz                                             | CSS renderiza                                                         |
| --------------- | -------------------------------------------------- | --------------------------------------------------------------------- |
| Selecionar item | `setAttribute('data-select-styled-active', '')`    | `data-[select-styled-active]:bg-* data-[select-styled-active]:text-*` |
| Navegar teclado | `setAttribute('data-select-styled-highlight', '')` | `data-[select-styled-highlight]:bg-*`                                 |
| Hover mouse     | Nada (puro CSS)                                    | `hover:bg-*`                                                          |
| Limpar seleção  | `removeAttribute('data-select-styled-active')`     | Classes reativas desaparecem automaticamente                          |
| Mostrar/ocultar | `classList.toggle('hidden')`                       | Classe utilitária direta (não é estado multi-valor)                   |

---

## Benefícios

1. **Fonte única de verdade** — o estado está no DOM como atributo inspecionável
2. **Sem conflito Blade ↔ JS** — ambos usam o mesmo mecanismo (data attribute)
3. **Hover puro CSS** — nunca interferido pelo JS, funciona sempre
4. **Depuração fácil** — DevTools mostra `data-select-styled-active` no elemento
5. **Manutenção segura** — alterar estilo visual = editar apenas o Blade, sem tocar no JS

---

## Exceção: classList.toggle para Hidden

O `classList.toggle('hidden', booleano)` continua sendo a técnica correta para
visibilidade binária (mostrar/ocultar elementos como check icons, placeholders,
containers de dropdown):

```javascript
// ✅ Correto — hidden é toggle binário simples
const checkIcon = option.querySelector("[data-select-styled-check]");
if (checkIcon) {
  checkIcon.classList.toggle("hidden", !isSelected);
}
```
