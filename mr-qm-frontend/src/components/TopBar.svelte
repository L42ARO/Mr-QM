<script>
  export let title="Mr QM";
  export let useMenu = true;


  import Icon from "svelte-icons-pack/Icon.svelte";
  import TiThMenu from "svelte-icons-pack/ti/TiThMenu";
  import { onMount } from "svelte";
  var menu;
  import { isActive, url } from '@roxi/routify'

  const links = [
    ['/home', 'Home'],
  ]
  onMount(() => {
    console.log("Using menu: " + useMenu);
  });
</script>

<ion-header collapse="fade" translucent="true">
  <ion-toolbar>
    {#if useMenu}
    <ion-button slot="start" on:click={() => menu.toggle()}>
      <Icon src={TiThMenu} />
    </ion-button>
    {/if}
    <ion-title>{title}</ion-title>
  </ion-toolbar>
</ion-header>
{#if useMenu}
  <ion-menu
    side="start"
    menu-id="first"
    content-id="router-content"
    bind:this={menu}
  >
    <ion-header>
      <ion-toolbar color="primary">
        <ion-title>Menu</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <ion-list>
        {#each links as [path,name]}
          <ion-item>
            <a href={$url(path)}>
              {name}
            </a>
          </ion-item>
        {/each}
      </ion-list>
    </ion-content>
  </ion-menu>
{/if}
