<script>
    import { Container,Row,Col,Table,Label,Input,Button,ButtonGroup } from "sveltestrap";
    import Exporttable from "../reportes/exporttable.svelte";
    const RUTA="http://localhost:8090/api/collections"
    let fechaDesde=""
    let fechaHasta=""
    let codUnidad=""
    let codModo=""
    let codProv=""
    let montoDesde=0
    let montoHasta=0
    let total=0
    let ordenar_por=""
    let itemxpagina=20
    let pagina_actual=1
    let paginas_totales=1
    let pages_promise=[]
    let egresos_todos=[]    
    let egresos_promise=getEgresosPagina(1)

    function sortByMonto(a,b){
        return b.monto-a.monto
        
    }
    function sortByFecha(a,b){
        if(new Date(a.fechaEgreso)>new Date(b.fechaEgreso)){
            return -1
        }
        else if(new Date(a.fechaEgreso)<new Date(b.fechaEgreso)){
            return 1
        }
        else{
            return 0
        }       
    }
    function filt(eg){
        if(fechaDesde!==""){
            if(!(new Date(eg.fechaEgreso)>=new Date(fechaDesde))){
                return false
            }
        }
        if(fechaHasta!==""){
            if(!(new Date(eg.fechaEgreso)<=new Date(fechaHasta))){
                return false
            }
        }
        if(codProv!==""){
            if(eg.codProveedor!==codProv){
                return false
            }
        }
        if(codModo!==""){
            if(eg.codModo!==codModo){
                return false
            }
        }
        if(codUnidad!==""){
            if(eg.codUnidad!==codUnidad){
                return false
            }
        }
        if(montoHasta!==0){
            if(eg.monto>montoHasta){
                return false
            }
        }
        if(eg.monto<montoDesde){
            return false
        }
        return true
    }
    async function getEgresosPagina(pagina){
        if(pagina==0){
            return []
        }
        if(pagina==1){
            egresos_todos=[]
            let res=await fetch(RUTA+"/egreso/records?perPage=1&page=1")
            let data=await res.json()
            if(data.totalItems>200){
                let paginas=Math.floor(data.totalItems/200)+1
                for(let i=1;i<=paginas;i++){
                    let res_eg=await fetch(RUTA+"/egreso/records?perPage=200&page="+i)
                    let data_eg=await res_eg.json()
                    
                    egresos_todos=egresos_todos.concat(data_eg.items.filter(filt))
                
                }
                //Que onda que no funciona
                let ordenar=sortByFecha
                if(ordenar_por=="monto"){
                    // @ts-ignore
                    ordenar=sortByMonto
                }
                egresos_todos.sort(ordenar)
                if (egresos_todos.length===0){
                    pagina_actual=0
                }
                else{
                    pagina_actual=pagina
                }
                paginas_totales=egresos_todos.length/itemxpagina|0
                if(egresos_todos.length%itemxpagina !== 0){
                    paginas_totales += 1
                }
                pages_promise=[]
                for(let i=1;i<=paginas_totales;i++){
                    pages_promise.push(i)
                }
                let egreso_pagina=[]
                for(let i=(pagina-1)*itemxpagina;i<(pagina*itemxpagina>egresos_todos.length?egresos_todos.length:pagina*itemxpagina);i++){
                    egreso_pagina.push(egresos_todos[i])
                }
                
                return egreso_pagina
            }
            else{
                let res_eg=await fetch(RUTA+"/egreso/records?perPage="+data.totalItems+"&page=1")
                let data_eg=await res_eg.json()

                egresos_todos = data_eg.items.filter(filt)
                if (egresos_todos.length===0){
                    pagina_actual=0
                }
                else{
                    pagina_actual=pagina
                }
                //Que onda que no funciona
                let ordenar=sortByFecha
                if(ordenar_por=="monto"){
                    // @ts-ignore
                    ordenar=sortByMonto
                }
                egresos_todos.sort(ordenar)
                paginas_totales=egresos_todos.length/itemxpagina|0
                if(egresos_todos.length%itemxpagina !== 0){
                    paginas_totales += 1
                }
                pages_promise=[]
                for(let i=1;i<=paginas_totales;i++){
                    pages_promise.push(i)
                }
                let egreso_pagina=[]
                for(let i=(pagina-1)*itemxpagina;i<(pagina*itemxpagina>egresos_todos.length?egresos_todos.length:pagina*itemxpagina);i++){
                    egreso_pagina.push(egresos_todos[i])
                }
                
                return egreso_pagina
            }
        }
        else{
            pagina_actual=pagina
            let egreso_pagina=[]
            for(let i=(pagina-1)*itemxpagina;i<(pagina*itemxpagina>egresos_todos.length?egresos_todos.length:pagina*itemxpagina);i++){

                egreso_pagina.push(egresos_todos[i])
            }

            return egreso_pagina
        }
    }
    async function getEgresosFiltros(){
        function filt(eg){
            if(fechaDesde!==""){
                if(!(eg.fechaEgreso>=fechaDesde)){
                    return false
                }
            }
            if(fechaHasta!==""){
                if(!(eg.fechaEgreso<=fechaHasta)){
                    return false
                }
            }
            if(codProv!==""){
                if(eg.codProveedor!==codProv){
                    return false
                }
            }
            if(codModo!==""){
                if(eg.codModo!==codModo){
                    return false
                }
            }
            if(codUnidad!==""){
                if(eg.codUnidad!==codUnidad){
                    return false
                }
            }
            if(montoHasta!==0){
                if(eg.monto>montoHasta){
                    return false
                }
            }
            if(eg.monto<montoDesde){
                return false
            }
            return true
        }
        let res=await fetch(RUTA+"/egreso/records?perPage=200&&page=1")
        let data=await res.json()
        let items=data.items.filter(
            eg=>filt(eg)
        )
        
        if(ordenar_por!==""){
            if(ordenar_por==="fecha"){
                items.sort((e1,e2)=>{
                    if(e1.fechaEgreso>e2.fechaEgreso){
                        return -1
                    }
                    else{
                        return 1
                    }
                })
            }
            else{
                items.sort((i1,i2)=>{
                    return i2.monto-i1.monto
                })
            }
        }
        total=items.reduce((prev,e)=>prev+e.monto,0)
        egresos_promise=items
        
    }
    async function getProveedorNombre(idProv){
        let res=await fetch(RUTA+"/proveedores/records/"+idProv)
        let data=await res.json()
        return data.nombre
    }
    async function getUnidadNombre(codUnidad){
        let res=await fetch(RUTA+"/unidades/records/"+codUnidad)
        let data=await res.json()
        return data.nombre
    }
    async function getModoNombre(codModo){
        let res=await fetch(RUTA+"/modopago/records/"+codModo)
        let data=await res.json()
        return data.nombre
    }
    async function getUnidades(){
        let res=await fetch(RUTA+"/unidades/records?perPage=100&&page1")
        let data=await res.json()
        return data.items
    }
    async function getModos(){
        let res=await fetch(RUTA+"/modopago/records?perPage=100&&page1")
        let data=await res.json()
        return data.items
    }
    async function getProveedores(){
        let res=await fetch(RUTA+"/proveedores/records?sort=apodo&&perPage=200&&page1")
        let data=await res.json()
        return data.items
    }
    function num2Curr(number){
        const numberFormat = new Intl.NumberFormat('es-ES');
        return numberFormat.format(number)
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
<Container>
    <Row>
        <Col>
            <h2>Reporte egresos</h2>
            
        </Col>
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
        <Col>
            <Label for="proveedor">Proveedor</Label>
            <br>    
            <select bind:value="{codProv}" name="proveedor" id="proveedor" size="1">
                <option value=""></option>
                {#await getProveedores() then ps}
                    {#each ps as p}
                        <option value="{p.id}">{p.apodo}</option>
                    {/each}
                    
                {/await}
            </select>
        </Col>
    </Row>
    <Row>
        <Col>
            <br>    
        </Col>
    </Row>
    <Row>
        
        <Col>
            <Label for="unidad">Unidad</Label>
            <br>
            <select bind:value="{codUnidad}" name="unidad" id="unidad" size="1" >
                {#await getUnidades() then us}
                    <option value=""></option>
                    {#each us as u}
                        <option value="{u.id}">{u.nombre}</option>
                    {/each}
                    
                {/await}                                 
            </select>
        </Col>
        <Col>
            <Label for ="modo">Modo</Label>
            <br>
            <select bind:value="{codModo}" name="modo" id="modo" size="1" >
                {#await getModos() then ms}
                    <option value=""></option>
                    {#each ms as m}
                        <option value="{m.id}">{m.nombre}</option>
                    {/each}
                    
                {/await}                                 
            </select>
        </Col>
        <Col>
            <Label for="montoDesde">Monto mínimo</Label>
            <Input
                type="number"
                name="montoDesde"
                id="montoDesde"
                bind:value="{montoDesde}"
            />
        </Col>
        <Col>
            <Label for="montoHasta">Monto máximo</Label>
            <Input
                type="number"
                name="montoHasta"
                id="montoHasta"
                bind:value="{montoHasta}"
            />
        </Col>
    </Row>
    <Row>
        <Col>
            <Label for="ordenar_por">Ordenar por</Label>
            <br>
            <select bind:value="{ordenar_por}" name="ordenar_por" id="ordenar_por" size="1" >
                    <option value=""></option>
                    <option value="fecha">Fecha</option>
                    <option value="monto">Monto</option>
                    
                                
            </select>
        </Col>
    </Row>
    <Row>
        <Col>
            <br>
            <Button on:click={()=>egresos_promise=getEgresosPagina(1)}>Buscar</Button>
        </Col>
        <Col>
            <br>
            <h4>Egreso Total : ${num2Curr(total)}</h4>
        </Col>
        <Col>
            {#await egresos_promise then egreso}
                <Exporttable table={egresos_todos} headers={["monto","fecha","agente","modo","unidad","tipo"]}></Exporttable>    
            {/await}
            
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
                    <th>Modo</th>
                    <th>Unidad</th>
                    
                    
                </tr>
            </thead>
            {#await egresos_promise then es}
                <tbody>
                    {#each es as e}
                        <tr>
                            <td>{addDays(new Date(e.fechaEgreso),1).toLocaleDateString()}</td>
                            <td>{num2Curr(e.monto)}</td>
                            <td>
                                {#await getProveedorNombre(e.codProveedor) then n}
                                    {n}
                                {/await}
                            </td>
                            <td>
                                {#await getModoNombre(e.codModo) then n}
                                    {n}
                                {/await}
                            </td>
                            <td>
                                {#await getUnidadNombre(e.codUnidad) then n}
                                    {n}
                                {/await}
                            </td>
                            
                        </tr>                        
                    {/each}

                </tbody>    
            {/await}
            
        </Table>
    </Row>
</Container>
<style>
    select{
        width: 200px;
    }
</style>