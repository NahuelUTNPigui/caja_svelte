<script>
    import {Container,Row,Col,Button,Table,Input,Label,Modal,ModalBody,ModalFooter,ModalHeader, ButtonGroup} from 'sveltestrap'
    import {Router,Link,navigate} from 'svelte-routing'
    function nuevo_ingreso(){
        navigate('/detail_ingreso/0')
    }
    const RUTA="http://localhost:8090/api/collections"
    export let url=""
    let fechaDesde=""
    let fechaHasta=""
    let idIngresoEliminar=""
    let itemxpagina=10
    let pagina_actual=1
    let paginas_totales=1
    let pages_promise=[]
    let ingreso_pagina=[]
    let ingresos_filtrados=[]
    let open=false
    let toggle=()=>(open = !open)
    let ingresos_todos=[]
    let ingresos_promise=getIngresosPagina(1)
    function addDays(date,days,i){
        
        var result = new Date(date);
        result.setDate(result.getDate() + days);
        return result;
    }
    async function getIngresosPagina(pagina){
        
        if(pagina===1){
            ingresos_todos=[]
            let res=await fetch(RUTA+"/ingreso/records?perPage=1&page1")
            let data_ing_q=await res.json()
            
            let paginas=Math.floor(data_ing_q.totalItems/200)+1
            for(let i=1;i<=paginas;i++){
                let res_ing=await fetch(RUTA+"/ingreso/records?perPage=200&page="+i)
                let data_ing=await res_ing.json()
                ingresos_todos=ingresos_todos.concat(data_ing.items.filter(ig=>(fechaDesde==="" || ig.fechaIngreso>=fechaDesde) && (fechaHasta==="" || ig.fechaIngreso<=fechaHasta)))
            }
            if (ingresos_todos.length===0){
                pagina_actual=0
            }
            else{
                pagina_actual=pagina
            }
            ingresos_todos.sort((a,b)=>{
                if(new Date(a.fechaIngreso)>new Date(b.fechaIngreso)){
                    return -1
                }
                else if(new Date(a.fechaIngreso)<new Date(b.fechaIngreso)){
                    return 1
                }
                else{
                    return 0
                }
            })
            paginas_totales=ingresos_todos.length/itemxpagina|0
            if(ingresos_todos.length%itemxpagina !== 0){
                paginas_totales += 1
            }
            pages_promise=[]
            for(let i=1;i<=paginas_totales;i++){
                pages_promise.push(i)
            }
            let ingreso_pagina=[]
            for(let i=0;i<(itemxpagina>ingresos_todos.length?ingresos_todos.length:itemxpagina);i++){
                ingreso_pagina.push(ingresos_todos[i])
            }
            
            return ingreso_pagina
                             
            
            
        }
        else{
            pagina_actual=pagina
            let ingreso_pagina=[]
            for(let i=(pagina-1)*itemxpagina;i<(pagina*itemxpagina>ingresos_todos.length?ingresos_todos.length:pagina*itemxpagina);i++){
                    ingreso_pagina.push(ingresos_todos[i])
            }
                
            return ingreso_pagina
        }

    }
    async function eliminarIngresoBD(){
        await fetch(RUTA+"/ingreso/records/"+idIngresoEliminar,{
            method:"DELETE"
        })
        pagina_actual=1
        paginas_totales=1
        pages_promise=[]
        ingreso_pagina=[]
        ingresos_filtrados=[]
        ingresos_promise=getIngresosPagina(1)
        idIngresoEliminar=""
        toggle()
    }
    async function eliminarIngreso(idIngreso){
        idIngresoEliminar=idIngreso
        toggle()
    }
    function cancelarModal(){
        idIngresoEliminar=""
        toggle()
    }
    async function getIngreso(idIngreso) {
        let res=await fetch(RUTA+"/ingreso/records/"+idIngreso)
        let data=await res.json()
        return data
    }
    async function getClienteNombre(codClient){
        let res=await fetch(RUTA+"/cliente/records/"+codClient)
        let data=await res.json()
        return data.nombre
    }
    async function getSaldo(){
        let res=await fetch(RUTA+"/saldo/records")
        let data=await res.json()
        return data.items[0]
    }
    function num2Curr(number){
        const numberFormat = new Intl.NumberFormat('es-ES');
        return numberFormat.format(number)
    }
    function onChangeItemXpagina(){
        ingresos_promise = getIngresosPagina(1)
    }
</script>
<div>
    <Modal isOpen={open} {toggle}>
      <ModalHeader {toggle}>Eliminar ingreso</ModalHeader>
      <ModalBody>
        Esta seguro que desea eliminar el ingreso?.
      </ModalBody>
      <ModalFooter>
        <Button color="danger" on:click={eliminarIngresoBD}>Eliminar</Button>
        <Button color="secondary" on:click={cancelarModal}>Cancel</Button>
      </ModalFooter>
    </Modal>
  </div>
<Container>
    <Row>
        <Col>
            <Button on:click={nuevo_ingreso}>Nuevo ingreso</Button></Col>
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
            <Button on:click={()=>ingresos_promise=getIngresosPagina(1)}>Buscar</Button>
        </Col>
    </Row>
    <hr>
    <Row>
        <Col>
            <Container>
                <Row>
                    <Col md="2">Pagina: {pagina_actual}</Col>
                    <Col>
                        <ButtonGroup>
                            <Button outline color='primary' on:click={()=>ingresos_promise=getIngresosPagina(pagina_actual>1?pagina_actual-1:1)}>&laquo</Button>
                            {#await pages_promise then pages}
                                {#each pages as p}
                                    <Button outline color='primary' on:click={()=>ingresos_promise=getIngresosPagina(p)}>{p}</Button>
                                {/each}                        
                            {/await}
    
                            <Button outline color='primary' on:click={()=>ingresos_promise=getIngresosPagina(pagina_actual<paginas_totales?pagina_actual+1:paginas_totales)}>&raquo</Button>
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
                    <th>Cliente</th>
                    <th>Acciones</th>
                    <th></th>
                </tr>
            </thead>
            {#await ingresos_promise then is}
                <tbody>
                {#each is as i}
                    <tr> 
                        <td>{addDays(new Date(i.fechaIngreso),1,i).toLocaleDateString()}</td>
                        <td>{num2Curr(i.monto)}</td>
                        <td>
                            {#await getClienteNombre(i.codCliente) then n}
                                {n}
                            {/await}   
                        </td>
                        <td>
                            <Router url="{url}">
                                <nav>
                                    <Link to="/detail_ingreso/{i.id}">Modificar</Link>
                                </nav>
                            </Router>
                        </td>
                        <td><Button outline color="danger" on:click={()=>eliminarIngreso(i.id)}>Eliminar</Button></td>
                    </tr>    
                {/each}
                </tbody>
            {/await}
        </Table>
    </Row>
</Container>