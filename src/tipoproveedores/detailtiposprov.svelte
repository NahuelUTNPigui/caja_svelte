<script lang="ts">
    import { navigate } from 'svelte-routing';
    import {Container,Row,Col,Form,Input,Label,Button} from 'sveltestrap'
    export let codTipo="0"
    let nombre=""
    const RUTA="http://localhost:8090/api/collections"
    let controlForm={
        noName:false
    }
    if(codTipo!=="0"){
        getTipo(codTipo)
    }
    async function getTipo(idTipo){
        let res=await fetch(RUTA+"/tipoproveedor/records/"+idTipo)
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
            let t={
                nombre
            }
            if(codTipo==="0"){
                await fetch(RUTA+"/tipoproveedor/records",{
                    method:"POST",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(t)
                })
            }
            else{
                await fetch(RUTA+"/tipoproveedor/records/"+codTipo,{
                    method:"PATCH",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(t)
                })
            }
            navigate("/tipoproveedores")
        }
    }
    function cancelar(e){
        e.preventDefault()
        navigate("/tipoproveedores")
    }
</script>
<main>
    <Container>
        <Row>
            <Col>
                <h2>Tipo Proveedores</h2>
                {#if codTipo=="0"}
                    <p>Nuevo tipo proveedor</p>
                {:else}
                    <p>Modificar tipo proveedor</p>
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