<script lang="ts">
    import useEmblaCarousel from "embla-carousel-svelte";
    import autoScroll from "embla-carousel-auto-scroll";

    const imgs = import.meta.glob("./../assets/tech/*.{png,jpg,svg}", {
        eager: true,
        import: "default",
    });
    const imgUrls = Object.values(imgs).map((img: any) => img.src || img);

    const techOptions = { loop: true };
    const techPlugins = [
        autoScroll({ playOnInit: true, speed: 1.5, stopOnInteraction: false }),
    ];

    const navLinks = [
        { label: "Inicio", href: "#" },
        { label: "Proyecto", href: "#proyectos" },
        { label: "Servicios", href: "#precios" },
        { label: "Nosotros", href: "#proceso" },
    ];

    const serviceLinks = [
        "Sitios web",
        "Soluciones empresariales",
        "Aplicaciones de servidor",
    ];

    // Generate enough SIRIO repetitions per row
    const sirioCount = 20;
    const rows = 8;
</script>

<footer class="footer-wrapper">
    <!-- Layer 1: Diagonal infinite scrolling SIRIO text rows -->
    <div class="sirio-bg" aria-hidden="true">
        <div class="sirio-diagonal">
            {#each Array(rows) as _, r}
                <div class="sirio-row" class:sirio-row-reverse={r % 2 === 1}>
                    <div class="sirio-track">
                        {#each Array(sirioCount) as _}
                            <span class="sirio-word">SIRIO</span>
                        {/each}
                    </div>
                    <div class="sirio-track" aria-hidden="true">
                        {#each Array(sirioCount) as _}
                            <span class="sirio-word">SIRIO</span>
                        {/each}
                    </div>
                </div>
            {/each}
        </div>
    </div>

    <!-- Layer 2: Dark overlay -->
    <div class="footer-overlay"></div>

    <!-- Content -->
    <div class="footer-content">
        <!-- Contactanos -->
        <div class="footer-contact">
            <svg
                class="contact-icon"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="1.5"
            >
                <path
                    d="M16 7a4 4 0 1 1-8 0 4 4 0 0 1 8 0ZM12 14a7 7 0 0 0-7 7h14a7 7 0 0 0-7-7Z"
                />
                <path d="M19 8v3m0 0v3m0-3h3m-3 0h-3" stroke-width="2" />
            </svg>
            <span class="contact-text">CONTACTANOS</span>
        </div>

        <!-- SIRIO logo -->
        <h2 class="footer-logo">SIRIO</h2>

        <!-- Navigation + Services columns -->
        <div class="footer-columns">
            <div class="footer-col">
                <h3 class="footer-col-title">NAVEGACION</h3>
                <ul class="footer-col-list">
                    {#each navLinks as link}
                        <li>
                            <a href={link.href} class="footer-link"
                                >{link.label}</a
                            >
                        </li>
                    {/each}
                </ul>
            </div>
            <div class="footer-col">
                <h3 class="footer-col-title">SERVICIOS</h3>
                <ul class="footer-col-list">
                    {#each serviceLinks as service}
                        <li>
                            <span class="footer-link-static">{service}</span>
                        </li>
                    {/each}
                </ul>
            </div>
        </div>
    </div>

    <!-- Layer 4: Tech carousel strip at bottom -->
    <div class="footer-tech">
        <div class="tech-embla">
            <div
                class="tech-viewport"
                use:useEmblaCarousel={{
                    options: techOptions,
                    plugins: techPlugins,
                }}
            >
                <div class="tech-container">
                    {#each imgUrls as url}
                        <div class="tech-slide">
                            <img class="tech-img" src={url} alt="tecnología" />
                        </div>
                    {/each}
                </div>
            </div>
        </div>
    </div>
</footer>

<style>
    /* ── Footer wrapper ── */
    .footer-wrapper {
        position: relative;
        width: 100%;
        overflow: hidden;
        background: #050505;
    }

    /* ── Layer 1: Diagonal SIRIO text background ── */
    .sirio-bg {
        position: absolute;
        inset: 0;
        z-index: 0;
        overflow: hidden;
    }

    .sirio-diagonal {
        position: absolute;
        top: -20%;
        left: -30%;
        width: 160%;
        height: 140%;
        display: flex;
        flex-direction: column;
        gap: 1.5rem;
        transform: rotate(-15deg);
    }

    @media (min-width: 768px) {
        .sirio-diagonal {
            gap: 2rem;
        }
    }

    .sirio-row {
        display: flex;
        white-space: nowrap;
        animation: sirio-scroll-left 40s linear infinite;
    }

    .sirio-row-reverse {
        animation: sirio-scroll-right 40s linear infinite;
    }

    .sirio-track {
        display: flex;
        flex-shrink: 0;
    }

    .sirio-word {
        font-size: 4rem;
        font-weight: 900;
        color: rgba(255, 255, 255, 0.04);
        letter-spacing: 0.15em;
        padding: 0 1.5rem;
        user-select: none;
    }

    @media (min-width: 768px) {
        .sirio-word {
            font-size: 5.5rem;
            padding: 0 2rem;
        }
    }

    @media (min-width: 1024px) {
        .sirio-word {
            font-size: 7rem;
            padding: 0 2.5rem;
        }
    }

    @keyframes sirio-scroll-left {
        0% {
            transform: translateX(0);
        }
        100% {
            transform: translateX(-50%);
        }
    }

    @keyframes sirio-scroll-right {
        0% {
            transform: translateX(-50%);
        }
        100% {
            transform: translateX(0);
        }
    }

    /* ── Layer 2: Dark overlay ── */
    .footer-overlay {
        position: absolute;
        inset: 0;
        z-index: 1;
        background: rgba(5, 5, 5, 0.85);
        pointer-events: none;
    }

    /* ── Layer 3: Content ── */
    .footer-content {
        position: relative;
        z-index: 2;
        display: flex;
        flex-direction: column;
        align-items: center;
        text-align: center;
        padding: 3rem 1.5rem 2rem;
    }

    @media (min-width: 768px) {
        .footer-content {
            padding: 4rem 2rem 2.5rem;
        }
    }

    @media (min-width: 1024px) {
        .footer-content {
            padding: 5rem 3rem 3rem;
        }
    }

    /* ── Contactanos ── */
    .footer-contact {
        display: flex;
        align-items: center;
        gap: 0.625rem;
        margin-bottom: 2rem;
    }

    .contact-icon {
        width: 1.5rem;
        height: 1.5rem;
        color: rgba(255, 255, 255, 0.8);
    }

    @media (min-width: 768px) {
        .contact-icon {
            width: 1.75rem;
            height: 1.75rem;
        }
    }

    .contact-text {
        font-size: 0.875rem;
        font-weight: 800;
        color: #fff;
        letter-spacing: 0.12em;
    }

    @media (min-width: 768px) {
        .contact-text {
            font-size: 1rem;
        }
    }

    /* ── SIRIO logo ── */
    .footer-logo {
        font-size: 4rem;
        font-weight: 900;
        color: #e85d04;
        letter-spacing: 0.1em;
        line-height: 1;
        margin-bottom: 2.5rem;
    }

    @media (min-width: 640px) {
        .footer-logo {
            font-size: 5rem;
        }
    }

    @media (min-width: 768px) {
        .footer-logo {
            font-size: 6rem;
            margin-bottom: 3rem;
        }
    }

    @media (min-width: 1024px) {
        .footer-logo {
            font-size: 7rem;
        }
    }

    /* ── Columns ── */
    .footer-columns {
        display: flex;
        gap: 3rem;
        margin-bottom: 2.5rem;
    }

    @media (min-width: 640px) {
        .footer-columns {
            gap: 5rem;
        }
    }

    @media (min-width: 768px) {
        .footer-columns {
            gap: 6rem;
            margin-bottom: 3rem;
        }
    }

    .footer-col {
        text-align: center;
    }

    .footer-col-title {
        font-size: 0.8125rem;
        font-weight: 800;
        color: #e85d04;
        letter-spacing: 0.1em;
        margin-bottom: 1rem;
    }

    @media (min-width: 768px) {
        .footer-col-title {
            font-size: 0.875rem;
            margin-bottom: 1.25rem;
        }
    }

    .footer-col-list {
        list-style: none;
        padding: 0;
        margin: 0;
        display: flex;
        flex-direction: column;
        gap: 0.625rem;
    }

    .footer-link {
        font-size: 0.8125rem;
        color: rgba(255, 255, 255, 0.6);
        text-decoration: none;
        transition: color 0.3s ease;
    }

    .footer-link:hover {
        color: #fff;
    }

    .footer-link-static {
        font-size: 0.8125rem;
        color: rgba(255, 255, 255, 0.6);
    }

    @media (min-width: 768px) {
        .footer-link,
        .footer-link-static {
            font-size: 0.875rem;
        }
    }

    /* ── Layer 4: Tech carousel ── */
    .footer-tech {
        position: relative;
        z-index: 2;
        width: 100%;
        padding: 1.5rem 0 2rem;
    }

    .tech-embla {
        width: 100%;
        overflow: hidden;
    }

    .tech-viewport {
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

    .tech-container {
        display: flex;
        touch-action: pan-y pinch-zoom;
        gap: 2rem;
        margin-right: 2rem;
        align-items: center;
    }

    .tech-slide {
        flex: 0 0 auto;
        min-width: 0;
        padding: 0 0.75rem;
    }

    @media (min-width: 640px) {
        .tech-slide {
            padding: 0 1rem;
        }
    }

    .tech-img {
        object-fit: contain;
        width: 4rem;
        height: auto;
        opacity: 0.5;
        transition: opacity 0.4s ease;
    }

    @media (min-width: 640px) {
        .tech-img {
            width: 5rem;
        }
    }

    @media (min-width: 768px) {
        .tech-img {
            width: 6rem;
        }
    }

    .tech-img:hover {
        opacity: 0.8;
    }
</style>
