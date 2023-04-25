<script >
    import {Container,Row,Col,Button,Table,Input,Modal,ModalBody,ModalFooter,ModalHeader,ButtonGroup} from 'sveltestrap'
    import {Router,Link,navigate} from 'svelte-routing'
    function nuevo_cliente(){
        navigate('/detail_client/0')
    }
    const RUTA="http://localhost:8090/api/collections"
    export let url=""
    let itemxpagina=10
    let nombre=""
    let open = false
    let idClienteEliminar=""
    const toggle =()=>{open = !open}
    let pages_promise=getPages()
    let pagina_actual=1
    let paginas_totales=1
    let clientes_promise=getClientes()
    
    async function getClientes(){
        let res=await fetch(RUTA+"/cliente/records?perPage="+itemxpagina)
        let data=await res.json()
        
        return data.items
    }
    async function getPages(){
        let res=await fetch(RUTA+"/cliente/records?perPage="+itemxpagina)
        let data=await res.json()
        let pages=[]
        paginas_totales=data.totalPages
        for(let i=1;i<=data.totalPages;i++){
            pages.push(i)
        }
        return pages
    }
    async function getClientesPagina(pagina){
        if (!!nombre){
            let res=await fetch(RUTA+"/cliente/records?perPage="+itemxpagina+"&filter=(nombre~'"+nombre+"')&page="+pagina)
            let data=await res.json()
            let pages=[]
            paginas_totales=data.totalPages
            for(let i=1;i<=data.totalPages;i++){
                pages.push(i)
            }
            pagina_actual=pagina
            if(data.items.length==0){
                pagina_actual=0
            }
            clientes_promise=data.items
            //@ts-ignore
            pages_promise=pages
        }
        else{
            let res=await fetch(RUTA+"/cliente/records?perPage="+itemxpagina+"&page="+pagina)
            let data=await res.json()
            pagina_actual=pagina
            clientes_promise=data.items
        }       
    }
    async function getClientesFilterPagina(pagina){
        let res=await fetch(RUTA+"/cliente/records?filter=(nombre~'"+nombre+"')&page="+pagina)
        let data=await res.json()
        let pages=[]
        paginas_totales=data.totalPages
        for(let i=1;i<=data.totalPages;i++){
            pages.push(i)
        }
        pagina_actual=1

        clientes_promise=data.items
        //@ts-ignore
        pages_promise=pages
    }   
    async function eliminarClienteBD(){
        await fetch(RUTA+"/cliente/records/"+idClienteEliminar,{
            method:"DELETE"
        })
        nombre=""
        pages_promise=getPages()
        pagina_actual=1
        paginas_totales=1
        clientes_promise=getClientes()
        idClienteEliminar=""
        toggle()
    }
    function cancelarModal(){
        idClienteEliminar=""
        toggle()
    }
    async function eliminarCliente(idCliente){
        idClienteEliminar=idCliente
        toggle()
    }
    function buscar(){
        if (!!nombre){
            getClientesPagina(1)
        }
        else{
            pages_promise=getPages()
            pagina_actual=1
            paginas_totales=1
            clientes_promise=getClientes()
        }
    }
    function onChangeItemXpagina(){
        nombre=""
        pages_promise=getPages()
        pagina_actual=1
        paginas_totales=1
        clientes_promise=getClientes()
    }
</script>
<main>
    <div>
        <Modal isOpen={open} {toggle}>
          <ModalHeader {toggle}>Eliminar cliente</ModalHeader>
          <ModalBody>
            ¿Esta seguro que desea eliminar el cliente?
            <br>
            Podria modificar informacion en los ingresos
          </ModalBody>
          <ModalFooter>
            <Button color="danger" on:click={eliminarClienteBD}>Eliminar</Button>
            <Button color="secondary" on:click={cancelarModal}>Cancelar</Button>
          </ModalFooter>
        </Modal>
      </div>
    <Container>
        <Row>
            <Col>                
                <Container>
                    <Row>
                        <Col><Button on:click={nuevo_cliente}>Nuevo cliente</Button></Col>
                        <Col name="nombre">Nombre</Col>
                        <Col><Input
                            type="text"
                            name="nombre"
                            id="nombre"
                            bind:value="{nombre}"
                        /></Col>
                        <Col><Button on:click={buscar}>Buscar</Button></Col>
                        
                    </Row>

                </Container>
                <hr>
                <Container>
                    <Row>
                        <Col md="2">Pagina: {pagina_actual} </Col>
                        <Col>
                            <ButtonGroup>
                                <Button outline color='primary' on:click={()=>!!nombre?getClientesFilterPagina(pagina_actual>1?pagina_actual-1:1):getClientesPagina(pagina_actual>1?pagina_actual-1:1)}>&laquo</Button>
                                {#await pages_promise then pages}
                                    {#each pages as p}
                                        <Button outline color='primary' on:click={()=>getClientesPagina(p)}>{p}</Button>
                                    {/each}                        
                                {/await}
    
                                <Button outline color='primary' on:click={()=>getClientesPagina(pagina_actual<paginas_totales?pagina_actual+1:paginas_totales)}>&raquo</Button>
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
                            <th>Nombre</th>
                            <th>Razon social</th>
                            <th>Acciones</th>
                            <th></th>
                        </tr>
                    </thead>
                    {#await clientes_promise then cs}
                        <tbody>
                            {#each cs as c}
                                <tr>
                                    <td>{c.nombre}</td>
                                    <td>{c.razonsocial}</td>
                                    <td>
                                        <Router url="{url}">
                                            <nav>
                                                <Link to="/detail_client/{c.id}">Modificar</Link>
                                            </nav>
                                        </Router>
                                    </td>
                                    <td><Button outline color='danger' on:click={()=>eliminarCliente(c.id)}>Eliminar</Button></td>
                                </tr>                            
                            {/each}

                        </tbody>                   
                    {/await}

                </Table>
            
            </Col>
        </Row>
    </Container>
</main>