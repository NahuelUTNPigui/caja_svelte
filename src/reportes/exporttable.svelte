<script>
    import {Button} from "sveltestrap"
    //import { saveAs } from 'file-saver';
    //import { saveAs } from 'file-saver';
    export let table=[]
    export let headers=[]
    export let titulo=""
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
    async function getRubroNombre(codCliente){
        let res_c=await fetch(RUTA+"/cliente/records/"+codCliente)
        let data_c=await res_c.json()
        let res=await fetch(RUTA+"/rubro/records/"+data_c.codRubro)
        let data=await res.json()
        return data.nombre
    }
    async function getTipoProveedorNombre(codProveedor){
        let res_p=await fetch(RUTA+"/proveedores/records/"+codProveedor)
        let data_p=await res_p.json()
        let res=await fetch(RUTA+"/tipoproveedor/records/"+data_p.codTipo)
        let data =await res.json()
        return data.nombre
    }
    function addDays(fecha,dias){
        var result = new Date(fecha);
        result.setDate(result.getDate() + dias);
        return result;
    }
    // NO funciona el num2Curr
    function only2Dig(numb){
        return Math.round(100*numb)/100
    }
    function num2Curr(numb){
        
        const numberFormat = new Intl.NumberFormat('es-ES');
        return numberFormat.format(only2Dig(numb))
    }
    async function table2table(old_table){
        let new_table=[]
        for(let i=0;i<old_table.length;i++){
            let elem=old_table[i]
            //console.log(old_table[i])
            elem["modo"]=await getModoNombre(elem.codModo)
            elem["unidad"]=await getUnidadNombre(elem.codUnidad)
            if(elem.codCliente){
                elem["agente"]=await getClienteNombre(elem.codCliente)
                elem["subtipo"]=await getRubroNombre(elem.codCliente)
                elem["fecha"]=addDays(new Date(elem.fechaIngreso),1).toLocaleDateString()
                elem["tipo"]="ingreso"
                
            }
            else{
                elem["agente"]=await getProveedorNombre(elem.codProveedor)
                elem["subtipo"]=await getTipoProveedorNombre(elem.codProveedor)
                elem["fecha"]=addDays(new Date(elem.fechaEgreso),1).toLocaleDateString()
                elem["tipo"]="egreso"
                
                elem.monto=-1*elem.monto
            }
            elem.monto='"'+num2Curr(elem.monto)+'"'
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
            saveAs(blob, titulo+" reporte: "+new Date().toLocaleDateString()+".csv");
        }
		
    
</script>
<Button on:click={saveStaticDataToFile}>Exportar</Button>