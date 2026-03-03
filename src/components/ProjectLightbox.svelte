<script>
    import { fade } from 'svelte/transition';

    export let media = []; // Array of { type: 'image' | 'video', src: string, caption: string }
    export let title;

    let isModalOpen = false;
    let currentIndex = 0;

    function openModal(index = 0) {
        currentIndex = index;
        isModalOpen = true;
        preloadAdjacent(index);
    }
    export const open = openModal; // Allow parent to trigger this

    function closeModal() { isModalOpen = false; }
    function next() { currentIndex = (currentIndex + 1) % media.length; preloadAdjacent(currentIndex); }
    function prev() { currentIndex = (currentIndex - 1 + media.length) % media.length; preloadAdjacent(currentIndex); }

    function handleKeydown(event) {
        if (!isModalOpen) return;
        if (event.key === 'Escape') closeModal();
        if (event.key === 'ArrowRight') next();
        if (event.key === 'ArrowLeft') prev();
    }

    function preloadAdjacent(index) {
        const nextIdx = (index + 1) % media.length;
        const prevIdx = (index - 1 + media.length) % media.length;
        [nextIdx, prevIdx].forEach(i => {
            if (media[i]?.type === "image") {
                const img = new Image();
                img.src = media[i].src;
            }
        });
    }
</script>

<svelte:window on:keydown={handleKeydown} />

<div on:click={() => openModal(0)} role="button" tabindex="0" class="trigger">
    <slot />
</div>

{#if isModalOpen}
<div class="lb-overlay" on:click|self={closeModal} transition:fade={{ duration: 200 }}>
    <div class="lb-container">
        <div class="lb-stage">
            {#if media[currentIndex].type === 'image'}
                <img src={media[currentIndex].src} alt={media[currentIndex].caption || title} />
            {:else if media[currentIndex].type === 'video'}
                <div class="video-wrapper">
                    <iframe
                        src={media[currentIndex].src}
                        title={title}
                        frameborder="0"
                        allow="autoplay; encrypted-media; fullscreen"
                        allowfullscreen
                    ></iframe>
                </div>
            {/if}

            {#if media.length > 1}
                <button class="nav prev" on:click|stopPropagation={prev}>‹</button>
                <button class="nav next" on:click|stopPropagation={next}>›</button>
            {/if}
        </div>

        <div class="lb-info">
            <div class="info-text">
                <h3>{title} <span class="counter">({currentIndex + 1}/{media.length})</span></h3>
                {#if media[currentIndex].caption}
                    <p class="caption">{media[currentIndex].caption}</p>
                {/if}
            </div>
            <button class="close-btn" on:click={closeModal}>Close</button>
        </div>
    </div>
</div>
{/if}

<style>
    .trigger { cursor: pointer; display: block; width: 100%; }
    
    .lb-overlay {
        position: fixed;
        inset: 0;
        background: rgba(0, 0, 0, 0.95);
        display: flex;
        justify-content: center;
        align-items: center;
        z-index: 2000;
        padding: 2rem;
    }

    .lb-container {
        display: flex;
        flex-direction: column;
        max-width: 90vw;
        gap: 1rem;
    }

    .lb-stage {
        position: relative;
        display: flex;
        justify-content: center;
        align-items: center;
        background: #000;
    }

    img, .video-wrapper {
        max-height: 75vh;
        max-width: 100%;
        object-fit: contain;
        border: 2px solid #333;
    }

    .video-wrapper {
        aspect-ratio: 16 / 9;
        width: 80vw;
    }

    iframe { width: 100%; height: 100%; }

    .lb-info {
        display: flex;
        justify-content: space-between;
        align-items: flex-start;
        color: white;
        background: #111;
        padding: 1.5rem;
        border-top: 2px solid var(--accent, #F49677);
    }

    .info-text h3 { margin: 0; font-size: 1.2rem; }
    .counter { opacity: 0.5; font-size: 0.9rem; }
    .caption { margin: 0.5rem 0 0; color: #ccc; font-style: italic; }

    .nav {
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
        background: none;
        border: none;
        color: white;
        font-size: 4rem;
        cursor: pointer;
        padding: 1rem;
        opacity: 0.5;
        transition: opacity 0.2s;
    }
    .nav:hover { opacity: 1; }
    .nav.prev { left: -4rem; }
    .nav.next { right: -4rem; }

    .close-btn {
        background: #333;
        border: none;
        color: white;
        padding: 0.5rem 1rem;
        cursor: pointer;
    }

    @media (max-width: 800px) {
        .nav { font-size: 2rem; }
        .nav.prev { left: 0; }
        .nav.next { right: 0; }
    }
</style>