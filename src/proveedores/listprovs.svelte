<script>
    import {Container,Row,Col,Button,Table,Input,Modal,ModalBody,ModalFooter,ModalHeader,ButtonGroup} from 'sveltestrap'
    import {Router,Link,navigate} from 'svelte-routing'
    function nuevo_proveedor(){
        navigate('/detail_prov/0')
    }
    const RUTA="http://localhost:8090/api/collections"
    export let url=""
    let nombre=""
    let idProveedorEliminar=""
    let  open =false
    let toggle=()=>(open = !open)
    let itemxpagina = 10
    let pages_promise=[]
    let paginas_totales=1
    let pagina_actual=1
    let proveedores_promise=getProveedoresPagina(1)
    async function getProveedores(){
        let res=await fetch(RUTA+"/proveedores/records?perPage=200&&page1")
        let data=await res.json()
        return data.items
    }
    async function getProveedoresPagina(pagina){
        let res=null
        if (!!nombre){
            res = await fetch(RUTA+"/proveedores/records?perPage="+itemxpagina+"&page="+pagina+"&filter=(nombre~'"+nombre+"')")

        }
        else{
            res = await fetch(RUTA+"/proveedores/records?perPage="+itemxpagina+"&page="+pagina)   
        }

        let data=await res.json()
        let items=data.items
        pagina_actual=pagina
        paginas_totales=data.totalPages
        pages_promise=[]
        for(let i=1;i<=paginas_totales;i++){
            pages_promise.push(i)
        }
        if(items.length==0){
                pagina_actual=0
        }
        proveedores_promise=items
        return items

    }
    async function eliminarProveedorBD(){
        await fetch(RUTA+"/proveedores/records/"+idProveedorEliminar,{
            method:"DELETE"
        })
        proveedores_promise=getProveedoresPagina(1)
        idProveedorEliminar=""
        nombre=""
        pagina_actual=1
        paginas_totales=1
        toggle()
    }
    function cancelarModal(){
        idProveedorEliminar=""
        toggle()
    }
    async function eliminarProveedor(codProv){
        idProveedorEliminar=codProv
        toggle()
    }
    async function getProveedoresFilter(){
        let res=await fetch(RUTA+"/proveedores/records?perPage=200&&page1")
        
        let data=await res.json()
        let items=data.items.filter(p=>p.apodo.includes(nombre))
        
        proveedores_promise=items
    }
    async function onChangeItemXpagina(){
        proveedores_promise=getProveedoresPagina(1)
    }
</script>
<main>
    <div>
        <Modal isOpen={open} {toggle}>
          <ModalHeader {toggle}>Eliminar proveedor</ModalHeader>
          <ModalBody>
            ¿Esta seguro que desea eliminar el proveedor?
            <br>
            Puede afectar a los egresos asociados
          </ModalBody>
          <ModalFooter>
            <Button color="danger" on:click={eliminarProveedorBD}>Eliminar</Button>
            <Button color="secondary" on:click={cancelarModal}>Cancelar</Button>
          </ModalFooter>
        </Modal>
      </div>
    <Container>
        <Row>
            <Col>
                
                <Container>
                    <Row>
                        <Col><Button on:click={nuevo_proveedor}>Nuevo proveedor</Button></Col>
                        <Col name="nombre">Nombre</Col>
                        <Col><Input
                            type="text"
                            name="nombre"
                            id="nombre"
                            bind:value="{nombre}"
                        /></Col>
                        <Col><Button on:click={()=>getProveedoresPagina(1)}>Buscar</Button></Col>
                        
                    </Row>
                </Container>
                <hr>
                <Container>
                    <Row>
                        <Col md="2">Pagina: {pagina_actual}</Col>
                        <Col md="6">
                            <ButtonGroup>
                                <Button outline color='primary' on:click={()=>getProveedoresPagina(pagina_actual>1?pagina_actual-1:1)}>&laquo</Button>
                                {#await pages_promise then pages}
                                    {#each pages as p}
                                        <Button outline color='primary' on:click={()=>getProveedoresPagina(p)}>{p}</Button>
                                    {/each}                        
                                {/await}
    
                                <Button outline color='primary' on:click={()=>getProveedoresPagina(pagina_actual<paginas_totales?pagina_actual+1:paginas_totales)}>&raquo</Button>
                            </ButtonGroup>
                        </Col>
                        <Col md="2">Items por pagina</Col>
                        <Col>
                            <select bind:value="{itemxpagina}" name="ixp" id="ixp" size="1" on:change={onChangeItemXpagina}>
                                <option value="5">5</option>
                                <option value="10" selected>10</option>
                                <option value="15" >15</option>
                            </select>
                        </Col>
                    </Row>
                </Container>
                <hr>
                <Table>
                    <thead>
                        <tr>
                            <th>Apodo</th>
                            <th>Nombre, Apellido</th>
                            <th>Acciones</th>
                            <th></th>
                        </tr>
                    </thead>
                    {#await proveedores_promise then ps}
                        <tbody>
                            {#each ps as p}
                                <tr>
                                    <td>{p.apodo}</td>
                                    <td>{p.nombre}, {p.apellido}</td>
                                    <td>
                                        <Router url="{url}">
                                            <nav>
                                                <Link to="/detail_prov/{p.id}">Modificar</Link>
                                            </nav>
                                        </Router>
                                    </td>
                                    <td><Button outline color="danger" on:click={()=>eliminarProveedor(p.id)}>Eliminar</Button></td>
                                </tr>                                
                            {/each}

                        </tbody>                        
                    {/await}

                </Table>
            
            </Col>
        </Row>
    </Container>
</main>