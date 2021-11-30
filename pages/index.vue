<template>
  <section>
    <h1>Selamat Datang!</h1>
    <h2>Silahkan pilih menu dibawah ini!</h2>
    <p></p>
    <ul>
      <li>
        <NuxtLink to="/visualisasi">
          Visualisasi Hasil Parser
        </NuxtLink>
      </li>
      <li>
        <NuxtLink to="/evaluasi">
          Evaluasi Hasil Parser
        </NuxtLink>
      </li>
      <li>
        <NuxtLink to="/comparison">
          Perbandingan Dua Hasil Parser
        </NuxtLink>
      </li>
      <li>
        <NuxtLink to="/parsing">
          Parsing Kalimat Berbahasa Indonesia
        </NuxtLink>
      </li>
    </ul>
  </section>
</template>

<script>
export default {
  layout: 'no-layout'
}
</script>
