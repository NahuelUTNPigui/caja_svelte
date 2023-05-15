<script>
    import {Container,Row,Col,Button,Table,Input,Label,Modal,ModalBody,ModalFooter,ModalHeader,ButtonGroup} from 'sveltestrap'
    import {Router,Link,navigate} from 'svelte-routing'
    function nuevo_egreso(){
        navigate('/detail_egreso/0')
    }
    const RUTA="http://localhost:8090/api/collections"
    export let url=""
    let fechaDesde=""
    let fechaHasta=""
    let itemxpagina=20
    let pagina_actual=1
    let paginas_totales=1
    let pages_promise=[]
    let egreso_pagina=[]
    let egresos_filtrados=[]
    let egresos = []
    let idEgresoEliminar=""
    let open = false
    let toggle =()=> (open= !open)
    let egresos_promise=getEgresosPagina(1)
    async function getEgresosPagina(pagina){
        if(pagina==1){
            egresos=[]
            let res=await fetch(RUTA+"/egreso/records?perPage=1&page=1")
            let data_eg_q=await res.json()

            let paginas=Math.floor(data_eg_q.totalItems/200)+1
            for(let i=1;i<=paginas;i++){
                let res_eg=await fetch(RUTA+"/egreso/records?perPage=200&page="+i)
                let data_eg=await res_eg.json()
                
                egresos=egresos.concat(data_eg.items.filter(eg=>{
                    return (fechaDesde==="" || eg.fechaEgreso>=fechaDesde) && (fechaHasta==="" || eg.fechaEgreso<=fechaHasta)
                }))
            
            }
            egresos.sort((a,b)=>{
                if(new Date(a.fechaEgreso)>new Date(b.fechaEgreso)){
                    return -1
                }
                else if(new Date(a.fechaEgreso  )<new Date(b.fechaEgreso)){
                    return 1
                }
                else{
                    return 0
                }
            })
            if (egresos.length===0){
                pagina_actual=0
            }
            else{
                pagina_actual=pagina
            }
            paginas_totales=egresos.length/itemxpagina|0
            if(egresos.length%itemxpagina !== 0){
                paginas_totales += 1
            }
            pages_promise=[]
            for(let i=1;i<=paginas_totales;i++){
                pages_promise.push(i)
            }
            let egreso_pagina=[]
            for(let i=(pagina-1)*itemxpagina;i<(pagina*itemxpagina>egresos.length?egresos.length:pagina*itemxpagina);i++){
                egreso_pagina.push(egresos[i])
            }
            
            return egreso_pagina
            

        }
        else{
            pagina_actual=pagina
            let egreso_pagina=[]
            for(let i=(pagina-1)*itemxpagina;i<(pagina*itemxpagina>egresos.length?egresos.length:pagina*itemxpagina);i++){

                egreso_pagina.push(egresos[i])
            }

            return egreso_pagina
        }
    }
    async function eliminarEgresoBD(){
        await fetch(RUTA+"/egreso/records/"+idEgresoEliminar,{
            method:"DELETE"
        })
        pagina_actual=1
        paginas_totales=1
        pages_promise=[]
        egresos_filtrados=[]
        egreso_pagina=[]
        egresos_promise=getEgresosPagina(1)
        idEgresoEliminar=""
        toggle()

    }
    async function eliminarEgreso(idEgreso){
        idEgresoEliminar=idEgreso
        toggle()
    }
    function cancelarModal(){
        idEgresoEliminar=""
        toggle()
    }
    async function getProveedorNombre(codProv){
        let res=await fetch(RUTA+"/proveedores/records/"+codProv)
        let data=await res.json()
        return data.nombre
    }
    function only2Dig(numb){
        return Math.round(100*numb)/100
    }
    function num2Curr(numb){
            
        const numberFormat = new Intl.NumberFormat('es-ar',{
            style:"currency",
            currency:"ARS"
        });
        return numberFormat.format(only2Dig(numb))
    }
    function addDays(date,days){
        var result = new Date(date);
        result.setDate(result.getDate() + days);
        return result;
    }
    function onChangeItemXpagina(){
        egresos_promise = getEgresosPagina(1)
    }
</script>
<div>
    <Modal isOpen={open} {toggle}>
      <ModalHeader {toggle}>Eliminar egreso</ModalHeader>
      <ModalBody>
        ¿Esta seguro que desea eliminar el Egreso?
      </ModalBody>
      <ModalFooter>
        <Button color="danger" on:click={eliminarEgresoBD}>Eliminar</Button>
        <Button color="secondary" on:click={cancelarModal}>Cancelar</Button>
      </ModalFooter>
    </Modal>
  </div>
<Container>
    <Row>
        <Col>
            <Button on:click={nuevo_egreso}>Nuevo egreso</Button></Col>
    </Row>
    <Row>
        <Col>
            <Label for="fechaDesde">Fecha desde</Label>
            <Input
                type="date"
                name="fechaDesde"
                id="fechaDesde"
                bind:value="{fechaDesde}"
            />
        </Col>
        <Col>
            <Label for ="fechaHasta">Fecha hasta</Label>
            <Input
                type="date"
                name="fechaHasta"
                id="fechaHasta"
                bind:value="{fechaHasta}"
            />
        </Col>
    </Row>
    <Row>
        <Col>
            <br>
            <Button on:click={()=>egresos_promise=getEgresosPagina(1)}>Buscar</Button>
        </Col>
    </Row>
    <hr>
    <Row>
        <Col>
            <Container fluid>
                <Row>
                    <Col md="2">Pagina: {pagina_actual}</Col>
                    <Col>
                        <ButtonGroup>
                            <Button outline color='primary' on:click={()=>egresos_promise=getEgresosPagina(pagina_actual>1?pagina_actual-1:1)}>&laquo</Button>
                            {#await pages_promise then pages}
                                {#each pages as p}
                                    <Button outline color='primary' on:click={()=>egresos_promise=getEgresosPagina(p)}>{p}</Button>
                                {/each}                        
                            {/await}
    
                            <Button outline color='primary' on:click={()=>egresos_promise=getEgresosPagina(pagina_actual<paginas_totales?pagina_actual+1:paginas_totales)}>&raquo</Button>
                        </ButtonGroup>
                    </Col>
                    <Col md="2">Item por pagina</Col>
                    <Col>
                        <select bind:value="{itemxpagina}" name="ixp" id="ixp" size="1" on:change={onChangeItemXpagina}>
                            <option value="5">5</option>
                            <option value="10" selected>10</option>
                            <option value="15" >15</option>
                            <option value="30" >30</option>
                            <option value="50" >50</option>
                            <option value="100" >100</option>
                            

                        </select>
                    </Col>

                </Row>
            </Container>
        </Col>
    </Row>
    <hr>
    <Row>
        <Table>
            <thead>
                <tr>
                    <th>Fecha</th>
                    <th>Monto</th>
                    <th>Proveedor</th>
                    <th>Acciones</th>
                    <th></th>
                </tr>
            </thead>
            {#await egresos_promise then es}
                <tbody>
                    {#each es as e}
                        <tr>
                            <td>{addDays(new Date(e.fechaEgreso),1).toLocaleDateString()}</td>
                            <td class="text-end">{num2Curr(e.monto)}</td>
                            <td>
                                {#await getProveedorNombre(e.codProveedor) then n}
                                    {n}
                                {/await}
                            </td>
                            <td>
                                <Router url="{url}">
                                    <nav>
                                        <Link to="/detail_egreso/{e.id}">Modificar</Link>
                                    </nav>
                                </Router>
                            </td>
                            <td><Button outline color="danger" on:click={()=>eliminarEgreso(e.id)}>Eliminar</Button></td>
                        </tr>                        
                    {/each}

                </tbody>    
            {/await}
            
        </Table>
    </Row>
</Container>