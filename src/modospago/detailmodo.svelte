<script lang="ts">
    
    import {Container,Row,Col,Form,Input,Label,Button} from 'sveltestrap'
    import { navigate } from 'svelte-routing';
    export let codModo="0"
    const RUTA ="http://localhost:8090/api/collections"
    let controlForm={
        notName:false
    }
    let nombre=""
    if(codModo!=="0"){
        getModo(codModo)
    }
    async function getModo(idModo) {
        let res=await fetch(RUTA+"/modopago/records/"+idModo)
        let data=await res.json()
        nombre=data.nombre
    }

    function checkForm(){
        if(nombre===""){
            controlForm.notName=true
            return false
        }
        return true
    }
    async function exito(e){
        e.preventDefault()
        if(checkForm()){
            let m={
                nombre
            }
            if(codModo==="0"){
                await fetch(RUTA+"/modopago/records",{
                    method:"POST",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(m)
                })
            }
            else{
                await fetch(RUTA+"/modopago/records/"+codModo,{
                    method:"PATCH",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(m)
                })
            }
            navigate('/modos')
        }
    }
    function cancelar(e){
        e.preventDefault()
        navigate('/modos')
    }
</script>
<main>
    <Container>
        <Row>
            <Col>
                <h2>Modo</h2>
                {#if codModo=="0"}
                    <p>Nuevo modo de pago</p>
                {:else}
                    <p>Modificar modo de pago</p>
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
                            {#if controlForm.notName && nombre===""}
                                <p class="badName">No tiene nombre</p>
                            {/if}
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
    .badName{
        color: #fe0000;
    }
</style>