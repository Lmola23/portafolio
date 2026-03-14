<script lang="ts">
    import useEmblaCarousel from "embla-carousel-svelte";
    import autoScroll from "embla-carousel-auto-scroll";

    const imgs = import.meta.glob("./../assets/tech/*.{png,jpg,svg}", {
        eager: true,
        import: "default",
    });

    const imgUrls = Object.values(imgs).map((img: any) => img.src || img);

    const options = { loop: true };
    const plugins = [
        autoScroll({ playOnInit: true, speed: 1.5, stopOnInteraction: false }),
    ];
</script>

<div class="w-full overflow-hidden py-2">
    <div class="embla w-full overflow-hidden">
        <div
            class="embla__viewport"
            use:useEmblaCarousel={{ options, plugins }}
        >
            <div class="embla__container flex items-center">
                {#each imgUrls as url}
                    {@render Tech(url)}
                {/each}
            </div>
        </div>
    </div>
</div>

{#snippet Tech(url: string)}
    <div class="embla__slide flex-none px-3 sm:px-4 md:px-5 lg:px-6">
        <img
            class="tech-img object-contain w-30 sm:w-14 md:w-20 lg:w-24 h-auto
                   opacity-30 hover:opacity-80
                   grayscale hover:grayscale-0
                   transition-transform duration-400 ease-out
                   will-change-transform"
            src={url}
            alt="tecnología"
            loading="lazy"
            decoding="async"
            fetchpriority="low"
        />
    </div>
{/snippet}

<style>
    .embla {
        max-width: 100%;
        margin: 0 auto;
    }

    .embla__viewport {
        overflow: hidden;
        mask-image: linear-gradient(
            to right,
            transparent,
            black 10%,
            black 90%,
            transparent
        );
        -webkit-mask-image: linear-gradient(
            to right,
            transparent,
            black 10%,
            black 90%,
            transparent
        );
    }

    .embla__container {
        display: flex;
        touch-action: pan-y pinch-zoom;
        gap: 2rem;
        margin-right: 2rem;
    }

    .embla__slide {
        flex: 0 0 auto;
        min-width: 0;
    }
</style>
