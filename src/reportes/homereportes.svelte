<script lang="ts">
    import {Container,Row,Col,Button,Table,Input,Label} from 'sveltestrap'
    import RepEgreso from '../egresos/reporteegreso.svelte'
    import Reporteingreso from '../ingresos/reporteingreso.svelte';
    import Reportesaldo from './reportesaldo.svelte';
    
    let option="ing"
    function changeOption(n_opt){
        option=n_opt
    }
</script>
<Container>
    <Row>
        <Col>
            <Button on:click={()=>changeOption("ing")}>Reporte ingresos</Button>
        </Col>
        <Col>
            <Button on:click={()=>changeOption("eg")}>Reporte egresos</Button>
        </Col>
        <Col>
            <Button on:click={()=>changeOption("sal")}>Reporte Saldo</Button>
        </Col>
    </Row>    
    <Row>
        <Col>
            <br>
            {#if option==="ing"}
                <Reporteingreso></Reporteingreso>
            {:else if option==="eg"}
                <RepEgreso></RepEgreso>
            {:else}
                <Reportesaldo></Reportesaldo>
            {/if}

        </Col>
    </Row>
</Container>
