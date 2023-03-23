<script lang="ts">
    import {onMount} from "svelte"
    import { navigate } from "svelte-routing";
    import {Container,Row,Col,Label,Input,Form,Button} from "sveltestrap"
    const RUTA="http://localhost:8090/api/collections"
    export let codEgreso="0"
    onMount(async ()=>{
        if(codEgreso!=="0"){
            await getEgreso(codEgreso)
        }
    })
    
    let monto=0
    let last_monto=0
    let codProv="0"
    let codModo="0"
    let codUnidad="0"
    let observacion=""
    let fecha   
    let formControl={
        noMonto:false,
        noProv:false,
        noModo:false,
        noFecha:false,
        noUnidad:false
    }
    /*
    if(codEgreso!=="0"){
        getEgreso(codEgreso)
    }
    */
    let modos_promise=getModos()
    let proveedores_promise=getProveedores()
    let unidades_promise=getUnidades()
    async function getModos(){
        let res=await fetch(RUTA+"/modopago/records?perPage=100&&page1")
        let data=await res.json()
        return data.items
    }
    async function getProveedores() {
        let res=await fetch(RUTA+"/proveedores/records?sort=apodo&&perPage=200&&page1")
        let data=await res.json()
        return data.items
    }
    async function getUnidades() {
        let res=await fetch(RUTA+"/unidades/records?perPage=100&&page1")
        let data=await res.json()
        return data.items
    }
    async function getEgreso(idEgreso) {
        let res=await fetch(RUTA+"/egreso/records/"+idEgreso)
        let data=await res.json()
        monto=data.monto
        codProv=data.codProveedor
        observacion=data.observacion
        fecha=data.fechaEgreso
        codModo=data.codModo
        codUnidad=data.codUnidad 
        last_monto=data.monto   
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
        if(codModo==="0"){
            formControl.noModo=true
            res=false
        }
        if(fecha===""){
            formControl.noFecha=true
            res=false
        }
        if(codProv==="0"){
            formControl.noProv=true
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
            let e={
                monto,
                codModo,
                codUnidad,
                codProveedor:codProv,
                fechaEgreso:fecha,
                observacion
            }
            if(codEgreso==="0"){
                let saldo=await getSaldo()
                await fetch(RUTA+"/egreso/records",{
                    method:"POST",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(e)
                })
                await fetch(RUTA+"/saldo/records/"+saldo.id,{
                    method:"PATCH",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify({
                        monto:saldo.monto-e.monto
                    })
                })
            }
            else{
                let saldo=await getSaldo()
                await fetch(RUTA+"/egreso/records/"+codEgreso,{
                    method:"PATCH",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(e)
                })
                /* await fetch(RUTA+"/saldo/records/"+saldo.id,{
                    method:"PATCH",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify({
                        monto:saldo.monto+last_monto-e.monto
                    })
                }) */
            }
            navigate("/egresos")    
        }
    }
    function cancelar(e){
        e.preventDefault()
        navigate("/egresos")
    }
    
</script>
<Container>
    <Row>
        <Col>
            <h2>Egreso</h2>
            {#if codEgreso =="0"}
                <p> Nuevo egreso</p>
            {:else}
                <p>Modificar egreso</p>
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
                    </Row>
                    <Row>
                        <Col>
                            <Label for="proveedor">Proveedor</Label>
                        </Col>
                        <Col>
                            <Label for="modo">Modo pago</Label>
                        </Col>
                        <Col>
                            <Label for="Unidad">Unidad</Label>
                        </Col>
                        <Col>
                            <Label for="fecha">Fecha</Label>
                        </Col>
                    </Row>
                    <Row>
                        <Col>
                            <select bind:value="{codProv}" name="proveedor" id="proveedor" size="1" >
                                <option value="">Seleccionar..</option>
                                {#await proveedores_promise then ps}
                                    {#each ps as p}
                                        <option value="{p.id}">{p.apodo}</option>
                                    {/each}
                                    
                                {/await}
                            </select>
                            {#if formControl.noProv && codProv==="0"}
                                <p class="badInput">No tiene proveedor</p>
                            {/if}
                        </Col>
                        <Col>
                            <select bind:value="{codModo}" name="modo" id="modo" size="1" >
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
                            <select bind:value="{codUnidad}" name="unidad" id="unidad" size="1" >
                                <option value="">Seleccionar..</option>
                                {#await unidades_promise then us}
                                    {#each us as u}
                                        <option value="{u.id}">{u.nombre}</option>
                                    {/each}
                                {/await}
                            </select>
                            {#if formControl.noUnidad && codUnidad==="0"}
                                <p class="badInput">No tiene unidad</p>
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