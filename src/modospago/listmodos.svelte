<script>
    import {Container,Row,Col,Button,Table,Modal,ModalBody,ModalFooter,ModalHeader} from 'sveltestrap'
    import {Router,Link,navigate} from 'svelte-routing'
    function nuevo_modo(){
        navigate('/detail_modo/0')
    }
    const RUTA="http://localhost:8090/api/collections"
    export let url=""
    let idEliminarModo=""
    let open = false
    let toggle = ()=> (open = !open)
    let modos_promise=getModos()
    async function getModos(){
        let res=await fetch(RUTA+"/modopago/records")
        let data=await res.json()
        return data.items
    }
    async function eliminarModoBD(){
        await fetch(RUTA+"/modopago/records/"+idEliminarModo,{
            method:"DELETE"
        })
        modos_promise=getModos()
        idEliminarModo=""
        toggle()
    }
    function cancelarModal(){
        idEliminarModo=""
        toggle()
    }
    async function eliminarModo(idModo){
        idEliminarModo=idModo
        toggle()
    }
</script>
<div>
    <Modal isOpen={open} {toggle}>
      <ModalHeader {toggle}>Eliminar Modo de pago</ModalHeader>
      <ModalBody>
        Esta seguro que desea eliminar el modo de pago?.
        <br>
        Podria modificar algun ingreso o egreso
      </ModalBody>
      <ModalFooter>
        <Button color="danger" on:click={eliminarModoBD}>Eliminar</Button>
        <Button color="secondary" on:click={cancelarModal}>Cancel</Button>
      </ModalFooter>
    </Modal>
  </div>
<main>
    <Container>
        <Row>
            <Col>
                <Button on:click={nuevo_modo}>Nuevo Modo de pago</Button>

                <hr>
                <Table>
                    <thead>
                        <tr>
                            <th>Nombre</th>
                            <th>Acciones</th>
                            <th></th>
                        </tr>
                    </thead>
                    {#await modos_promise then ms}
                        <tbody>
                            {#each ms as m}
                                <tr>
                                    <td>{m.nombre}</td>
                                    <td>
                                        <Router url="{url}">
                                            <nav>
                                                <Link to="/detail_modo/{m.id}">Modificar</Link>
                                            </nav>
                                        </Router>
                                    </td>
                                    <td><Button outline color='danger' on:click={()=>eliminarModo(m.id)}>Eliminar</Button></td>
                                </tr>
                            {/each}
                        </tbody>
                    {/await}
                    
                </Table>
            
            </Col>
        </Row>
    </Container>
</main>