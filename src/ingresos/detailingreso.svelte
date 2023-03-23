<script lang="ts">
    import {onMount} from "svelte"
    import { navigate } from "svelte-routing";
    import {Container,Row,Col,Label,Input,Form,Button} from "sveltestrap"
    const RUTA="http://localhost:8090/api/collections"
    export let codIngreso="0"
    onMount(async ()=>{
        if(codIngreso!=="0"){
            getIngreso(codIngreso)
        }
    })
    let monto=0
    let last_monto=0
    let codClient="0"
    let nroFactura=""
    let codModo="0"
    let codUnidad="0"
    let observacion=""
    let fecha=""

    let formControl={
        noMonto:false,
        noCliente:false,
        noFactura:false,
        noModo:false,
        noFecha:false,
        noUnidad:false
    }
    
    /*
    if(codIngreso!=="0"){
        getIngreso(codIngreso)
    }
    */
    let clientes_promise=getClientes()
    let modos_promise=getModos()
    let unidades_promise=getUnidades()
    async function getClientes() {
        let res=await fetch(RUTA+"/cliente/records?sort=nombre&&perPage=200&&page1")
        let data=await res.json()
        return data.items
    }
    async function getModos(){
        let res=await fetch(RUTA+"/modopago/records?perPage=100&&page1")
        let data=await res.json()
        return data.items
    }
    async function getIngreso(idIngreso) {
        let res=await fetch(RUTA+"/ingreso/records/"+idIngreso)
        let data=await res.json()
        monto=data.monto
        codClient=data.codCliente
        nroFactura=data.nroFactura
        observacion=data.observacion
        fecha=data.fechaIngreso
        codModo=data.codModo
        codUnidad=data.codUnidad
        last_monto=data.monto
    }
    async function getUnidades(){
        let res=await fetch(RUTA+"/unidades/records?perPage=100&&page1")
        let data=await res.json()
        return data.items
    }
    async function getSaldo(){
        let res=await fetch(RUTA+"/saldo/records")
        let data=await res.json()
        return data.items[0]
    }

    function checkForm(){
        let res=true
        if(!monto || monto===0){
            formControl.noMonto=true
            res=false
        }
        if(nroFactura===""){
            formControl.noFactura=true
            res=false
        }
        if(codModo==="0"){
            formControl.noModo=true
            res=false
        }
        if(fecha===""){
            formControl.noFecha=true
            res=false
        }
        if(codClient==="0"){
            formControl.noCliente=true
            res=false
        }
        if(codUnidad==="0"){
            formControl.noUnidad=true
            res=false
        }

        return res
    }
    async function exito(e){
        e.preventDefault()
        if(checkForm()){
            let i={
                monto,
                nroFactura,
                observacion,
                fechaIngreso:fecha,
                codModo,
                codCliente:codClient,
                codUnidad
            }
            if(codIngreso==="0"){
                let saldo=await getSaldo()
                await fetch(RUTA+"/ingreso/records",{
                    method:"POST",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(i)
                })
                /* await fetch(RUTA+"/saldo/records/"+saldo.id,{
                    method:"PATCH",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(
                        {
                            monto:saldo.monto+i.monto
                        }
                    )
                }) */
            }
            else{
                let saldo=await getSaldo()
                await fetch(RUTA+"/ingreso/records/"+codIngreso,{
                    method:"PATCH",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(i)
                })
                await fetch(RUTA+"/saldo/records/"+saldo.id,{
                    method:"PATCH",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(
                        {
                            monto:saldo.monto+i.monto-last_monto
                        }
                    )
                })
            }
            navigate("/ingresos")    
        }
    }
    function cancelar(e){
        e.preventDefault()
        navigate("/ingresos")
    }
    
</script>
<Container>
    <Row>
        <Col>
            <h2>Ingreso</h2>
            {#if codIngreso =="0"}
                <p> Nuevo ingreso</p>
            {:else}
                <p>Modificar ingreso</p>
            {/if}
            <Form>
                <Container>
                    <Row>
                        <Col>
                            <Label for="nombre">Monto</Label>
                            <Input
                              type="number"
                              name="monto"
                              id="monto"
                              placeholder="monto"
                              bind:value="{monto}"
                            />
                            {#if formControl.noMonto && !monto}
                                <p class="badInput">Debe tener monto</p>
                            {/if}
                            {#if formControl.noMonto && monto<=0 }
                                <p class="badInput">El monto debe ser positivo</p>
                            {/if}

                        </Col>
                        <Col>
                            <Label for="nroFactura">Factura</Label>
                            <Input
                              type="text"
                              name="nroFactura"
                              id="nroFactura"
                              placeholder="Numero de Factura"
                              bind:value="{nroFactura}"
                            />
                            {#if formControl.noFactura && nroFactura==="" }
                                <p class="badInput">Debe tener un numero de factura</p>
                            {/if}

                        </Col>
                    </Row>
                    <Row>
                        <Col>
                            <Label for="cliente">Cliente</Label>
                        </Col>
                        <Col>
                            <Label for="modo">Modo pago</Label>
                        </Col>
                        <Col>
                            <Label for="unidad">Unidad</Label>
                        </Col>
                        <Col>
                            <Label for="fecha">Fecha</Label>
                        </Col>
                    </Row>
                    <Row>
                        <Col>
                            <select bind:value="{codClient}" name="cliente" id="cliente" size="1" >
                                <option value="">Seleccionar..</option>
                                {#await clientes_promise then cs}
                                    {#each cs as c}
                                        <option value="{c.id}">{c.nombre}</option>
                                    {/each}
                                {/await}
                                
                            </select>
                            {#if formControl.noCliente && codClient==="0"}
                                <p class="badInput">No tiene cliente</p>
                            {/if}
                        </Col>
                        <Col>
                            <select bind:value="{codModo}" name="modo" id="modo" size="1">
                                <option value="">Seleccionar..</option>
                                {#await modos_promise then ms}
                                    {#each ms as m}
                                        <option value="{m.id}">{m.nombre}</option>
                                    {/each}
                                    
                                {/await}
                            </select>
                            {#if formControl.noModo && codModo==="0"}
                                <p class="badInput">No tiene modo</p>
                            {/if}
                        </Col>
                        <Col>
                            <select bind:value="{codUnidad}" name="unidad" id="unidad" size="1">
                                <option value="">Seleccionar..</option>
                                {#await unidades_promise then us}
                                    {#each us as u}
                                        <option value="{u.id}">{u.nombre}</option>
                                    {/each}
                                    
                                {/await}
                            </select>
                            {#if formControl.noUnidad && codUnidad==="0"}
                                <p class="badInput">No tiene Unidad</p>
                            {/if}
                        </Col>
                        <Col>
                            <Input
                                type="date"
                                name="fecha"
                                id="fecha"
                                bind:value="{fecha}"
                            />
                            {#if formControl.noFecha && fecha===""}
                                <p class="badInput">Debe tener fecha</p>
                            {/if}
                        </Col>
                    </Row>
                    <Row>
                        <Col>
                            <Label for="observacion">Observaciones</Label>
                            <Input type="textarea" name="observacion" id="observacion" bind:value="{observacion}" />
                        </Col>
                        
                    </Row>
                    <Row>
                        
                        <Col>
                            <br>
                            <Button outline color="success" on:click={exito}>Guardar</Button>
                        </Col>
                        <Col>
                            <br>
                            <Button outline color="danger" on:click={cancelar}>cancelar</Button>
                        </Col>
                    </Row>
                </Container>
            </Form>
        </Col>
    </Row>
</Container>
<style>
    .badInput{
        color: #fe0000;
    }
    select{
        width: 200px;
    }
</style>