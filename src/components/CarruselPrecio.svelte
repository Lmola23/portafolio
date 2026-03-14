<script lang="ts">
    import useEmblaCarousel from "embla-carousel-svelte";

    const precios = [
        {
            title: "Sitios webs",
            text: "Impulsa tu negocio, facilítale a tus clientes encontrarte en el mercado",
            precio: 50,
            ejemplo: ["Landing page", "Blog", "Ecomers"],
        },
        {
            title: "Soluciones Empresariales",
            text: "Optimiza tus procesos internos con herramientas a medida y escalables",
            precio: 150,
            ejemplo: ["Sistemas CRM", "Paneles de control", "Automatización"],
        },
        {
            title: "Aplicaciones de Servidor",
            text: "Infraestructura robusta para manejar grandes volúmenes de datos y usuarios",
            precio: 250,
            ejemplo: ["API Rest", "Microservicios", "Cloud Hosting"],
        },
    ];

    let selectedIndex = $state(1);
    let emblaApi: any = null;

    const options = {
        align: "center" as const,
        startIndex: 1,
    };

    function onInit(event: CustomEvent) {
        emblaApi = event.detail;
        selectedIndex = emblaApi.selectedScrollSnap();
        emblaApi.on("select", () => {
            selectedIndex = emblaApi.selectedScrollSnap();
        });
    }
</script>

<div class="py-20 sm:py-24 md:py-32 overflow-hidden w-full">
    <!-- Section header -->
    <div class="text-center px-4 sm:px-6 mb-12 md:mb-16">
        <h2
            class="text-3xl sm:text-4xl md:text-5xl font-bold text-white tracking-tight mb-4 leading-tight"
        >
            Invierte en tu crecimiento
        </h2>
        <p
            class="text-sm sm:text-base md:text-lg text-white/40 max-w-xl mx-auto leading-relaxed"
        >
            Soluciones para cada etapa de tu negocio, desde una presencia web
            hasta infraestructura empresarial.
        </p>
    </div>

    <!-- Carousel -->
    <div class="embla w-full">
        <div
            class="embla__viewport"
            use:useEmblaCarousel={{ options, plugins: [] }}
            onemblaInit={onInit}
        >
            <div class="embla__container">
                {#each precios as p, i}
                    {@const isActive = selectedIndex === i}
                    <div class="embla__slide">
                        <div
                            class="precio-card"
                            class:precio-card-active={isActive}
                            class:precio-card-inactive={!isActive}
                        >
                            <!-- Title -->
                            <h4 class="precio-title">{p.title}</h4>

                            <!-- Description -->
                            <p class="precio-desc">{p.text}</p>

                            <!-- Price -->
                            <div class="precio-price">
                                <span class="precio-desde">desde:</span>
                                <span class="precio-amount">${p.precio}</span>
                            </div>

                            <!-- Divider -->
                            <div class="precio-divider"></div>

                            <!-- Feature summary line -->
                            <p class="precio-summary">{p.ejemplo.join(", ")}</p>

                            <!-- Feature list -->
                            <ul class="precio-list">
                                {#each p.ejemplo as e}
                                    <li class="precio-list-item">{e}</li>
                                {/each}
                            </ul>

                            <!-- Spacer -->
                            <div class="precio-spacer"></div>

                            <!-- CTA -->
                            <button class="precio-cta">
                                Comprar Servicio
                            </button>
                        </div>
                    </div>
                {/each}
            </div>
        </div>
    </div>

    <!-- Dot indicators -->
    <div class="flex items-center justify-center gap-2.5 mt-8">
        {#each precios as _, i}
            <button
                onclick={() => emblaApi?.scrollTo(i)}
                class="rounded-full transition-all duration-500
                       {i === selectedIndex
                    ? 'w-7 h-2 bg-gradient-to-r from-[#F53E1D] to-[#F7810A]'
                    : 'w-2 h-2 bg-white/20 hover:bg-white/50'}"
                aria-label="Ver plan {i + 1}"
            ></button>
        {/each}
    </div>
</div>

<style>
    .embla {
        overflow: hidden;
    }

    .embla__viewport {
        overflow: visible;
        cursor: grab;
        padding: 1.5rem 0;
    }

    .embla__viewport:active {
        cursor: grabbing;
    }

    .embla__container {
        display: flex;
        touch-action: pan-y pinch-zoom;
        align-items: stretch;
    }

    .embla__slide {
        flex: 0 0 82%;
        min-width: 0;
        padding-left: 0.625rem;
        padding-right: 0.625rem;
        display: flex;
        flex-direction: column;
    }

    .embla__slide > div {
        flex: 1;
    }

    @media (min-width: 480px) {
        .embla__slide {
            flex: 0 0 70%;
        }
    }

    @media (min-width: 640px) {
        .embla__slide {
            flex: 0 0 55%;
        }
    }

    @media (min-width: 768px) {
        .embla__slide {
            flex: 0 0 38%;
            padding-left: 0.875rem;
            padding-right: 0.875rem;
        }
    }

    @media (min-width: 1024px) {
        .embla__slide {
            flex: 0 0 30%;
            padding-left: 1rem;
            padding-right: 1rem;
        }
    }

    @media (min-width: 1280px) {
        .embla__slide {
            flex: 0 0 26%;
        }
    }

    /* ── Precio card ── */
    .precio-card {
        display: flex;
        flex-direction: column;
        background: linear-gradient(145deg, #1a1a1a 0%, #111111 50%, #0d0d0d 100%);
        border-radius: 1.25rem;
        padding: 1.75rem;
        transition: all 0.5s ease-out;
    }

    @media (min-width: 640px) {
        .precio-card {
            padding: 2rem;
        }
    }

    @media (min-width: 768px) {
        .precio-card {
            padding: 2.25rem;
            border-radius: 1.5rem;
        }
    }

    .precio-card-active {
        opacity: 1;
        transform: scale(1);
    }

    .precio-card-inactive {
        opacity: 0.35;
        transform: scale(0.95);
    }

    /* ── Title ── */
    .precio-title {
        font-size: 1.375rem;
        font-weight: 700;
        color: #fff;
        line-height: 1.2;
        margin-bottom: 0.75rem;
    }

    @media (min-width: 640px) {
        .precio-title {
            font-size: 1.5rem;
        }
    }

    @media (min-width: 768px) {
        .precio-title {
            font-size: 1.75rem;
        }
    }

    /* ── Description ── */
    .precio-desc {
        font-size: 0.75rem;
        color: rgba(255, 255, 255, 0.35);
        line-height: 1.6;
        margin-bottom: 1.75rem;
    }

    @media (min-width: 640px) {
        .precio-desc {
            font-size: 0.8125rem;
        }
    }

    /* ── Price ── */
    .precio-price {
        display: flex;
        align-items: baseline;
        gap: 0.5rem;
        margin-bottom: 1.75rem;
    }

    .precio-desde {
        font-size: 0.75rem;
        color: rgba(255, 255, 255, 0.3);
        font-style: italic;
    }

    .precio-amount {
        font-size: 2.5rem;
        font-weight: 800;
        color: #fff;
        letter-spacing: -0.02em;
        line-height: 1;
    }

    @media (min-width: 640px) {
        .precio-amount {
            font-size: 3rem;
        }
    }

    /* ── Divider ── */
    .precio-divider {
        width: 100%;
        height: 1px;
        background: rgba(255, 255, 255, 0.08);
        margin-bottom: 1.5rem;
    }

    /* ── Summary line ── */
    .precio-summary {
        font-size: 0.875rem;
        font-weight: 600;
        color: rgba(255, 255, 255, 0.7);
        margin-bottom: 1.25rem;
        line-height: 1.5;
    }

    /* ── Feature list ── */
    .precio-list {
        list-style: none;
        padding: 0;
        margin: 0 0 1.5rem;
        display: flex;
        flex-direction: column;
        gap: 0.875rem;
    }

    .precio-list-item {
        font-size: 0.8125rem;
        color: rgba(255, 255, 255, 0.5);
        padding-left: 0.25rem;
    }

    /* ── Spacer ── */
    .precio-spacer {
        flex: 1;
    }

    /* ── CTA button ── */
    .precio-cta {
        width: 100%;
        padding: 0.9375rem 0;
        border-radius: 9999px;
        font-size: 0.875rem;
        font-weight: 700;
        color: #fff;
        background: linear-gradient(135deg, #F53E1D, #F55714, #F7810A);
        border: none;
        cursor: pointer;
        transition: opacity 0.3s ease, transform 0.3s ease;
    }

    .precio-cta:hover {
        opacity: 0.9;
        transform: translateY(-1px);
    }
</style>
