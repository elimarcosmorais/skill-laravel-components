# PhpStorm — JSDoc e Supressão de Avisos

Referência para eliminar avisos da IDE PhpStorm em componentes JavaScript.
Carregar na Fase 3 quando o componente utilizar APIs externas (Google Maps,
Chart.js, Leaflet, etc.) ou quando o usuário reportar avisos da IDE.

---

## Sumário

1. [Bibliotecas de tipos (`@types/*`)](#1-bibliotecas-de-tipos-types)
2. [`@typedef` para objetos de configuração](#2-typedef-para-objetos-de-configuração)
3. [`@type` em campos privados da classe](#3-type-em-campos-privados-da-classe)
4. [Cast após `JSON.parse`](#4-cast-após-jsonparse)
5. [Supressão com `noinspection`](#5-supressão-com-noinspection)
6. [Referência rápida de padrões](#6-referência-rápida-de-padrões)

---

## 1. Bibliotecas de tipos (`@types/*`)

Quando o componente consome uma API externa carregada via CDN ou `<script>` (não via
`node_modules`), o PhpStorm não reconhece os tipos globais. A solução é instalar
os stubs TypeScript do DefinitelyTyped como **biblioteca da IDE** — sem afetar o
bundle de produção.

### Instalação no PhpStorm

```
Settings (Ctrl+Alt+S)
  → Languages & Frameworks
    → JavaScript
      → Libraries
        → Download...
          → Pesquisar o nome do pacote
            → Download and Install
```

### Pacotes comuns

| API externa | Pacote `@types/*` |
|---|---|
| Google Maps | `@types/google.maps` |
| Chart.js | `@types/chart.js` |
| Leaflet | `@types/leaflet` |
| ECharts | (não possui — usar `// noinspection JSUnresolvedReference`) |

Após instalar, a IDE resolve os namespaces globais (ex: `google.maps.Map`,
`google.maps.Polygon`, `L.map`, `Chart`) sem nenhuma alteração no código.

---

## 2. `@typedef` para objetos de configuração

Quando um componente recebe configuração via `JSON.parse` de um data attribute,
o PhpStorm não conhece a forma (shape) do objeto resultante. Definir `@typedef`
no topo do arquivo elimina avisos de "Unresolved variable" em todas as
propriedades acessadas.

### Posicionamento

Os `@typedef` ficam na seção **Type Definitions**, logo após o DocBlock do
componente e antes das constantes:

```javascript
/**
 * NomeDoComponente Component
 * ...
 */

// =============================================================================
// Type Definitions
// =============================================================================

/**
 * @typedef {Object} NomeDoComponenteConfig
 * @property {number} [zoom]
 * @property {boolean} [scrollZoom]
 * @property {string[]} [items]
 */
```

### Regras

- Um `@typedef` por shape de objeto (config, zone, card, etc.)
- Prefixar com o nome do componente para evitar colisões: `PolygonGoogleMapsConfig`
- Propriedades opcionais com `[colchetes]`: `@property {string} [title]`
- Tipos de API externa são válidos em `@property`: `@property {google.maps.LatLngLiteral} [center]`

---

## 3. `@type` em campos privados da classe

Anotar campos privados com `@type` permite que o PhpStorm ofereça autocompletion
e resolva métodos nos objetos armazenados.

```javascript
class MeuComponente {
    /** @type {google.maps.Map|null} */
    #map;
    /** @type {google.maps.Polygon[]} */
    #polygons;
    /** @type {MeuComponenteConfig|null} */
    #config;
    /** @type {MutationObserver|null} */
    #themeObserver;
```

### Quando anotar

- Campos que armazenam instâncias de APIs externas (`#map`, `#chart`, `#polygon`)
- Campos cujo tipo é um `@typedef` definido no mesmo arquivo (`#config`)
- Campos inicializados como `null` no constructor (o PhpStorm infere `null`, não o tipo real)
- Arrays tipados (`#markers`, `#polygons`) — sem `@type`, a IDE não sabe o tipo dos elementos

### Quando NÃO anotar

- Campos DOM simples (`#input`, `#button`) — a IDE infere `Element|null` do `querySelector`
- Booleanos e strings primitivas (`#isVisible`, `#currentTheme`) — inferência automática funciona
- Campos cujo tipo é óbvio pelo valor inicial (`#closeTimeout = null` → pode anotar se causar aviso)

---

## 4. Cast após `JSON.parse`

`JSON.parse` retorna `any`. Para que o PhpStorm reconheça as propriedades do
objeto parseado, usar cast inline com `/** @type */`:

```javascript
this.#config = /** @type {MeuComponenteConfig} */ (JSON.parse(rawConfig));
```

Os parênteses ao redor de `JSON.parse(...)` são obrigatórios — é a sintaxe de
cast do JSDoc/Closure Compiler que o PhpStorm reconhece.

---

## 5. Supressão com `noinspection`

Quando a anotação JSDoc não resolve o aviso (API sem `@types`, símbolo gerado
dinamicamente, ou uso intencional de API deprecated), usar `// noinspection` na
linha anterior.

### Padrões usados no projeto

| Aviso | Supressão | Quando usar |
|---|---|---|
| Unresolved variable/function | `// noinspection JSUnresolvedReference` | API sem `@types/*` disponível |
| Deprecated symbol | `// noinspection JSDeprecated` | Uso intencional com justificativa documentada |
| Unused global symbol | `// noinspection JSUnusedGlobalSymbols` | Método público chamado externamente ou override |

### Formato

O `noinspection` vai na linha **imediatamente anterior** ao símbolo, como
comentário de linha:

```javascript
// noinspection JSDeprecated — intencional: AdvancedMarkerElement requer mapId, incompatível com JSON styles
const marker = new google.maps.Marker({ ... });
```

### Regra de ouro

Seguir a ordem de preferência:

1. **Instalar `@types/*`** — resolve a causa raiz
2. **Adicionar `@typedef` / `@type`** — resolve para objetos de config
3. **`// noinspection`** — último recurso, sempre com justificativa no mesmo comentário

Nunca usar `noinspection` como atalho para evitar tipar corretamente. A
justificativa no comentário é obrigatória para que revisores entendam por que a
supressão existe.

---

## 6. Referência rápida de padrões

### Variáveis locais tipadas

```javascript
/** @type {google.maps.MapOptions} */
const mapOptions = { zoom: 14, center: { lat: 0, lng: 0 } };
```

### Retorno de método

```javascript
/**
 * @returns {google.maps.MapTypeStyle[]}
 */
#resolveMapStyles() { ... }
```

### Parâmetros de método

```javascript
/**
 * @param {google.maps.LatLngLiteral} position
 * @param {MeuComponenteZoneConfig} zone
 */
#createMarker(position, zone) { ... }
```

### Override de classe base

```javascript
// noinspection JSUnusedGlobalSymbols — override google.maps.OverlayView
onAdd() { ... }
```
