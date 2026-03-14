<script lang="ts">
    import rectSvg from "../assets/rect.svg";
    import vectorSvg from "../assets/Vector.svg";
</script>

<section class="about-section">
    <div class="about-card">
        <!-- Noise texture background from rect.svg -->
        <img src={rectSvg.src} alt="" class="about-bg" aria-hidden="true" />

        <!-- Decorative Vector.svg figure -->
        <img
            src={vectorSvg.src}
            alt=""
            class="about-vector"
            aria-hidden="true"
        />

        <!-- Text content -->
        <div class="about-content">
            <p class="about-text">
                Nosotros Son A Dise&ntilde;o Estudio Enfocado En Claridad,
                Artesan&iacute;a, Y Mundo Real Ejecuci&oacute;n. Nuestro
                Trabajar Abarca Marca Sistemas, Digital Experiencias, Y
                F&iacute;sico Productos &mdash; Todo Conformado A Trav&eacute;s
                De Considerado Dise&ntilde;o Decisiones Y Preciso 3D.
            </p>
            <a href="#proyectos" class="about-cta">Ver Portafolio</a>
        </div>
    </div>
</section>

<style>
    .about-section {
        width: 100%;
        display: flex;
        justify-content: center;
        padding: 0 1rem;
    }

    .about-card {
        position: relative;
        width: 100%;
        max-width: 72rem;
        min-height: 28rem;
        border-radius: 1.5rem;
        overflow: hidden;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    @media (min-width: 768px) {
        .about-card {
            min-height: 36rem;
            border-radius: 2rem;
        }
    }

    @media (min-width: 1024px) {
        .about-card {
            min-height: 40rem;
        }
    }

    /* ── Noise texture background (rect.svg) ── */
    .about-bg {
        position: absolute;
        inset: 0;
        width: 100%;
        height: 100%;
        object-fit: cover;
        z-index: 0;
        pointer-events: none;
    }

    /* ── Decorative vector figure ── */
    .about-vector {
        position: absolute;
        bottom: -5%;
        left: 50%;
        transform: translateX(-50%);
        width: 100%;
        opacity: 0.6;
        z-index: 1;
        pointer-events: none;
    }

    @media (min-width: 768px) {
        .about-vector {
            bottom: -8%;
            width: 50%;
            max-width: 32rem;
        }
    }

    @media (min-width: 1024px) {
        .about-vector {
            bottom: -10%;
            width: 45%;
            max-width: 36rem;
        }
    }

    /* ── Text content ── */
    .about-content {
        position: relative;
        z-index: 2;
        display: flex;
        flex-direction: column;
        align-items: center;
        text-align: center;
        padding: 3rem 1.5rem;
        max-width: 38rem;
    }

    @media (min-width: 768px) {
        .about-content {
            padding: 4rem 2rem;
            max-width: 40rem;
        }
    }

    @media (min-width: 1024px) {
        .about-content {
            padding: 5rem 3rem;
            max-width: 44rem;
        }
    }

    .about-text {
        font-size: 0.9375rem;
        line-height: 1.7;
        color: rgba(255, 255, 255, 0.75);
        font-weight: 400;
        letter-spacing: 0.01em;
        margin-bottom: 2rem;
    }

    @media (min-width: 640px) {
        .about-text {
            font-size: 1.0625rem;
        }
    }

    @media (min-width: 768px) {
        .about-text {
            font-size: 1.125rem;
            line-height: 1.8;
            margin-bottom: 2.5rem;
        }
    }

    @media (min-width: 1024px) {
        .about-text {
            font-size: 1.25rem;
        }
    }

    /* ── CTA button ── */
    .about-cta {
        display: inline-block;
        font-size: 0.875rem;
        font-weight: 600;
        color: #fff;
        background: linear-gradient(135deg, #f53e1d, #f55714, #f7810a);
        padding: 0.75rem 2rem;
        border-radius: 9999px;
        text-decoration: none;
        transition:
            opacity 0.3s ease,
            transform 0.3s ease;
    }

    @media (min-width: 768px) {
        .about-cta {
            font-size: 0.9375rem;
            padding: 0.875rem 2.5rem;
        }
    }

    .about-cta:hover {
        opacity: 0.9;
        transform: translateY(-1px);
    }
</style>
