<script lang="ts">
    import { navigate } from 'svelte-routing';
    import {Container,Row,Col,Form,Input,Label,Button} from 'sveltestrap'
    
    const RUTA="http://localhost:8090/api/collections"
    export let codRubro="0"
    let nombre=""
    let controlForm={
        noName:false
    }
    if(codRubro!="0"){
        getRubro(codRubro)
    }
    async function getRubro(codRubro){
        let res=await fetch(RUTA+"/rubro/records/"+codRubro)
        let data=await res.json()
        nombre=data.nombre
    }
    function checkForm(){
        let res=true
        if(nombre===""){
            controlForm.noName=true
            res=false
        }
        return res
    }
    async function exito(e){
        e.preventDefault()
        if(checkForm()){
            let r={
                nombre
            }
            if(codRubro=="0"){
                await fetch(RUTA+"/rubro/records",{
                    method:"POST",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(r)
                })
            }
            else{
                await fetch(RUTA+"/rubro/records/"+codRubro,{
                    method:"PATCH",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(r)
                })
            }
            navigate("/rubros")            
        }

    }
    function cancelar(e){
        e.preventDefault()
        navigate("/rubros")
    }
</script>
<main>
    <Container>
        <Row>
            <Col>
                <h2>Rubro</h2>
                {#if codRubro=="0"}
                    <p>Nuevo rubro</p>
                {:else}
                    <p>Modificar rubro</p>
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