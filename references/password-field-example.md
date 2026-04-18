# Exemplo de Referência — Password Field

Este é o componente canônico. Ao refatorar qualquer componente, compare a estrutura
final com este exemplo para garantir consistência.

---

## JavaScript (`password-field.js`)

```javascript
/**
 * Password Field Component
 *
 * Componente de campo de senha com geração aleatória,
 * cópia, toggle de visibilidade e medidor de força.
 */

const COMPONENT_SELECTOR = '[data-password-field-root]';

const initialized = new WeakSet();

class PasswordField {
    #input;
    #box;
    #generateBtn;
    #copyBtn;
    #eyeIcon;
    #eyeSlashIcon;
    #meterContainer;
    #meterBox;
    #meterBars;

    constructor(root) {
        this.root = root;
        this.#input = root.querySelector('[data-password-field-input]');
        this.#box = root.querySelector('[data-password-field-box]');
        this.#generateBtn = root.querySelector('[data-password-field-generate]');
        this.#copyBtn = root.querySelector('[data-password-field-copy]');
        this.#eyeIcon = root.querySelector('[data-password-field-eye]');
        this.#eyeSlashIcon = root.querySelector('[data-password-field-eye-slash]');
        this.#meterContainer = root.querySelector('[data-password-field-meter]');
        this.#meterBox = root.querySelector('[data-password-field-meter-box]');

        this.#init();
    }

    #init() {
        this.#setupGenerate();
        this.#setupCopy();
        this.#setupToggle();
        this.#setupMeter();
    }

    // =========================================================================
    // Geração de senha
    // =========================================================================

    #setupGenerate() {
        if (!this.#input || !this.#generateBtn) return;

        this.#generateBtn.addEventListener('click', () => {
            this.#input.value = this.#createRandomPassword();
            this.#input.dispatchEvent(new Event('input', { bubbles: true }));
        });
    }

    /**
     * Gera senha criptograficamente segura com distribuição uniforme.
     * Composição: 3 maiúsculas, 3 minúsculas, 4 números e 2 especiais.
     */
    #createRandomPassword() {
        const charsets = {
            upper: 'ABCDEFGHIJKLMNOPQRSTUVWXYZ',
            lower: 'abcdefghijklmnopqrstuvwxyz',
            digits: '0123456789',
            special: '!@#$%^&*_-',
        };

        const composition = [
            { charset: charsets.upper, count: 3 },
            { charset: charsets.lower, count: 3 },
            { charset: charsets.digits, count: 4 },
            { charset: charsets.special, count: 2 },
        ];

        const chars = [];

        for (const { charset, count } of composition) {
            for (let i = 0; i < count; i++) {
                chars.push(charset[this.#secureRandomIndex(charset.length)]);
            }
        }

        return this.#shuffleArray(chars).join('');
    }

    /**
     * Retorna índice aleatório usando crypto API.
     * Rejeita valores com viés modular para distribuição uniforme.
     */
    #secureRandomIndex(max) {
        const array = new Uint32Array(1);
        const limit = Math.floor(0xFFFFFFFF / max) * max;

        do {
            crypto.getRandomValues(array);
        } while (array[0] >= limit);

        return array[0] % max;
    }

    /** Fisher-Yates shuffle com crypto API */
    #shuffleArray(array) {
        for (let i = array.length - 1; i > 0; i--) {
            const j = this.#secureRandomIndex(i + 1);
            [array[i], array[j]] = [array[j], array[i]];
        }

        return array;
    }

    // =========================================================================
    // Cópia
    // =========================================================================

    #setupCopy() {
        if (!this.#input || !this.#copyBtn) return;

        this.#copyBtn.addEventListener('click', () => this.#copyToClipboard());

        this.#input.addEventListener('keyup', (event) => {
            if (event.key === 'Enter') this.#input.blur();
        });
    }

    async #copyToClipboard() {
        this.#input.select();
        this.#input.setSelectionRange(0, 99999);

        try {
            await navigator.clipboard.writeText(this.#input.value);
        } catch {
            document.execCommand('copy');
        }

        setTimeout(() => {
            const end = this.#input.value.length;
            this.#input.setSelectionRange(end, end);
        }, 200);
    }

    // =========================================================================
    // Toggle de visibilidade
    // =========================================================================

    #setupToggle() {
        if (!this.#input || !this.#eyeIcon || !this.#eyeSlashIcon) return;

        const toggle = () => {
            const isPassword = this.#input.type === 'password';
            this.#input.type = isPassword ? 'text' : 'password';

            this.#eyeIcon.classList.toggle('hidden', isPassword);
            this.#eyeSlashIcon.classList.toggle('hidden', !isPassword);
        };

        this.#eyeIcon.addEventListener('click', toggle);
        this.#eyeSlashIcon.addEventListener('click', toggle);
    }

    // =========================================================================
    // Medidor de força
    // =========================================================================

    #setupMeter() {
        if (!this.#input || !this.#meterBox || !this.#meterContainer) return;

        this.#meterBars = this.#meterBox.querySelectorAll('div');

        this.#input.addEventListener('input', () => this.#updateMeter());

        document.addEventListener('click', (event) => {
            if (this.#box && !this.#box.contains(event.target)) {
                this.#meterContainer.classList.add('hidden');
            }
        });

        this.#updateMeter();
    }

    #updateMeter() {
        const password = this.#input.value;
        const score = this.#calculateStrength(password);

        this.#toggleMeterVisibility(password.length > 0);
        this.#renderMeterBars(score);
    }

    #toggleMeterVisibility(visible) {
        this.#meterContainer.classList.toggle('hidden', !visible);
    }

    #calculateStrength(password) {
        if (!password) return 0;

        let score = 0;

        if (password.length >= 6) score++;
        if (password.length >= 12) score++;

        const checks = [/[A-Z]/, /[a-z]/, /[0-9]/, /[^A-Za-z0-9]/];
        const complexity = checks.filter((regex) => regex.test(password)).length;

        score += Math.min(2, Math.floor(complexity / 2));

        return Math.min(4, score);
    }

    #renderMeterBars(score) {
        const baseClasses = 'w-full h-1 rounded-sm';
        const inactiveClasses = 'bg-gray-200 dark:bg-zinc-700';

        const levelColors = [
            'bg-red-500 dark:bg-red-700',
            'bg-yellow-500 dark:bg-yellow-600',
            'bg-blue-500 dark:bg-blue-700',
            'bg-green-500 dark:bg-green-700',
        ];

        this.#meterBars.forEach((bar, index) => {
            const isActive = index < score;
            const colorClasses = isActive ? levelColors[index] : inactiveClasses;

            bar.className = `${baseClasses} ${colorClasses}`;
        });
    }
}

// =============================================================================
// Bootstrap
// =============================================================================

function mountPasswordFields() {
    document.querySelectorAll(COMPONENT_SELECTOR).forEach((root) => {
        if (initialized.has(root)) return;

        initialized.add(root);
        new PasswordField(root);
    });
}

const observer = new MutationObserver((mutations) => {
    const hasNewComponent = mutations.some((mutation) =>
        Array.from(mutation.addedNodes).some(
            (node) =>
                node.nodeType === 1 &&
                (node.matches?.(COMPONENT_SELECTOR) ||
                    node.querySelector?.(COMPONENT_SELECTOR)),
        ),
    );

    if (hasNewComponent) mountPasswordFields();
});

observer.observe(document.body, { childList: true, subtree: true });

document.addEventListener('DOMContentLoaded', mountPasswordFields);
```

---

## Blade (`password-field.blade.php`)

```blade
@props([
    'label' => null,
    'icon' => 'key',
    'size' => null,
    'generate' => false,
    'copy' => false,
    'eye' => false,
    'meter' => false,
    'maxlength' => 100,
    'autocomplete' => 'off',
    'disabled' => false,
    'col' => null,
])

@php
    $isSmall = $size === 'sm';
    $iconBoxSize = $isSmall ? 'size-8' : 'size-10';
    $iconSize = $isSmall ? 'size-[16px]' : 'size-[19px]';
    $inputHeight = $isSmall ? 'h-8 text-[14px]' : 'h-10 text-[15px]';
    $eyePadding = $eye ? ($isSmall ? 'pr-8' : 'pr-10') : '';

    $colSpan = $col ? 'md:col-span-' . $col : '';

    $inputAttr = $attributes
        ->merge([
            'type' => 'password',
            'maxlength' => $maxlength,
            'autocomplete' => $autocomplete,
            'disabled' => $disabled,
        ])
        ->except('class');
@endphp

@if ($col)
    <div class="col-span-12 {{ $colSpan }}">
@endif

<div data-password-field-root class="flex flex-col {{ $attributes->get('class') }}">

    @isset($label)
        <label
            class="text-gray-900 dark:text-zinc-200 {{ $isSmall ? 'text-[13px]/[17px]' : 'text-sm' }} after:text-red-500 {{ str_contains($label, '*') ? 'after:content-["*"]' : '' }}">
            {{ trim(str_replace('*', '', $label)) }}
        </label>
    @endisset

    <div data-password-field-box class="flex relative">

        {{-- Botão do ícone (com geração opcional) --}}
        <button type="button"
            @if ($generate) data-password-field-generate @endif
            @class([
                'form-input-icon-start',
                $iconBoxSize,
                'cursor-pointer' => $generate,
            ])
        >
            <x-icon :name="$icon" @class([$iconSize, 'fill-gray-500/90 dark:fill-zinc-300', 'form-disabled' => $disabled]) />
        </button>

        {{-- Botão de cópia --}}
        @if ($copy)
            <button type="button" data-password-field-copy @class(['form-input-icon-center border-l cursor-pointer', $iconBoxSize])>
                <x-icon name="copy" @class([$iconSize, 'fill-gray-500/90 dark:fill-zinc-300', 'form-disabled' => $disabled]) />
            </button>
        @endif

        {{-- Toggle de visibilidade --}}
        @if ($eye)
            <div @class(['absolute inset-y-0 flex items-center', $isSmall ? 'right-2.5' : 'right-3'])>
                <div data-password-field-eye>
                    <x-icon name="eye" @class([$iconSize, 'fill-gray-500/90 dark:fill-zinc-300 cursor-pointer']) />
                </div>
                <div data-password-field-eye-slash class="hidden">
                    <x-icon name="eye-slash" @class([$iconSize, 'fill-gray-500/90 dark:fill-zinc-300 cursor-pointer']) />
                </div>
            </div>
        @endif

        {{-- Campo de senha --}}
        <input data-password-field-input {{ $inputAttr }}
            @class([
                'form-input',
                $inputHeight,
                'rounded-l-none' => $icon,
                $eyePadding,
                'mb-1' => $meter,
            ])
        >

        {{-- Medidor de força --}}
        @if ($meter)
            <div data-password-field-meter class="hidden absolute -bottom-0.5 w-full">
                <div data-password-field-meter-box class="grid grid-cols-4 gap-1.5 w-full px-0.5">
                    @for ($i = 0; $i < 4; $i++)
                        <div class="w-full h-1 rounded-sm bg-gray-200 dark:bg-zinc-700"></div>
                    @endfor
                </div>
            </div>
        @endif

    </div>
</div>

@if ($col)
    </div>
@endif
```
