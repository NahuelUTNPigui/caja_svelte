<script>
    import {Container,Row,Col,Button,Table,Modal,ModalBody,ModalFooter,ModalHeader} from 'sveltestrap'
    import {Router,Link,navigate} from 'svelte-routing'
    function nuevo_rubro(){
        navigate('/detail_rubro/0')
    }
    const RUTA="http://localhost:8090/api/collections"
    export let url=""
    let idEliminarRubro=""
    let open=false
    let toggle=()=>(open = !open)
    let rubros_promise=getRubros()
    async function getRubros(){
        let res=await fetch(RUTA+"/rubro/records")
        let data=await res.json()
        return data.items
    }
    async function eliminarRubroBD(){
        await fetch(RUTA+"/rubro/records/"+idEliminarRubro,{
            method:"DELETE"
        })
        rubros_promise=getRubros()
        idEliminarRubro=""
        toggle()
    }
    async function eliminarRubro(idRubro){
        idEliminarRubro=idRubro
        toggle()
    }
    function cancelarModal(){
        idEliminarRubro=""
        toggle()
    }
</script>
<div>
    <Modal isOpen={open} {toggle}>
      <ModalHeader {toggle}>Eliminar Rubro</ModalHeader>
      <ModalBody>
        ¿Esta seguro que desea eliminar el rubro?
        <br>
        Podria modificar algun ingreso o egreso
      </ModalBody>
      <ModalFooter>
        <Button color="danger" on:click={eliminarRubroBD}>Eliminar</Button>
        <Button color="secondary" on:click={cancelarModal}>Cancelar</Button>
      </ModalFooter>
    </Modal>
  </div>
<main>
    <Container>
        <Row>
            <Col>
                <Button on:click={nuevo_rubro}>Nuevo Rubro</Button>

                <hr>
                <Table>
                    <thead>
                        <tr>
                            <th>Nombre</th>
                            <th>Acciones</th>
                            <th></th>
                        </tr>
                    </thead>
                    {#await rubros_promise then  rs}
                        <tbody>
                            {#each rs as r}
                                <tr>
                                    <td>{r.nombre}</td>
                                    <td>
                                        <Router url="{url}">
                                            <nav>
                                                <Link to="/detail_rubro/{r.id}">Modificar</Link>
                                            </nav>
                                        </Router>
                                    </td>
                                    <td><Button outline color='danger' on:click={()=>eliminarRubro(r.id)}>Eliminar</Button></td>
                                </tr>                                
                            {/each}

                        </tbody>                        
                    {/await}

                </Table>
            
            </Col>
        </Row>
    </Container>
</main>