<script>
  import { onMount } from 'svelte';
  import About from '$lib/sections/about.svelte';
  import Projects from '$lib/sections/projects.svelte';
  import Skills from '$lib/sections/skills.svelte';
  import Contact from '$lib/sections/contact.svelte';

  let ticking = false;
  let sectionPositions = [];
  let scrollMap = [];

  onMount(() => {
    const profile = document.getElementById('profile');
    const zoomSection = document.getElementById('zoom-section');
    const horizontal = document.getElementById('horizontal-scroll');
    const horizontalWrapper = document.getElementById('horizontal-wrapper');
    const sections = Array.from(horizontal.children);

    const adjustScrollHeight = () => {
      const sectionHeight = window.innerHeight * 1.2;
      const spacerHeight = document.getElementById('scroll-spacer').offsetHeight;
      scrollMap = sections.map((_, i) => spacerHeight + i * sectionHeight);
      horizontalWrapper.style.height = `${sectionHeight * sections.length + window.innerHeight}px`;
      sectionPositions = sections.map((_, i) => i * window.innerWidth);
    };

    adjustScrollHeight();
    window.addEventListener('resize', adjustScrollHeight);


    const handleScroll = () => {
      const scrollY = window.scrollY;
      const windowHeight = window.innerHeight;
      const scale = 1 + (scrollY / windowHeight) * 4.5;
      const opacity = 1 - scrollY / (windowHeight * 0.65);
      profile.style.transform = `scale(${scale})`;
      profile.style.opacity = Math.max(opacity, 0.25);
      zoomSection.style.opacity = scrollY >= windowHeight ? '0' : '1';
      zoomSection.style.pointerEvents = scrollY >= windowHeight ? 'none' : 'auto';
    };


    window.addEventListener('scroll', () => {
      if (!ticking) {
        window.requestAnimationFrame(() => {
          handleScroll();
          ticking = false;
        });
        ticking = true;
      }
    });
  });


</script>

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
      <h1 class="text-3xl font-bold">Hi, I'm Kai Roth</h1>
      <p class="mt-2 text-lg">Data Scientist • Developer • Creative Thinker</p>
      <p class="mt-2 text-sm text-gray-300">Scroll to explore more about my work, skills, and ways to connect.</p>
    </div>
  </section>

  <section id="scroll-spacer" class="h-[180vh] bg-[#100] flex items-end justify-center p-8">

    <h2 class="text-2xl font-bold text-center text-gray-300">
      "The only thing we have to fear is fear itself"    
    </h2>
  </section>

  <section id="horizontal-scroll"></section>
  <section id="horizontal-wrapper" class="flex flex-nowrap overflow-x-auto snap-x snap-mandatory scrollbar-hide">
      <About />
      <Projects />
      <Skills />
  </section>

<!-- 
  <section class="h-screen flex items-center justify-center">
    <Contact />
  </section> -->
</main>

<svelte:head>
  <script src="https://cdn.tailwindcss.com"></script>
</svelte:head>
