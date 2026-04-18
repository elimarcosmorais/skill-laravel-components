# laravel-components (v1.0.1)

Skill para [Claude Code](https://docs.anthropic.com/en/docs/claude-code) que cria novos componentes e refatora componentes existentes JavaScript + Blade (Laravel) seguindo padrões consistentes de organização, encapsulamento e nomenclatura.

## O que faz

Cria novos componentes ou refatora componentes existentes JS + Blade aplicando:

- **JavaScript:** classes ES2025+ com private fields (`#`), data attributes semânticos, bootstrap com MutationObserver, guard clauses, convenções JSDoc para PhpStorm
- **Blade:** `@php` centralizado, `@class` para condicionais, data attributes com nome completo, boas práticas Laravel 13.x
- **Documentação:** gera `.md` do componente otimizado para agentes AI

> **Stack:** PHP 8.4 · Laravel 13.x · JavaScript ES2025+ · Tailwind CSS 4.x

---

## Instalação

Requisito: [Node.js](https://nodejs.org/) instalado (para `npx`).

```bash
npx degit elimarcosmorais/skill-laravel-components .claude/skills/laravel-components
```

Isso baixa a skill diretamente para `.claude/skills/laravel-components` sem histórico git.

### Verificar a instalação

```bash
ls .claude/skills/laravel-components/
# SKILL.md  references/
```

---

## Atualização

Para atualizar para a versão mais recente, rode o mesmo comando com a flag `--force`:

```bash
npx degit elimarcosmorais/skill-laravel-components .claude/skills/laravel-components --force
```

A flag `--force` sobrescreve os arquivos existentes.

### Instalar uma versão específica

Se o repositório usar tags de versão:

```bash
npx degit elimarcosmorais/skill-laravel-components#v1.0.0 .claude/skills/laravel-components --force
```

---

## Estrutura

```
laravel-components/
├── SKILL.md                                  # Instruções para o Claude Code
└── references/
    ├── js-structure.md                       # Padrão de classes JS
    ├── blade-structure.md                    # Padrão de templates Blade
    ├── state-data-attributes.md              # Estados visuais via data attributes
    ├── component-doc-template.md             # Template de documentação .md
    ├── password-field-example.md             # Componente canônico completo
    └── phpstorm-jsdoc.md                     # JSDoc, @typedef, @types/*
```

---

## Como usar

Com a skill instalada, basta pedir ao Claude Code para criar ou refatorar um componente:

```
Crie o componente file-upload seguindo a skill laravel-components
```

```
Refatore o componente file-upload seguindo a skill laravel-components
```

A skill deve ser acionada explicitamente mencionando `laravel-components` no pedido.

---

## Desinstalação

```bash
rm -rf .claude/skills/laravel-components
```

---

## Licença

MIT
