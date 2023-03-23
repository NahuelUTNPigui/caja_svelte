<script lang="ts">
    import { Col, Container, Row } from 'sveltestrap';
    import { Router, Link } from "svelte-routing";
    export let url = "";
</script>

<Container>
    <Router url="{url}">
        <nav>
            <Row>
                <Col>
                    <Link to="/">Home</Link>                 
                </Col>
                <Col>
                    <Link to="/clientes">Clientes</Link>                 
                </Col>
                <Col>
                    <Link to="/rubros">Rubros</Link>
                </Col>
                <Col>
                    <Link to="/proveedores">Proveedores</Link>
                    
                </Col>
                <Col>
                    <Link to="/tipoproveedores">Tipo Proveedores</Link>
                </Col>
                <Col>
                    <Link to="/unidades">Unidades</Link>
                </Col>
                <Col>
                    <Link to="/modos">Modos pago</Link>
                </Col>
                <!--<Col>
                    <Link to="/subcategorias">Sub categorias</Link>
                </Col>-->
                <Col>
                    <Link to="/ingresos">Ingresos</Link>
                    
                </Col>
                <Col>
                    <Link to="/egresos">Egresos</Link>
                </Col>
                <Col>
                    <Link to="/reportes">Reportes</Link>
                </Col>
            </Row>   
        </nav>
    </Router>
</Container>