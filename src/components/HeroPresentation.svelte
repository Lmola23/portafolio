<script lang="ts">
    import { onMount } from "svelte";
    import * as THREE from "three";

    let canvasContainer: HTMLElement;
    let scrollY = $state(0);
    let innerHeight = $state(0);

    let progress = $derived(
        innerHeight > 0 ? Math.min(Math.max(scrollY / innerHeight, 0), 1) : 0,
    );

    onMount(() => {
        const scene = new THREE.Scene();
        const camera = new THREE.OrthographicCamera(-1, 1, 1, -1, 0, 1);
        const renderer = new THREE.WebGLRenderer({
            alpha: true,
            antialias: false,
        });
        renderer.setSize(
            canvasContainer.clientWidth,
            canvasContainer.clientHeight,
        );
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        canvasContainer.appendChild(renderer.domElement);

        // ── Fullscreen quad with photorealistic nebula/star shader ──
        const geo = new THREE.PlaneGeometry(2, 2);
        const mat = new THREE.ShaderMaterial({
            uniforms: {
                uTime: { value: 0 },
                uProgress: { value: 0 },
                uResolution: {
                    value: new THREE.Vector2(
                        canvasContainer.clientWidth,
                        canvasContainer.clientHeight,
                    ),
                },
            },
            vertexShader: `
                varying vec2 vUv;
                void main() {
                    vUv = uv;
                    gl_Position = vec4(position, 1.0);
                }
            `,
            fragmentShader: `
                precision highp float;
                varying vec2 vUv;
                uniform float uTime;
                uniform float uProgress;
                uniform vec2 uResolution;

                // ── Noise functions ──
                vec3 mod289(vec3 x){ return x - floor(x*(1.0/289.0))*289.0; }
                vec4 mod289(vec4 x){ return x - floor(x*(1.0/289.0))*289.0; }
                vec4 permute(vec4 x){ return mod289(((x*34.0)+1.0)*x); }
                vec4 taylorInvSqrt(vec4 r){ return 1.79284291400159 - 0.85373472095314*r; }

                float snoise(vec3 v){
                    const vec2 C = vec2(1.0/6.0, 1.0/3.0);
                    const vec4 D = vec4(0.0, 0.5, 1.0, 2.0);
                    vec3 i = floor(v + dot(v, C.yyy));
                    vec3 x0 = v - i + dot(i, C.xxx);
                    vec3 g = step(x0.yzx, x0.xyz);
                    vec3 l = 1.0 - g;
                    vec3 i1 = min(g, l.zxy);
                    vec3 i2 = max(g, l.zxy);
                    vec3 x1 = x0 - i1 + C.xxx;
                    vec3 x2 = x0 - i2 + C.yyy;
                    vec3 x3 = x0 - D.yyy;
                    i = mod289(i);
                    vec4 p = permute(permute(permute(
                        i.z + vec4(0.0, i1.z, i2.z, 1.0))
                        + i.y + vec4(0.0, i1.y, i2.y, 1.0))
                        + i.x + vec4(0.0, i1.x, i2.x, 1.0));
                    float n_ = 0.142857142857;
                    vec3 ns = n_*D.wyz - D.xzx;
                    vec4 j = p - 49.0*floor(p*ns.z*ns.z);
                    vec4 x_ = floor(j*ns.z);
                    vec4 y_ = floor(j - 7.0*x_);
                    vec4 x = x_*ns.x + ns.yyyy;
                    vec4 y = y_*ns.x + ns.yyyy;
                    vec4 h = 1.0 - abs(x) - abs(y);
                    vec4 b0 = vec4(x.xy, y.xy);
                    vec4 b1 = vec4(x.zw, y.zw);
                    vec4 s0 = floor(b0)*2.0 + 1.0;
                    vec4 s1 = floor(b1)*2.0 + 1.0;
                    vec4 sh = -step(h, vec4(0.0));
                    vec4 a0 = b0.xzyw + s0.xzyw*sh.xxyy;
                    vec4 a1 = b1.xzyw + s1.xzyw*sh.zzww;
                    vec3 p0 = vec3(a0.xy,h.x);
                    vec3 p1 = vec3(a0.zw,h.y);
                    vec3 p2 = vec3(a1.xy,h.z);
                    vec3 p3 = vec3(a1.zw,h.w);
                    vec4 norm = taylorInvSqrt(vec4(dot(p0,p0),dot(p1,p1),dot(p2,p2),dot(p3,p3)));
                    p0 *= norm.x; p1 *= norm.y; p2 *= norm.z; p3 *= norm.w;
                    vec4 m = max(0.6 - vec4(dot(x0,x0),dot(x1,x1),dot(x2,x2),dot(x3,x3)), 0.0);
                    m = m*m;
                    return 42.0*dot(m*m, vec4(dot(p0,x0),dot(p1,x1),dot(p2,x2),dot(p3,x3)));
                }

                // Fractal Brownian Motion
                float fbm(vec3 p) {
                    float v = 0.0;
                    float a = 0.5;
                    vec3 shift = vec3(100.0);
                    for (int i = 0; i < 5; i++) {
                        v += a * snoise(p);
                        p = p * 2.0 + shift;
                        a *= 0.5;
                    }
                    return v;
                }

                void main() {
                    vec2 uv = vUv;
                    float aspect = uResolution.x / uResolution.y;
                    vec2 p = (uv - 0.5) * vec2(aspect, 1.0);

                    // Center of the star slightly above middle (like the reference)
                    vec2 center = vec2(0.0, 0.05);
                    vec2 d = p - center;
                    float dist = length(d);

                    // Scroll: star expands and dims (fast expansion)
                    float prog = uProgress;
                    float expansion = 1.0 + prog * 8.0;
                    dist /= expansion;

                    float t = uTime * 0.06; // very slow time

                    // ── 1. Hot white-yellow core ──
                    float core = exp(-dist * 18.0);
                    vec3 coreColor = vec3(1.0, 0.95, 0.85) * core * 2.2;

                    // ── 2. Inner orange glow (smooth radial) ──
                    float inner = exp(-dist * 5.5);
                    vec3 innerColor = vec3(0.91, 0.36, 0.02) * inner * 1.4; // #e85d04

                    // ── 3. Warm amber mid-glow ──
                    float mid = exp(-dist * 3.0);
                    vec3 midColor = vec3(0.97, 0.51, 0.04) * mid * 0.7; // #F7810A

                    // ── 4. Diffuse outer halo ──
                    float outer = exp(-dist * 1.5);
                    vec3 outerColor = vec3(0.6, 0.25, 0.02) * outer * 0.35;

                    // ── 5. Gas/nebula noise texture ──
                    // Angle-dependent noise for asymmetric nebula arms
                    float angle = atan(d.y, d.x);
                    vec3 noiseCoord = vec3(d * 3.0 / expansion, t);
                    float gasNoise = fbm(noiseCoord) * 0.5 + 0.5;

                    // Wispy tendrils using angle modulation
                    float tendrils = fbm(vec3(angle * 2.0, dist * 4.0, t * 0.5));
                    tendrils = smoothstep(0.1, 0.8, tendrils);

                    // Gas visible in the mid-range ring
                    float gasMask = smoothstep(0.02, 0.15, dist) * smoothstep(0.6, 0.15, dist);
                    gasMask /= expansion;
                    vec3 gasColor = vec3(0.85, 0.42, 0.05) * gasNoise * tendrils * gasMask * 0.6;

                    // ── 6. Concentric ring hints (like the reference) ──
                    float ringDist = dist * 12.0;
                    float ring1 = exp(-pow(ringDist - 2.5, 2.0) * 4.0) * 0.12;
                    float ring2 = exp(-pow(ringDist - 4.5, 2.0) * 3.0) * 0.06;
                    vec3 ringColor = vec3(0.8, 0.4, 0.05) * (ring1 + ring2);

                    // ── 7. Very subtle ambient dust glow ──
                    float dust = fbm(vec3(p * 1.5, t * 0.3)) * 0.5 + 0.5;
                    float dustMask = smoothstep(0.7, 0.1, dist) * 0.03;
                    vec3 dustColor = vec3(0.5, 0.25, 0.05) * dust * dustMask;

                    // ── Compose ──
                    vec3 color = coreColor + innerColor + midColor + outerColor + gasColor + ringColor + dustColor;

                    // Fade everything during scroll explosion
                    float fade = 1.0 - smoothstep(0.0, 0.5, prog);
                    color *= fade;

                    // Slight vignette to black edges
                    float vignette = smoothstep(0.9, 0.3, length(p * 0.8));
                    color *= vignette;

                    gl_FragColor = vec4(color, 1.0);
                }
            `,
            depthWrite: false,
            depthTest: false,
        });

        const quad = new THREE.Mesh(geo, mat);
        scene.add(quad);

        // ── Dust particles (subtle, floating) ──
        const PARTICLE_COUNT = 300;
        const pScene = new THREE.Scene();
        const pCamera = new THREE.PerspectiveCamera(
            60,
            canvasContainer.clientWidth / canvasContainer.clientHeight,
            0.1,
            100,
        );
        pCamera.position.z = 5;

        const pGeo = new THREE.BufferGeometry();
        const pPos = new Float32Array(PARTICLE_COUNT * 3);
        const pVel = new Float32Array(PARTICLE_COUNT * 3);
        const pSizes = new Float32Array(PARTICLE_COUNT);
        const pRand = new Float32Array(PARTICLE_COUNT);

        for (let i = 0; i < PARTICLE_COUNT; i++) {
            // Distributed around the star area
            const theta = Math.random() * Math.PI * 2;
            const r = 0.3 + Math.random() * 2.5;
            pPos[i * 3] = Math.cos(theta) * r;
            pPos[i * 3 + 1] = Math.sin(theta) * r * 0.7 + 0.2;
            pPos[i * 3 + 2] = (Math.random() - 0.5) * 2;

            // Slow drift velocities for explosion
            const speed = 0.5 + Math.random() * 2.0;
            const phi = Math.acos(2 * Math.random() - 1);
            const th = Math.random() * Math.PI * 2;
            pVel[i * 3] = Math.sin(phi) * Math.cos(th) * speed;
            pVel[i * 3 + 1] = Math.sin(phi) * Math.sin(th) * speed;
            pVel[i * 3 + 2] = Math.cos(phi) * speed;

            pSizes[i] = 1.0 + Math.random() * 2.0;
            pRand[i] = Math.random();
        }

        pGeo.setAttribute("position", new THREE.BufferAttribute(pPos, 3));
        pGeo.setAttribute("aVelocity", new THREE.BufferAttribute(pVel, 3));
        pGeo.setAttribute("aSize", new THREE.BufferAttribute(pSizes, 1));
        pGeo.setAttribute("aRandom", new THREE.BufferAttribute(pRand, 1));

        const pMat = new THREE.ShaderMaterial({
            uniforms: {
                uTime: { value: 0 },
                uProgress: { value: 0 },
                uPixelRatio: { value: Math.min(window.devicePixelRatio, 2) },
            },
            vertexShader: `
                attribute vec3 aVelocity;
                attribute float aSize;
                attribute float aRandom;
                uniform float uTime;
                uniform float uProgress;
                uniform float uPixelRatio;
                varying float vAlpha;

                void main() {
                    vec3 pos = position;

                    // Gentle floating drift
                    float t = uTime * 0.15;
                    pos.x += sin(t + aRandom * 6.28) * 0.05;
                    pos.y += cos(t * 0.7 + aRandom * 3.14) * 0.04;

                    // On scroll, particles drift outward
                    float p = smoothstep(0.05, 0.7, uProgress);
                    pos += aVelocity * p * 1.5;

                    // Fade: visible at rest, fade out during scroll
                    vAlpha = (0.15 + aRandom * 0.2) * (1.0 - smoothstep(0.3, 1.0, uProgress));

                    vec4 mv = modelViewMatrix * vec4(pos, 1.0);
                    gl_PointSize = aSize * uPixelRatio * (120.0 / -mv.z);
                    gl_Position = projectionMatrix * mv;
                }
            `,
            fragmentShader: `
                varying float vAlpha;
                void main() {
                    float dist = length(gl_PointCoord - vec2(0.5));
                    if(dist > 0.5) discard;
                    float strength = 1.0 - smoothstep(0.0, 0.5, dist);
                    strength = pow(strength, 2.0);
                    vec3 color = vec3(0.9, 0.55, 0.15);
                    gl_FragColor = vec4(color, strength * vAlpha);
                }
            `,
            transparent: true,
            depthWrite: false,
            blending: THREE.AdditiveBlending,
        });
        const points = new THREE.Points(pGeo, pMat);
        pScene.add(points);

        // ── Animation ──
        const clock = new THREE.Clock();
        let currentProgress = 0;
        let animationId: number;
        renderer.autoClear = false;

        const animate = () => {
            animationId = requestAnimationFrame(animate);
            const elapsed = clock.getElapsedTime();
            currentProgress += (progress - currentProgress) * 0.035;

            mat.uniforms.uTime.value = elapsed;
            mat.uniforms.uProgress.value = currentProgress;
            pMat.uniforms.uTime.value = elapsed;
            pMat.uniforms.uProgress.value = currentProgress;

            renderer.clear();
            renderer.render(scene, camera); // nebula quad
            renderer.render(pScene, pCamera); // dust particles on top
        };
        animate();

        const handleResize = () => {
            const w = canvasContainer.clientWidth;
            const h = canvasContainer.clientHeight;
            renderer.setSize(w, h);
            mat.uniforms.uResolution.value.set(w, h);
            pCamera.aspect = w / h;
            pCamera.updateProjectionMatrix();
        };
        window.addEventListener("resize", handleResize);

        return () => {
            cancelAnimationFrame(animationId);
            window.removeEventListener("resize", handleResize);
            renderer.dispose();
            geo.dispose();
            mat.dispose();
            pGeo.dispose();
            pMat.dispose();
            if (
                canvasContainer &&
                renderer.domElement.parentNode === canvasContainer
            ) {
                canvasContainer.removeChild(renderer.domElement);
            }
        };
    });

    const navLinks = [
        { label: "Inicio", href: "#", active: true },
        { label: "Proyectos", href: "#proyectos" },
        { label: "Servicios", href: "#precios" },
        { label: "Nosotros", href: "#proceso" },
    ];

    const services = [
        "Sitios webs",
        "Soluciones empresariales",
        "Aplicaciones de servidor",
    ];

    let menuOpen = $state(false);

    function toggleMenu() {
        menuOpen = !menuOpen;
        if (menuOpen) {
            document.body.style.overflow = "hidden";
        } else {
            document.body.style.overflow = "";
        }
    }

    function closeMenu() {
        menuOpen = false;
        document.body.style.overflow = "";
    }
</script>

<svelte:window bind:scrollY bind:innerHeight />

<div class="hero-wrapper">
    <!-- Three.js canvas background -->
    <div bind:this={canvasContainer} class="canvas-container"></div>

    <!-- Dark overlay for text readability -->
    <div class="hero-overlay"></div>

    <!-- Navbar -->
    <nav class="hero-nav">
        <div class="nav-inner">
            <a href="#" class="logo">SIRIO</a>

            <!-- Desktop nav links (hidden on mobile) -->
            <div class="nav-links-desktop">
                {#each navLinks as link}
                    <a
                        href={link.href}
                        class="nav-link"
                        class:nav-link-active={link.active}
                    >
                        {link.label}
                    </a>
                {/each}
                <a href="#contacto" class="nav-contact">Contacto</a>
            </div>
        </div>
    </nav>

    <!-- Hamburger button (mobile only) — outside nav so z-index beats overlay -->
    <button
        class="hamburger"
        class:hamburger-open={menuOpen}
        onclick={toggleMenu}
        aria-label="Abrir menú"
    >
        <span class="bar bar-1"></span>
        <span class="bar bar-2"></span>
    </button>

    <!-- Fullscreen mobile menu overlay -->
    <div class="mobile-overlay" class:mobile-overlay-open={menuOpen}>
        <div class="mobile-overlay-bg"></div>
        <div class="mobile-menu-content">
            {#each navLinks as link}
                <a
                    href={link.href}
                    class="mobile-link"
                    class:mobile-link-active={link.active}
                    onclick={closeMenu}
                >
                    {link.label}
                </a>
            {/each}
            <a
                href="#contacto"
                class="mobile-link mobile-link-contact"
                onclick={closeMenu}
            >
                Contacto
            </a>
        </div>
    </div>

    <!-- Hero content -->
    <div class="hero-content">
        <h1 class="hero-title">
            DISEÑAMOS, AUTOMATIZAMOS<br />
            Y DIGITALIZAMOS, TU NEGOCIO.
        </h1>
    </div>

    <!-- Service pills at bottom
    <div class="hero-pills">
        {#each services as service}
            <button class="pill">{service}</button>
        {/each}
        </div>-->
</div>

<style>
    .hero-wrapper {
        position: relative;
        width: 100%;
        min-height: 120vh;
        min-height: 120dvh;
        overflow: hidden;
        background: #000;
        display: flex;
        flex-direction: column;
    }

    /* ── Three.js canvas ── */
    .canvas-container {
        position: absolute;
        inset: 0;
        z-index: 0;
    }
    .canvas-container :global(canvas) {
        display: block;
        width: 100% !important;
        height: 100% !important;
    }

    /* ── Dark overlay for readability ── */
    .hero-overlay {
        position: absolute;
        inset: 0;
        z-index: 1;
        background: radial-gradient(
            ellipse 60% 50% at 50% 45%,
            transparent 0%,
            rgba(0, 0, 0, 0.3) 50%,
            rgba(0, 0, 0, 0.8) 100%
        );
        pointer-events: none;
    }

    /* ── Navbar ── */
    .hero-nav {
        position: relative;
        z-index: 10;
        width: 100%;
        padding: 1.25rem 1rem;
    }
    @media (min-width: 640px) {
        .hero-nav {
            padding: 1.5rem 1.5rem;
        }
    }
    @media (min-width: 768px) {
        .hero-nav {
            padding: 1.75rem 2.5rem;
        }
    }
    @media (min-width: 1024px) {
        .hero-nav {
            padding: 2rem 3rem;
        }
    }

    .nav-inner {
        display: flex;
        align-items: center;
        justify-content: space-between;
        max-width: 80rem;
        margin: 0 auto;
        width: 100%;
    }

    .logo {
        font-size: 1.25rem;
        font-weight: 800;
        color: #fff;
        text-decoration: none;
        letter-spacing: 0.08em;
    }
    @media (min-width: 640px) {
        .logo {
            font-size: 1.5rem;
        }
    }

    .nav-links-desktop {
        display: none;
        align-items: center;
        gap: 2rem;
    }
    @media (min-width: 768px) {
        .nav-links-desktop {
            display: flex;
        }
    }

    .nav-link {
        font-size: 0.75rem;
        color: rgba(255, 255, 255, 0.5);
        text-decoration: none;
        transition: color 0.3s ease;
    }
    @media (min-width: 640px) {
        .nav-link {
            font-size: 0.875rem;
        }
    }
    .nav-link:hover {
        color: #fff;
    }
    .nav-link-active {
        color: #e85d04 !important;
    }

    .nav-contact {
        font-size: 0.75rem;
        font-weight: 600;
        color: #fff;
        text-decoration: none;
        padding: 0.375rem 1rem;
        border: 1px solid rgba(255, 255, 255, 0.3);
        border-radius: 9999px;
        transition: all 0.3s ease;
    }
    @media (min-width: 640px) {
        .nav-contact {
            font-size: 0.875rem;
            padding: 0.5rem 1.25rem;
        }
    }
    .nav-contact:hover {
        border-color: #e85d04;
        color: #e85d04;
    }

    /* ── Hero content ── */
    .hero-content {
        position: relative;
        z-index: 10;
        flex: 1;
        display: flex;
        align-items: center;
        justify-content: center;
        padding: 0 1rem;
    }

    .hero-title {
        font-size: 1.5rem;
        font-weight: 900;
        color: #fff;
        text-align: center;
        line-height: 1.15;
        letter-spacing: -0.02em;
        max-width: 50rem;
        text-shadow: 0 2px 40px rgba(0, 0, 0, 0.6);
    }
    @media (min-width: 480px) {
        .hero-title {
            font-size: 2rem;
        }
    }
    @media (min-width: 640px) {
        .hero-title {
            font-size: 2.5rem;
        }
    }
    @media (min-width: 768px) {
        .hero-title {
            font-size: 3rem;
        }
    }
    @media (min-width: 1024px) {
        .hero-title {
            font-size: 3.5rem;
        }
    }
    @media (min-width: 1280px) {
        .hero-title {
            font-size: 4rem;
        }
    }

    /* ── Service pills ── */
    .hero-pills {
        position: relative;
        z-index: 10;
        display: flex;
        justify-content: center;
        gap: 0.5rem;
        padding: 0 1rem 2.5rem;
        flex-wrap: wrap;
    }
    @media (min-width: 640px) {
        .hero-pills {
            gap: 0.75rem;
            padding-bottom: 3rem;
        }
    }
    @media (min-width: 768px) {
        .hero-pills {
            gap: 1rem;
            padding-bottom: 3.5rem;
        }
    }
    @media (min-width: 1024px) {
        .hero-pills {
            gap: 1.25rem;
            padding-bottom: 4rem;
        }
    }

    .pill {
        font-size: 0.75rem;
        font-weight: 500;
        color: rgba(255, 255, 255, 0.8);
        background: transparent;
        border: 1px solid rgba(255, 255, 255, 0.2);
        border-radius: 9999px;
        padding: 0.625rem 1.25rem;
        cursor: pointer;
        transition: all 0.3s ease;
        white-space: nowrap;
    }
    @media (min-width: 640px) {
        .pill {
            font-size: 0.8125rem;
            padding: 0.75rem 1.75rem;
        }
    }
    @media (min-width: 768px) {
        .pill {
            font-size: 0.875rem;
            padding: 0.875rem 2.25rem;
        }
    }
    @media (min-width: 1024px) {
        .pill {
            font-size: 0.9375rem;
            padding: 1rem 2.75rem;
        }
    }
    .pill:hover {
        border-color: #e85d04;
        color: #fff;
        background: rgba(232, 93, 4, 0.1);
    }

    /* ── Hamburger button (mobile only) ── */
    .hamburger {
        position: fixed;
        top: 1.25rem;
        right: 1rem;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        gap: 8px;
        width: 40px;
        height: 40px;
        background: none;
        border: none;
        cursor: pointer;
        padding: 0;
        z-index: 60;
    }
    @media (min-width: 768px) {
        .hamburger {
            display: none;
        }
    }

    .bar {
        display: block;
        width: 24px;
        height: 2px;
        background: #a1a1aa;
        border-radius: 2px;
        transition: transform 0.3s cubic-bezier(0.22, 1, 0.36, 1);
        transform-origin: center;
    }

    /* Animate bars to X on open */
    .hamburger-open .bar-1 {
        transform: translateY(5px) rotate(45deg);
    }
    .hamburger-open .bar-2 {
        transform: translateY(-5px) rotate(-45deg);
    }

    /* ── Mobile overlay ── */
    .mobile-overlay {
        position: fixed;
        inset: 0;
        z-index: 50;
        opacity: 0;
        pointer-events: none;
        transition: opacity 0.3s cubic-bezier(0.22, 1, 0.36, 1);
        display: flex;
        align-items: center;
        justify-content: center;
    }
    .mobile-overlay-open {
        opacity: 1;
        pointer-events: auto;
    }

    .mobile-overlay-bg {
        position: absolute;
        inset: 0;
        background: rgba(0, 0, 0, 0.7);
        backdrop-filter: blur(20px);
        -webkit-backdrop-filter: blur(20px);
    }

    /* ── Mobile menu content ── */
    .mobile-menu-content {
        position: relative;
        z-index: 1;
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 2rem;
    }

    .mobile-link {
        font-size: 1.5rem;
        font-weight: 600;
        color: rgba(255, 255, 255, 0.85);
        text-decoration: none;
        transition: color 0.3s ease;
        letter-spacing: 0.02em;
    }
    .mobile-link:hover {
        color: #fff;
    }
    .mobile-link-active {
        color: #e85d04 !important;
    }
    .mobile-link-contact {
        font-size: 1.125rem;
        padding: 0.625rem 2rem;
        border: 1px solid rgba(255, 255, 255, 0.3);
        border-radius: 9999px;
        margin-top: 1rem;
        transition: all 0.3s ease;
    }
    .mobile-link-contact:hover {
        border-color: #e85d04;
        color: #e85d04;
    }
</style>
