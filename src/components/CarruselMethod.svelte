<script lang="ts">
    import useEmblaCarousel from "embla-carousel-svelte";
    import autoplay from "embla-carousel-autoplay";

    const imgUrls = import.meta.glob("./../assets/Method/*.png", {
        eager: true,
        import: "default",
    });
    const images = Object.values(imgUrls).map((url: any) => url.src || url);

    const methods = [
        {
            title: "Descubrimiento",
            text: "Analizamos tu negocio, competencia y usuarios para definir una estrategia clara y objetivos medibles desde el día uno.",
            img: images[0] || "",
        },
        {
            title: "Estrategia",
            text: "Diseñamos la arquitectura de la solución, definiendo tecnologías, flujos de usuario y los hitos del proyecto.",
            img: images[1] || "",
        },
        {
            title: "Diseño UI/UX",
            text: "Creamos interfaces atractivas, intuitivas y centradas en el usuario, garantizando una experiencia memorable.",
            img: images[2] || "",
        },
        {
            title: "Desarrollo",
            text: "Escribimos código limpio, escalable y optimizado. Construimos soluciones robustas preparadas para el futuro.",
            img: images[3] || "",
        },
        {
            title: "Lanzamiento",
            text: "Desplegamos tu producto con las mejores prácticas, asegurando rendimiento, seguridad y disponibilidad total.",
            img: images[4] || "",
        },
    ];

    let selectedIndex = $state(0);
    let emblaApi: any = null;

    const options = { loop: true, align: "start" as const };
    const plugins = [autoplay({ delay: 4000, stopOnInteraction: false })];

    function onInit(event: CustomEvent) {
        emblaApi = event.detail;
        selectedIndex = emblaApi.selectedScrollSnap();
        emblaApi.on("select", () => {
            selectedIndex = emblaApi.selectedScrollSnap();
        });
    }
</script>

<div class="w-full">
    <!-- Carousel -->
    <div class="embla w-full">
        <div
            class="embla__viewport"
            use:useEmblaCarousel={{ options, plugins }}
            onemblaInit={onInit}
        >
            <div class="embla__container">
                {#each methods as method, i}
                    <div class="embla__slide">
                        <div
                            class="method-card"
                            style="background-image: url({method.img});"
                        >
                            <!-- Dark gradient overlay -->
                            <div class="method-overlay"></div>

                            <!-- Title at top, centered, uppercase -->
                            <div class="method-top">
                                <h4 class="method-title">
                                    {method.title}
                                </h4>
                                <div class="method-divider"></div>
                            </div>

                            <!-- Description centered in middle -->
                            <div class="method-body">
                                <p class="method-text">
                                    {method.text}
                                </p>
                            </div>

                            <!-- Step number bottom right, orange -->
                            <div class="method-number">
                                {String(i + 1).padStart(2, "0")}
                            </div>
                        </div>
                    </div>
                {/each}
            </div>
        </div>
    </div>

    <!-- Dot indicators -->
    <div class="flex items-center justify-center gap-2.5 mt-7">
        {#each methods as _, i}
            <button
                onclick={() => emblaApi?.scrollTo(i)}
                class="rounded-full transition-all duration-500
                       {i === selectedIndex
                    ? 'w-7 h-2 bg-gradient-to-r from-[#F53E1D] to-[#F7810A]'
                    : 'w-2 h-2 bg-white/20 hover:bg-white/50'}"
                aria-label="Ir al paso {i + 1}"
            ></button>
        {/each}
    </div>
</div>

<style>
    .embla {
        max-width: 100%;
        margin: 0 auto;
    }

    .embla__viewport {
        overflow: hidden;
        width: 100%;
    }

    .embla__container {
        display: flex;
        touch-action: pan-y pinch-zoom;
        margin-left: -0.75rem;
    }

    .embla__slide {
        flex: 0 0 88%;
        min-width: 0;
        padding-left: 0.75rem;
    }

    @media (min-width: 640px) {
        .embla__slide {
            flex: 0 0 75%;
        }
    }

    @media (min-width: 768px) {
        .embla__container {
            margin-left: -1rem;
        }
        .embla__slide {
            flex: 0 0 50%;
            padding-left: 1rem;
        }
    }

    @media (min-width: 1024px) {
        .embla__container {
            margin-left: -1.25rem;
        }
        .embla__slide {
            flex: 0 0 38%;
            padding-left: 1.25rem;
        }
    }

    @media (min-width: 1280px) {
        .embla__slide {
            flex: 0 0 33.333%;
        }
    }

    /* ── Method card ── */
    .method-card {
        position: relative;
        width: 100%;
        height: 360px;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        padding: 1.5rem;
        background-size: cover;
        background-position: center;
        background-repeat: no-repeat;
        border-radius: 1rem;
        overflow: hidden;
        cursor: pointer;
        user-select: none;
    }

    @media (min-width: 640px) {
        .method-card {
            height: 420px;
            padding: 1.75rem;
        }
    }

    @media (min-width: 768px) {
        .method-card {
            height: 480px;
            padding: 2rem;
        }
    }

    @media (min-width: 1024px) {
        .method-card {
            height: 520px;
            padding: 2.25rem;
        }
    }

    .method-overlay {
        position: absolute;
        inset: 0;
        background: linear-gradient(
            to bottom,
            rgba(0, 0, 0, 0.55) 0%,
            rgba(0, 0, 0, 0.45) 40%,
            rgba(0, 0, 0, 0.6) 70%,
            rgba(0, 0, 0, 0.8) 100%
        );
        z-index: 0;
    }

    /* ── Title area: top center ── */
    .method-top {
        position: relative;
        z-index: 1;
        text-align: center;
    }

    .method-title {
        font-size: 0.8125rem;
        font-weight: 700;
        color: #fff;
        text-transform: uppercase;
        letter-spacing: 0.1em;
        line-height: 1.4;
        margin-bottom: 0.75rem;
    }

    @media (min-width: 640px) {
        .method-title {
            font-size: 0.875rem;
        }
    }

    @media (min-width: 768px) {
        .method-title {
            font-size: 0.9375rem;
            letter-spacing: 0.12em;
        }
    }

    .method-divider {
        width: 100%;
        height: 1px;
        background: rgba(255, 255, 255, 0.12);
    }

    /* ── Description: centered middle ── */
    .method-body {
        position: relative;
        z-index: 1;
        text-align: center;
        padding: 0 0.5rem;
    }

    .method-text {
        font-size: 0.8125rem;
        line-height: 1.7;
        color: rgba(255, 255, 255, 0.65);
        font-weight: 400;
    }

    @media (min-width: 640px) {
        .method-text {
            font-size: 0.875rem;
        }
    }

    @media (min-width: 768px) {
        .method-text {
            font-size: 0.9375rem;
            line-height: 1.75;
            padding: 0 1rem;
        }
    }

    /* ── Step number: bottom right, orange ── */
    .method-number {
        position: relative;
        z-index: 1;
        text-align: right;
        font-size: 2rem;
        font-weight: 800;
        color: #e85d04;
        line-height: 1;
        font-variant-numeric: tabular-nums;
    }

    @media (min-width: 640px) {
        .method-number {
            font-size: 2.5rem;
        }
    }

    @media (min-width: 768px) {
        .method-number {
            font-size: 3rem;
        }
    }
</style>
