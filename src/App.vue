<template>
  <LayoutPadrao>
    <Sobreosistema />
    <EntradaAnalise />
    <TecnologiasUtilizadas />
  </LayoutPadrao>
</template>

<script>
import LayoutPadrao from "@/layouts/LayoutPadrao";
import Sobreosistema from "@/components/Sobreosistema";
import EntradaAnalise from "@/components/EntradaAnalise";
import TecnologiasUtilizadas from "@/components/TecnologiasUtilizadas";
export default {
  components: {
    LayoutPadrao,
    Sobreosistema,
    EntradaAnalise,
    TecnologiasUtilizadas,
  },
};
</script>

<style>
</style>
