---
name: laravel-components
description: >
  ONLY activate this skill when the user explicitly requests it by name (e.g. 'use laravel-components',
  'aplique laravel-components', 'skill laravel-components'). Do NOT auto-trigger based on context
  inference. This skill creates new JavaScript + Blade (Laravel) components and refactors existing
  ones, following patterns with ES2025 classes (private fields/methods with #, Iterator Helpers,
  RegExp.escape, Promise.try), data attributes with full name, bootstrap with MutationObserver,
  and Blade templates with centralized @php. Includes JSDoc conventions to eliminate PhpStorm
  warnings (@typedef, @type, @types/*, noinspection) and .md documentation generation for AI agents.
  Always generates both files (JS + Blade) together, and optionally the .md documentation.
---

# Criação e Refatoração de Componentes — JavaScript + Blade

Cria novos componentes e refatora componentes existentes que possuem arquivos JavaScript e Blade,
aplicando padrões consistentes de organização, encapsulamento e nomenclatura. O contexto é PHP 8.4,
Laravel 13.x, JavaScript ES2025 e Tailwind CSS 4.

---

## Contexto de Versões

- **Laravel 13.x** (lançado em 17/03/2026) — PHP 8.3+ mínimo, PHP Attributes nativos
  como alternativa a propriedades de classe, zero breaking changes no Blade.
- **JavaScript ES2025** (ECMAScript 2025, ratificado em junho 2025) — Iterator Helpers,
  Set methods, `Promise.try()`, `RegExp.escape()`, Import Attributes, Duplicate Named
  Capture Groups. Todos já suportados nos navegadores modernos.

---

## Workflow — Fases

### Fase 0 — Plano de Trabalho (OBRIGATÓRIA)

Antes de criar ou alterar qualquer arquivo, o agente deve:

1. Apresentar um plano de trabalho detalhando:
   - Arquivos que serão criados ou modificados
   - Features que serão implementadas ou alteradas
   - Registro no `loader.js` (se houver arquivo JS)
   - Documentação que será gerada
2. **Aguardar autorização explícita do usuário** — o agente NÃO pode avançar para as fases seguintes sem confirmação.

Só após aprovação do usuário o agente prossegue com as fases de implementação.

### Fase 1 — Análise
Identificar se é criação de novo componente ou refatoração de existente. Para refatoração, receber
o código JS + Blade existente do usuário. Para criação, receber a descrição e features do novo
componente. Em ambos os casos, confirmar o nome do componente e suas features.

### Fase 2 — Implementação JS (carregar sob demanda)
Consultar **`references/js-structure.md`** para a estrutura completa de classes, encapsulamento,
bootstrap e convenções de código (inclui features ES2025).

Se o componente tem estados visuais multi-valor (selecionado, highlight, ativo), consultar
também **`references/state-data-attributes.md`** para o padrão Estado via Data Attributes.

Se o componente consome APIs externas (Google Maps, Chart.js, Leaflet, etc.) ou recebe
configuração via `JSON.parse`, consultar **`references/phpstorm-jsdoc.md`** para
`@typedef`, `@type`, instalação de `@types/*` e supressão de avisos da IDE.

### Fase 3 — Implementação Blade (carregar sob demanda)
Consultar **`references/blade-structure.md`** para o padrão de templates Blade, @props,
@php centralizado e boas práticas.

### Fase 4 — Documentação (carregar sob demanda)
Consultar **`references/component-doc-template.md`** para gerar a documentação .md
do componente, otimizada para agentes AI (Claude Code). O arquivo deve ser salvo em
**`.claude/docs/components/{nome-kebab-case}.md`** dentro do projeto.

### Fase 5 — Registro no loader.js
Se o componente possui arquivo JavaScript, declarar o módulo no arquivo `resources/js/loader.js`,
adicionando a entrada no objeto `LAZY_MODULES` com o seletor `COMPONENT_SELECTOR` do componente:

```js
"[data-nome-do-componente-root]": () =>
    import("./components/{categoria}/{nome-do-arquivo}"),
```

Consultar o conteúdo atual do `loader.js` para agrupar corretamente o novo módulo com os
demais da mesma categoria (Inputs, Modals, etc.).

### Fase 6 — Validação e Entrega
Validar contra o checklist final. Entregar os arquivos juntos (JS + Blade + documentação .md).
Para referência completa de um componente canônico, consultar
**`references/password-field-example.md`**.

---

## Regras Invioláveis

Estas regras se aplicam a TODOS os componentes — consulte os references para detalhes.

**Geral:**
1. **Plano de trabalho obrigatório** — apresentar o plano e aguardar aprovação do usuário antes de criar ou alterar qualquer arquivo
2. **Comentários em português do Brasil** — todos os DocBlocks, comentários de classe, método e função devem ser escritos em português do Brasil
3. **Registro no `loader.js`** — todo componente com arquivo JS deve ser declarado em `resources/js/loader.js` no objeto `LAZY_MODULES`

**JavaScript (ES2025):**
1. **Classe ES2025** encapsulando tudo (sem funções soltas no escopo global)
2. **Private fields `#`** para todas as propriedades DOM, `#init()`, `#setup*` e auxiliares
3. **Apenas `this.root` público** (e constructor por natureza)
4. **`COMPONENT_SELECTOR`** com nome completo kebab-case: `[data-{nome-arquivo}-root]`
5. **Data attributes internos** com nome completo: `data-{nome-arquivo}-{nome-do-atributo}`
6. **`root.querySelector`** (nunca `document.querySelector`) — escopo isolado evita colisões
7. **Guard clause** no início de cada `#setup*` validando elementos necessários
8. **Bootstrap**: mount + MutationObserver + guard `readyState` (compatível com lazy load e eager load)
9. **Estados visuais multi-valor**: `setAttribute/removeAttribute` — nunca `classList` de cores
10. **`classList.toggle('hidden', bool)`** apenas para toggles binários simples
11. **`@typedef` para configs parseadas** — sempre tipar objetos vindos de `JSON.parse` com `/** @type */`
12. **Iterator Helpers** — preferir `.filter()`, `.map()`, `.take()` em iteradores quando processar
    coleções DOM (ex: `NodeList.values().filter(...)`) em vez de converter para array com spread
13. **`RegExp.escape()`** — usar ao construir regex dinâmicos a partir de strings de usuário
14. **`Promise.try()`** — preferir sobre `Promise.resolve().then(fn)` para wrapping sync/async uniforme

**Blade (Laravel 13.x):**
1. **`@php` único** no topo (após `@props`) — nunca blocos espalhados
2. **`@class`** para classes condicionais
3. **Data attributes com nome completo**: `data-{nome-arquivo}-root` no root, `data-{nome-arquivo}-{atributo}` nos internos
4. **Estado inicial** via `@if` inline quando JS também gerencia o estado
5. **`<x-icon>` com data attributes**: envolver em `<span>` quando JS precisa encontrá-los

**Título do componente:**
- Sempre separar palavras no cabeçalho: `Password Field Component` (nunca `PasswordField Component`)
- A classe JS permanece PascalCase: `class PasswordField {}`

---

## Referências — Quando Carregar

| Arquivo | Conteúdo | Carregar Quando |
|---|---|---|
| `references/js-structure.md` | Classe, #init, #setup*, bootstrap, ES2025, convenções, loader.js | Fase 2 (implementação JS) |
| `references/blade-structure.md` | @props, @php, @class, boas práticas (Laravel 13.x) | Fase 3 (implementação Blade) |
| `references/state-data-attributes.md` | Padrão Estado via Data Attributes | Fase 2, quando houver estados multi-valor |
| `references/component-doc-template.md` | Template de documentação .md para agentes AI | Fase 4 (documentação) |
| `references/password-field-example.md` | Componente canônico completo (JS + Blade) | Fase 6 ou quando precisar de referência |
| `references/phpstorm-jsdoc.md` | `@typedef`, `@type`, `@types/*`, `noinspection` | Fase 2, quando houver APIs externas ou avisos da IDE |

---

## Checklist Final

**Geral:**
- [ ] Plano de trabalho apresentado e aprovado pelo usuário antes de qualquer implementação
- [ ] Todos os comentários (DocBlocks, classes, métodos, funções) em português do Brasil
- [ ] Componente JS declarado em `resources/js/loader.js` (se houver arquivo JS)

**JavaScript (ES2025):**
- [ ] Classe ES2025 com campos privados `#` declarados no topo
- [ ] `COMPONENT_SELECTOR` e `initialized` como constantes no topo
- [ ] Constructor com `root.querySelector` (nunca `document.querySelector`)
- [ ] Data attributes internos com nome completo: `data-{nome-arquivo}-{atributo}`
- [ ] `#init()` chama todos `#setup*` sem lógica condicional
- [ ] Cada `#setup*` com guard clause no início
- [ ] Seções separadas por comentários de 75 `=`
- [ ] Event listeners apenas nos `#setup*`
- [ ] Bootstrap com `mount*`, `MutationObserver` e guard `readyState` (`if (document.readyState === 'loading')` antes de registrar `DOMContentLoaded`)
- [ ] `Math.random()` substituído por `crypto` onde aplicável
- [ ] Estados visuais multi-valor via `setAttribute/removeAttribute`
- [ ] `@typedef` para cada shape de config recebido via `JSON.parse`
- [ ] `@type` em campos privados que armazenam instâncias de APIs externas
- [ ] `// noinspection` apenas como último recurso, sempre com justificativa
- [ ] Título do componente com palavras separadas: `Password Field Component`
- [ ] Iterator Helpers usados quando aplicável (DOM NodeLists, coleções)
- [ ] `RegExp.escape()` em regex construídos a partir de input dinâmico
- [ ] `Promise.try()` preferido sobre wrapping manual de sync em Promise

**Blade (Laravel 13.x):**
- [ ] Bloco `@php` único no topo (após `@props`)
- [ ] Variáveis de conveniência para condições repetidas
- [ ] `@class` para classes condicionais
- [ ] Data attributes com nome completo: `data-{nome-arquivo}-root` e `data-{nome-arquivo}-{atributo}`
- [ ] Estados multi-valor via `data-[*]:` do Tailwind
- [ ] `<x-icon>` com data attributes envolvidos em `<span>` quando JS precisa encontrá-los

**Documentação:**
- [ ] Arquivo .md gerado com nome kebab-case do componente dentro de `.claude/docs/components/`
- [ ] Segue o template de `references/component-doc-template.md`
