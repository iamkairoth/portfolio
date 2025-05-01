<script>
  import { onMount } from 'svelte';
  import About from '$lib/sections/about.svelte';
  import Projects from '$lib/sections/projects.svelte';
  import Skills from '$lib/sections/skills.svelte';
  import Contact from '$lib/sections/contact.svelte';

  let ticking = false;
  let sectionPositions = [];
  let scrollMap = [];
  let showTopButton = false;
  let showBottomButton = false;

  onMount(() => {
    const profile = document.getElementById('profile');
    const zoomSection = document.getElementById('zoom-section');
    const horizontal = document.getElementById('horizontal-scroll');
    const horizontalWrapper = document.getElementById('horizontal-wrapper');
    const sections = Array.from(horizontal.children);
    const indicators = document.getElementById('scroll-indicators');

    const adjustScrollHeight = () => {
      const sectionHeight = window.innerHeight * 1.2;
      const spacerHeight = document.getElementById('scroll-spacer').offsetHeight;
      scrollMap = sections.map((_, i) => spacerHeight + i * sectionHeight);
      horizontalWrapper.style.height = `${sectionHeight * sections.length + window.innerHeight}px`;
      sectionPositions = sections.map((_, i) => i * window.innerWidth);
    };

    adjustScrollHeight();
    window.addEventListener('resize', adjustScrollHeight);

    let lastSnappedIndex = 0;

    const updateIndicators = (activeIndex) => {
      Array.from(indicators.children).forEach((dot, idx) => {
        dot.classList.toggle('bg-white', idx === activeIndex);
        dot.classList.toggle('bg-gray-500', idx !== activeIndex);
      });
      showTopButton = activeIndex === 0;
      showBottomButton = activeIndex === sections.length - 1;
    };

    const handleSnapScroll = () => {
      const scrollY = window.scrollY;
      for (let i = 0; i < scrollMap.length; i++) {
        const targetScroll = scrollMap[i];
        if (Math.abs(scrollY - targetScroll) < window.innerHeight * 0.4) {
          lastSnappedIndex = i;
          updateIndicators(i);
          horizontal.scrollTo({ left: sectionPositions[i], behavior: 'smooth' });
          break;
        }
      }
    };

    const handleScroll = () => {
      const scrollY = window.scrollY;
      const windowHeight = window.innerHeight;
      const scale = 1 + (scrollY / windowHeight) * 4.5;
      const opacity = 1 - scrollY / (windowHeight * 0.65);
      profile.style.transform = `scale(${scale})`;
      profile.style.opacity = Math.max(opacity, 0.25);
      zoomSection.style.opacity = scrollY >= windowHeight ? '0' : '1';
      zoomSection.style.pointerEvents = scrollY >= windowHeight ? 'none' : 'auto';

      handleSnapScroll();
    };

    const handleWheelSnap = (e) => {
      const scrollY = window.scrollY;
      const sectionHeight = window.innerHeight * 1.2;
      const spacerHeight = document.getElementById('scroll-spacer').offsetHeight;

      if (scrollY >= spacerHeight && scrollY < scrollMap[scrollMap.length - 1] + sectionHeight) {
        e.preventDefault();
        const delta = e.deltaY;

        if (delta > 0 && lastSnappedIndex < sections.length - 1) {
          lastSnappedIndex++;
        } else if (delta < 0 && lastSnappedIndex > 0) {
          lastSnappedIndex--;
        }
        updateIndicators(lastSnappedIndex);
        const targetOffset = sectionPositions[lastSnappedIndex];
        horizontal.scrollTo({ left: targetOffset, behavior: 'smooth' });
        window.scrollTo({ top: scrollMap[lastSnappedIndex], behavior: 'smooth' });
      }
    };

    Array.from(indicators.children).forEach((dot, idx) => {
      dot.addEventListener('click', () => {
        lastSnappedIndex = idx;
        updateIndicators(idx);
        horizontal.scrollTo({ left: sectionPositions[idx], behavior: 'smooth' });
        window.scrollTo({ top: scrollMap[idx], behavior: 'smooth' });
      });
    });

    window.addEventListener('scroll', () => {
      if (!ticking) {
        window.requestAnimationFrame(() => {
          handleScroll();
          ticking = false;
        });
        ticking = true;
      }
    });

    window.addEventListener('wheel', handleWheelSnap, { passive: false });
  });

  function scrollToTop() {
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }

  function scrollToBottom() {
    window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' });
  }
</script>

<!-- Scroll Indicators -->
<div id="scroll-indicators" class="fixed bottom-6 left-1/2 -translate-x-1/2 flex gap-3 z-40 cursor-pointer">
  <div class="w-3 h-3 rounded-full bg-white"></div>
  <div class="w-3 h-3 rounded-full bg-gray-500"></div>
  <div class="w-3 h-3 rounded-full bg-gray-500"></div>
  <div class="w-3 h-3 rounded-full bg-gray-500"></div>
</div>

<!-- Floating Buttons -->
{#if showTopButton}
  <button on:click={scrollToTop} class="fixed top-4 right-4 z-50 px-4 py-2 bg-gray-700 text-white rounded shadow">
    ↑ Home
  </button>
{/if}
{#if showBottomButton}
  <button on:click={scrollToBottom} class="fixed bottom-20 right-4 z-50 px-4 py-2 bg-gray-700 text-white rounded shadow">
    ↓ Exit
  </button>
{/if}

<style>
  ::-webkit-scrollbar {
    display: none;
  }
  html {
    scrollbar-width: none;
    -ms-overflow-style: none;
  }
</style>

<main class="m-0 p-0 font-sans bg-black text-white">
  <section id="zoom-section" class="fixed top-0 left-0 right-0 h-screen z-50 bg-black flex flex-col items-center justify-center gap-6 transition-opacity duration-700 ease-out">
    <div id="profile" class="w-52 h-52 rounded-full overflow-hidden border-4 border-white transition-transform duration-700 ease-out opacity-100 shadow-2xl">
      <img src="./profile.png" alt="Profile" class="w-full h-full object-cover" />
    </div>
    <div class="text-center max-w-md">
      <h1 class="text-3xl font-bold">Hi, I'm </h1>
      <p class="mt-2 text-lg">Data Scientist • Developer • Creative Thinker</p>
      <p class="mt-2 text-sm text-gray-300">Scroll to explore more about my work, skills, and ways to connect.</p>
    </div>
  </section>

  <section id="scroll-spacer" class="h-[180vh] bg-[#222] flex items-end justify-center p-8">
    <h2 class="text-2xl">Scroll down to begin horizontal journey</h2>
  </section>

  <section id="horizontal-wrapper">
    <div class="sticky top-0 h-screen overflow-hidden">
      <div id="horizontal-scroll" class="flex h-full overflow-x-auto scroll-smooth">
        <div class="w-screen flex-shrink-0">
          <About />
        </div>
        <div class="w-screen flex-shrink-0">
          <Projects />
        </div>
        <div class="w-screen flex-shrink-0">
          <Skills />
        </div>
        <div class="w-screen flex-shrink-0">
          <Contact />
        </div>
      </div>
    </div>
  </section>

  <section class="h-screen bg-gray-900 flex items-center justify-center">
    <div class="max-w-2xl text-center">
      <h2 class="text-4xl font-bold">Thanks for scrolling!</h2>
      <p class="mt-4 text-lg text-gray-300">This section supports vertical scroll again. Add more content here!</p>
    </div>
  </section>
</main>

<svelte:head>
  <script src="https://cdn.tailwindcss.com"></script>
</svelte:head>
