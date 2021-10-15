<template>
    <v-layout align-start>
        <v-flex>
            <v-toolbar flat color="white">
                <v-toolbar-title>Ventas</v-toolbar-title>
                <v-divider
                class="mx-2"
                inset
                vertical
                ></v-divider>
                <v-spacer></v-spacer>
                <v-text-field v-if="verNuevo==0" class="text-xs-center" v-model="search" append-icon="search" 
                label="Búsqueda" single-line hide-details></v-text-field>
                <v-spacer></v-spacer>
                <v-btn color="primary" v-if="verNuevo==0" @click="mostrarNuevo()" dark class="mb-2">Nuevo</v-btn>
                <v-dialog v-model="dialog" max-width="1000px">
                    <v-card>
                        <v-card-title> 
                            <span class="headline">Seleccione un artículo</span>
                        </v-card-title>            
                        <v-card-text>
                            <v-container grid-list-md>
                                <v-layout wrap>
                                    <v-flex xs12 sm12 md12 lg12 xl12>                                        
                                        <v-text-field v-model="texto" @keyup.enter="listarArticulos()" class="text-xs-center" append-icon="search" 
                                        label="Búsqueda"></v-text-field>
                                        <template>
                                            <v-data-table
                                                :headers="cabeceraArticulos"
                                                :items="articulos"
                                                class="elevation-1"
                                            >
                                                <template v-slot:items="props">
                                                <td class="justify-center layout px-0">
                                                    <v-icon
                                                    small
                                                    class="mr-2"
                                                    @click="agregarDetalle(props.item)"
                                                    >
                                                    add
                                                    </v-icon>                                                    
                                                </td>
                                                <td>{{ props.item.codigo }}</td>
                                                <td>{{ props.item.nombre }}</td>
                                                <td>{{ props.item.categoria.nombre }}</td>
                                                <td>{{ props.item.stock }}</td>
                                                <td>{{ props.item.precio_venta }}</td>
                                                <td>{{ props.item.descripcion }}</td>
                                                <td>
                                                    <div v-if="props.item.estado">
                                                        <span class="blue--text">Activo</span>
                                                    </div>
                                                    <div v-else>
                                                        <span class="red--text">Inactivo</span>
                                                    </div>
                                                </td>                
                                                </template>
                                            </v-data-table>
                                        </template>
                                    </v-flex>
                                </v-layout>
                            </v-container>
                        </v-card-text>            
                        <v-card-actions>
                        <v-spacer></v-spacer>
                            <v-btn color="blue darken-1" flat @click="close">Cancelar</v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
                <v-dialog v-model="adModal" max-width="290">
                    <v-card>
                        <v-card-title class="headline" v-if="adAccion==1">
                            Activar Item
                        </v-card-title>
                        <v-card-title class="headline" v-if="adAccion==2">
                            Desactivar Item
                        </v-card-title>
                        <v-card-text>
                            Estás a punto de <span v-if="adAccion==1">activar </span>
                            <span v-if="adAccion==2">desactivar </span> el item {{adNombre}}
                        </v-card-text>
                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn @click="activarDesactivarCerrar()" color="green darken-1" flat="flat">
                                Cancelar
                            </v-btn>
                            <v-btn v-if="adAccion==1" @click="activar()" color="orange darken-4" flat="flat">
                                Activar
                            </v-btn>
                            <v-btn v-if="adAccion==2" @click="desactivar()" color="orange darken-4" flat="flat">
                                Desactivar
                            </v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
                <v-dialog v-model="comprobanteModal" max-width="1000px">
                    <v-card>
                        <v-card-title class="headline">
                            <v-btn @click="crearPDF()"><v-icon>print</v-icon></v-btn> Reporte de venta
                        </v-card-title>

                        <v-card-text>
                            <div id="factura">
                                <header>
                                    <div id="logo">
                                        <img src="data:image/png;base64,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" id="imagen">
                                    </div>
                                    <div id="datos">
                                        <p id="encabezado">
                                            <b>SoftwareCardozo</b>
                                            <br>Camilo Fuentes - Ibague - Colombia Grupo 72 Ciclo 3<br>Telefono:(+57)3178350571<br>Email:camilofcardozo@hotmail.com
                                        </p>
                                    </div>
                                    <div id="fact">
                                        <p>{{tipo_comprobante}}<br>
                                        {{serie_comprobante}}-{{num_comprobante}}<br>
                                        {{fecha}}</p>
                                    </div>
                                </header>
                                <br>
                                <section>
                                    <div>
                                        <table id="facliente">
                                            <tbody>
                                                <tr>
                                                    <td id="cliente">
                                                        <strong>Sr(a). {{persona.nombre}}</strong><br>
                                                        <strong>Documento:</strong> {{persona.num_documento}}<br>
                                                        <strong>Dirección:</strong> {{persona.direccion}}<br>
                                                        <strong>Teléfono:</strong> {{persona.telefono}}<br>
                                                        <strong>Email:</strong> {{persona.email}}
                                                    </td>
                                                </tr>
                                            </tbody>
                                        </table>
                                    </div>
                                </section>
                                <br>
                                <section>
                                    <div>
                                        <table id="facarticulo">
                                            <thead>
                                                <tr id="fa">
                                                    <th>CANT</th>
                                                    <th>DESCRIPCION</th>
                                                    <th>PRECIO UNIT</th>
                                                    <th>DESC.</th>
                                                    <th>PRECIO TOTAL</th>
                                                </tr>
                                            </thead>
                                            <tbody>
                                                <tr v-for="det in detalles" :key="det._id">
                                                    <td style="text-align:center;">{{det.cantidad}}</td>
                                                    <td>{{det.articulo}}</td>
                                                    <td style="text-align:right;">{{det.precio}}</td>
                                                    <td style="text-align:right;">{{det.descuento}}</td>
                                                    <td style="text-align:right;">{{(det.cantidad*det.precio)-det.descuento}}</td>
                                                </tr>
                                            </tbody>
                                            <tfoot>
                                                <tr>
                                                    <th></th>
                                                    <th></th>
                                                    <th></th>
                                                    <th style="text-align:right;">SUBTOTAL</th>
                                                    <th style="text-align:right;">$ {{totalParcial=(total-totalImpuesto).toFixed(2)}}</th>
                                                </tr>
                                                <tr>
                                                    <th></th>
                                                    <th></th>
                                                    <th></th>
                                                    <th style="text-align:right;">IVA({{impuesto}}%)</th>
                                                    <th style="text-align:right;">$ {{totalImpuesto=((total*impuesto)/(1+impuesto)).toFixed(2)}}</th>
                                                </tr>
                                                <tr>
                                                    <th></th>
                                                    <th></th>
                                                    <th></th>
                                                    <th style="text-align:right;">TOTAL</th>
                                                    <th style="text-align:right;">$ {{total=calcularTotal}}</th>
                                                </tr>
                                            </tfoot>
                                        </table>
                                    </div>
                                </section>
                                <br>
                                <br>
                                <footer>
                                    <div id="gracias">
                                        <p><b>Gracias por su compra!</b></p>
                                    </div>
                                </footer>
                            </div>
                        </v-card-text>

                        <v-card-actions>
                            <v-btn @click="ocultarComprobante()" color="blue darken-1" flat>Cancelar</v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
            </v-toolbar>
            <v-data-table
                :headers="headers"
                :items="ventas"
                :search="search"
                class="elevation-1"
                v-if="verNuevo==0"
            >
                <template v-slot:items="props">
                    <td class="justify-center layout px-0">
                        <v-icon small class="mr-2" @click="verIngreso(props.item)">
                            tab
                        </v-icon>
                        <v-icon small class="mr-2" @click="mostrarComprobante(props.item)">
                            print
                        </v-icon>
                        <template v-if="props.item.estado">
                            <v-icon
                            small
                            @click="activarDesactivarMostrar(2,props.item)"
                            >
                            block
                            </v-icon>
                        </template>
                        <template v-else>
                            <v-icon
                            small
                            @click="activarDesactivarMostrar(1,props.item)"
                            >
                            check
                            </v-icon>
                        </template>
                    </td>
                    <td>{{ props.item.usuario.nombre }}</td>
                    <td>{{ props.item.persona.nombre }}</td>
                    <td>{{ props.item.tipo_comprobante }}</td>
                    <td>{{ props.item.serie_comprobante }}</td>
                    <td>{{ props.item.num_comprobante }}</td>
                    <td>{{ props.item.createdAt }}</td>
                    <td>{{ props.item.impuesto }}</td>
                    <td>{{ props.item.total }}</td>
                    <td>
                        <div v-if="props.item.estado">
                            <span class="blue--text">Aceptado</span>
                        </div>
                        <div v-else>
                            <span class="red--text">Anulado</span>
                        </div>
                    </td>                
                    </template>
                    <template v-slot:no-data>
                    <v-btn color="primary" @click="listar()">Resetear</v-btn>
                </template>
            </v-data-table>
            <v-container grid-list-sm class="pa-4 white" v-if="verNuevo">
                <v-layout row wrap>
                    <v-flex xs12 sm4 md4 lg4 xl4>
                        <v-select v-model="tipo_comprobante" 
                        :items="comprobantes" label="Tipo Comprobante">
                        </v-select>
                    </v-flex>
                    <v-flex xs12 sm4 md4 lg4 xl4>
                        <v-text-field v-model="serie_comprobante" label="Serie Comprobante">
                        </v-text-field>
                    </v-flex>
                    <v-flex xs12 sm4 md4 lg4 xl4>
                        <v-text-field v-model="num_comprobante" label="Número Comprobante">
                        </v-text-field>
                    </v-flex>
                    <v-flex xs12 sm8 md8 lg8 xl8>
                        <v-autocomplete :items="personas" v-model="persona" label="Cliente">
                        </v-autocomplete>
                    </v-flex>
                    <v-flex xs12 sm4 md4 lg4 xl4>
                        <v-text-field type="number" v-model="impuesto" label="Impuesto">
                        </v-text-field>
                    </v-flex>
                    <v-flex xs12 sm8 md8 lg8 x8>
                        <v-text-field v-model="codigo" label="Código" @keyup.enter="buscarCodigo()">
                        </v-text-field>
                    </v-flex>
                    <v-flex xs12 sm2 md2 lg2 xl2>
                        <v-btn small fab dark color="teal" @click="mostrarModalArticulos()">
                            <v-icon dark>list</v-icon>
                        </v-btn>
                    </v-flex>
                    <v-flex xs12 sm2 md2 lg2 xl2 v-show="errorArticulo">
                        <div class="red--text" v-text="errorArticulo">

                        </div>
                    </v-flex>
                    <v-flex xs12 sm12 md12 lg12 xl12>
                        <template>
                            <v-data-table
                                :headers="cabeceraDetalles"
                                :items="detalles"
                                hide-actions
                                class="elevation-1"
                            >
                                <template slot="items" slot-scope="props">
                                    <td>
                                        <v-icon
                                        small
                                        class="mr-2"
                                        @click="eliminarDetalle(detalles,props.item)"
                                        >
                                        delete
                                        </v-icon>
                                    </td>
                                    <td class="text-xs-center">{{ props.item.articulo }}</td>
                                    <td class="text-xs-center"><v-text-field v-model="props.item.cantidad" type="number"></v-text-field></td>
                                    <td class="text-xs-center"><v-text-field v-model="props.item.precio" type="number"></v-text-field></td>
                                    <td class="text-xs-center"><v-text-field v-model="props.item.descuento" type="number"></v-text-field></td>
                                    <td class="text-xs-right">$ {{ (props.item.cantidad * props.item.precio)-props.item.descuento}}</td>
                                </template>
                                <template slot="no-data">
                                    <h3>No hay artículos agregados al detalle.</h3>
                                </template>
                            </v-data-table>
                            <v-flex class="text-xs-right">
                                <strong>Total Parcial:</strong> $ 
                                {{totalParcial=(total-totalImpuesto).toFixed(2)}}
                            </v-flex>
                            <v-flex class="text-xs-right">
                                <strong>Total Impuesto:</strong> $ 
                                {{totalImpuesto=((total*impuesto)/(1+impuesto)).toFixed(2)}}
                            </v-flex>
                            <v-flex class="text-xs-right">
                                <strong>Total Neto:</strong> $ {{total=calcularTotal}}
                            </v-flex>
                        </template>
                    </v-flex>
                    <v-flex xs12 sm12 md12 v-show="valida">
                        <div class="red--text" v-for="v in validaMensaje" :key="v" v-text="v">

                        </div>
                    </v-flex>                    
                    <v-flex xs12 sm12 md12 lg12 xl12>
                        <v-btn color="blue darken-1" flat @click.native="ocultarNuevo()">Cancelar</v-btn>
                        <v-btn color="success" v-if="verDetalle==0" @click.native="guardar()">Guardar</v-btn>
                    </v-flex>
		        </v-layout>
            </v-container>
        </v-flex>
    </v-layout>
</template>
<script>
    import axios from 'axios'
    import jsPDF from 'jspdf'
    import html2canvas from 'html2canvas';
    export default {
        data(){
            return{
                dialog: false,
                search:'',
                ventas:[],
                headers: [
                    { text: 'Opciones', value: 'opciones', sortable: false },
                    { text: 'Usuario', value: 'usuario.nombre', sortable: true },
                    { text: 'Cliente', value: 'persona.nombre', sortable: true },
                    { text: 'Tipo Comprobante', value: 'tipo_comprobante', sortable: true },
                    { text: 'Serie comprobante', value: 'serie_comprobante', sortable: false  },
                    { text: 'Número comprobante', value: 'num_comprobante', sortable: false  },
                    { text: 'Fecha', value: 'createdAt', sortable: false  },
                    { text: 'Impuesto', value: 'impuesto', sortable: false  },
                    { text: 'Total', value: 'total', sortable: false  },
                    { text: 'Estado', value: 'estado', sortable: false  }                
                ],
                _id:'',
                persona:'',
                personas:[],
                tipo_comprobante:'',
                comprobantes: ['BOLETA','FACTURA','TICKET','GUIA'],
                serie_comprobante:'',
                num_comprobante: '',
                impuesto: 19,
                codigo:'',
                cabeceraDetalles:[
                    { text: 'Borrar', value: 'borrar', sortable: false },
                    { text: 'Artículo', value: 'articulo', sortable: false },
                    { text: 'Cantidad', value: 'cantidad', sortable: false },
                    { text: 'Precio', value: 'precio', sortable: false },
                    { text: 'Descuento', value: 'descuento', sortable: false },
                    { text: 'Sub Total', value: 'subtotal', sortable: false  }
                ],
                detalles:[],
                verNuevo:0,
                errorArticulo:null,
                total:0,
                totalParcial:0,
                totalImpuesto:0,
                articulos:[],
                texto:'',
                cabeceraArticulos: [
                    { text: 'Seleccionar', value: 'seleccionar', sortable: false },
                    { text: 'Código',value: 'codigo', sortable: false},
                    { text: 'Nombre',value: 'nombre', sortable: true},
                    { text: 'Categoría',value: 'categoria.nombre', sortable: true},
                    { text: 'Stock',value: 'stock', sortable: false},
                    { text: 'Precio Venta',value: 'precio_venta', sortable: false},
                    { text: 'Descripción', value: 'descripcion', sortable: false },              
                    { text: 'Estado', value: 'estado', sortable: false }
                ],
                verDetalle:0,
                valida:0,
                validaMensaje:[],
                adModal:0,
                adAccion:0,
                adNombre:'',
                adId:'',
                comprobanteModal:0,
                fecha:null
            }
        },
        computed: {
            calcularTotal: function(){
                let resultado=0.0;
                for(var i=0;i<this.detalles.length;i++){
                    resultado=resultado+((this.detalles[i].cantidad*this.detalles[i].precio)-this.detalles[i].descuento);
                }
                return resultado;
            }
        },
        watch: {
            dialog (val) {
            val || this.close()
            }
        },
        created () {
            this.listar();
            this.selectPersona();
        },
        methods: {
            crearPDF(){
                var quotes = document.getElementById('factura');
                html2canvas(quotes).then(function (canvas) {
                    var imgData = canvas.toDataURL('image/png');
                    var imgWidth = 210;
                    var pageHeight = 295;

                    var imgHeight = canvas.height * imgWidth / canvas.width;
                    var heightLeft = imgHeight;
                    
                    var doc = new jsPDF('p', 'mm', 'a4');
                    var position = 0;

                    doc.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight);
                    heightLeft -= pageHeight;

                    while (heightLeft >= 0) {
                        position = heightLeft - imgHeight;
                        doc.addPage();
                        doc.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight);
                        heightLeft -= pageHeight;
                    }
                    doc.save('ComprobanteVenta.pdf');
                });
            },
            mostrarComprobante(item){
                this.limpiar();
                this.tipo_comprobante=item.tipo_comprobante;
                this.serie_comprobante=item.serie_comprobante;
                this.num_comprobante=item.num_comprobante;
                this.fecha=item.createdAt;
                this.persona=item.persona;
                this.impuesto=item.impuesto;
                this.listarDetalles(item._id);
                this.comprobanteModal=1;
            },
            ocultarComprobante(){
                this.comprobanteModal=0;
            },
            selectPersona(){
                let me=this;
                let personaArray=[];
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};            
                axios.get('persona/listClientes',configuracion).then(function (response){
                    personaArray=response.data;
                    personaArray.map(function(x){
                        me.personas.push({text:x.nombre, value:x._id});
                    });
                }).catch(function(error){
                    console.log(error);
                });
            },
            buscarCodigo(){
                let me=this;
                me.errorArticulo=null;
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};            
                axios.get('articulo/queryCodigo?codigo='+this.codigo,configuracion).then(function (response){
                    me.agregarDetalle(response.data);
                }).catch(function(error){
                    me.errorArticulo='No existe el artículo.';
                });

            },
            agregarDetalle(data){
                this.errorArticulo=null;
                if (this.encuentra(data._id)==true){
                    this.errorArticulo='El artículo ya ha sido agregado.';
                }
                else{
                    this.detalles.push(
                        {
                            _id:data._id,
                            articulo:data.nombre,
                            cantidad:1,
                            precio:data.precio_venta,
                            descuento:0
                        }
                    );
                    this.codigo='';
                }
                
            },
            encuentra(id){
                let sw=0;
                for (var i=0;i<this.detalles.length;i++){
                    if(this.detalles[i]._id==id){
                        sw=true;
                    }
                }
                return sw;
            },
            eliminarDetalle(arr,item){
                let i=arr.indexOf(item);
                if (i!= -1){
                    arr.splice(i,1);
                }
            },
            listarArticulos(){
                let me=this;
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};            
                axios.get('articulo/list?valor='+this.texto,configuracion).then(function (response){
                    me.articulos=response.data;
                }).catch(function(error){
                    console.log(error);
                });
            },
            mostrarModalArticulos(){
                this.dialog=1;
            },
            listarDetalles(id){
                let me=this;
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};            
                axios.get('venta/query?_id='+id,configuracion).then(function (response){
                    me.detalles=response.data.detalles;
                }).catch(function(error){
                    console.log(error);
                });
            },
            verIngreso(item){
                this.limpiar();
                this.tipo_comprobante=item.tipo_comprobante;
                this.serie_comprobante=item.serie_comprobante;
                this.num_comprobante=item.num_comprobante;
                this.persona=item.persona._id;
                this.impuesto=item.impuesto;
                this.listarDetalles(item._id);
                this.verNuevo=1;
                this.verDetalle=1;
            },
            listar(){
                let me=this;
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};            
                axios.get('venta/list',configuracion).then(function (response){
                    me.ventas=response.data;
                }).catch(function(error){
                    console.log(error);
                });

            },
            limpiar(){
                this._id='';
                this.tipo_comprobante='';
                this.serie_comprobante='';
                this.num_comprobante='';
                this.impuesto=0.18;
                this.codigo='';
                this.total=0;
                this.totalParcial=0;
                this.totalImpuesto=0;
                this.detalles=[];
                this.verNuevo=0;
                this.valida=0;
                this.validaMensaje=[];
                this.verDetalle=0;
            },
            validar(){
                this.valida=0;
                this.validaMensaje=[];
                if(!this.persona){
                    this.validaMensaje.push('Seleccione un cliente.');
                }
                if(!this.tipo_comprobante){
                    this.validaMensaje.push('Seleccione un tipo de comprobante.');
                }
                if(!this.num_comprobante){
                    this.validaMensaje.push('Ingrese el número del comprobante.');
                }
                if(!this.impuesto || this.impuesto<0 || this.impuesto>1){
                    this.validaMensaje.push('Ingrese un impuesto válido.');
                }
                if(this.detalles.length<=0){
                    this.validaMensaje.push('Ingrese al menos un artículo al detalle');
                }
                if (this.validaMensaje.length){
                    this.valida=1;
                }
                return this.valida;
            },
            mostrarNuevo(){
                this.verNuevo=1;
            },
            ocultarNuevo(){
                this.verNuevo=0;
                this.limpiar();
            },
            guardar(){
                let me=this;
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};
                if (this.validar()){
                    return;
                }
                //Código para guardar
                axios.post('venta/add',
                {
                    'persona':this.persona,
                    'usuario':this.$store.state.usuario._id,
                    'tipo_comprobante': this.tipo_comprobante,
                    'serie_comprobante': this.serie_comprobante,
                    'num_comprobante':this.num_comprobante,
                    'impuesto': this.impuesto,
                    'total':this.total,
                    'detalles':this.detalles
                },configuracion)
                .then(function(response){
                    me.limpiar();
                    me.close();
                    me.listar();
                })
                .catch(function(error){
                    console.log(error);
                });
            },
            activarDesactivarMostrar(accion,item){
                this.adModal=1;
                this.adNombre=item.num_comprobante;
                this.adId=item._id;
                if (accion==1){
                    this.adAccion=1;
                } else if(accion==2){
                    this.adAccion=2;
                } else{
                    this.adModal=0;
                }
            },
            activarDesactivarCerrar(){
                this.adModal=0;
            },
            activar(){
                let me=this;
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};
                axios.put('venta/activate',{'_id':this.adId},configuracion)
                    .then(function(response){
                        me.adModal=0;
                        me.adAccion=0;
                        me.adNombre='';
                        me.adId='';
                        me.listar();
                    })
                    .catch(function(error){
                        console.log(error);
                    });
            },
            desactivar(){
                let me=this;
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};
                axios.put('venta/deactivate',{'_id':this.adId},configuracion)
                    .then(function(response){
                        me.adModal=0;
                        me.adAccion=0;
                        me.adNombre='';
                        me.adId='';
                        me.listar();
                    })
                    .catch(function(error){
                        console.log(error);
                    });
            },
            close () {
                this.dialog = false;
            }
        }
    }
</script>
<style>
	    #factura {
            padding: 20px;
            font-family: Arial, sans-serif;
            font-size: 16px ;
        }

        #logo {
            float: left;
            margin-left: 2%;
            margin-right: 2%;
        }
        #imagen {
            width: 100px;
        }

        #fact {
            font-size: 18px;
            font-weight: bold;
            text-align: center;
        }   

        #datos {
            float: left;
            margin-top: 0%;
            margin-left: 2%;
            margin-right: 2%;
            /*text-align: justify;*/
        }

        #encabezado {
            text-align: center;
            margin-left: 10px;
            margin-right: 10px;
            font-size: 16px;
        }

        section {
            clear: left;
        }

        #cliente {
            text-align: left;
        }

        #facliente {
            width: 40%;
            border-collapse: collapse;
            border-spacing: 0;
            margin-bottom: 15px;
        }

        #fa {
            color: #FFFFFF;
            font-size: 14px;
        }

        #facarticulo {
            width: 100%;
            border-collapse: collapse;
            border-spacing: 0;
            padding: 20px;
            margin-bottom: 15px;
        }

        #facarticulo thead {
            padding: 20px;
            background: #2183E3;
            text-align: center;
            border-bottom: 1px solid #FFFFFF;
        }

        #gracias {
            text-align: center;
        }
</style>
