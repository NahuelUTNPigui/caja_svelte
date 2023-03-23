<script>
    import { Container,Row,Col,Table,Label,Input,Button,ButtonGroup } from "sveltestrap";
    import Exporttable from "../reportes/exporttable.svelte";
    const RUTA="http://localhost:8090/api/collections"
    let fechaDesde=""
    let fechaHasta=""
    let codUnidad=""
    let codModo=""
    let montoDesde=0
    let pagina_actual=1
    let itemxpagina=30
    let montoHasta=0
    let total=0
    let ordenar_por=""
    let saldos_todos=[]
    let pages_promise=[]
    let paginas_totales=1
    let saldo_promise=getSaldoPagina(0)
    function sortByFecha(trans1,trans2){
        let fecha1;
        if(trans1.fechaIngreso){
            fecha1=new Date(trans1.fechaIngreso)
        }
        else{
            fecha1=new Date(trans1.fechaEgreso)
        }
        let fecha2;
        if(trans2.fechaIngreso){
            fecha2=new Date(trans2.fechaIngreso)
        }
        else{
            fecha2=new Date(trans2.fechaEgreso)
        }
        if(fecha1>fecha2){
            return -1
        }
        else if(fecha1<fecha2){
            return 1
        }
        else{
            return 0
        } 
    }
    function sortByMonto(trans1,trans2){

        return trans1.monto-trans2.monto

    }
    function filt(trans,esIngreso){
        if(fechaDesde!==""){
            if (esIngreso){
                if(!(new Date(trans.fechaIngreso)>=new Date(fechaDesde))){
                    return false
                }
            }
            else{
                if(!(new Date(trans.fechaEgreso)>=new Date(fechaDesde))){
                    return false
                }
            }

        }
        if(fechaHasta!==""){
            if (esIngreso){
                if(!(new Date(trans.fechaIngreso)<=new Date(fechaHasta))){
                    return false
                }
            }
            else{
                if(!(new Date(trans.fechaEgreso)<=new Date(fechaHasta))){
                    return false
                }
            }
        }
        if(codModo!==""){
            if(trans.codModo!==codModo){
                return false
            }
        }
        if(codUnidad!==""){
            if(trans.codUnidad!==codUnidad){
                return false
            }
        }
        if(montoHasta!==0){
            if(trans.monto>montoHasta){
                return false
            }
        }
        if(trans.monto<montoDesde){
            return false
        }
        return true
    }
    async function getSaldoPagina(pagina){
        if (pagina==0){
            return []
        }
        if(pagina==1){
            saldos_todos=[]
            let res_ing=await fetch(RUTA+"/ingreso/records?perPage=1&&page=1")
            let res_eg=await fetch(RUTA+"/egreso/records?perPage=1&page=1")
            let data_ing=await res_ing.json()
            let data_eg=await res_eg.json()
            let paginas_ing=Math.floor(data_ing.totalItems/200)+1
            let paginas_eg=Math.floor(data_eg.totalItems/200)+1
            for(let i=1;i<=paginas_ing;i++){
                let res =await fetch(RUTA+"/ingreso/records?perPage=200&&page="+i)
                let data=await res.json()
                saldos_todos=saldos_todos.concat(data.items.filter((ing)=>filt(ing,true)))
            }
            for(let i=1;i<=paginas_eg;i++){
                let res =await fetch(RUTA+"/egreso/records?perPage=200&page="+i)
                let data=await res.json()
                saldos_todos=saldos_todos.concat(data.items.filter((eg)=>filt(eg,false)))
            }
            if(saldos_todos.length==0){
                pagina_actual=0
            }
            else{
                pagina_actual=1
            }
            //Que onda que no funciona
            let ordenar=sortByFecha
            if(ordenar_por=="monto"){
                // @ts-ignore
                ordenar=sortByMonto
            }
            saldos_todos.sort(ordenar)
            paginas_totales=saldos_todos.length/itemxpagina|0
            if(saldos_todos.length%itemxpagina !== 0){
                paginas_totales += 1
            }
            pages_promise=[]
            for(let i=1;i<=paginas_totales;i++){
                pages_promise.push(i)
            }
            let saldo_pagina=[]
            for(let i=(pagina-1)*itemxpagina;i<(pagina*itemxpagina>saldos_todos.length?saldos_todos.length:pagina*itemxpagina);i++){
                saldo_pagina.push(saldos_todos[i])
            }
            
            return saldo_pagina

        }
        else{
            pagina_actual=pagina
            let saldo_pagina=[]
            for(let i=(pagina-1)*itemxpagina;i<(pagina*itemxpagina>saldos_todos.length?saldos_todos.length:pagina*itemxpagina);i++){

                saldo_pagina.push(saldos_todos[i])
            }

            return saldo_pagina
        }

    }
    async function getSaldoFilter(){
        function filt(sal,is_ing){
            if(fechaDesde!==""){
                if(is_ing){
                    if(!(sal.fechaIngreso>=fechaDesde)){
                        return false
                    }
                }
                else{
                    if(!(sal.fechaEgreso>=fechaDesde)){
                        return false
                    }

                }

            }
            if(fechaHasta!==""){
                if(is_ing){
                    if(!(sal.fechaIngreso<=fechaHasta)){
                        return false
                    }
                }
                else{
                    if(!(sal.fechaEgreso<=fechaHasta)){
                        return false
                    }
                }

            }
            if(codModo!==""){
                if(sal.codModo!==codModo){
                    return false
                }
            }
            if(codUnidad!==""){
                if(sal.codUnidad!==codUnidad){
                    return false
                }
            }
            if(montoHasta!==0){
                if(sal.monto>montoHasta){
                    return false
                }
            }
            if(sal.monto<montoDesde){
                return false
            }
            return true
        }
        let res_ingreso_q=await fetch(RUTA+"/ingreso/records?perPage=0&&page=1")
        let res_egreso_q=await fetch(RUTA+"/egreso/records?perPage=0&&page=1")
        let data_ing_q=await res_ingreso_q.json()
        let data_eg_q=await res_egreso_q.json()
        let res_ing=await fetch(RUTA+"/ingreso/records?perPage="+data_ing_q.totalItems+"&&page=1")
        let res_eg=await fetch(RUTA+"/egreso/records?perPage="+data_eg_q.totalItems+"&&page=1")
        let data_ing=await res_ing.json()
        let data_eg=await res_eg.json()
        
        
        let items_ing=data_ing.items.filter(
            ig=>filt(ig,true)
        )
        let items_eg=data_eg.items.filter(
            eg=>filt(eg,false)
        )
        let items=items_ing.concat(items_eg)
        if(ordenar_por!==""){
            if(ordenar_por==="fecha"){
                items.sort((s1,s2)=>{
                    if(s1.fechaIngreso){
                        if(s1.fechaIngreso>s2.fechaIngreso){
                            return -1
                        }
                        else{
                            return 1
                        }
                    }
                    else{
                        if(s1.fechaEgreso>s2.fechaEgreso){
                            return -1
                        }
                        else{
                            return 1
                        }
                    }


                })
            }
            else{
                items.sort((i1,i2)=>{
                    return i2.monto-i1.monto
                })
            }
        }
        total=0
        for(let s of items){
            if(s.codCliente){
                total+=s.monto
            }
            else{
                total-=s.monto
            }
        }
        saldo_promise=items
        
    }
    async function getClienteNombre(codClient){
        let res=await fetch(RUTA+"/cliente/records/"+codClient)
        let data=await res.json()
        return data.nombre
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
        let res=await fetch(RUTA+"/unidades/records")
        let data=await res.json()
        return data.items
    }
    async function getModos(){
        let res=await fetch(RUTA+"/modopago/records")
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
        saldo_promise=getSaldoPagina(1)
    }
</script>
<Container>
    <Row>
        <Col>
            <h2>Reporte Saldo</h2>
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
            <Button on:click={()=>saldo_promise=getSaldoPagina(1)}>Buscar</Button>
        </Col>
        <Col>
            <br>
            <h4>Saldo Total $ <span class="{total>0?'positivo':'negativo'}">{num2Curr(total)}</span></h4>
        </Col>
        <Col>

            {#await saldo_promise then ss}
                <Exporttable table={saldos_todos} headers={["monto","fecha","agente","modo","unidad","tipo"]} ></Exporttable>    
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
                            <Button outline color='primary' on:click={()=>saldo_promise=getSaldoPagina(pagina_actual>1?pagina_actual-1:1)} >&laquo</Button>
                            {#await pages_promise then pages}
                                {#each pages as p}
                                    <Button outline color='primary' on:click={()=>saldo_promise=getSaldoPagina(p)}>{p}</Button>
                                {/each}                        
                            {/await}
    
                            <Button outline color='primary' on:click={()=>saldo_promise=getSaldoPagina(pagina_actual<paginas_totales?pagina_actual+1:paginas_totales)}>&raquo</Button>
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
                    <th>Agente</th>
                    <th>Modo</th>
                    <th>Unidad</th>
                    
                </tr>
            </thead>
            {#await saldo_promise then ss}
                <tbody>
                    {#each ss as s}
                        <tr>
                            <td>
                                {#if s.fechaIngreso}
                                    {addDays(new Date(s.fechaIngreso),1).toLocaleDateString()}                                    
                                {:else}
                                    {addDays(new Date(s.fechaEgreso),1).toLocaleDateString()}
                                {/if}
                            </td>
                            <td>{num2Curr(s.monto)}</td>
                            <td>
                                {#if s.codCliente}
                                    {#await getClienteNombre(s.codCliente) then n}
                                        Cliente: {n}
                                    {/await}
                                {:else}
                                    {#await getProveedorNombre(s.codProveedor) then n}
                                        Proveedor: {n}
                                    {/await}
                                {/if}

                            </td>
                            <td>
                                {#await getModoNombre(s.codModo) then n}
                                    {n}
                                {/await}
                            </td>
                            <td>
                                {#await getUnidadNombre(s.codUnidad) then n}
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

    .negativo{
        color: #fe0000;
    }
    .positivo{
        color: #498306;
    }
</style>