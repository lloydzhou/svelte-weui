<svelte:options tag="weui-mask" />


<script>
  import { fade } from 'svelte/transition';
  export let transparent;
</script>

<div transition:fade class="weui-mask{transparent ? '_transparent' : ''}">
</div>

<style>
.weui-mask, .weui-mask_transparent {
  position: fixed;
  z-index: 1000;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
}
.weui-mask_transparent {
  background: transparent;
}
</style>
