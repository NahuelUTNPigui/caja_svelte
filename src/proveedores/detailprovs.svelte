<script>
    import {onMount} from "svelte"
    import { navigate } from 'svelte-routing';
    import { add_attribute } from "svelte/internal";
    import {Container,Row,Col,Form,Input,Label,Button} from 'sveltestrap'
    export let codProv="0"
    onMount(async ()=>{
        if(codProv!=="0"){
            await getProveedor(codProv)
        }
    })
    let apodo=""
    let nombre=""
    let apellido=""
    let codTipo="0"
    const RUTA="http://localhost:8090/api/collections"

    let controlForm={
        noApodo:false,
        noName:false,
        noApellido:false,
        noTipo:false
    }
    async function getProveedor(idProv){
        let res=await fetch(RUTA+"/proveedores/records/"+idProv)
        let data=await res.json()
        apodo=data.apodo
        nombre=data.nombre
        apellido=data.apellido
        codTipo=data.codTipo
    }
    function checkForm(){
        let res=true
        if(apodo===""){
            controlForm.noApodo=true
            res=false
        }
        if(nombre===""){
            controlForm.noName=true
            res=false
        }
        if(apellido===""){
            controlForm.noApellido=true
            res=false
        }
        if(codTipo==="0"){
            controlForm.noTipo=true
            res=false
        }
        return res
    }
    async function exito(e){
        e.preventDefault()
        if(checkForm()){
            let p={
                nombre,
                apellido,
                apodo,
                codTipo
            }
            if(codProv==="0"){
                await fetch(RUTA+"/proveedores/records",{
                    method:"POST",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(p)
                })
            }
            else{
                await fetch(RUTA+"/proveedores/records/"+codProv,{
                    method:"PATCH",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(p)
                })
            }
            navigate('/proveedores')
        }
    }
    function cancelar(e){
        e.preventDefault()
        navigate('/proveedores')
    }
    let tipos_promise=getTipos()
    async function getTipos(){
        let tipoproveedores=[]
        let res_p=await fetch(RUTA+"/tipoproveedor/records?page=1&perPage=1")
        let data_p = await res_p.json()
        let paginas= Math.floor(data_p.totalItems/200)+1
        for(let pag=1;pag<=paginas;pag++){
            let res=await fetch(RUTA+"/tipoproveedor/records?perPage=200&page="+pag)
            let data=await res.json()
            tipoproveedores=tipoproveedores.concat(data.items)
        }
        return tipoproveedores
    }
</script>
<main>
    <Container>
        <Row>
            <Col>
                <h2>Proveedores</h2>
                {#if codProv=="0"}
                    <p>Nuevo proveedor</p>
                {:else}
                    <p>Modificar proveedor</p>
                {/if}
                <Form>
                    <Container>
                        <Row>
                            <Col>
                                <Label for="apodo">Alias</Label>
                                <Input
                                  type="text"
                                  name="apodo"
                                  id="apodo"
                                  placeholder="apodo"
                                  bind:value="{apodo}"
                                />
                            </Col>
                            {#if controlForm.noApodo && apodo===""}
                                <p class="badInput">No tiene apodo</p>
                                
                            {/if}
                        </Row>
                        <Row>
                            <Col>
                                <Label for="nombre">Nombre</Label>
                                <Input
                                  type="text"
                                  name="nombre"
                                  id="nombre"
                                  placeholder="nombre"
                                  bind:value="{nombre}"
                                />
                            </Col>
                            {#if controlForm.noName && nombre===""}
                                <p class="badInput">No tiene nombre</p>
                                
                            {/if}
                        </Row>
                        <Row>
                            <Col>
                                <Label for="apellido">Apellido</Label>
                                <Input
                                  type="text"
                                  name="apellido"
                                  id="apellido"
                                  placeholder="apellido"
                                  bind:value="{apellido}"
                                />
                            </Col>
                            {#if controlForm.noApellido && apellido===""}
                                <p class="badInput"> No tiene apellido</p>
                            {/if}
                        </Row>
                        <Row>
                            <Col>
                                <Label for="tipo">Tipo proveedor</Label>
                            </Col>
                        </Row>
                        <Row>
                            <Col>
                                <select bind:value="{codTipo}" name="tipo" id="tipo" size="1" >
                                    <option value="">Seleccionar..</option>
                                    {#await tipos_promise then tps}
                                        {#each tps as tp}
                                            <option value="{tp.id}">{tp.nombre}</option>
                                        {/each}                                        
                                    {/await}
                                </select>
                            </Col>
                            {#if controlForm.noTipo && codTipo==="0"}
                                <p class="badInput">No tiene tipo</p>
                            {/if}
                        </Row>
                        <Row>
                            
                            <Col>
                                <br>
                                <Button outline color="success" on:click={exito}>Guardar</Button>
                            </Col>
                            <Col>
                                <br>
                                <Button outline color="danger" on:click={cancelar}>Cancelar</Button>
                            </Col>
                        </Row>
                    </Container>
                </Form>
            </Col>
        </Row>
    </Container>

</main>
<style>
    .badInput{
        color: #fe0000;
    }
    select{
        width: 200px;
    }
</style>
