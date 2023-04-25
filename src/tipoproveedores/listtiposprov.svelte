<script lang="ts">
    import {Container,Row,Col,Button,Table,Modal,ModalBody,ModalFooter,ModalHeader,ButtonGroup} from 'sveltestrap'
    import {Router,Link,navigate} from 'svelte-routing'
    const RUTA="http://localhost:8090/api/collections"
    
    let itemxpagina=10
    let paginas_totales=1
    let pagina_actual=1
    let pages_promise=[]
    let tipos_promise=getTipoPagina(1)
    export let url=""
    let idEliminarTipoProveedor=""
    let open = false
    let toggle=()=>(open =!open)
    function nuevo_tipo_prov(){
        navigate('/detail_tipo_prov/0')
    }
    async function getTipoPagina(pagina) {
        let res = await fetch(RUTA+"/tipoproveedor/records?perPage="+itemxpagina+"&page="+pagina)
        let data = await res.json()
        let items=data.items
        pagina_actual=pagina
        paginas_totales=data.totalPages
        pages_promise=[]
        for(let i=1;i<=paginas_totales;i++){
            pages_promise.push(i)
        }
        items.sort((a,b)=>{
            let aup=a.nombre.toUpperCase()
            let bup=b.nombre.toUpperCase()
            if(aup>bup){
                return 1
            }
            else if(bup>aup){
                return -1
            }
            return 0

        })
        tipos_promise=items
        return items
    }
    
    async function eliminarTipoBD() {
        await fetch(RUTA+"/tipoproveedor/records/"+idEliminarTipoProveedor,{
            method:"DELETE"
        })
        tipos_promise=getTipoPagina(1)        
        idEliminarTipoProveedor=""
        toggle()
    }
    async function eliminarTipo(idTipo){
        idEliminarTipoProveedor=idTipo
        toggle()
    }
    function cancelarModal(){
        idEliminarTipoProveedor=""
        toggle()
    }
    function onChangeItemXpagina(){  
        pagina_actual=1
        paginas_totales=1
        tipos_promise=getTipoPagina(1)
    }
</script>
<div>
    <Modal isOpen={open} {toggle}>
      <ModalHeader {toggle}>Eliminar Tipo proveedor</ModalHeader>
      <ModalBody>
        ¿Esta seguro que desea eliminar el tipo proveedor?
        <br>
        Podria modificar algun proveedor
      </ModalBody>
      <ModalFooter>
        <Button color="danger" on:click={eliminarTipoBD}>Eliminar</Button>
        <Button color="secondary" on:click={cancelarModal}>Cancelar</Button>
      </ModalFooter>
    </Modal>
  </div>
<Container>
    <Row>
        <Col>
            <Button on:click={nuevo_tipo_prov}>Nuevo Tipo proveedor</Button>

            <hr>
            <Container>
                <Row>
                    <Col md="2">Pagina: {pagina_actual}</Col>
                    <Col>
                        <ButtonGroup>
                            <Button outline color='primary' on:click={()=>getTipoPagina(pagina_actual>1?pagina_actual-1:1)}>&laquo</Button>
                            {#await pages_promise then pages}
                                {#each pages as p}
                                    <Button outline color='primary' on:click={()=>getTipoPagina(p)}>{p}</Button>
                                {/each}                        
                            {/await}

                            <Button outline color='primary' on:click={()=>getTipoPagina(pagina_actual<paginas_totales?pagina_actual+1:paginas_totales)}>&raquo</Button>
                        </ButtonGroup>
                    </Col>
                    <Col md="2">Item por pagina: </Col>
                    <Col>
                        <select bind:value="{itemxpagina}" name="ixp" id="ixp" size="1" on:change={onChangeItemXpagina}>
                            <option value="5">5</option>
                            <option value="10" selected>10</option>
                            <option value="15" >15</option>
                        </select>
                    </Col>
                    
                </Row>
            </Container>
            <Table>
                <thead>
                    <tr>
                        <th>Nombre</th>
                        <th>Acciones</th>
                        <th></th>
                    </tr>
                </thead>
                {#await tipos_promise then tps}
                    <tbody>
                        {#each tps as tp}
                            <tr>
                                <td>{tp.nombre}</td>
                                <td>
                                    <Router url="{url}">
                                        <nav>
                                            <Link to="/detail_tipo_prov/{tp.id}">Modificar</Link>
                                        </nav>
                                    </Router>
                                </td>
                                <td><Button outline color="danger" on:click={()=>eliminarTipo(tp.id)}>Eliminar</Button></td>
                            </tr>                            
                        {/each}

                    </tbody>                    
                {/await}

            </Table>
        
        </Col>
    </Row>
</Container>