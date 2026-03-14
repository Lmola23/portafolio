<script>
    let scrollY = $state(0);
    let innerHeight = $state(0);
    let containerNode = $state(null);
    let globalProgress = $state(0);

    // SVG path measurements
    let svgW = $state(0);
    let svgH = $state(0);
    let pathEl = $state(null);
    let totalLen = $state(0);

    $effect(() => {
        if (!pathEl) return;
        totalLen = pathEl.getTotalLength();
    });

    $effect(() => {
        const _s = scrollY;
        const _h = innerHeight;
        if (!containerNode) return;
        const rect = containerNode.getBoundingClientRect();
        const scrolled = _h * 0.5 - rect.top;
        if (scrolled < 0) globalProgress = 0;
        else if (scrolled > rect.height) globalProgress = 100;
        else globalProgress = (scrolled / rect.height) * 100;
    });

    let dashOffset = $derived(totalLen - (globalProgress / 100) * totalLen);

    const THRESHOLDS = [5, 27, 49, 71];

    // Per-card fill progress: once threshold is hit, animate from 0→100 over the next ~15% of scroll
    function cardFill(g, threshold) {
        if (g < threshold) return 0;
        const end = threshold + 15;
        if (g >= end) return 100;
        return ((g - threshold) / (end - threshold)) * 100;
    }

    const steps = [
        {
            id: 1,
            title: "Análisis",
            icon: '<svg stroke="currentColor" fill="none" stroke-width="2" viewBox="0 0 24 24" class="icon-svg" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"/></svg>',
            desc: [
                "Analizamos tu modelo de negocio y objetivos.",
                "Identificamos necesidades y oportunidades de mejora.",
                "Definimos el alcance del proyecto.",
                "Establecemos prioridades y funcionalidades clave.",
                "Acordamos tiempos y entregables.",
            ],
            footer: "El objetivo es reducir riesgos y asegurar que la solución responda realmente a lo que tu empresa necesita.",
        },
        {
            id: 2,
            title: "Diseño",
            icon: '<svg stroke="currentColor" fill="none" stroke-width="2" viewBox="0 0 24 24" class="icon-svg" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"/><path stroke-linecap="round" stroke-linejoin="round" d="M19 13v5a2 2 0 01-2 2H5a2 2 0 01-2-2v-5"/></svg>',
            desc: [
                "Diseñamos la experiencia de usuario (UX).",
                "Creamos estructuras, wireframes y prototipos.",
                "Definimos identidad visual y diseño de interfaz (UI).",
                "Validamos la propuesta antes de iniciar el desarrollo.",
            ],
            footer: "El diseño nos permite visualizar el producto antes de construirlo, optimizando tiempo y evitando retrabajos.",
        },
        {
            id: 3,
            title: "Desarrollo y Pruebas",
            icon: '<svg stroke="currentColor" fill="none" stroke-width="2" viewBox="0 0 24 24" class="icon-svg" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"/><circle cx="12" cy="12" r="3"/></svg>',
            desc: [
                "Desarrollamos el frontend y backend.",
                "Estructuramos y configuramos bases de datos.",
                "Implementamos medidas de seguridad y rendimiento.",
                "Realizamos pruebas técnicas y funcionales.",
                "Validamos estabilidad, usabilidad y desempeño.",
            ],
            footer: "Garantizamos que el sistema funcione correctamente en entornos reales antes de su lanzamiento.",
        },
        {
            id: 4,
            title: "Entrega",
            icon: '<svg stroke="currentColor" fill="none" stroke-width="2" viewBox="0 0 24 24" class="icon-svg" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" d="M20 7l-8-4-8 4m16 0l-8 4m8-4v10l-8 4m0-10L4 7m8 4v10M4 7v10l8 4"/></svg>',
            desc: [
                "Puesta en producción.",
                "Configuración final de servidores o entornos.",
                "Revisión conjunta con el cliente.",
                "Capacitación básica si es necesaria.",
                "Soporte inicial post-entrega.",
            ],
            footer: "No solo entregamos un producto, entregamos una solución lista para operar.",
        },
    ];

    // Row heights
    let rowHeights = $state([0, 0, 0, 0]);
    let rowNodes = $state([null, null, null, null]);
    let finalRowH = $state(0);
    let finalRowNode = $state(null);

    // Card DOM refs for measuring their size
    let cardEls = $state([null, null, null, null]);
    let cardSizes = $state([{w:0,h:0},{w:0,h:0},{w:0,h:0},{w:0,h:0}]);
    let containerPL = $state(0); // padding-left of container, needed to offset nodes
    let finalCardEl = $state(null);
    let finalCardSize = $state({w:0, h:0});

    // Final card fill animation derived values
    let finalFill = $derived(cardFill(globalProgress, 90));
    let fPerim = $derived(finalCardSize.w && finalCardSize.h ? 2 * (finalCardSize.w + finalCardSize.h) : 0);

    $effect(() => {
        if (rowNodes.some((n) => !n) || !finalRowNode) return;
        const allNodes = [...rowNodes, finalRowNode, containerNode, ...cardEls.filter(Boolean), finalCardEl].filter(Boolean);
        const obs = new ResizeObserver(() => {
            rowHeights = rowNodes.map((n) => n.getBoundingClientRect().height);
            finalRowH = finalRowNode.getBoundingClientRect().height;
            if (containerNode) {
                const cs = getComputedStyle(containerNode);
                const pl = parseFloat(cs.paddingLeft);
                const pr = parseFloat(cs.paddingRight);
                svgW = containerNode.getBoundingClientRect().width - pl - pr;
                containerPL = pl;
            }
            svgH = containerNode?.getBoundingClientRect().height ?? 0;
            // Measure cards
            cardSizes = cardEls.map(el => {
                if (!el) return {w:0,h:0};
                const r = el.getBoundingClientRect();
                return {w: r.width, h: r.height};
            });
            // Measure final card
            if (finalCardEl) {
                const fr = finalCardEl.getBoundingClientRect();
                finalCardSize = {w: fr.width, h: fr.height};
            }
        });
        allNodes.forEach((n) => n && obs.observe(n));
        return () => obs.disconnect();
    });

    // SVG zigzag path
    let pathD = $derived.by(() => {
        if (!svgW || rowHeights.some((h) => !h)) return "";

        const L = 0;
        const R = svgW;
        const C = svgW / 2;

        let ys = [0];
        for (let h of rowHeights) ys.push(ys.at(-1) + h);

        const y0 = ys[0], y1 = ys[1], y2 = ys[2], y3 = ys[3], y4 = ys[4];
        const yEnd = y4 + finalRowH;

        return [
            `M ${L} ${y0}`,
            `V ${y1}`,
            `H ${R}`,
            `V ${y2}`,
            `H ${L}`,
            `V ${y3}`,
            `H ${R}`,
            `V ${y4}`,
            `H ${C}`,
            `V ${yEnd}`,
        ].join(" ");
    });

    // Node positions at zigzag vertices
    let nodePositions = $derived.by(() => {
        if (!svgW || rowHeights.some((h) => !h)) return [];

        let ys = [0];
        for (let h of rowHeights) ys.push(ys.at(-1) + h);

        return [
            { x: 0,    y: 0 },
            { x: svgW, y: ys[1] },
            { x: 0,    y: ys[2] },
            { x: svgW, y: ys[3] },
        ];
    });
</script>

<svelte:window bind:scrollY bind:innerHeight />

<!-- Hidden SVG defs: noise filter extracted from rect.svg -->
<svg class="absolute w-0 h-0 overflow-hidden" aria-hidden="true">
    <defs>
        <filter id="noiseFilter" x="0" y="0" width="100%" height="100%" filterUnits="objectBoundingBox" color-interpolation-filters="sRGB">
            <feFlood flood-opacity="0" result="BackgroundImageFix"/>
            <feBlend mode="normal" in="SourceGraphic" in2="BackgroundImageFix" result="shape"/>
            <feTurbulence type="fractalNoise" baseFrequency="0.32258066534996033 0.32258066534996033" stitchTiles="stitch" numOctaves="3" result="noise" seed="220" />
            <feComponentTransfer in="noise" result="coloredNoise1">
                <feFuncR type="linear" slope="2" intercept="-0.5" />
                <feFuncG type="linear" slope="2" intercept="-0.5" />
                <feFuncB type="linear" slope="2" intercept="-0.5" />
                <feFuncA type="discrete" tableValues="1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 "/>
            </feComponentTransfer>
            <feComposite operator="in" in2="shape" in="coloredNoise1" result="noise1Clipped" />
            <feComponentTransfer in="noise1Clipped" result="color1">
                <feFuncA type="table" tableValues="0 0.1" />
            </feComponentTransfer>
            <feMerge result="effect1_noise">
                <feMergeNode in="shape" />
                <feMergeNode in="color1" />
            </feMerge>
        </filter>
    </defs>
</svg>

<section class="bg-black py-16 sm:py-20 md:py-28 font-sans overflow-hidden">
    <div class="max-w-5xl w-full mx-auto px-4 sm:px-6 md:px-12 lg:px-16">
        <!-- Section header -->
        <div class="text-center mb-14 sm:mb-18 md:mb-24">
            <h2 class="section-title text-white font-extrabold tracking-tight mb-3 leading-tight">
                PROCESO DE DESARROLLO
            </h2>
            <p class="text-sm sm:text-base text-zinc-400 max-w-md mx-auto leading-relaxed">
                Establecemos prioridades y funcionalidades clave.
            </p>
        </div>

        <!-- Zigzag timeline container -->
        <div bind:this={containerNode} class="relative w-full timeline-container">
            <!-- SVG track -->
            {#if pathD}
                <svg
                    class="absolute top-0 bottom-0 pointer-events-none z-0 overflow-visible svg-track"
                    width={svgW}
                    height={svgH}
                    viewBox="0 0 {svgW} {svgH}"
                    preserveAspectRatio="none"
                >
                    <path
                        d={pathD}
                        fill="none"
                        stroke="#27272a"
                        stroke-width="2"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                    />
                    <path
                        bind:this={pathEl}
                        d={pathD}
                        fill="none"
                        stroke="#e85d04"
                        stroke-width="2"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-dasharray={totalLen}
                        stroke-dashoffset={dashOffset}
                    />
                </svg>

                <!-- Numbered nodes at zigzag vertices -->
                {#each steps as step, i}
                    {@const pos = nodePositions[i]}
                    {@const nodeActive = globalProgress >= THRESHOLDS[i]}
                    {#if pos}
                        <div
                            class="node-circle absolute z-20 rounded-full border-4 border-[#050505]
                                   flex items-center justify-center font-bold
                                   transition-all duration-500"
                            class:node-active={nodeActive}
                            style="left: {pos.x + containerPL}px; top: {pos.y}px; transform: translate(-50%, -50%);"
                        >
                            {step.id}
                        </div>
                    {/if}
                {/each}
            {/if}

            <!-- Steps -->
            {#each steps as step, i}
                {@const isLtr = i % 2 === 0}
                {@const nodeActive = globalProgress >= THRESHOLDS[i]}
                {@const fill = cardFill(globalProgress, THRESHOLDS[i])}
                {@const cw = cardSizes[i].w}
                {@const ch = cardSizes[i].h}
                {@const perim = cw && ch ? 2 * (cw + ch) : 0}

                <div
                    bind:this={rowNodes[i]}
                    class="step-row relative w-full"
                >
                    <div class="content-row" class:is-ltr={isLtr} class:is-rtl={!isLtr}>

                        <!-- Title side -->
                        <div class="title-side">
                            <div class="icon-wrap transition-all duration-500" class:icon-active={nodeActive}>
                                {@html step.icon}
                            </div>
                            <h3
                                class="step-title font-black uppercase tracking-widest leading-tight transition-colors duration-500"
                                class:title-active={nodeActive}
                            >
                                {step.title}
                            </h3>
                        </div>

                        <!-- Card side with SVG fill animation -->
                        <div class="card-side">
                            <div
                                bind:this={cardEls[i]}
                                class="card-outer relative overflow-hidden rounded-2xl"
                            >
                                <!-- SVG overlay: draws orange border + fills background -->
                                {#if perim > 0}
                                    <svg
                                        class="absolute inset-0 w-full h-full pointer-events-none z-0"
                                        viewBox="0 0 {cw} {ch}"
                                        preserveAspectRatio="none"
                                    >
                                        <!-- Orange fill rect that fades in -->
                                        <rect
                                            x="0" y="0"
                                            width={cw} height={ch}
                                            rx="16" ry="16"
                                            fill="#e85d04"
                                            opacity={fill / 100}
                                        />
                                        <!-- Noise texture overlay (appears with orange fill) -->
                                        <rect
                                            x="0" y="0"
                                            width={cw} height={ch}
                                            rx="16" ry="16"
                                            fill="#2A2A2A"
                                            fill-opacity="0.4"
                                            filter="url(#noiseFilter)"
                                            opacity={fill / 100}
                                        />
                                    </svg>
                                {/if}

                                <!-- Card content on top -->
                                <div class="card relative z-10">
                                    <ul class="card-list list-none">
                                        {#each step.desc as item}
                                            <li
                                                class="card-item relative transition-colors duration-300"
                                                class:item-active={fill > 50}
                                            >
                                                <span class="bullet" class:bullet-active={fill > 50}>&#8226;</span>
                                                {item}
                                            </li>
                                        {/each}
                                    </ul>
                                    <p
                                        class="card-footer leading-relaxed transition-colors duration-300"
                                        class:footer-active={fill > 50}
                                    >
                                        {step.footer}
                                    </p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            {/each}

            <!-- Final closing section -->
            <div bind:this={finalRowNode} class="relative w-full pt-2 pb-8">
                <div class="final-content flex flex-col items-center text-center max-w-xl mx-auto">
                    <div
                        bind:this={finalCardEl}
                        class="final-card-outer relative overflow-hidden rounded-2xl"
                    >
                        {#if fPerim > 0}
                            <svg
                                class="absolute inset-0 w-full h-full pointer-events-none z-0"
                                viewBox="0 0 {finalCardSize.w} {finalCardSize.h}"
                                preserveAspectRatio="none"
                            >
                                <rect
                                    x="0" y="0"
                                    width={finalCardSize.w} height={finalCardSize.h}
                                    rx="16" ry="16"
                                    fill="#e85d04"
                                    opacity={finalFill / 100}
                                />
                                <!-- Noise texture overlay (appears with orange fill) -->
                                <rect
                                    x="0" y="0"
                                    width={finalCardSize.w} height={finalCardSize.h}
                                    rx="16" ry="16"
                                    fill="#2A2A2A"
                                    fill-opacity="0.4"
                                    filter="url(#noiseFilter)"
                                    opacity={finalFill / 100}
                                />
                            </svg>
                        {/if}
                        <p
                            class="relative z-10 leading-relaxed final-text transition-colors duration-300"
                            class:final-text-active={finalFill > 50}
                        >
                            Cada proyecto representa una oportunidad para optimizar
                            procesos, fortalecer tu presencia digital y mejorar la
                            experiencia de tus clientes. Nuestro equipo está
                            preparado para acompañarte desde la idea inicial hasta
                            la implementación final, con una metodología clara,
                            comunicación constante y soluciones diseñadas para
                            crecer junto a tu negocio. Si estás buscando un equipo comprometido,
                            estructurado y enfocado en resultados reales, estamos listos para
                            comenzar.
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</section>

<style>
    /* ── SVG: offset by container padding ── */
    .svg-track {
        left: 1rem;
        right: 1rem;
    }
    @media (min-width: 640px)  { .svg-track { left: 1.5rem; right: 1.5rem; } }
    @media (min-width: 768px)  { .svg-track { left: 2rem;   right: 2rem;   } }

    /* ── Timeline container padding ── */
    .timeline-container {
        padding-left: 1rem;
        padding-right: 1rem;
    }
    @media (min-width: 640px)  { .timeline-container { padding-left: 1.5rem; padding-right: 1.5rem; } }
    @media (min-width: 768px)  { .timeline-container { padding-left: 2rem;   padding-right: 2rem;   } }

    /* ── Section title ── */
    .section-title {
        font-size: 1.5rem;
        letter-spacing: 0.05em;
    }
    @media (min-width: 640px)  { .section-title { font-size: 2rem; } }
    @media (min-width: 768px)  { .section-title { font-size: 2.5rem; } }
    @media (min-width: 1024px) { .section-title { font-size: 3rem; } }

    /* ── Step row ── */
    .step-row {
        padding-top: 1.5rem;
        padding-bottom: 1.5rem;
    }
    @media (min-width: 640px)  { .step-row { padding-top: 2rem; padding-bottom: 2rem; } }
    @media (min-width: 768px)  { .step-row { padding-top: 2.5rem; padding-bottom: 2.5rem; } }
    @media (min-width: 1024px) { .step-row { padding-top: 3rem; padding-bottom: 3rem; } }

    /* ── Content row ── */
    .content-row {
        display: flex;
        align-items: center;
        gap: 0.75rem;
    }
    @media (min-width: 640px)  { .content-row { gap: 1rem; } }
    @media (min-width: 768px)  { .content-row { gap: 1.5rem; } }
    @media (min-width: 1024px) { .content-row { gap: 2rem; } }

    .content-row.is-ltr {
        flex-direction: row;
    }
    .content-row.is-rtl {
        flex-direction: row-reverse;
    }

    /* ── Title side ── */
    .title-side {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        text-align: center;
        width: 35%;
        min-width: 100px;
        flex-shrink: 0;
        gap: 0.5rem;
        padding: 0.75rem 0.25rem;
    }
    @media (min-width: 640px)  { .title-side { width: 35%; gap: 0.625rem; padding: 1rem 0.5rem; } }
    @media (min-width: 768px)  { .title-side { width: 38%; gap: 0.75rem; padding: 1.25rem 0.75rem; } }
    @media (min-width: 1024px) { .title-side { width: 40%; gap: 1rem; padding: 1.5rem 1rem; } }

    /* ── Icon ── */
    .icon-wrap {
        color: #3f3f46;
    }
    .icon-active {
        color: #e85d04 !important;
    }

    :global(.icon-svg) {
        width: 2.5rem;
        height: 2.5rem;
    }
    @media (min-width: 640px)  { :global(.icon-svg) { width: 3rem; height: 3rem; } }
    @media (min-width: 768px)  { :global(.icon-svg) { width: 3.5rem; height: 3.5rem; } }
    @media (min-width: 1024px) { :global(.icon-svg) { width: 4rem; height: 4rem; } }

    /* ── Step title ── */
    .step-title {
        font-size: 0.875rem;
        color: #52525b;
    }
    @media (min-width: 640px)  { .step-title { font-size: 1rem; } }
    @media (min-width: 768px)  { .step-title { font-size: 1.25rem; } }
    @media (min-width: 1024px) { .step-title { font-size: 1.5rem; } }

    .title-active {
        color: #fff !important;
    }

    /* ── Node circles at vertices ── */
    .node-circle {
        width: 2rem;
        height: 2rem;
        font-size: 0.75rem;
        background: #27272a;
        color: #71717a;
    }
    @media (min-width: 640px)  { .node-circle { width: 2.25rem; height: 2.25rem; font-size: 0.8rem; } }
    @media (min-width: 768px)  { .node-circle { width: 2.5rem; height: 2.5rem; font-size: 0.875rem; } }

    .node-active {
        background: #e85d04 !important;
        color: #fff !important;
        box-shadow: 0 0 16px rgba(232, 93, 4, 0.5);
    }

    /* ── Card side ── */
    .card-side {
        flex: 1;
        min-width: 0;
        display: flex;
        align-items: center;
        position: relative;
        z-index: 10;
    }

    /* ── Card outer: container for SVG + content ── */
    .card-outer {
        width: 100%;
        position: relative;
        background: #0a0a0a;
        border: 1px solid #27272a;
    }

    /* ── Card content ── */
    .card {
        padding: 1rem;
        color: #a1a1aa;
    }
    @media (min-width: 640px)  { .card { padding: 1.25rem; } }
    @media (min-width: 768px)  { .card { padding: 1.5rem; } }
    @media (min-width: 1024px) { .card { padding: 1.75rem; } }

    /* ── Card list ── */
    .card-list {
        font-size: 0.7rem;
        margin-bottom: 0.625rem;
        line-height: 1.6;
    }
    @media (min-width: 640px)  { .card-list { font-size: 0.75rem; margin-bottom: 0.75rem; } }
    @media (min-width: 768px)  { .card-list { font-size: 0.8125rem; margin-bottom: 1rem; } }
    @media (min-width: 1024px) { .card-list { font-size: 0.875rem; } }

    .card-item {
        padding-left: 0.875rem;
        margin-bottom: 0.25rem;
        position: relative;
        color: #a1a1aa;
    }
    @media (min-width: 640px)  { .card-item { padding-left: 1rem; margin-bottom: 0.375rem; } }
    @media (min-width: 768px)  { .card-item { margin-bottom: 0.5rem; } }

    .item-active {
        color: #fff !important;
    }

    .bullet {
        position: absolute;
        left: 0;
        color: #e85d04;
        font-weight: bold;
    }
    .bullet-active {
        color: rgba(255, 255, 255, 0.7) !important;
    }

    /* ── Card footer ── */
    .card-footer {
        font-size: 0.625rem;
        color: #71717a;
    }
    @media (min-width: 640px)  { .card-footer { font-size: 0.7rem; } }
    @media (min-width: 768px)  { .card-footer { font-size: 0.75rem; } }

    .footer-active {
        color: rgba(255, 255, 255, 0.6) !important;
    }

    /* ── Final content ── */
    .final-content {
        padding-top: 2rem;
        gap: 1rem;
    }
    @media (min-width: 640px)  { .final-content { padding-top: 2.5rem; } }
    @media (min-width: 768px)  { .final-content { padding-top: 3rem; } }

    .final-card-outer {
        width: 100%;
        position: relative;
        background: #0a0a0a;
        border: 1px solid #27272a;
    }

    .final-text {
        font-size: 0.7rem;
        padding: 1.25rem;
        color: #a1a1aa;
    }
    @media (min-width: 640px)  { .final-text { font-size: 0.75rem; padding: 1.5rem; } }
    @media (min-width: 768px)  { .final-text { font-size: 0.8125rem; padding: 1.75rem; } }

    .final-text-active {
        color: #fff !important;
    }
</style>
