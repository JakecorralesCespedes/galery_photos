<script lang="ts">
  import axios from 'axios';
  import { onMount } from 'svelte';
  import { Gallery, Button } from 'flowbite-svelte';
  import { writable } from 'svelte/store';
  import { Pagination } from 'flowbite-svelte';
  import { ChevronLeftOutline, ChevronRightOutline } from 'flowbite-svelte-icons';

  let images = writable([]);
  let filteredImages = writable([]);
  let tags = ['nature', 'animals', 'technology', 'people', 'architecture', 'flowers'];
  let selectedTag = writable('nature');
  let currentPage = writable(1);
  let totalPages = writable(10); // Asume que hay 10 páginas por defecto

  const API_URL = 'https://api.pexels.com/v1/search';
  const API_KEY = 'gTkqB6DSGKmKNNblR0IVfP5qlU15I80yj2m2Bm9xxzLKHjKrd3MnYrGq'; // Reemplaza con tu clave de API de Pexels

  const fetchImages = async (tag: string, page: number = 1) => {
    const params = {
      query: tag,
      per_page: 10,
      page
    };

    try {
      const response = await axios.get(API_URL, {
        headers: {
          Authorization: API_KEY
        },
        params
      });
      const fetchedImages = response.data.photos.map((photo: any) => ({
        alt: photo.alt,
        src: photo.src.medium
      }));
      images.set(fetchedImages);
      filteredImages.set(fetchedImages);
      totalPages.set(Math.ceil(response.data.total_results / 10)); // Actualiza el total de páginas basado en los resultados
    } catch (error) {
      console.error('Error fetching images:', error);
    }
  };

  onMount(() => {
    fetchImages($selectedTag, $currentPage);
  });

  const filterImages = (tag: string) => {
    selectedTag.set(tag);
    currentPage.set(1); // Reinicia a la primera página cuando se cambia la etiqueta
    fetchImages(tag, 1);
  };

  const previous = () => {
    if ($currentPage > 1) {
      currentPage.update(n => n - 1);
      fetchImages($selectedTag, $currentPage - 1);
    }
  };

  const next = () => {
    if ($currentPage < $totalPages) {
      currentPage.update(n => n + 1);
      fetchImages($selectedTag, $currentPage + 1);
    }
  };

  const handleClick = (event) => {
    const page = parseInt(event.detail.page);
    currentPage.set(page);
    fetchImages($selectedTag, page);
  };

  $: pages = Array.from({ length: $totalPages }, (_, i) => ({
    name: (i + 1).toString(),
    href: `?page=${i + 1}`,
    active: i + 1 === $currentPage
  }));
</script>

<div class="flex items-center justify-center py-4 md:py-8 flex-wrap gap-3 mb-3 mx-auto">
  {#each tags as tag}
    <Button 
      pill 
      size="xl" 
      outline={tag !== $selectedTag} 
      color={tag === $selectedTag ? 'primary' : 'alternative'} 
      on:click={() => filterImages(tag)}
    >
      {tag.charAt(0).toUpperCase() + tag.slice(1)}
    </Button>
  {/each}
</div>

<Gallery items={$filteredImages} class="gap-4 grid-cols-2 md:grid-cols-3" />

<Pagination {pages} on:previous={previous} on:next={next} on:click={handleClick} icon>
  <svelte:fragment slot="prev">
    <span class="sr-only">Previous</span>
    <ChevronLeftOutline class="w-2.5 h-2.5" />
  </svelte:fragment>
  <svelte:fragment slot="next">
    <span class="sr-only">Next</span>
    <ChevronRightOutline class="w-2.5 h-2.5" />
  </svelte:fragment>
</Pagination>


