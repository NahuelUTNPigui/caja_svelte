<script lang="ts">
        import { navigate } from 'svelte-routing';
    import {Container,Row,Col,Form,Input,Label,Button} from 'sveltestrap'
    
    const RUTA="http://localhost:8090/api/collections"
    export let codSub="0"
    let nombre=""
    let controlForm={
        noName:false
    }
    if(codSub!="0"){
        getSub(codSub)
    }
    async function getSub(codRubro){
        let res=await fetch(RUTA+"/subcategorias/records/"+codRubro)
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
            let sb={
                nombre
            }
            if(codSub=="0"){
                await fetch(RUTA+"/subcategorias/records",{
                    method:"POST",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(sb)
                })
            }
            else{
                await fetch(RUTA+"/subcategorias/records/"+codSub,{
                    method:"PATCH",
                    headers:{
                        "Content-Type":"application/json"
                    },
                    body:JSON.stringify(sb)
                })
            }
            navigate("/subcategorias")            
        }

    }
    function cancelar(e){
        e.preventDefault()
        navigate("/subcategorias")
    }
</script>
<main>
    <Container>
        <Row>
            <Col>
                <p>{codSub}</p>
                <h2>Sub categorias</h2>
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