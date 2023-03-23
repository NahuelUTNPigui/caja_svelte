<script>
    import {onMount} from "svelte"
    import { navigate } from 'svelte-routing';
    import {Container,Row,Col,Form,Input,Label,Button} from 'sveltestrap'
    export let codClient="0"
    onMount(()=>{
        if(codClient!=="0"){
            getCliente(codClient)
        }
    })
    const RUTA="http://localhost:8090/api/collections"
    let nombre=""
    let razon_social=""
    let codRubro="0"
    let rubros_promise=getRubros()
    let formControl={
        noName:false,
        noRazonSocial:false,
        noRubro:false
    }
    /*
    if(codClient!="0"){
        getCliente(codClient)
    }
    */
    async function getCliente(codClient){
        let res=await fetch(RUTA+"/cliente/records/"+codClient)
        let data=await res.json()
        nombre=data.nombre
        razon_social=data.razonsocial
        codRubro=data.codRubro
    }
    async function getRubros(){
        let res=await fetch(RUTA+"/rubro/records?perPage=100&&page1")
        let data=await res.json()
        return data.items
    }
    function checkForm(){
        let res=true
        if(nombre===""){
            formControl.noName=true
            res= false
        }
        if(razon_social===""){
            formControl.noRazonSocial=true
            res= false
        }
        if(codRubro==="0"){
            formControl.noRubro=true
            res= false
        }
        return res
        
    }
    async function exito(e){
        e.preventDefault()
        if(checkForm()){
            let c={
                nombre,
                razonsocial:razon_social,
                codRubro
            }
            if(codClient==="0"){
                await fetch(RUTA+"/cliente/records",{
                    method:"POST",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(c)
                })
            }
            else{
                await fetch(RUTA+"/cliente/records/"+codClient,{
                    method:"PATCH",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(c)
                })
            }
            navigate('/clientes')
        }
    }
    function cancelar(e){
        e.preventDefault()
        navigate('/clientes')
    }
</script>
<main>
    <Container>
        <Row>
            <Col>
                <h2>Clientes</h2>
                {#if codClient == "0"}
                    <p>Nuevo cliente</p>
                {:else}
                    <p>Modificar cliente</p>
                {/if}
                <Form>
                    <Container>
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
                            {#if formControl.noName && nombre==="" }
                                <p class="badInput">No tiene nombre</p>
                            {/if}
                        </Row>
                        <Row>
                            <Col>
                                <Label for="razon_social">Razon Social</Label>
                                <Input
                                  type="text"
                                  name="razon_social"
                                  id="razon_social"
                                  placeholder="Razon social"
                                  bind:value="{razon_social}"
                                />
                            </Col>
                            {#if formControl.noRazonSocial && razon_social===""}
                                <p class="badInput"> No tiene razon social</p>
                            {/if}
                        </Row>
                        <Row>
                            <Col>
                                <Label for="rubro">Rubro</Label>
                            </Col>
                        </Row>
                        <Row>
                            <Col>
                                <select bind:value="{codRubro}" name="rubro" id="rubro" size="1" >
                                    <option value="">Seleccionar..</option>
                                    {#await rubros_promise then rs}
                                        {#each rs as r}
                                            <option value="{r.id}" selected>{r.nombre}</option>                                            
                                        {/each}
                                    {/await}
                                    

                                    
                                    
                                </select>
                                {#if formControl.noRubro && codRubro==="0"}
                                    <p class="badInput">No tiene rubro</p>
                                {/if}
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

</main>
<style>
    .badInput{
        color: #fe0000;
    }
    select{
        width: 200px;
    }
</style>