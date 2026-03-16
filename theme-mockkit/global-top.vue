<script setup>
import { useNav } from '@slidev/client'
import { onMounted, onUnmounted } from 'vue'

const { go } = useNav()

function onKey(e) {
  if (e.key === 't' || e.key === 'T') {
    go('toc')
  }
}

onMounted(() => window.addEventListener('keydown', onKey))
onUnmounted(() => window.removeEventListener('keydown', onKey))
</script>

<template />
