<template>
  <div class="layout flex flex-col min-h-screen bg-green-200 justify-between">
    <header class="header border-t-8 border-teal-200 bg-teal-500">
      <nav class="container mx-4 flex space-between items-center py-4">
        <g-link to="/" class="text-white text-4xl font-bold no-underline hover:border hover:border-white">{{ $static.metaData.siteName }}</g-link>
      </nav>
    </header>
    <div class="flex-grow p-4">
      <slot />
    </div>
    <footer class="bg-teal-700 text-white p-2">
      <div class="container mx-4 flex items-center justify-between py-4">
        <div>Copyright 2019 by Frank Steinberg. All rights reserved.</div>
      </div>
    </footer>
  </div>
</template>

<static-query>
query {
  metaData {
    siteName
  }
}
</static-query>

<style src="../main.css" />
