<script>
    import { Container,Row,Col,Table,Label,Input,Button,ButtonGroup } from "sveltestrap";
    // @ts-ignore
    import {Router,Link,navigate} from 'svelte-routing'
    // @ts-ignore
    import { jsPDF } from "jspdf";
    import autoTable from 'jspdf-autotable'
    import Exporttable from "../reportes/exporttable.svelte";
    const RUTA="http://localhost:8090/api/collections"
    export let url=""
    let fechaDesde=""
    let fechaHasta=""
    let codUnidad=""
    let codRubro=""
    let clientes_todos=[]
    let codModo=""
    let codCliente=""
    let montoDesde=0
    let montoHasta=0
    let monto_total=0
    // @ts-ignore
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
    function getClienteCod(codCliente){
        let cs=clientes_todos.filter(p=>p.id===codCliente)
        if(cs.length==0){
            return null
        }
        return cs[0]
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
            if(codRubro!==""){
                let c = getClienteCod(ig.codCliente)
                if(c.codRubro!==codRubro){
                    return false
                }
            }
            return true
    }
    async function getIngresosPagina(pagina){
        if (pagina==0){
            ingresos_todos=[]
            monto_total=0
            return []
        }
        if (pagina==1){
            monto_total=0
            ingresos_todos=[]
            let res=await fetch(RUTA+"/ingreso/records?perPage=1&&page=1")
            let data=await res.json()
            
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
            for(let i_i=0;i_i<ingresos_todos.length;i_i++){
                monto_total += ingresos_todos[i_i].monto
            }
            let ingreso_pagina=[]
            for(let i=(pagina-1)*itemxpagina;i<(pagina*itemxpagina>ingresos_todos.length?ingresos_todos.length:pagina*itemxpagina);i++){
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
        let unidades=[]
        let res=await fetch(RUTA+"/unidades/records?perPage=100&&page1")
        let data=await res.json()
        unidades=data.items
        unidades.sort((u1,u2)=>u1.nombre.toLowerCase()<u2.nombre.toLowerCase()?-1:1)
        return unidades
    }
    async function getModos(){
        let modos=[]
        let res=await fetch(RUTA+"/modopago/records?perPage=100&&page1")
        let data=await res.json()
        modos=data.items
        modos.sort((m1,m2)=>m1.nombre.toLowerCase()<m2.nombre.toLowerCase()?-1:1)

        return modos
    }
    async function getRubros(){
        let rubros=[]
        let res_p=await fetch(RUTA+"/rubro/records?page=1&perPage=1")
        let data_p=await res_p.json()
        let paginas= Math.floor(data_p.totalItems/200)+1
        for(let pag=1;pag<=paginas;pag++){
            let res=await fetch(RUTA+"/rubro/records?perPage=200&page="+pag)
            let data=await res.json()
            rubros=rubros.concat(data.items)
        }
        rubros.sort((r1,r2)=>r1.nombre.toLowerCase()<r2.nombre.toLowerCase()?-1:1)
        return rubros
    }
    async function getCliente(codClient){
        let res=await fetch(RUTA+"/cliente/records/"+codClient)
        let data=await res.json()
        return data
    }
    async function getRubroNombre(codRubro){
        let res=await fetch(RUTA+"/rubro/records/"+codRubro)
        let data=await res.json()
        return data.nombre
    }
    async function getClientes(){
        let clientes=[]
        let res_p=await fetch(RUTA+"/cliente/records?page=1&perPage=1")
        let data_p=await res_p.json()
        let paginas= Math.floor(data_p.totalItems/200)+1
        for(let pag=1;pag<=paginas;pag++){
            let res =await fetch(RUTA+"/cliente/records?perPage=200&page="+pag)
            let data=await res.json()
            clientes=clientes.concat(data.items)
        }
        clientes.sort((c1,c2)=>c1.nombre.toLowerCase()<c2.nombre.toLowerCase()?-1:1)
        clientes_todos=clientes
        return clientes
    }
    async function crearPDF(){
        if(ingresos_todos.length==0){
            return
        }
        // Default export is a4 paper, portrait, using millimeters for units
        const doc = new jsPDF();
        let body_table=[]
        for(let i_i=0;i_i<ingresos_todos.length;i_i++){
            let i = ingresos_todos[i_i]
            let row =[]
            let c = await getCliente(i.codCliente)
            row.push(addDays(i.fechaIngreso,1).toLocaleDateString())
            row.push(num2Curr(i.monto))
            row.push(c.nombre)
            row.push(await getRubroNombre(c.codRubro))
            row.push(await getModoNombre(i.codModo))
            row.push(await getUnidadNombre(i.codUnidad))
            row.push(i.observacion)
            body_table.push(row)
            
        }
        doc.text("Reporte ingreso: "+new Date().toLocaleDateString(), 5, 5);
        autoTable(doc, {
            columnStyles:{1:{halign:'right'}},
            head: [['Fecha','Monto','Cliente','Rubro','Modo','Unidad','Observacion']],
            body: body_table
        })
       
        doc.save("ingresos_"+new Date().toLocaleDateString()+".pdf");
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
            <h2>Reporte Ingresos</h2>
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
        <Col>
            <Label> Rubro</Label>
            <br>    
            <select bind:value="{codRubro}" name="rubro" id="rubro" size="1">
                <option value=""></option>
                {#await getRubros() then rs}
                    {#each rs as r}
                        <option value="{r.id}">{r.nombre}</option>
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
            <h4>Ingreso Total : ${num2Curr(monto_total)}</h4>
        </Col>
        <Col>
            <br>
            {#await ingresos_promise then ingreso}
                <Exporttable table={ingresos_todos} headers={["fecha","monto","agente","subtipo","modo","unidad","tipo","observacion"]} titulo="Ingreso"></Exporttable>
            {/await}
            
        </Col>
        <Col>
            <br>
            {#await ingresos_promise then ingreso}
                <Button on:click={crearPDF}>Crear documento PDF</Button>
            {/await}
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
                    <th>Rubro</th>
                    <th>Modo</th>
                    <th>Unidad</th>
                    <th>Accion</th>
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
                                {#await  getCliente(i.codCliente) then c}
                                    {#await getRubroNombre(c.codRubro) then n}
                                        {n}
                                    {/await}
                                    
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
                            <td>
                                <Router url="{url}">
                                    <nav>
                                        <Link to="/detail_ingreso/{i.id}">Modificar</Link>
                                    </nav>
                                </Router>
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