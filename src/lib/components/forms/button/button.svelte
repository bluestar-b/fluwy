<script lang="ts">
    import type { Any, Component } from '@/lib/core/contracts.js';
    import { cn, deferred } from '@/lib/core/utils/index.js';
    import { Icon, type IconProps } from '../../common/icon/index.js';
    import { useClient, mergeThemes } from '../../../core/client/index.js';
    import { useContext } from '../../../core/context/index.js';
    import { type Snippet } from 'svelte';
    import { compile, Render, type ElementProps } from '@/lib/core/index.js';
    import { setCurrentColor } from '@/lib/core/utils/color/index.js';
    import { BorderRadius, Sizes, Variants } from './styles.js';
    import { Colors } from '@/lib/core/styles.js';
    import { useCommon } from '../../common/styles.js';

    interface ButtonProps extends ElementProps {
        text?: string;
        icon?: IconProps | string;
        trailing_icon?: IconProps | string;
        loading?: boolean;
        disabled?: boolean;
        class?: string;
        variant?: string;
        size?: string;
        color?: string;
        on_click?: Any;
        onclick?: () => Any;
        component?: Partial<Component>;
        children?: Snippet;
    }

    const { component, children, ...props }: ButtonProps = $props();

    const componentName = component?.name ?? 'button';
    const context = useContext();
    const client = useClient();

    const sizes = mergeThemes('forms.common.sizes', Sizes);
    const colors = mergeThemes('colors', Colors);
    const variants = mergeThemes(`forms.${componentName}.variants`, Variants);
    const defaultSize = mergeThemes('forms.common.default_size', 'md');
    const borderRadius = mergeThemes('forms.common.border_radius', BorderRadius);
    const commonBorderColor = useCommon('border_color');

    let innerLoading = false;

    let text = $derived(compile(props.text ?? '', context.data));
    let size = $derived(compile(props.size || defaultSize, context.data));
    let color = $derived(compile(props.color || 'default', context.data));
    let variant: keyof typeof Variants = $derived(compile(props.variant || 'filled', context.data));
    let loading = $derived(props.loading || innerLoading);
    let disabled = $derived(props.disabled || loading);

    async function handleClick(e: MouseEvent) {
        if (!props.on_click) {
            return props.onclick?.();
        }

        e.stopPropagation();

        let done = false;

        deferred(() => (innerLoading = !done));

        try {
            return await client.handleOperations(props.on_click, context);
        } catch (error) {
            console.error(error);
        } finally {
            innerLoading = false;
            done = true;
        }
    }

    let setButtonColor = $derived(
        color === 'default' ? setCurrentColor('gray', colors) : setCurrentColor(color, colors)
    );
    let defaultColorClasses: typeof Variants = $derived(
        color === 'default'
            ? {
                  filled: 'text-color-900 dark:text-color-contrast focus:ring-color-200 dark:focus:ring-color-700 bg-color-contrast dark:bg-color-700/50 enabled:hover:bg-color-100 enabled:dark:hover:bg-color-700 border-color-200 dark:border-color-700',
                  outline:
                      'text-color-900 dark:text-color-contrast bg-white enabled:hover:bg-color-100 enabled:dark:hover:bg-color-700 focus:ring-color dark:focus:ring-color border-color-500 dark:border-color-500',
                  ghost: 'text-color-900 dark:text-color-contrast/75 enabled:dark:hover:text-color-contrast focus:ring-color-200 dark:focus:ring-color-700 enabled:hover:bg-color-100 enabled:dark:hover:bg-color-700 border-transparent dark:border-transparent',
                  link: 'text-color-900 underline decoration-color-500 enabled:hover:decoration-2 dark:text-color-contrast focus:ring-color-200',
              }
            : ({} as typeof Variants)
    );

    function getIcon(propValue: Any): IconProps {
        if (typeof propValue === 'string') return { name: propValue };

        return propValue as IconProps;
    }
</script>

<button
    onclick={handleClick}
    class={cn(
        commonBorderColor,
        `flex items-center justify-center gap-1 shadow-sm ring-offset-white transition-all duration-75 hover:opacity-100 focus:outline-none focus:ring-2 focus:ring-color focus:ring-offset-2 enabled:active:scale-[0.99] dark:ring-offset-black`,
        sizes[size],
        variants[variant],
        borderRadius[size],
        defaultColorClasses[variant],
        props.class,
        disabled ? 'hover:none cursor-not-allowed opacity-50' : ''
    )}
    {disabled}
    style={setButtonColor}
>
    {#if props.icon && !loading}
        <Icon {...getIcon(props.icon)} />
    {:else if loading}
        <Icon {...{ name: 'svg-spinners:90-ring-with-bg' }} />
    {/if}

    {#if props.content}
        <Render props={props.content} />
    {:else if text}
        <Render props={text} />
    {/if}

    {#if props.trailing_icon}
        <Icon {...getIcon(props.trailing_icon)} />
    {/if}

    {#if children}
        {@render children()}
    {/if}
</button>
