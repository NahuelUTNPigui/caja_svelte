<script>
  import Home from './Home.svelte'
  import Navegacion from './Navegacion.svelte';
  import Listclients from './clientes/listclients.svelte';
  import Detailclient from './clientes/detailclient.svelte';
  import Listsubs from './subcategorias/listsubs.svelte';
  import Detailsubs from './subcategorias/detailsubs.svelte';
  import Listrubros from './rubros/listrubros.svelte';
  import Detailrubro from './rubros/detailrubro.svelte';
  import Listtiposprov from './tipoproveedores/listtiposprov.svelte';
  import Detailtiposprov from './tipoproveedores/detailtiposprov.svelte';
  import Listunidades from './unidades/listunidades.svelte';
  import Detailunidades from './unidades/detailunidades.svelte';
  import Listmodos from './modospago/listmodos.svelte';
  import Detailmodo from './modospago/detailmodo.svelte';
  import Listprovs from './proveedores/listprovs.svelte';
  import Detailprovs from './proveedores/detailprovs.svelte';
  import Listingresos from './ingresos/listingresos.svelte';
  import Detailingreso from './ingresos/detailingreso.svelte';
  import Listegresos from './egresos/listegresos.svelte';
  import Detailegreso from './egresos/detailegreso.svelte';
  import Homereportes from './reportes/homereportes.svelte';
  import { Router,Route } from "svelte-routing";
  export let url=""
</script>

<main>
  <Navegacion></Navegacion>
  <br>
  <br>
  <Router url="{url}">
    <Route path="/">
      <Home></Home>
    </Route>
    <Route path="/clientes">
      <Listclients/>
    </Route>
    <Route path="/detail_client/:id" let:params>
      <Detailclient codClient={params.id}/>
    </Route>
    <Route path="/subcategorias">
      <Listsubs/>
    </Route>
    <Route path="/detail_sub/:id" let:params>
      <Detailsubs codSub={params.id}/>
    </Route>
    <Route path="/rubros">
      <Listrubros/>
    </Route>
    <Route path="/detail_rubro/:id" let:params>
      <Detailrubro codRubro={params.id}/>
    </Route>
    <Route path="/tipoproveedores">
      <Listtiposprov/>
    </Route>
    <Route path="/detail_tipo_prov/:id" let:params>
      <Detailtiposprov codTipo={params.id}/>
    </Route>
    <Route path="/unidades">
      <Listunidades/>
    </Route>
    <Route path="/detail_unidad/:id" let:params>
      <Detailunidades codUnidad={params.id}/>
    </Route>
    <Route path="/modos">
      <Listmodos/>
    </Route>
    <Route path="/detail_modo/:id" let:params>
      <Detailmodo codModo={params.id}/>
    </Route>
    <Route path="/proveedores">
      <Listprovs/>
    </Route>
    <Route path="/detail_prov/:id" let:params>
      <Detailprovs codProv={params.id}/>
    </Route>
    <Route path="/ingresos">
      <Listingresos/>
    </Route>
    <Route path="/detail_ingreso/:id" let:params>
      <Detailingreso codIngreso={params.id}/>
    </Route>
    <Route path="/egresos">
      <Listegresos/>
    </Route>
    <Route path="/detail_egreso/:id" let:params>
      <Detailegreso codEgreso={params.id}/>
    </Route>
    <Route path="/reportes">
      <Homereportes/>
    </Route>
  </Router>
</main>

<style>

</style>
