<script>
    import {Button} from "sveltestrap"
    export let table=[]
    export let headers=[]
    const RUTA="http://localhost:8090/api/collections"
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
    function addDays(fecha,dias){
        var result = new Date(fecha);
        result.setDate(result.getDate() + dias);
        return result;
    }
    async function table2table(old_table){
        let new_table=[]
        for(let i=0;i<old_table.length;i++){
            let elem=old_table[i]
            elem["modo"]=await getModoNombre(elem.codModo)
            elem["unidad"]=await getUnidadNombre(elem.codUnidad)
            if(elem.codCliente){
                elem["agente"]=await getClienteNombre(elem.codCliente)
                elem["fecha"]=addDays(new Date(elem.fechaIngreso),1).toLocaleDateString()
                elem["tipo"]="ingreso"
                
            }
            else{
                elem["agente"]=await getProveedorNombre(elem.codProveedor)
                elem["fecha"]=addDays(new Date(elem.fechaEgreso),1).toLocaleDateString()
                elem["tipo"]="egreso"
                elem.monto=-1*elem.monto
            }
            elem.monto=elem.monto
            new_table.push(elem)
        }
        return new_table
    }
    function table2csv(table,headers){
        let csv=""
        for(let i=0;i<headers.length;i++){
            csv+=headers[i]
            if(i!==headers.length-1){
                csv+=","
            }
            
            
        }
        csv+="\n"
        for(let i=0;i<table.length;i++){
                for(let j=0;j<headers.length;j++){
                    csv+=table[i][headers[j]]
                    if(j!==headers.length-1){
                        csv+=","
                    }
                }
                if(i!==table.length-1){
                    csv+="\n"
                }
                
            }
        return [csv]

    }
    async function saveStaticDataToFile(){
        let new_table=await table2table(table)
        let items=table2csv(new_table,headers)
        var blob = new Blob(items,
                { type: "text/plain;charset=utf-8" });
            // @ts-ignore
            saveAs(blob, "reporte_"+new Date().toLocaleDateString()+".csv");
        }
		
    
</script>
<Button on:click={saveStaticDataToFile}>Exportar</Button>