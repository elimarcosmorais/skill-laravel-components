# JavaScript — Estrutura e Convenções (ES2025)

Referência completa para a refatoração de componentes JavaScript.
Carregar na Fase 2 do workflow.

**Versão alvo:** ECMAScript 2025 (ES16, ratificado em junho 2025).

---

## Estrutura do Arquivo

Cada componente JS segue esta sequência fixa:

```
1. DocBlock do componente (cabeçalho)
2. Type Definitions (@typedef) — se houver configs parseadas ou APIs externas
3. Constantes (COMPONENT_SELECTOR + initialized WeakSet)
4. Classe principal (constructor → init → setup* → auxiliares)
5. Bootstrap (mount + MutationObserver + DOMContentLoaded)
```

---

## Cabeçalho

O título do componente deve ter as palavras separadas (não PascalCase):

```javascript
/**
 * Nome Do Componente Component
 *
 * Descrição breve do que o componente faz.
 */
```

Exemplos: `Password Field Component`, `Date Range Picker Component`, `Accordion Component`.

## Idioma dos Comentários

**Todos os comentários devem ser escritos em português do Brasil** — DocBlocks de classe,
métodos, funções e comentários inline. Isso inclui descrições de `@param`, `@returns`,
`@typedef` e comentários de seção (separadores `===`).

```javascript
// =========================================================================
// Alternância de Visibilidade
// =========================================================================

/**
 * Configura o botão de alternar visibilidade da senha.
 */
#setupToggle() {
    if (!this.#toggle || !this.#input) return;
    // Alterna entre tipo 'text' e 'password'
    this.#toggle.addEventListener('click', () => this.#handleToggle());
}

/**
 * Alterna a visibilidade do campo de senha.
 */
#handleToggle() {
    const isVisible = this.#input.type === 'text';
    this.#input.type = isVisible ? 'password' : 'text';
}
```

## Constantes Globais

```javascript
const COMPONENT_SELECTOR = "[data-nome-do-componente-root]";
const initialized = new WeakSet();
```

O `COMPONENT_SELECTOR` usa o nome completo do arquivo em kebab-case. O `WeakSet` evita
inicialização duplicada — essencial para componentes injetados dinamicamente via AJAX ou Livewire.

---

## Classe Principal

### Constructor

Recebe o elemento raiz e mapeia todos os elementos internos via `root.querySelector` com
data attributes de nome completo. Nunca use `document.querySelector` — o escopo isolado
via `root` garante que os seletores não colidam entre componentes.

```javascript
class NomeDoComponente {
  #input;
  #button;

  constructor(root) {
    this.root = root;
    this.#input = root.querySelector("[data-nome-do-componente-input]");
    this.#button = root.querySelector("[data-nome-do-componente-button]");
    this.#init();
  }
}
```

### Encapsulamento com `#` (Private Class Fields)

Use a sintaxe nativa `#` do JavaScript para campos e métodos internos à classe.
Isso garante encapsulamento real (não apenas convenção).

```javascript
class NomeDoComponente {
  // Campos privados — declarados no topo da classe
  #input;
  #button;
  #meterBars;

  constructor(root) {
    this.root = root; // root permanece público
    this.#input = root.querySelector("[data-nome-do-componente-input]");
    this.#button = root.querySelector("[data-nome-do-componente-button]");
    this.#init();
  }

  #init() {
    this.#setupFeatureA();
    this.#setupFeatureB();
  }

  #setupFeatureA() {
    if (!this.#input || !this.#button) return;
    this.#button.addEventListener("click", () => this.#handleAction());
  }

  #handleAction() {
    // lógica interna
  }
}
```

**O que deve ser `#` privado:**

- Todas as propriedades mapeadas do DOM (`#input`, `#button`, `#box`, etc.)
- O método `#init()` — só é chamado pelo constructor
- Todos os métodos `#setup*` — só são chamados pelo `#init()`
- Métodos auxiliares/helpers (`#calculateStrength`, `#renderBars`, etc.)
- Propriedades derivadas internas (`#meterBars`, `#isVisible`, etc.)

**O que permanece público:**

- `this.root` — o elemento raiz (útil para referência externa)
- O `constructor` (sempre público por natureza)

**Declaração dos campos:** Todos os campos privados devem ser declarados no topo da classe,
antes do constructor, como exigido pela especificação.

### Método #init()

Chama todos os `#setup*` sem lógica condicional — cada setup cuida da própria validação:

```javascript
#init() {
    this.#setupFeatureA();
    this.#setupFeatureB();
}
```

### Métodos #setup\* e Auxiliares

Cada feature é isolada em um bloco com separador de 75 caracteres `=`:

```javascript
// =========================================================================
// Nome da Feature
// =========================================================================

#setupFeature() {
    if (!this.#elementoA || !this.#elementoB) return;  // guard clause obrigatória
    this.#elementoA.addEventListener('click', () => this.#handleAction());
}

#handleAction() {
    // lógica de suporte
}
```

Regras importantes:

- O `#setup*` sempre inicia com guard clause validando elementos necessários
- Event listeners são registrados apenas dentro dos `#setup*`, nunca em auxiliares
- Use arrow functions nos listeners para preservar `this`
- Prefira `classList.toggle(classe, booleano)` para toggles binários simples (ex: `hidden`)
- Para estados visuais multi-valor, consulte `references/state-data-attributes.md`

### Documentação

Use DocBlock apenas quando o método tem lógica complexa ou parâmetros não-óbvios.
Métodos autodescritivos como `#setupToggle()` ou `#updateMeter()` não precisam de DocBlock.
Quando usar DocBlock, escrever sempre em **português do Brasil**.

---

## Bootstrap

Bloco idêntico em todos os componentes — mude apenas o nome da função mount e da classe:

```javascript
// =============================================================================
// Bootstrap
// =============================================================================

function mountNomeDoComponente() {
  document.querySelectorAll(COMPONENT_SELECTOR).forEach((root) => {
    if (initialized.has(root)) return;
    initialized.add(root);
    new NomeDoComponente(root);
  });
}

const observer = new MutationObserver((mutations) => {
  const hasNewComponent = mutations.some((mutation) =>
    Array.from(mutation.addedNodes).some(
      (node) => node.nodeType === 1 && (node.matches?.(COMPONENT_SELECTOR) || node.querySelector?.(COMPONENT_SELECTOR)),
    ),
  );
  if (hasNewComponent) mountNomeDoComponente();
});

observer.observe(document.body, { childList: true, subtree: true });

if (document.readyState === "loading") {
  document.addEventListener("DOMContentLoaded", mountNomeDoComponente);
} else {
  mountNomeDoComponente();
}
```

O MutationObserver garante que componentes inseridos dinamicamente no DOM (via AJAX,
Livewire, modais) sejam inicializados automaticamente.

O guard de `readyState` é obrigatório porque o `loader.js` importa módulos de forma lazy
(`import()` assíncrono) — quando o módulo executa, o `DOMContentLoaded` já disparou e um
listener registrado diretamente nunca seria chamado. O padrão `readyState === 'loading'`
torna o bootstrap compatível com lazy load e eager load.

---

## Data Attributes — Nome Completo

Todos os data attributes usam o nome completo do componente em kebab-case, tanto no root
quanto nos elementos internos. Isso elimina a necessidade de controle de prefixos.

- **Root** — `data-{nome-arquivo}-root`
- **Internos** — `data-{nome-arquivo}-{nome-do-atributo}`

```html
<div data-password-field-root>
  <!-- root -->
  <input data-password-field-input />
  <!-- nome completo -->
  <button data-password-field-copy>Copiar</button>
  <!-- nome completo -->
  <button data-password-field-generate>Gerar</button>
  <!-- nome completo -->
</div>
```

### Nomenclatura dos Elementos Internos

Nomes descritivos em kebab-case após o nome do componente:

```
data-{nome-arquivo}-input       → Campo de entrada principal
data-{nome-arquivo}-box         → Container de agrupamento
data-{nome-arquivo}-trigger     → Elemento que dispara ação
data-{nome-arquivo}-content     → Área de conteúdo dinâmico
data-{nome-arquivo}-[acao]      → Botão de ação específica (ex: data-password-field-copy)
```

### Separação de Responsabilidades

- `data-*` → seletores para JavaScript (comportamento)
- Classes CSS → exclusivamente para estilização
- Nunca use classes CSS como seletor JavaScript

---

## Features ES2025 Relevantes para Componentes

### Iterator Helpers

ES2025 adicionou métodos nativos a iteradores: `.map()`, `.filter()`, `.take()`, `.drop()`,
`.reduce()`, `.some()`, `.every()`, `.find()`, `.toArray()`. Estes métodos são **lazy** — processam
elementos sob demanda sem criar arrays intermediários.

**Quando usar em componentes:**

Use Iterator Helpers ao processar coleções DOM (`NodeList`, `HTMLCollection`) quando precisar
encadear operações de filtragem/transformação. Isso evita conversões desnecessárias para array.

```javascript
// ✅ ES2025 — lazy, sem array intermediário
#getVisibleOptions() {
    return this.#options.values()
        .filter((option) => !option.hidden)
        .toArray();
}

// ✅ ES2025 — filtrar e transformar em cadeia
#getActiveLabels() {
    return this.root.querySelectorAll('[data-tabs-item]').values()
        .filter((item) => item.hasAttribute('data-tabs-active'))
        .map((item) => item.textContent.trim())
        .toArray();
}

// ❌ Antes — spread + array methods (cria array intermediário)
#getVisibleOptions() {
    return [...this.#options].filter((option) => !option.hidden);
}
```

**Quando NÃO usar:**

- Iterações simples com `forEach` — `querySelectorAll().forEach()` já é eficiente
- Quando não há encadeamento (filtro único em lista pequena) — spread é mais legível
- No `MutationObserver` do bootstrap — manter `Array.from()` por clareza e compatibilidade

### RegExp.escape()

Método estático que escapa caracteres especiais de regex em strings. Usar sempre que construir
regex a partir de input dinâmico (busca, filtro, highlight).

```javascript
// ✅ ES2025 — seguro contra injeção de regex
#filterByText(searchText) {
    const escaped = RegExp.escape(searchText);
    const regex = new RegExp(escaped, 'i');
    this.#items.forEach((item) => {
        const matches = regex.test(item.textContent);
        item.classList.toggle('hidden', !matches);
    });
}

// ❌ Antes — vulnerável a caracteres especiais como . * + ? ( )
#filterByText(searchText) {
    const regex = new RegExp(searchText, 'i');  // "user.name" casa com "username"
    // ...
}
```

### Promise.try()

Executa uma função e garante que o resultado seja sempre uma Promise, capturando erros
síncronos automaticamente. Usar quando uma operação pode ser sync ou async.

```javascript
// ✅ ES2025 — captura erros sync e async uniformemente
#loadConfig() {
    return Promise.try(() => {
        const raw = this.root.getAttribute('data-component-config');
        if (!raw) throw new Error('Config not found');
        return JSON.parse(raw);
    });
}

// ❌ Antes — erro síncrono de JSON.parse não é capturado
#loadConfig() {
    return Promise.resolve()
        .then(() => {
            const raw = this.root.getAttribute('data-component-config');
            return JSON.parse(raw);  // se raw for null, throw síncrono escapa
        });
}
```

### Set Methods

ES2025 adicionou métodos nativos a `Set`: `union()`, `intersection()`, `difference()`,
`symmetricDifference()`. Úteis para gerenciar estados de seleção múltipla.

```javascript
// ✅ ES2025 — diferença nativa entre conjuntos
#getNewlySelected(previousIds, currentIds) {
    return currentIds.difference(previousIds);  // retorna Set
}

#getCommonSelected(setA, setB) {
    return setA.intersection(setB);  // retorna Set
}
```

---

## Registro no loader.js

Todo componente com arquivo JavaScript deve ser declarado em `resources/js/loader.js` no
objeto `LAZY_MODULES`. O seletor é o `COMPONENT_SELECTOR` do componente.

```javascript
const LAZY_MODULES = {
  // Inputs
  "[data-password-field-root]": () => import("./components/inputs/password-field"),

  // Novo componente adicionado — agrupar por categoria
  "[data-nome-do-componente-root]": () => import("./components/{categoria}/{nome-do-arquivo}"),
};
```

**Regras:**

- Agrupar entradas por categoria com comentário (ex: `// Inputs`, `// Modals`, `// Charts`)
- O seletor deve corresponder exatamente ao `COMPONENT_SELECTOR` definido no arquivo JS
- O caminho de import é relativo ao `loader.js`, sem extensão `.js`
- Ler o `loader.js` atual antes de adicionar para manter a organização existente

---

## Convenções de Código

| Tipo                    | Convenção                  | Exemplo                |
| ----------------------- | -------------------------- | ---------------------- |
| Classe                  | PascalCase                 | `PasswordField`        |
| Campo privado           | `#` + camelCase            | `#input`, `#meterBars` |
| Método privado setup    | `#` + camelCase + `setup`  | `#setupToggle`         |
| Método privado auxiliar | `#` + camelCase descritivo | `#calculateStrength`   |
| Propriedade pública     | camelCase                  | `this.root`            |
| Função mount            | camelCase + `mount`        | `mountPasswordField`   |
| Constante global        | UPPER_SNAKE_CASE           | `COMPONENT_SELECTOR`   |

Strings com aspas simples. Template literals apenas com interpolação. Trailing comma em arrays/objetos.

---

## Utilitários Globais

### window.ScrollLock

Utilitário global em `resources/js/components/util/scroll-lock.js` que gerencia o travamento
da scrollbar com **contagem de referências**. Múltiplos componentes podem chamar `lock/unlock`
independentemente — o `document.body` só é modificado quando o contador sai de/chega a zero,
evitando conflitos entre overlays simultâneos (ex: modal + date-picker abertos ao mesmo tempo).

**API:**

```javascript
window.ScrollLock.lock(); // incrementa contador e trava scroll (se for o primeiro)
window.ScrollLock.unlock(); // decrementa contador e libera scroll (se for o último)
window.ScrollLock.locked; // getter boolean — true se qualquer componente travou o scroll
```

**Regra inviolável:** Nunca manipule `document.body.overflow` ou `document.body.paddingRight`
diretamente em componentes. Sempre delegue ao `window.ScrollLock`.

**Quando usar:** Qualquer componente que precise impedir o scroll da página enquanto estiver
aberto — modais, drawers, calendários que ultrapassam a viewport, bottom sheets, etc.

**Padrão com flag de controle:**

Componentes que travam o scroll condicionalmente (ex: só quando o calendário ultrapassa a
viewport) devem usar um campo privado `#didLockScroll` para rastrear se foram eles que
chamaram `lock()` — e só chamar `unlock()` nesse caso:

```javascript
class MeuComponente {
  #didLockScroll = false;

  #open() {
    // trava apenas se necessário (ex: conteúdo ultrapassa viewport)
    if (precisaTravar) {
      window.ScrollLock.lock();
      this.#didLockScroll = true;
    }
    // ...
  }

  #close() {
    if (this.#didLockScroll) {
      window.ScrollLock.unlock();
      this.#didLockScroll = false;
    }
    // ...
  }
}
```

Componentes que **sempre** travam o scroll ao abrir (ex: modal) chamam `lock/unlock`
diretamente, sem necessidade de flag:

```javascript
show() {
    window.ScrollLock.lock();
    // ...
}

close() {
    window.ScrollLock.unlock();
    // ...
}
```

**Anti-padrão — nunca condicionar `unlock()` ao estado interno da stack:**

```javascript
// ❌ ERRADO — quebra a contagem de referências com múltiplas instâncias abertas
close() {
    this.#removeFromStack();
    if (stack.length === 0) {        // ← não faça isso
        window.ScrollLock.unlock();
    }
}

// ✅ CORRETO — cada instância que chamou lock() deve chamar unlock()
close() {
    this.#removeFromStack();
    window.ScrollLock.unlock();     // ScrollLock gerencia a contagem internamente
}
```

Com 2 instâncias abertas, `count = 2`. Se `unlock()` for chamado apenas uma vez (quando a
stack interna chega a zero), o `count` fica em 1 e os estilos do `body` nunca são removidos.

**Dependência de carregamento:** `scroll-lock.js` deve ser carregado **antes** de qualquer
componente que o utilize. Verificar o `loader.js` ou o entry point do bundle.

### Funções Globais Expostas (`window.*`)

Componentes que expõem funções globais (ex: `window.showAlert`, `window.openModal`) enfrentam
uma **race condition** com o lazy load: se o código da página chamar a função global antes do
módulo terminar de importar, a função ainda não existe e gera `ReferenceError`.

**Causa raiz:** o `loader.js` usa `import()` dinâmico (assíncrono). Quando o `DOMContentLoaded`
da view dispara e chama `window.showAlert(...)`, o módulo pode ainda não ter resolvido.

**Solução: stub de fila no `loader.js` + drenagem no componente**

No `loader.js`, antes de `loadModules()`, definir um stub que enfileira as chamadas:

```javascript
// loader.js — antes de loadModules()
window._alertQueue = [];
window.showAlert = (opts) => window._alertQueue.push(opts);
```

No componente, após definir a função real, drenar a fila:

```javascript
// alert.js — no final do arquivo, após definir a classe
window.showAlert = (options) => Alert.show(options);
window._alertQueue?.forEach((opts) => Alert.show(opts));
window._alertQueue = null;
```

**Quando aplicar:**

- Qualquer componente que defina uma função global (`window.nomeFuncao`)
- Sempre que o componente for registrado no `LAZY_MODULES` do `loader.js`
- Sempre que a função global puder ser chamada em `DOMContentLoaded` ou em scripts inline da view

**Regras:**

- A fila deve ser inicializada **no `loader.js`**, antes de `loadModules()`
- A drenagem deve ocorrer **após** `window.nomeFuncao = ...` no componente
- Após drenar, setar a fila como `null` (libera memória e sinaliza que o módulo carregou)
- Nomear a fila com prefixo `_` e sufixo `Queue`: `window._alertQueue`, `window._toastQueue`

---

## Segurança

Nunca use `Math.random()` para senhas, tokens ou valores sensíveis. Use `crypto.getRandomValues`.
Para shuffle, use Fisher-Yates com crypto API — nunca `sort(() => 0.5 - Math.random())`.
