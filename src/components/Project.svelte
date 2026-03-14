<script lang="ts">
    import { onMount } from "svelte";

    const proyectos: { title: string; text: string; url: string }[] = [
        {
            title: "Salon Xanadu",
            text: "Lorem Ipsum es simplemente el texto de relleno de las imprentas y archivos de texto. Lorem Ipsum ha sido el texto de relleno estándar de las industrias desde el año 1500, cuando un impresor desconocido usó una galería de textos y los mezcló.",
            url: "www.salonxanadu.com",
        },
        {
            title: "Huella al Rescate",
            text: "No sólo sobrevivió 500 años, sino que tambien ingresó como texto de relleno en documentos electrónicos, quedando esencialmente igual al original. Fue popularizado en los 60s con la creación de las hojas.",
            url: "huellasalrescate.org",
        },
    ];

    let currentIndex = $state(0);
    let isTransitioning = $state(false);
    let isLocked = $state(false);
    let sectionEl: HTMLElement;

    onMount(() => {
        let accumulatedDelta = 0;
        const threshold = 120;
        const cooldown = 1200;
        let resetTimer: ReturnType<typeof setTimeout>;

        // Touch tracking for mobile swipe
        let touchStartY = 0;
        let touchAccumulated = 0;
        const touchThreshold = 60;

        const lockScroll = () => {
            isLocked = true;
            document.body.style.overflow = "hidden";
        };

        const unlockScroll = () => {
            isLocked = false;
            document.body.style.overflow = "";
        };

        const isSectionVisible = () => {
            const rect = sectionEl.getBoundingClientRect();
            return (
                rect.top < window.innerHeight * 0.6 &&
                rect.bottom > window.innerHeight * 0.3
            );
        };

        const tryLock = (goingDown: boolean) => {
            if (!isSectionVisible()) return false;
            if (goingDown && currentIndex < proyectos.length - 1) {
                sectionEl.scrollIntoView({ behavior: "smooth" });
                lockScroll();
                return true;
            }
            if (!goingDown && currentIndex > 0) {
                sectionEl.scrollIntoView({ behavior: "smooth" });
                lockScroll();
                return true;
            }
            return false;
        };

        const navigate = (direction: number) => {
            if (isTransitioning) return;
            if (direction > 0) {
                if (currentIndex < proyectos.length - 1) {
                    isTransitioning = true;
                    currentIndex++;
                    setTimeout(() => (isTransitioning = false), cooldown);
                } else {
                    unlockScroll();
                }
            } else {
                if (currentIndex > 0) {
                    isTransitioning = true;
                    currentIndex--;
                    setTimeout(() => (isTransitioning = false), cooldown);
                } else {
                    unlockScroll();
                }
            }
        };

        // --- Wheel handler (desktop + mouse) ---
        const handleWheel = (e: WheelEvent) => {
            if (!isLocked) {
                if (tryLock(e.deltaY > 0)) {
                    e.preventDefault();
                }
                return;
            }

            e.preventDefault();
            if (isTransitioning) return;

            accumulatedDelta += e.deltaY;

            clearTimeout(resetTimer);
            resetTimer = setTimeout(() => {
                accumulatedDelta = 0;
            }, 300);

            if (accumulatedDelta > threshold) {
                accumulatedDelta = 0;
                navigate(1);
            } else if (accumulatedDelta < -threshold) {
                accumulatedDelta = 0;
                navigate(-1);
            }
        };

        // --- Touch handlers (mobile) ---
        const handleTouchStart = (e: TouchEvent) => {
            touchStartY = e.touches[0].clientY;
            touchAccumulated = 0;
        };

        const handleTouchMove = (e: TouchEvent) => {
            const deltaY = touchStartY - e.touches[0].clientY;

            if (!isLocked) {
                if (tryLock(deltaY > 0)) {
                    e.preventDefault();
                    touchStartY = e.touches[0].clientY;
                }
                return;
            }

            e.preventDefault();
            if (isTransitioning) return;

            touchAccumulated = deltaY;

            if (touchAccumulated > touchThreshold) {
                touchAccumulated = 0;
                touchStartY = e.touches[0].clientY;
                navigate(1);
            } else if (touchAccumulated < -touchThreshold) {
                touchAccumulated = 0;
                touchStartY = e.touches[0].clientY;
                navigate(-1);
            }
        };

        window.addEventListener("wheel", handleWheel, { passive: false });
        window.addEventListener("touchstart", handleTouchStart, {
            passive: true,
        });
        window.addEventListener("touchmove", handleTouchMove, {
            passive: false,
        });

        return () => {
            window.removeEventListener("wheel", handleWheel);
            window.removeEventListener("touchstart", handleTouchStart);
            window.removeEventListener("touchmove", handleTouchMove);
            clearTimeout(resetTimer);
            document.body.style.overflow = "";
        };
    });
</script>

<section
    bind:this={sectionEl}
    style="background: linear-gradient(180deg, #000000 0%, #0a0500 15%, #1a0a00 28%, #2d1200 38%, #4a1d00 46%, #7a3200 52%, #a84500 56%, #c45000 59%, #d45800 61%, #c45000 63%, #a84500 66%, #7a3200 70%, #4a1d00 76%, #2d1200 82%, #1a0a00 88%, #0a0500 94%, #000000 100%);"
    class="relative font-sans py-16 sm:py-20 md:py-28"
>
    <!-- Section header -->
    <div
        class="px-4 sm:px-8 md:px-12 lg:px-20 max-w-7xl mx-auto w-full mb-10 sm:mb-14 md:mb-16"
    >
        <h2
            class="text-3xl sm:text-4xl md:text-5xl lg:text-6xl font-bold text-white tracking-tight leading-tight"
        >
            Nuestros Proyectos
        </h2>
    </div>

    <!-- Overlay slides (same for mobile + desktop) -->
    <div
        class="relative h-[50vh] min-h-[350px] sm:min-h-[380px] md:min-h-[420px] w-full max-w-7xl mx-auto px-4 sm:px-8 md:px-12 lg:px-20"
    >
        {#each proyectos as proyecto, i}
            <div
                class="absolute inset-0 px-4 sm:px-8 md:px-12 lg:px-20 flex items-center
                       transition-all duration-1000 ease-in-out
                       {i === currentIndex
                    ? 'opacity-100 translate-y-0 z-10'
                    : i < currentIndex
                      ? 'opacity-0 -translate-y-10 z-0 pointer-events-none'
                      : 'opacity-0 translate-y-10 z-0 pointer-events-none'}"
            >
                <div
                    class="max-w-3xl w-full flex flex-col gap-4 sm:gap-5 lg:gap-6"
                >
                    <h3
                        class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl xl:text-6xl font-bold text-white leading-tight"
                    >
                        {proyecto.title}
                    </h3>

                    <div
                        class="w-14 sm:w-20 h-0.5 bg-gradient-to-r from-[#F53E1D] via-[#F55714] to-[#F7810A] rounded-full"
                    ></div>

                    <p
                        class="text-sm sm:text-base lg:text-lg leading-relaxed text-white/55 max-w-2xl"
                    >
                        {proyecto.text}
                    </p>

                    <a
                        href={`https://${proyecto.url}`}
                        target="_blank"
                        rel="noopener noreferrer"
                        class="inline-flex items-center gap-2 sm:gap-3 text-sm sm:text-base font-semibold
                               text-[#F7810A] hover:text-white transition-all duration-300
                               group mt-1 sm:mt-2 w-fit"
                    >
                        <span>Visitar proyecto</span>
                        <span
                            class="transition-transform duration-300 group-hover:translate-x-2 text-lg"
                            >→</span
                        >
                    </a>
                </div>
            </div>
        {/each}

    </div>
</section>
