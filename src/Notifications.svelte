<script>
  import { notification } from "./store.js";
  import { onMount, onDestroy } from "svelte";
  import { fade } from "svelte/transition";

  export let themes = {
    danger: "#E65D3B",
    success: "#000",
    warning: "#E65D3B",
    info: "#000",
    default: "#000"
  };

  export let timeout = 3000;

  let count = 0;
  let toasts = [];
  let unsubscribe;

  function createToast(msg, theme, to) {
    const background = themes[theme] || themes["default"];
    toasts = [
      {
        id: count,
        msg,
        background,
        timeout: to || timeout,
        width: "100%"
      },
      ...toasts
    ];
    count = count + 1;
  }

  unsubscribe = notification.subscribe(value => {
    if (!value) {
      return;
    }
    createToast(value.message, value.type, value.timeout);
    notification.set();
  });

  onDestroy(unsubscribe);

  function removeToast(id) {
    toasts = toasts.filter(t => t.id != id);
  }
</script>

<style>
  :global(.toasts) {
    list-style: none;
    position: fixed;
    left: 50%;
    transform: translateX(-50%);
    bottom: 0;
    margin-bottom: 20px;
    z-index: 9999;
    width: auto;
  }

  :global(.toasts) > .toast {
    position: relative;
    margin: 10px;
    width: auto;
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1),
      0 10px 10px -5px rgba(0, 0, 0, 0.04);
    position: relative;
    color: #fff;
  }

  :global(.toasts) > .toast > .content {
    color: #fff;
    padding: 12px;
    display: block;
  }

  :global(.toasts) > .toast > .progress {
    position: absolute;
    bottom: 0;
    background-color: transparent;
    height: 6px;
    width: 100%;
  }

  :global(.toasts) > .toast:before,
  :global(.toasts) > .toast:after {
    content: "";
    position: absolute;
    z-index: -1;
    top: 50%;
    bottom: 0;
    left: 10px;
    right: 10px;
    border-radius: 100px / 10px;
  }

  :global(.toasts) > .toast:after {
    right: 10px;
    left: auto;
  }
</style>

<ul class="toasts">
  {#each toasts as toast (toast.id)}
    <li class="toast" style="background: {toast.background};" out:fade>
      <div class="content">{toast.msg}</div>
      <div
        class="progress"
        style="animation-duration: {toast.timeout}ms;"
        transition:fade
        on:animationend={() => removeToast(toast.id)} />
    </li>
  {/each}
</ul>
