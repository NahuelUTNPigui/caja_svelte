<script lang="ts">
    import {Container,Row,Col,Button,Table,Modal,ModalBody,ModalFooter,ModalHeader} from 'sveltestrap'
    import {Router,Link,navigate} from 'svelte-routing'
    const RUTA="http://localhost:8090/api/collections"
    let unidades_promise=getUnidades()
    export let url=""
    let idEliminarUnidad=""
    let open=false
    let toggle=()=>(open = !open)
    function nueva_unidad(){
        navigate('/detail_unidad/0')
    }
    async function getUnidades(){
        let res=await fetch(RUTA+"/unidades/records")
        let data = await res.json()
        return data.items
    }
    async function eliminarUnidadBD() {
        await fetch(RUTA+"/unidades/records/"+idEliminarUnidad,{
            method:"DELETE"
        })
        unidades_promise=getUnidades()
        idEliminarUnidad=""
        toggle()
    }
    async function eliminarUnidad(idUnidad){
        idEliminarUnidad=idUnidad
        toggle()
    }
    function cancelarModal(){
        idEliminarUnidad=""
        toggle()
    }
</script>
<div>
    <Modal isOpen={open} {toggle}>
      <ModalHeader {toggle}>Eliminar unidad</ModalHeader>
      <ModalBody>
        ¿Esta seguro que desea eliminar la unidad?
        <br>
        Podria modificar algun ingreso o egreso
      </ModalBody>
      <ModalFooter>
        <Button color="danger" on:click={eliminarUnidadBD}>Eliminar</Button>
        <Button color="secondary" on:click={cancelarModal}>Cancelar</Button>
      </ModalFooter>
    </Modal>
  </div>
<Container>
    <Row>
        <Col>
            <Button on:click={nueva_unidad}>Nueva unidad</Button>

            <hr>
            <Table>
                <thead>
                    <tr>
                        <th>Nombre</th>
                        <th>Acciones</th>
                        <th></th>
                    </tr>
                </thead>
                {#await unidades_promise then us}
                <tbody>
                    {#each us as u}
                        <tr>
                            <td>{u.nombre}</td>
                            <td>
                                <Router url="{url}">
                                    <nav>
                                        <Link to="/detail_unidad/{u.id}">Modificar</Link>
                                    </nav>
                                </Router>
                            </td>
                            <td><Button outline color='danger' on:click={()=>eliminarUnidad(u.id)}>Eliminar</Button></td>
                        </tr>                        
                    {/each}

                </tbody>                    
                {/await}

            </Table>
        
        </Col>
    </Row>
</Container>