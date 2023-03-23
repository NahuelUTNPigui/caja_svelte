<script>
    import { Container,Row,Col,Table,Label,Input,Button,ButtonGroup } from "sveltestrap";
    import Exporttable from "../reportes/exporttable.svelte";
    const RUTA="http://localhost:8090/api/collections"
    let fechaDesde=""
    let fechaHasta=""
    let codUnidad=""
    let codModo=""
    let codCliente=""
    let montoDesde=0
    let montoHasta=0
    let total=0
    let ordenar_por=""
    let ingresos_todos=[]
    let pages_promise=[]
    let itemxpagina=10

    let pagina_actual=1
    let paginas_totales=1
    let ingresos_promise=getIngresosPagina(0)


    function sortByMonto(a,b){
        return b.monto-a.monto
        
    }
    function sortByFecha(a,b){
        if(new Date(a.fechaIngreso)>new Date(b.fechaIngreso)){
            return -1
        }
        else if(new Date(a.fechaIngreso)<new Date(b.fechaIngreso)){
            return 1
        }
        else{
            return 0
        }       
    }
    function filt(ig){
            if(fechaDesde!==""){
                if(!(new Date(ig.fechaIngreso)>=new Date(fechaDesde))){
                    return false
                }
            }
            if(fechaHasta!==""){
                if(!(new Date(ig.fechaIngreso)<=new Date(fechaHasta))){
                    return false
                }
            }
            if(codCliente!==""){
                if(ig.codCliente!==codCliente){
                    return false
                }
            }
            if(codModo!==""){
                if(ig.codModo!==codModo){
                    return false
                }
            }
            if(codUnidad!==""){
                if(ig.codUnidad!==codUnidad){
                    return false
                }
            }
            if(montoHasta!==0){
                if(ig.monto>montoHasta){
                    return false
                }
            }
            if(ig.monto<montoDesde){
                return false
            }
            return true
    }
    async function getIngresosPagina(pagina){
        if (pagina==0){
            ingresos_todos=[]
            return []
        }
        if (pagina==1){
            ingresos_todos=[]
            let res=await fetch(RUTA+"/ingreso/records?perPage=1&&page=1")
            let data=await res.json()
            if(data.totalItems>200){
                let paginas=Math.floor(data.totalItems/200)+1
                for(let i=1;i<=paginas;i++){
                    let res_ing=await fetch(RUTA+"/ingreso/records?perPage=200&page="+i)
                    let data_ing=await res_ing.json()
                    ingresos_todos=ingresos_todos.concat(data_ing.items.filter(filt))
                }
                if (ingresos_todos.length===0){
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
                ingresos_todos.sort(ordenar)
                paginas_totales=ingresos_todos.length/itemxpagina|0
                if(ingresos_todos.length%itemxpagina !== 0){
                    paginas_totales += 1
                }
                pages_promise=[]
                for(let i=1;i<=paginas_totales;i++){
                    pages_promise.push(i)
                }
                let ingreso_pagina=[]
                for(let i=(pagina-1)*itemxpagina;i<(pagina*itemxpagina>ingresos_todos.length?ingresos_todos.length:pagina*itemxpagina);i++){
                    ingreso_pagina.push(ingresos_todos[i])
                }
                
                return ingreso_pagina
            }
            else{
                let res_ing=await fetch(RUTA+"/ingreso/records?perPage=200&&page=1")
                let data_ing=await res_ing.json()
                ingresos_todos=data_ing.items.filter(ig=>filt(ig))
                if (ingresos_todos.length===0){
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
                ingresos_todos.sort(ordenar)
                paginas_totales=ingresos_todos.length/itemxpagina|0
                if(ingresos_todos.length%itemxpagina !== 0){
                    paginas_totales += 1
                }
                pages_promise=[]
                for(let i=1;i<=paginas_totales;i++){
                    pages_promise.push(i)
                }
                let ingreso_pagina=[]
                for(let i=(pagina-1)*itemxpagina;i<(pagina*itemxpagina>ingresos_todos.length?ingresos_todos.length:pagina*itemxpagina);i++){
                    ingreso_pagina.push(ingresos_todos[i])
                }
                
                return ingreso_pagina

            }
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
    async function getIngresosFiltros(){
        
        let res=await fetch(RUTA+"/ingreso/records?perPage=200&&page=1")
        let data=await res.json()
        
        let items=data.items.filter(
            ig=>filt(ig)
        )
        if(ordenar_por!==""){
            if(ordenar_por==="fecha"){
                items.sort((i1,i2)=>{
                    if(i1.fechaIngreso>i2.fechaIngreso){
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
        total=items.reduce((prev,i)=>prev+i.monto,0)
        ingresos_promise=items
        
    }
    async function getClienteNombre(codClient){
        let res=await fetch(RUTA+"/cliente/records/"+codClient)
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
    async function getClientes(){
        let res=await fetch(RUTA+"/cliente/records?sort=nombre&&perPage=200&&page1")
        let data=await res.json()
        return data.items
    }
    function num2Curr(number){
        const numberFormat = new Intl.NumberFormat('es-ES');
        return numberFormat.format(number)
    }
    function onChangeItemXpagina(){
        //@ts-ignore
        ingresos_promise = getIngresosPagina(1)
    }
    function addDays(date,days){
        var result = new Date(date);
        result.setDate(result.getDate() + days);
        return result;
    }
</script>
<Container>
    <Row>
        <Col>
            <h2>Reporte ingresos</h2>
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
            <Label for="cliente">Cliente</Label>
            <br>    
            <select bind:value="{codCliente}" name="cliente" id="cliente" size="1">
                <option value=""></option>
                {#await getClientes() then cs}
                    {#each cs as c}
                        <option value="{c.id}">{c.nombre}</option>
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
            <Button on:click={()=>ingresos_promise=getIngresosPagina(1)}>Buscar</Button>
        </Col>
        <Col>
            <br>
            <h4>Ingreso Total : ${num2Curr(total)}</h4>
        </Col>
        <Col>
            {#await ingresos_promise then ingreso}
                <Exporttable table={ingresos_todos} headers={["monto","fecha","agente","modo","unidad","tipo"]}></Exporttable>
            {/await}
            <!--Ya se me va a ocurrir algo-->
            <!--<Exporttable table={async ()=>await ingresos_promise} headers={["monto","fecha","agente","modo","unidad","tipo"]}></Exporttable>-->
        </Col>
    </Row>
    <hr>
    <Row>
        <Col>
            <Container>
                <Row >
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
    <Row>
        <Table>
            <thead>
                <tr>
                    <th>Fecha</th>
                    <th>Monto</th>
                    <th>Cliente</th>
                    <th>Modo</th>
                    <th>Unidad</th>
                    
                </tr>
            </thead>
            {#await ingresos_promise then is}
                <tbody>
                    {#each is as i}
                        <tr>
                            <td>{addDays(new Date(i.fechaIngreso),1).toLocaleDateString()}</td>
                            <td>{num2Curr(i.monto)}</td>
                            <td>
                                {#await getClienteNombre(i.codCliente) then n}
                                    {n}
                                {/await}
                            </td>
                            <td>
                                {#await getModoNombre(i.codModo) then n}
                                    {n}
                                {/await}
                            </td>
                            <td>
                                {#await getUnidadNombre(i.codUnidad) then n}
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