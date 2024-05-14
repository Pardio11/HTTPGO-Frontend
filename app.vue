<script>


export default {
  setup() {
    useSeoMeta({
      ogSiteName: 'Go HTTP CRUD',
    })
  }
}
</script>

<template>
  <div class="max-w-[1920px] relative bg-slate-700 w-full h-full min-h-screen">

    <div class="flex justify-center items-center pt-28 w-full">
      <Crud />
    </div>
  </div>
</template>

<style>
.red {
  border: solid red 1px;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
</style>