<script lang="ts">
    import { navigate } from 'svelte-routing';
    import {Container,Row,Col,Form,Input,Label,Button} from 'sveltestrap'
    export let codUnidad="0"
    let nombre=""
    const RUTA="http://localhost:8090/api/collections"
    let controlForm={
        noName:false
    }
    if(codUnidad!=="0"){
        getUnidad(codUnidad)
    }
    async function getUnidad(idUnidad){
        let res=await fetch(RUTA+"/unidades/records/"+idUnidad)
        let data=await res.json()
        nombre=data.nombre
    }
    function checkForm(){
        let res =true
        if(nombre===""){
            controlForm.noName=true
            res=false
        }
        return res
    }
    async function exito(e){
        e.preventDefault()
        if(checkForm()){
            let u={
                nombre
            }
            if(codUnidad==="0"){
                await fetch(RUTA+"/unidades/records",{
                    method:"POST",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(u)
                })
            }
            else{
                await fetch(RUTA+"/unidades/records/"+codUnidad,{
                    method:"PATCH",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(u)
                })
            }
            navigate("/unidades")
        }
    }
    function cancelar(e){
        e.preventDefault()
        navigate("/unidades")
    }
</script>
<main>
    <Container>
        <Row>
            <Col>
                <h2>Unidad</h2>
                {#if codUnidad=="0"}
                    <p>Nueva unidad</p>
                {:else}
                    <p>Modificar unidad</p>
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
                            {#if controlForm.noName && nombre===""}
                                <p class="badInput">No tiene nombre</p>
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
</style>