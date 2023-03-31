<script>
    import {Container,Row,Col,Table} from 'sveltestrap'
    const version="1.0.4"
    const RUTA="http://localhost:8090/api/collections"
    let nuevoSaldo=false
    let numNuevoSaldo=0
    async function getLastIngresos(){
        let res=await fetch(RUTA+"/ingreso/records?sort=-fechaIngreso&page=1&perPage=5")
        let data=await res.json()
        return data.items
    }
    async function getLastEgresos(){
        let res=await fetch(RUTA+"/egreso/records?sort=-fechaEgreso&page=1&perPage=5")
        let data=await res.json()
        return data.items
    }
    async function getClienteNombre(codClient){
        let res=await fetch(RUTA+"/cliente/records/"+codClient)
        let data=await res.json()
        return data.nombre
    }
    async function getProveedorNombre(codProv){
        let res=await fetch(RUTA+"/proveedores/records/"+codProv)
        let data=await res.json()
        return data.nombre
    }
    function addDays(date,days){
        var result = new Date(date);
        result.setDate(result.getDate() + days);
        return result;
    }
    //Chusmear esto     
    async function recalcularSaldo(){
        nuevoSaldo=true
        let ingresos=[]
        let egresos =[]
        let res_ingreso_q=await fetch(RUTA+"/ingreso/records?perPage=1&page=1")
        let res_egreso_q=await fetch(RUTA+"/egreso/records?perPage=1&page=1")
        let data_ing_q=await res_ingreso_q.json()
        let data_eg_q=await res_egreso_q.json()
        
        let paginas=Math.floor(data_ing_q.totalItems/200)+1
        for(let i=1;i<=paginas;i++){
            let res_ing=await fetch(RUTA+"/ingreso/records?perPage=200&page="+i)
            let data_ing=await res_ing.json()
            ingresos=ingresos.concat(data_ing.items)
        }
        
        
        
        paginas=Math.floor(data_eg_q.totalItems/200)+1
        for(let i=1;i<=paginas;i++){
            let res_eg=await fetch(RUTA+"/egreso/records?perPage=200&page="+i)
            let data_eg=await res_eg.json()
            
            egresos=egresos.concat(data_eg.items)
        }        
        let saldo=0
        for(let i=0;i<ingresos.length;i++){
            saldo += ingresos[i].monto
        }   

        for(let i=0;i<egresos.length;i++){
            
            saldo -=egresos[i].monto

        }   
        
        
        numNuevoSaldo=saldo
        
    }
    function num2Curr(number){
        
        const numberFormat = new Intl.NumberFormat('es-ES');
        return numberFormat.format(number)
    }
</script>

<main>

    <Container>
        <Row>
            <Col>
                {#await recalcularSaldo() then _}
                    <h3 >Saldo: $<span class="{numNuevoSaldo>0?'positivo':'negativo'}">{num2Curr(numNuevoSaldo)}</span></h3>
                {/await}
                <hr>
            </Col>
            <!--     -->
<!--             <Col>
                {#if nuevoSaldo}
                    <h3 >Saldo recalculado: $<span class="{numNuevoSaldo>0?'positivo':'negativo'}">{num2Curr(numNuevoSaldo)}</span></h3>   
                {/if}
            </Col> -->
        </Row> 
        <Row>
            <Col>
                <h4>Últimos ingresos</h4>
                <Table>
                    <thead>
                        <tr>
                            <th>Fecha</th>
                            <th>Monto</th>
                            <th>Cliente</th>
                        </tr>
                    </thead>
                    {#await getLastIngresos() then igs}
                        <tbody>
                            {#each igs as ig}
                                <tr>
                                    <td>{addDays(new Date(ig.fechaIngreso),1).toLocaleDateString()}</td>
                                    <td>{ig.monto}</td>
                                    <td>
                                        {#await getClienteNombre(ig.codCliente) then n}
                                            {n}
                                        {/await}
                                    </td>
                                </tr>                                
                            {/each}

                        </tbody>
                    {/await}

                </Table>
            </Col>
            <Col>
                <h4>Últimos egresos</h4>
                <Table>
                    <thead>
                        <tr>
                            <th>Fecha</th>
                            <th>Monto</th>
                            <th>Proveedor</th>
                        </tr>
                    </thead>
                    {#await getLastEgresos() then egs}
                        <tbody>
                            {#each egs as eg}
                                <tr>
                                    <td>{addDays(new Date(eg.fechaEgreso),1).toLocaleDateString()}</td>
                                    <td>{eg.monto}</td>
                                    <td>
                                        {#await getProveedorNombre(eg.codProveedor) then n}
                                            {n}
                                        {/await}
                                    </td>
                                </tr>                                
                            {/each}

                        </tbody>                        
                    {/await}

                </Table>
            </Col>
        </Row>
    </Container>
</main>
<p>Version :{version}</p>
<style>
    .negativo{
        color: #fe0000;
    }
    .positivo{
        color: #498306;
    }
</style>