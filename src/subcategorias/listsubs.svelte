<script lang="ts">
    import {Container,Row,Col,Button,Table} from 'sveltestrap'
    import {Router,Link,navigate} from 'svelte-routing'
    export let url=""
    const RUTA="http://localhost:8090/api/collections"
    let sub_categorias=getSubs()
    async function getSubs(){
        let res=await fetch(RUTA+"/subcategorias/records")
        let data=await res.json()
        return data.items
    }
    function nuevo_sub(){
        navigate('/detail_sub/0')
    }
    async function eliminarSubCategoria(codSub){
        await fetch(RUTA+"/subcategorias/records/"+codSub,{
            method:"DELETE"
        })
        sub_categorias=getSubs()
    } 
</script>
<Container>
    <Row>
        <Col>
            <Button on:click={nuevo_sub}>Nueva sub categoriaa</Button>

            <hr>
            <Table>
                <thead>
                    <tr>
                        <th>Nombre</th>
                        <th>Acciones</th>
                        <th></th>
                    </tr>
                </thead>
                {#await sub_categorias then sbs}
                    <tbody>
                        {#each sbs as sb}
                            <tr>
                                <td>{sb.nombre}</td>
                                <td>
                                    <Router url="{url}">
                                        <nav>
                                            <Link to="/detail_sub/{sb.id}">Modificar</Link>
                                        </nav>
                                    </Router>
                                </td>
                                <td><Button outline color="danger" on:click={()=>eliminarSubCategoria(sb.id)}>Eliminar</Button></td>
                            </tr>                            
                        {/each}

                    </tbody>                    
                {/await}

            </Table>
        
        </Col>
    </Row>
</Container>